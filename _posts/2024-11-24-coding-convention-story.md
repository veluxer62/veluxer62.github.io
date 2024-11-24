---
layout: single
title: 좀 더 나은 코딩 컨벤션을 위하여
categories:
  - EXPLANATION
toc: true
toc_sticky: true
toc_h_max: 2
---

이 글은 사내 블로그에 작성한 [스포카의 백엔드팀에서 코딩 컨벤션을 관리하는 방법](https://spoqa.github.io/2024/11/18/coding-convention-story.html){: target="\_blank" } 내용을 그대로 가져오면서 나의 블로그의 언어톤에 맞게 변경한 글이다.

개발자라면 누구나 한 번쯤 더 나은 코드를 작성하고, 팀의 생산성과 유지보수성을 높이기 위해 고민해 보았을 것이다. 중복된 코드를 줄이고, 가독성을 높이며, 테스트 코드를 꼼꼼히 작성하거나, 알맞은 변수명을 고심하는 과정은 모두 그런 노력의 일환이다. 하지만 이런 개선 작업이 효과적으로 이루어지려면, 팀 전체가 공통된 코딩 기준을 공유하고 지키는 것이 무엇보다 중요하다.

일관된 코딩 컨벤션은 단순한 규칙 이상의 역할을 한다. 특히 여러 명의 개발자로 이루어진 팀에서는 코드의 가독성과 유지보수성을 높이고, 불필요한 논쟁을 줄이며, 협업의 효율성을 극대화하는 강력한 도구가 된다. 하지만 이러한 컨벤션을 설정하고, 이를 꾸준히 유지하는 과정은 절대 간단하지 않다. 문서로 정리한 규칙이 팀 내에서 제대로 적용되지 않거나, 시간이 지나며 점차 구식이 되는 문제를 겪어보신 분들도 많을 것이라 생각한다.

우리 팀 역시 비슷한 과정을 겪었다. 코딩 컨벤션을 관리하기 위해 문서, 코드 리뷰, 그리고 자동화 도구를 활용하며, 더 나아가 Konsist라는 새로운 도구까지 도입하게 된 여정을 통해 다양한 시행착오를 경험했다. 이 글에서는 단순한 도구 사용법을 넘어, 우리가 직면했던 문제들과 이를 어떻게 극복했는지에 대해 이야기해 보려 한다.

코딩 컨벤션 관리라는 쉽지 않은 도전이 어떻게 우리 팀의 개발 문화를 변화시켰는지, 그리고 여러분의 팀에도 어떤 인사이트를 줄 수 있을지 함께 살펴보자.

# 코딩 컨벤션 관리의 시작: 문서 작성

모든 시작은 단순하다. 우리 팀 역시 처음에는 README.md 파일을 통해 코딩 컨벤션을 관리하기 시작했다. 사내 문서 관리 도구인 Confluence를 사용하지 않은 이유는 간단했다. 코딩 컨벤션은 코드와 가장 밀접하게 연관되어 있기 때문에, 코드와 가장 가까운 곳에서 관리할 필요가 있다고 판단했기 때문이다.

README.md에는 서버 실행을 위한 준비 작업, IDEA 환경 설정, GIT 브랜치 전략, 그리고 코딩 컨벤션 등 백엔드 개발자가 우리 팀에서 알아야 할 모든 내용이 기재되어 있었다. 초기에는, 이 접근법이 꽤 효과적이었다. 새로운 팀원이 들어오더라도 문서를 참고해 작업 환경을 설정하거나 코드를 작성할 때 기준을 따를 수 있었으니까.

아래는 우리가 README.md에 작성했던 문서의 일부분이다.

#### 일관된 EOF(End of File) 설정을 위한 가이드

```markdown
#### EOF 설정

파일의 마지막 라인에 자동으로 개행이 되도록 하기 위한 설정입니다.

1. `Editor` -> `General`로 이동합니다.
2. `On Save` 색션에서 `Ensure every saved file ends with a line break`를 체크합니다.
```

#### Ktlint 플러그인 설정 가이드

```markdown
## Ktlint Settings

### ktlint pre-commit 설정

아래 명령어를 실행하여 pre-commit hook을 등록합니다.

"""
./gradlew addKtlintCheckGitPreCommitHook
"""


### Formatting

만약 lint 위반 오류가 발생하는 경우 아래 명령어를 실행하여 자동으로 포메팅을 수행합니다.

"""
./gradlew ktlintFormat
"""
```

이처럼 문서는 개발자가 코드 작성 시 참고할 수 있는 명확한 기준을 제시해 주었고, 코드 리뷰 과정에서도 사소한 논쟁을 줄이는 데 기여했다. 하지만 시간이 지나면서 문제점이 하나둘씩 드러나기 시작했다.

### 문서 관리의 한계

코딩 컨벤션은 한 번 정하고 끝나는 작업이 아니다. 시간이 흐르고 조직의 요구사항이나 기술 스택이 변화함에 따라 컨벤션도 업데이트되어야 한다. 하지만 문서로 관리할 경우, 이를 지속적으로 유지하는 책임자가 없다면 문서가 점차 구식이 되는 문제가 발생한다.

예를 들어, 문서에 설명된 규칙이 실제 코드와 일치하지 않을 때, 팀원들 사이에서 혼란이 발생하곤 했다. 이에 따라 코딩 컨벤션의 신뢰성이 떨어지고, "문서에 적혀 있는 내용은 무시해도 되는 것"이라는 인식이 퍼지기도 했다.

이러한 한계를 극복하기 위해 우리 팀은 문서만으로 코딩 컨벤션을 관리하는 데서 벗어나기 시작했다. 코딩 컨벤션을 더 효과적으로 지키기 위해 Lint와 같은 자동화된 도구, 그리고 코드 리뷰와 같은 방식을 도입했고, 이는 더 나은 방향으로의 첫걸음이 되었다.

# 자동화의 첫걸음: ktlint의 도입

문서로 코딩 컨벤션을 관리하는 데 한계를 느낀 우리 팀은 Lint라는 자동화 도구를 도입했다. Lint는 코드의 오류, 버그, 스타일 문제를 찾아주는 정적 코드 분석 도구로, 간단한 설정만으로도 코드 스타일을 일관성 있게 유지할 수 있게 해준다. 특히, Kotlin 언어에서는 ktlint가 대표적인 Lint 도구로 자리 잡고 있다.

### Lint 도구의 효과

Lint 도구를 활용하면서 코드 스타일 문제를 해결하는 과정이 크게 간소화되었다. 이전에는 코드 리뷰 과정에서 개행, Trailing commas, 공백 처리 같은 사소한 스타일 문제로 많은 시간을 할애해야 했다. 하지만 Lint를 도입한 이후, 이런 문제들은 더 이상 개발자들이 신경 써야 할 부분이 아니게 되었다. CI/CD 파이프라인에 통합하면, 코드가 규칙을 위반할 경우 자동으로 알려주기 때문에 문제를 사전에 방지할 수 있었다.

#### Gradle에서 ktlint 설정하기

Gradle에서 ktlint를 설정하는 방법은 간단하다. 먼저, Gradle 빌드 스크립트에 ktlint 플러그인을 추가한다.

```kotlin
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "{version}"
}
```

그 후, 특정 파일이나 디렉터리를 제외하고 싶은 경우, 다음과 같이 필터를 설정할 수 있다. 아래 예시는 다른 프레임워크에서 자동으로 생성된 코드들은 Lint의 검증 대상에서 제외하도록 하는 설정을 나타낸다.

```kotlin
ktlint {
    filter {
        exclude("**/generated/**")
    }
}
```

마지막으로, 테스트 실행 전 lint 검사를 자동으로 수행하도록 구성할 수 있다.

```kotlin
tasks.withType<Test> {
    dependsOn(tasks.withType<KtLintCheckTask>())
}
```

만약 ktlint에서 제공해 주는 Hook 기능을 사용한다면 Git에서 Commit 시 매번 스타일 오류를 수작업으로 수정하지 않아도, 코드를 저장하거나 테스트를 실행하는 과정에서 자동으로 스타일이 교정되거나 문제를 알려주도록 만들 수 있었다.

```
# Git Commit 전 ktlint check를 통해 스타일 위반 사항이 있는지 확인
./graldew addKtlintCheckGitPreCommitHook

# Git Commit 전 kotlin format을 통해 스타일을 자동으로 보정
./graldew addKtlintFormatGitPreCommitHook
```

### Lint 도구의 한계

Lint는 코드 스타일을 유지하고 간단한 문제를 잡아내는 데 매우 유용하지만, 모든 문제를 해결해 주는 도구는 아니다. 개행이나 공백 처리 같은 기본적인 규칙을 강제할 수는 있지만, 코드의 중복 문제나 보안상의 허점, 복잡한 구조적 컨벤션은 감지하지 못한다.

Lint 도구를 사용하면서 우리는 "자동화로 해결할 수 있는 범위는 어디까지일까?"라는 고민을 하게 되었고, 이 고민은 더 정교한 도구와 방법을 찾는 과정으로 이어졌으며, 그 여정은 곧 SonarQube와 Konsist 같은 도구의 도입으로 확장되었다.

# 코드 리뷰: 사람의 눈은 여전히 중요하다

자동화 도구를 더 이야기하기 전에, Lint와 같은 자동화 도구를 도입했음에도, 코드 리뷰는 여전히 코딩 컨벤션을 지키고 코드 품질을 높이는 데 필수적인 과정으로 남아 있다. 자동화 도구가 사소한 스타일 문제를 처리해 준다면, 코드 리뷰는 도메인 요구사항을 잘 충족하였는지, 자동화된 도구 잡아내기 힘든 컨벤션 위반을 발견해 주는 등의 더 깊이 있는 검토를 통해 팀의 코드 베이스를 개선하는 데 집중할 수 있다.

아래 이미지는 Lint로는 잡아내기 힘든 컨벤션 위반을 코드리뷰를 통해 개선할 수 있었던 사례이다. 우리 팀은 테스팅 라이브러리로 Kotest를 사용 중이며 테스트 코드 작성 시 아래와 같은 컨벤션을 지키도록 약속되어 있다.

```kotlin
// 테스트 코드 컨벤션

class SalesOrderFacadeTest : UnitTestBase() {
    // ... 생략
    
    init {
        context("getSalesOrderById") {
            test("아이디로 매출 전표를 조회한다") {
                // ... 생략
            }
        }
        
        // ... 생략
    }
}
```

![code-review](/assets/images/posts/coding-convention-story/code-review.png)

### 코드 리뷰의 한계

코드 리뷰는 수많은 회사에서 도입하고 있는 코드 품질을 높일 수 있는 수단임에도 불구하고 아래와 같이 몇 가지 한계가 있다.

첫 번째로, 사람이 직접 코드를 검토하기 때문에 시간이 많이 소요된다는 점이다. 특히, 개행, 공백, Trailing commas 같은 사소한 스타일 문제를 반복적으로 지적해야 한다면, 리뷰어와 작성자 모두에게 소모적인 작업이 될 수 있다. 이러한 한계는 앞서 소개한 Lint를 통해 해결할 수 있다.

두 번째로, 리뷰 품질이 리뷰어에 따라 다를 수 있다는 문제도 있다. 어떤 리뷰어는 코드 설계나 성능 최적화에 집중할 수 있지만, 다른 리뷰어는 스타일 문제에 더 많은 시간을 할애할 수 있다. 이에 따라 팀 내에서 일관성이 떨어질 가능성이 있다.

마지막으로, 코드 리뷰 과정에서 부정적인 피드백이 적절히 전달되지 않으면 팀원 간의 긴장감이 생길 수 있다. 이는 팀워크에 부정적인 영향을 미칠 수 있고, 오히려 협업의 효율성을 저해하는 결과를 초래할 수 있다.

이러한 한계를 극복하기 위해 자동화 도구를 활용해 사소한 문제를 미리 해결하거나, 팀의 리뷰 가이드라인을 문서화하여 일관성을 유지하는 등의 보완책이 필요하다. 코드 리뷰는 팀의 코드 품질과 협업 문화를 개선하는 중요한 과정인 만큼, 이러한 한계를 인지하고 효과적으로 관리하는 것이 중요하다.

"자동화된 도구가 더 많은 것을 해결해 줄 순 없을까?"라는 고민은 여기서 더욱 깊어졌다.

# SonarQube의 도입: 더 넓은 관점에서의 코드 품질 관리

Lint와 코드 리뷰를 통해 팀의 코딩 컨벤션을 어느 정도 유지할 수 있었지만, 여전히 놓치고 있는 문제들이 있었다. 스타일 문제나 사소한 규칙 위반은 Lint로 충분히 해결할 수 있었지만, 코드의 중복, 복잡성, 보안 취약점, 그리고 더 넓은 관점에서의 코드 품질 관리는 다른 도구가 필요했다. 이에 우리 팀은 SonarQube를 도입하게 되었다.

### SonarQube의 기능과 장점

SonarQube는 코드 품질을 유지하고 개선하는 데 있어 매우 강력한 도구이다. 가장 큰 장점은 코드의 다양한 품질 요소를 종합적으로 분석할 수 있다는 점이다. 예를 들어, 코드에서 중복된 부분을 찾아내거나 복잡도가 높은 영역을 감지하며, 테스트 커버리지를 평가하고 버그나 보안 취약점을 자동으로 탐지할 수 있다. 이를 통해 개발자는 단순한 스타일 문제를 넘어서, 코드의 구조적 결함이나 유지보수성을 저해할 수 있는 장기적인 문제들을 사전에 파악할 수 있다.

아래 사례는 Kotlin의 Data Class에서 Array 프로퍼티를 사용하는 경우 equals, hashCode, toString을 재정의해 주지 않아 발생한 SonarQube의 이슈 목록을 보여준다.

![sonarqube-issues](/assets/images/posts/coding-convention-story/sonarqube-issues.png)

또한, SonarQube는 오픈소스로 제공되기 때문에 로컬 환경이나 사내 서버에 설치해서 사용할 수 있다. 이 외에도 클라우드 기반으로도 운영이 가능하여 팀의 필요에 맞는 다양한 방식으로 유연하게 활용할 수 있다. 기본 제공되는 검사 규칙뿐만 아니라, 프로젝트의 특성에 따라 다양한 플러그인과 설정을 통해 맞춤형 규칙을 추가할 수도 있어 확장성도 뛰어나다.

### 운영 과정에서 마주한 도전 과제

SonarQube의 강력함에도 불구하고, 실제 운영 과정에서는 몇 가지 불편한 점들이 존재했다. 

첫 번째로는 서버 관리와 관련된 문제이다. SonarQube를 사용하려면 자체 서버를 설치하고 운영해야 하는데, 이 과정에서 추가적인 설정과 유지보수가 필요했다. 특히, 코드 분석 과정에서 소스코드를 서버에 업로드하고 검증하는 데 시간이 소요되었는데, 이는 Pull Request 단계에서 실시간 검증을 수행하기에는 부담으로 작용했다.

두 번째 도전 과제는 사전 검증과 사후 검증 간의 균형을 맞추는 일이었다. 초기에는 코드 변경 사항이 병합되기 전에 Pull Request 단계에서 SonarQube를 통해 코드를 검증하려 했다. 그러나 검증 과정이 느리고 시간이 오래 걸리는 탓에, 개발 속도를 저하하는 문제가 있었다. 이에 따라 우리는 코드가 병합된 후에 Code Smell이나 품질 문제를 감지하는 방식으로 운영 방식을 조정했다. 이 방식은 코드 품질 문제를 감지하는 데 유연함을 제공했지만, 문제를 사전에 완전히 방지할 수는 없다는 한계도 있었다.

SonarQube는 단순히 스타일을 검사하는 도구에서 벗어나, 팀이 더 넓은 관점에서 코드 품질을 관리할 수 있도록 도와주는 강력한 도구이다. 비록 작업코드가 병합되기 전에 품질관리를 하기 어렵다는 불편함은 있지만 사후 검증을 통해서도 Lint와 코드 리뷰가 해결하지 못했던 복잡한 문제를 감지하고, 코드베이스의 장기적인 건강 상태를 유지하는 데 중요한 역할을 하고 있다는 점은 코드 컨벤션 관리를 통해 우리 팀이 더 나은 코드 품질을 유지하기 위한 노력에 잘 부합한다고 할 수 있다.

다만, SonarQube를 도입한 이후에도 우리는 더 세부적인 규칙 검증과 구조적인 코딩 컨벤션 관리를 필요로 했고, 이 과정은 결국 Konsist와 같은 도구를 찾게 되는 계기가 되었다.

# Konsist: 코딩 컨벤션 관리의 새로운 도약

앞서 소개한 도구들(문서, Lint, SonarQube, 코드 리뷰)은 코딩 컨벤션을 유지하고 코드 품질을 관리하는 데 큰 도움이 되었지만, 여전히 해결되지 않는 문제들이 있었다. 예를 들어, 프로젝트의 구조적 규칙을 정의하거나, 클래스 네이밍 규칙, 의존성 방향과 같은 팀 고유의 복잡한 코딩 컨벤션을 자동으로 검증하는 것은 기존 도구로는 쉽지 않았다. 바로 이 지점에서 Konsist라는 새로운 도구가 빛을 발했다.

### Konsist란 무엇인가?

Konsist는 Kotlin 언어를 위해 설계된 정적 코드 분석 도구로, 기존의 Lint나 SonarQube가 다루기 어려운 구조적 규칙과 세세한 컨벤션을 검증할 수 있도록 설계되었다. Konsist의 가장 큰 특징은 API를 활용한 단위 테스트를 통해, 팀에서 정의한 코딩 규칙을 명확히 표현하고 자동화된 방식으로 검증할 수 있다는 점이다.

예를 들어, "Domain 레이어는 Application 레이어를 의존하지 않는다"와 같은 복잡한 구조적 규칙을 코드로 표현하고, 이를 테스트를 통해 검증할 수 있다. 이 도구는 기존 도구들의 한계를 보완하며, 코딩 컨벤션 관리의 새로운 가능성을 열어주었다.

### Konsist의 주요 특징

#### 간단한 설정

Konsist는 Gradle이나 Maven에 의존성을 추가하고, JUnit이나 Kotest와 같은 테스트 프레임워크와 연동하면 바로 사용할 수 있다. 복잡한 설정 과정 없이 빠르게 적용할 수 있기 때문에 새로운 도구를 도입할 때 진입 장벽이 낮다. 우리 팀도 Konsist를 처음 접한 날 바로 프로젝트에 적용 테스트를 진행할 수 있을 만큼 사용이 간단했다.

#### 구조적 규칙 검증

Konsist는 기존 도구들이 다루기 어려운 프로젝트의 구조적 규칙을 검증할 수 있는 강력한 기능을 제공한다. 예를 들어, 특정 레이어가 다른 레이어를 의존하지 않도록 규칙을 정의하거나, 클래스와 속성의 네이밍 규칙을 설정하고 이를 자동으로 검증할 수 있다. 이러한 기능은 프로젝트 구조와 일관성을 유지하는 데 큰 도움이 된다.

#### 효율성 향상

Konsist를 활용하면 코드 리뷰 과정에서 반복적으로 지적해야 했던 많은 규칙 위반을 자동화할 수 있다. 이는 코드 리뷰의 품질을 높이는 동시에, 리뷰어가 더 중요한 논의나 설계 검토에 집중할 수 있도록 해준다. 덕분에 팀의 생산성과 효율성이 크게 향상되었다.

#### 단순하면서도 강력한 구조

Konsist는 복잡한 학습 없이 바로 사용할 수 있는 단순한 구조로 되어 있다. 개발자는 검증해야 할 코딩 규칙을 정의하는 데만 집중하면 되며, 불필요한 설정이나 복잡한 사용법으로 인해 시간을 낭비하지 않아도 된다. 이는 도구를 처음 접하는 개발자들에게도 사용이 용이하다는 점에서 큰 장점으로 작용한다.

### Konsist 활용 사례

이 글의 목적은 Konsist를 자세히 소개하는 데 있지 않다. Konsist를 시작하는 방법이나 설치 가이드는 [공식 홈페이지](https://docs.konsist.lemonappdev.com/){: target="\_blank" } 에서 확인하실 수 있으니 참고바란다. 대신, 이 글에서는 우리 팀이 Konsist를 활용하고 있는 몇 가지 사례를 통해, 이 도구를 어떻게 효과적으로 사용할 수 있을지 감을 잡을 수 있도록 도와드리고자 한다.

#### 구조 컨벤션

우리 팀은 명확하고 이해하기 쉬우며 유지보수성을 높일 수 있도록 아래처럼 단순한 프로젝트 구조를 사용하고 있다.

![architecture](/assets/images/posts/coding-convention-story/architecture.png)

Domain 레이어는 데이터 및 비즈니스 로직과 같은 도메인 로직을 처리하는 코드들이 위치하며,
Application 레이어는 표현 계층, Facade, 그리고 외부 리소스와의 연동을 담당하는 기반 코드들이 포함되어 있다.

의존성 방향에 대한 규칙도 명확히 정의했다. Application 레이어는 Domain 레이어를 의존할 수 있지만, 반대로 Domain 레이어는 Application 레이어를 의존하지 않도록 컨벤션을 정했다.

이러한 규칙을 코드로 표현하면 아래와 같다.

```kotlin
test("application 레이어는 domain 레이어를 의존한다.") {
    Konsist
        .scopeFromProduction()
        .assertArchitecture {
            val application = Layer("Application", "com.spoqa.cart.application..")
            val domain = Layer("Domain", "com.spoqa.cart.domain..")

            application.dependsOn(domain)
            domain.dependsOnNothing()
        }
}
```

여담으로, 처음 구조 테스트를 실행했을 때 나름 코드 리뷰에서 꼼꼼하게 의존성 위반 여부를 점검했다고 생각했지만, 예상외로 다수의 의존성 위반 코드가 발견되었다. 이를 통해 코드 리뷰가 사람이 수행하는 작업인 만큼, 놓치는 부분이 생길 수밖에 없다는 점을 다시 한번 실감했다. 동시에, Konsist가 이런 문제를 효과적으로 해결할 수 있다는 점도 확실히 체감할 수 있었다.

![architectural-test-result](/assets/images/posts/coding-convention-story/architectural-test-result.png)

#### Class 컨벤션

또한 우리 팀에서는 Entity 클래스나 테스트 클래스들은 여러 이유로 인해 특정 클래스를 필수적으로 상속받도록 강제하고 있다.

```kotlin
@Entity
class Bill(
    orderableVendor: OrderableVendor,
    store: Store,
    amount: BigDecimal,
    type: BillType,
    attachements: List<String>,
) : BaseEntity()

@Entity
class Role(
    name: String,
    permissions: Array<String>,
) : BaseEntity()

// ... 생략

class OrderSheetFacadeTest : UnitTestBase()

class AdminMutationTest : FunctionalTestBase()

class CartRepositoryTest(
    private val cartRepository: CartRepository,
    private val testEntityManager: TestEntityManager,
) : RepositoryTestBase()

// ... 생략
```

위와 같은 컨벤션을 테스트하는 코드는 아래와 같다.

```kotlin
test("Entity 클래스는 BaseEntity 클래스를 필수로 상속한다") {
    Konsist.scopeFromProduction()
        .classes()
        .filter { it.hasAnnotationOf(Entity::class) }
        .assertTrue(testName = this.testCase.name.testName) {
            it.hasParentClassOf(BaseEntity::class)
        }
}

test("test 클래스는 Base 클래스 중 하나를 필수로 상속한다") {
    Konsist.scopeFromTest()
        .classes()
        .filter { it.name.endsWith("Test") }
        .assertTrue(testName = this.testCase.name.testName) {
            it.hasParentClassOf(FunctionalTestBase::class) ||
                it.hasParentClassOf(UnitTestBase::class) ||
                it.hasParentClassOf(RepositoryTestBase::class) ||
                it.hasParentClassOf(IndexRepositoryTestBase::class)
        }
}
```

#### Entity Property 컨벤션

Kotlin으로 JPA를 사용하다 보면 종종 실수하기 쉬운 부분이 있다. 바로 JPA의 `@Column` 어노테이션에서 `nullable` 속성과 Entity 프로퍼티의 타입을 다르게 정의하는 경우이다. 우리 팀은 이러한 실수를 방지하기 위해 아래와 같이 Declarations Test를 정의해 두었다.

```kotlin
test("Entity 클래스의 Column 프로퍼티가 non-nullable 타입이라면 '@Column(nullable=false)'가 선언되어야 한다") {
    Konsist.scopeFromProject()
        .classes()
        .withAllAnnotationsOf(Entity::class)
        .properties()
        .withAllAnnotationsOf(Column::class)
        .filter { it.type?.isNullable == false }
        .assertTrue {
            it.hasAnnotation { annotation ->
                annotation.hasArgument { arg ->
                    arg.name == "nullable" && arg.value == "false"
                }
            }
        }
}
```

이와 같이 테스트 코드를 작성하면 아래와 같이 어노테이션과 컬럼의 타입이 불일치하게 작성하는 경우를 효율적으로 방지할 수 있다.

```kotlin
@Entity
class Admin(
    email: String,
    name: String,
    role: Role?,
) : BaseEntity() {
    @Column
    val email: String = email
    
    // ... 생략
}
```

![property-test-result](/assets/images/posts/coding-convention-story/property-test-result.png)

# 마무리하며

Konsist와 같은 도구는 팀이 코딩 컨벤션을 잘 지키도록 돕는 강력한 도구지만, 모든 문제를 해결할 수 있는 만능 솔루션은 아니다. 지나치게 세부적인 규칙은 생산성을 떨어뜨릴 수 있으며, 도구가 잡아내지 못하는 부분은 결국 사람이 검토해야 한다는 것을 기억하자.

우리 팀은 Konsist, ktlint, SonarQube와 같은 도구를 적절히 조합하고, 코드 리뷰를 통해 이를 보완하면서 코딩 컨벤션을 지속적으로 발전시키고 있다. 이러한 과정은 단순히 코딩 규칙을 준수하는 것을 넘어, 팀의 협업 문화를 개선하고 더 나은 코드를 만드는 데 기여하고 있다.

다른 팀들은 코딩 컨벤션을 어떻게 관리하고 있는지 궁금하다. 기회가 된다면 각자의 경험과 노하우를 공유하며 서로 배울 수 있는 시간이 생겼으면 하는 바램이다.
