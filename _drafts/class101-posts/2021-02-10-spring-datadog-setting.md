---
layout: single
title: "스프링 with k8s and datadog"
categories:
  - POST
tags:
---

Jib 세팅

Jib 기준 설명.

### 로그 디펜던시 추가하기 (아래 명시된 버전으로 사용된 사용 권장)

```kotlin
// build.gradle.kts
implementation "org.springframework.cloud:spring-cloud-starter-sleuth:2.2.5.RELEASE"

implementation "ch.qos.logback:logback-classic:1.1.11"
implementation "ch.qos.logback:logback-core:1.1.11"
implementation "net.logstash.logback:logstash-logback-encoder:6.4"
```

### Logback 추가.

```kotlin
// logback.xml
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>

<appender name="DATA_DOG_LOG_APPENDER" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <remoteHost>intake.logs.datadoghq.com</remoteHost>
    <port>10514</port>
    <keepAliveDuration>20 seconds</keepAliveDuration>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <prefix class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>34eeb8bebf31d6a69cf3bda963c76304 %mdc{keyThatDoesNotExist}</pattern>
            </layout>
        </prefix>
    </encoder>
</appender>

<root level="INFO">
    <appender-ref ref="DATA_DOG_LOG_APPENDER"/>
</root>
```

- Jib 세팅 (Java)

    ```groovy
    // build.gradle
    plugins {
    		id "de.undercouch.download" version "4.1.1"
    }

    jib {
        to {
            image = '212011163676.dkr.ecr.ap-northeast-2.amazonaws.com/user-service'
            project.afterEvaluate {
                tags = [version]
            }
        }
        container {
            creationTime = 'USE_CURRENT_TIMESTAMP'
            jvmFlags = [
                    "-javaagent:/agents/dd-java-agent.jar"
            ]
        }
    }

    tasks {
        task downloadAgent(type: Download) {
            src(apmAgentUrl)
            dependsOn("setupAgentKey")
            dest("$apmAgentDownloadDir/$apmAgentFileName")
        }

        task setupAgentKey(type: Copy) {
            from("src/main/resources/$apmApiKeyFileName")
            into(apmAgentDownloadDir)
        }
    }

    tasks.jib {
        dependsOn("downloadAgent")
    }
    tasks.jibDockerBuild {
        dependsOn("downloadAgent")
    }
    tasks.jibBuildTar {
        dependsOn("downloadAgent")
    }
    tasks.build {
        dependsOn("downloadAgent")
    }
    defaultTasks("jib")
    ```

    ```
    // gradle.properties

    ## DataDog
    apmAgentUrl=https://s3.ap-northeast-2.amazonaws.com/net.class101.libs/dd-java-agent-0.58.0.jar
    apmAgentFileName=dd-java-agent.jar
    apmAgentDownloadDir=src/main/jib/agents
    apmApiKeyFileName=datadog-key.txt
    ```

### Jib 세팅 (Kotlin)

```kotlin
// build.gradle.kts
import de.undercouch.gradle.tasks.download.Download

tasks {
    task<Download>("downloadAgent") {
        dependsOn("setupAgentKey")
        src(apmAgentUrl)
        dest(apmAgentDownloadDir)
    }

    task<Copy>("setupAgentKey") {
        from("src/main/resources/${apmApiKeyFileName}")
        into("src/main/jib/agents")
    }
}

tasks.jib {
    dependsOn("downloadAgent")
}
tasks.jibDockerBuild {
    dependsOn("downloadAgent")
}
tasks.jibBuildTar {
    dependsOn("downloadAgent")
}
tasks.build {
    dependsOn("downloadAgent")
}
defaultTasks("jib")

jib {
    to {
        image = "212011163676.dkr.ecr.ap-northeast-2.amazonaws.com/point-server:${version}"
    }
    container {
        jvmFlags = mutableListOf(
                "-javaagent:/agents/${apmAgentFileName}"
        )
    }
}
```

### helm charㅅ

```go
// _helpers.tpl
{{/*
Datadog labels
*/}}
{{- define "class101-server.apm.labels" -}}
tags.datadoghq.com/{{.Release.Name}}.env: {{ .Values.container.activeProfile }}
tags.datadoghq.com/{{.Release.Name}}.service: {{ .Release.Name }}
tags.datadoghq.com/{{.Release.Name}}.version: {{ .Values.image.versionTag }}
{{- end -}}

{{/*
Datadog annotations
*/}}
{{- define "class101-server.apm.annotations" -}}
tags.datadoghq.com/{{.Release.Name}}.check_names: {{ .Values.container.activeProfile }}
tags.datadoghq.com/{{.Release.Name}}.init_configs: {{ .Release.Name }}
tags.datadoghq.com/{{.Release.Name}}.instances: |
    [
      {
        "host": "%%host%%"
      }
    ]

{{- end -}}
```

```go
// deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "class101-server.fullname" . }}
  labels:
    {{- include "class101-server.labels" . | nindent 4 }}
    {{- include "class101-server.apm.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.container.replicas }}
	...

	template:
		metadata:
			{{- include "class101-server.apm.labels" . | nindent 8 }}
			...
		spec:
			...
			containers:
        - name: {{ .Chart.Name }}
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: {{ .Values.container.activeProfile }}
            - name: DD_AGENT_HOST
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: DD_PROFILING_ENABLED
              value: "true"
            - name: DD_LOGS_INJECTION
              value: "true"
						- name: DD_TRACE_DB_CLIENT_SPLIT_BY_INSTANCE
              value: "true"
            - name: DD_APM_IGNORE_RESOURCES
              value: "GET /lifecycle/health,GET /health"
            - name: DD_ENV
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['tags.datadoghq.com/{{.Release.Name}}.env']
            - name: DD_SERVICE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['tags.datadoghq.com/{{.Release.Name}}.service']
            - name: DD_VERSION
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['tags.datadoghq.com/{{.Release.Name}}.version']
```

## 댓글

나처럼 최근 버젼을 사용하고자 하는 친구가 있을까봐 메모 남겨. spring boot 에서 logback-classic 과 logback-core 의 기본 버젼 값은 1.2.3 인데, logstash-logback-encoder 의 설정에서 LayoutWrappingEncoder 를 사용하기 위해서는 1.2 이전 버젼으로 명시적인 선언이 필요하니 설정시 참고 바래. 관련 내용은 https://github.com/logstash/logstash-logback-encoder/issues/158 이야.


datadog apm 에 서버가 보여지려면 deplyment.yaml 에 아래 내용 추가가 필요해, 템플릿에는 곳 추가할예정이야
template:
      metadata:
{{- include "class101-server.apm.labels" . | nindent 8 }}