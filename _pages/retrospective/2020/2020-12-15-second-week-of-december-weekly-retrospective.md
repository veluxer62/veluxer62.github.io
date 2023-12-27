---
layout: single
title: "2020년 12월 2주차 회고"
toc: true
toc_sticky: true
retrospective: true
permalink: /retrospective/2020-second-week-of-december-weekly-retrospective/
---

Azure Pipeline 작업을 원하는 수준까지 맞춘 후 마무리 지었다. 문서화 작업을 끝냈고 개발자 회의에서 해당 문서를 공유하고 앞으로 Azure Pipeline으로 서버 배포를 하도록 유도하였다. Azure Pipeline 작업은 쉽게 생각하고 시작하였는데, 예상치 못한 곳에서 이슈가 많이 있었고, 특히 Release의 경우에는 아직 전체를 yaml로 관리하는 것을 지원하지 않아서 결국 CI/CD 모두 yaml이 아닌 GUI에서 생성하고 관리하도록 통일하였다. yaml로 관리하면 코드화 할 수 있어서 많은 이점이 있겠지만 GUI로 생성하면 파이프라인의 이력을 좀더 편리하게 볼 수 있는 장점이 있어서 GUI로 생성하기로 결정하였다. Azure Pipeline에 대한 작업내용은 나중에 블로그로 작성해봐야 겠다.

아래 내용은 지난주 내가 겪었던 이슈 또는 배운 내용들을 적은 것이다.

## javax.validation.Valid

Kotlin을 사용하면서 굳이 Spring의 Validation을 사용하지 않았다. 그이유는 자바보다 Validation 코드를 적는데 번거로움이 적다. Kotlin에서는 String class에 isEmpty나 isBlank와 같은 함수를 기본적으로 제공해 준다. String 뿐만 아니라 List와 같은 Collection에도 해당 함수들이 내장되어 있는데 DTO의 Validation에서 대부분은 이와 같은 공백 체크나 null 체크가 대부분이다. (null 체크의 경우 kotlin은 null safe하기 때문에 기본적으로 제공된다고 보면 된다) 그러다보니 Validation을 거의 사용할일이 없었는데, 이번에 자바로 개발된 어플리케이션에서 API를 추가하던 중 필요해서 사용해 보게 되었다.

이번 구현에서는 단순히 기본 제공되는 `@NotBlank`, `@NotNull`과 같은 함수가 아니라 문자열 배열 내에 공백을 포함하지 못하도록 제약하는 조건을 추가하는 것이었다. Custom Validation Annotation을 추가하려면 먼저 어노테이션을 추가하고 해당 어노테이션을 사용하여 제약 조건을 검증하는 `ConstraintValidator`를 추가로 구현해주어야 한다. 코드를 보면 아래와 같다.

```java
public class CustomValidation {
    @SuppressWarnings("unused")
    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = NotEmptyFieldsValidator.class)
    public @interface NotEmptyFields {
        String message() default "List cannot contain empty fields";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};
    }

    public static class NotEmptyFieldsValidator implements ConstraintValidator<NotEmptyFields, List<String>> {
        @Override
        public boolean isValid(List<String> value, ConstraintValidatorContext context) {
            return value.stream().allMatch(item -> item != null && !item.isBlank());
        }
    }
}
```

아주 쉬운편에 속하는 Custom Validation이었지만 앞으로 Validation을 사용할때 유용하게 써먹을 수 있을 것 같다.

나중에 안 사실인데 `Bean Validation 2.0` 부터는 컬랙션 내에서도 Validation을 추가할 수 있다고 한다.

```java
public class DeleteContacts {
    @Min(1)
    private Collection<@Length(max = 64) @NotBlank String> uids;
}
```

참고 : [https://meetup.toast.com/posts/223](https://meetup.toast.com/posts/223){: target="\_blank" }

## @EntityGraph

JPA와 같은 ORM을 사용하다 보면 아마 반드시 마주치게될 이슈가 바로 N+1 문제이다.
하나의 Entity에 다른 Entity를 연관관계로 맺어줄 경우 조회 쿼리 시 반드시 N+1 문제가 발생하게 되는데 조회해야 할 대상이 적은 경우에는 쿼리 성능에 큰 문제가 발생하지 않지만 조회해야할 대상이 많은 경우에는 성능적으로 큰 차이가 난다.

이러한 문제를 해결하기 위해서 여러가지 방법이 있는데 QueryDSL을 이용한 Fetch Join 말고 해결할 수 있는 방법이 `@EntityGraph`를 이용하는 방법이 있다.

`@EntiyGraph`를 이용할때 두가지 방법을 사용할 수 있는데, 하나는 `Repository`에서 `@EntityGraph`만 사용하였을 때와 `Entity`내에서 `@NamedEntityGraph`를 함께 사용하는 것이다.

### `Repository`에서 `@EntityGraph`만 사용하는 경우

`Entity`내에 `@NamedEntityGraph`를 적어주지 않고 `Respository`에서만 `@EntityGraph`를 이용하여 N+1을 해결하려고 하는 경우 아래와 같이 FetchJoin을 걸어줄 속성의 이름을 `attributePaths`에 적어주면 된다.

```kotlin
interface KitPackRepository : JpaRepository<KitPackEntity, Long> {
    @EntityGraph(attributePaths = ["optionGroups", "kits"])
    @Query("select p from kit_packs p")
    fun getAll(): List<KitPackEntity>
}
```

만약 하위 Entity에 다시 또 N+1이 발생하는 경우에는 어떻게 하면 될까? 바로 `.`을 이용하여 하위 Entity의 속성을 추가로 적어주면 된다.

```kotlin
interface KitPackRepository : JpaRepository<KitPackEntity, Long> {
    @EntityGraph(attributePaths = ["optionGroups", "kits.settlements", "kits.shippings", "kits.options"])
    @Query("select p from kit_packs p")
    fun getAll(): List<KitPackEntity>
}
```

### `Entity`내 `@NamedEntityGraph`를 사용하는 경우

`Entity`내에 `@NamedEntityGraph`를 적어주고 `@EntityGraph`에서 해당 이름을 사용하여 N+1를 해결하려고 하는 경우에는 아래와 같이 사용하면 된다.

```kotlin
@Entity
@NamedEntityGraph(
    name = "kit-pack-graph",
    attributeNodes = [
        NamedAttributeNode("optionGroups"),
        NamedAttributeNode("kits")
    ]
)
class KitPackEntity {
  // ...
}
```

```kotlin
interface KitPackRepository : JpaRepository<KitPackEntity, Long> {
    @EntityGraph("kit-pack-graph")
    @Query("select p from kit_packs p")
    fun getAll(): List<KitPackEntity>
}
```

만약 하위 Entity에서 다시 또 N+1이 발생하는 경우 `subgraphs`를 추가해 주면 된다.

```kotlin
@Entity
@NamedEntityGraph(
    name = "kit-pack-graph",
    attributeNodes = [
        NamedAttributeNode("optionGroups"),
        NamedAttributeNode("kits", subgraph = "kit-graph")
    ],
    subgraphs = [
        NamedSubgraph(name = "kit-graph",
            attributeNodes = [
                NamedAttributeNode("settlements"),
                NamedAttributeNode("shippings"),
                NamedAttributeNode("options")
            ])
    ]
)
class KitPackEntity {
  // ...
}
```

```kotlin
interface KitPackRepository : JpaRepository<KitPackEntity, Long> {
    @EntityGraph("kit-pack-graph")
    @Query("select p from kit_packs p")
    fun getAll(): List<KitPackEntity>
}
```

두가지 방법 모두 장단 점이 있다. 아무래도 재사용성은 `@NamedEntityGraph`을 사용하는게 나아보이고, 중복 가능성이 적다면 `@EntityGraph`이 훨씬 단순하게 사용할 수 있어 보인다. 나중에 해당 방법에 대해서도 블로그로 정리해 봐야 겠다.

## Pod Identity Webhook

EKS를 사용하는 경우 AWS의 서비스를 사용하기 위해서 AWS권한을 가진 롤을 Pod마다 할당해 주어야 하는 경우가 있다. Pod는 Controller에 의해서 언제든지 제거되었다가 다시 생성되기도 하고 개수가 증가하기도 하고 감소하기도 한다. 이때 롤을 매번 할당해 줄 수는 없기 때문에 자동으로 롤을 할당해주는 방법이 필요한데 그때 사용할 수 있는게 `Pod Identity Webhook`이다.

AWS의 github repository에서 `Pod Identity Webhook`이미지를 받아서 빌드한 후 설치할 수 있는데 사용방법에 대해서는 데브시스터즈에서 작성한 블로그에 잘 나와 있으므로 해당 링크로 대체하고자 한다.

데브시스터즈 블로그 링크 : [https://tech.devsisters.com/posts/pod-iam-role/](https://tech.devsisters.com/posts/pod-iam-role/){: target="\_blank" }

## istio

현재 사내에서는 쿠버네티스를 이용하여 서비스를 운영중인데, 서킷브레이커나 트래픽 보안관리, A/B Testing, 로그 관리 등 사이트매시 기능을 제공하는 istio 도입을 고려중이다. 현재 그래서 개발 클러스터를 별도로 설정하고 있으며 istio 설치를 완료하면 내용을 정리해 보고자 한다.

참고 자료 : [https://istio.io/latest/docs/](https://istio.io/latest/docs/){: target="\_blank" }

## terraform

istio 도입을 위해 개발 클러스터를 별도로 설정하고 있다고 위에서 말하였는데, 현재 운영중인 클러스터는 terraform, CDK와 같이 인프라스트럭쳐를 코드화해서 관리할 수 있도록 도와주는 툴들을 이용해서 배포되어 있지 않다. 그렇다보니 개발 클러스터를 손쉽게 재구성할 수 없는 상황인데, istio를 설정하기 위해서 개발 클러스터를 구성하면서 terraform으로 코드화 하는 방식을 도입하여 진행중이다. CDK가 아닌 terraform을 선택한 이유는 단순히 이미 다른 팀에서 적용한 사례가 있어 도움을 받을 수 있는 상황이기에 빠르게 적용할 수 있기 때문이다. 그리고 추가적으로 CDK는 AWS에 국한되어 있지만 terraform은 멀티 플랫폼을 지원하기 때문에 좀더 매력적으로 다가왔다.
