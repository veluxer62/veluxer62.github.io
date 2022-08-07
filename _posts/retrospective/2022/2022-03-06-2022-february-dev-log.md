---
layout: single
title: "2022년 2월 개발일지"
categories:
  - RETROSPECTIVE
tags:
  - 개발일지
toc: true
---

언어전환이 중반기를 넘어가면서 슬슬 일정에 대한 압박을 받고 있다. 처음에 3개월이라는 시간을 약속 받았고 12월쯤 부터 작업을 시작했지만 마지막 스프린트 작업의 딜레이와 회사내 팀 매각 이슈, 명절, 등을 이유로 작업 일정이 예상보다 지연이 되었다. 그러면서 작업일정을 4월 중순으로 미룰것을 요청하였지만 경영진의 결정은 결국 3월안에 마무리를 짓고 4월 초에 다른 프로젝트를 시작하는 것으로 결론이 났다. 아쉬운 부분이 있지만 결정이 났으니 어찌하겠는가, 최대한 할 수 있는 대까진 하고 타협해야 할 부분은 타협해서 최대한 3월안에 마무리 할 수 있도록 최선을 다해야 겠다. 희망 회로지만 만약 3월에 마무리가 된다면 4월 첫주에는 그동안 늦게까지 야근하고 주말에도 일하면서 지친 몸과 마음을 달래기 위해 일주일 정도 휴가를 내던지 해야겠다. 3월에 마무리가 잘 된다면...

22일에 야곰 캐스트에서 [소통하는 개발자로 협UP하기](https://yagom.net/courses/techcast-9/){: target="\_blank" }라는 주제로 발표를 하였다. 온라인이긴 하지만 모르는 분들 앞에서 이렇게 발표하는 것은 처음이었다. 긴장은 되었지만 사내에서 발표한다는 마음가짐으로 발표를 하였고 아쉬운 부분은 많았지만 그래도 발표를 잘 마무리 하였다. 지인 중에 몇분이 들으러 와주셔서 큰 힘이 되었다.

회사 내 제품의 매각절차가 어느정도 마무리 되었다. 설을 기점으로 해당 제품을 담당하던 팀원들이 다른회사로 이직하게 되면서 그동안 미뤄두었던 2022년 연봉협상을 진행하였다. 다행스럽게도 지난 1년간의 노력을 인정받아 좋은 평가를 받았고 연봉협상도 만족스럽게 진행되었다. 언어 전환이라는 큰 프로젝트를 잘 마무리 짓고 앞으로 진행할 프로젝트도 열심히 진행해서 내년에도 만족스러운 협상이 진행되길 바라면서 올해도 화이팅이다.

작년에는 책을 많이 읽지 않은 것 같아서 원인을 생각해보니 책보단 전자책을 읽는 비중이 컷던게 원인이었다고 생각이 든다. 그래서 올해는 전자책보다는 그냥 책을 사서 읽으려고 책을 구매하기로 결심했다. 올해의 첫 책은 바로 [피플웨어](http://www.yes24.com/Product/Goods/13657193){: target="\_blank" }이다. 소프트웨어를 개발하면서 기술이 아니라 사람의 관점에서 이야기를 해주는 책인데, 요즘 챕터장을 하면서 고민하는 부분에 대한 혜안을 얻을 수 있지 않을까 싶어서 구매해 보았다. 이 글을 적는 현재 다 읽어서 조만간 후기글을 적어보려고 한다. 앞으로 좀더 많이 책을 구매해야겠다.

아래는 2월동안 정리한 이슈 내용들이다.

## `SpringBootTest.WebEnvironment.RANDOM_PORT`

Spring Boot 환경에서 통합/기능 테스트 시 `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`를 사용하곤한다. 해당 기능을 사용할 때 주의해야할 점은 테스트가 실행될 때 별도의 스레드에서 기능이 수행되기 때문에 테스트의 `@Transactional`을 사용하여 데이터를 초기화 하는 등의 혜택을 누릴 수 없다. 어찌보면 당연한게 `SpringBootTest.WebEnvironment.RANDOM_PORT`는 내부에서 별도의 웹서버를 띄워서 기능을 수행하는 것이기 때문에 요청에 대한 별도의 스레드가 생성이 되는데 이는 동일 스레드 내에서만 동작하는 `@Transactional`이 동작할 수 없는 구조이다. 그래서 테스트 시 데이터 검증에 유의하거나 데이터를 초기화 해주는 등의 코드를 넣어줘야 한다.

만약에 `@Transactional`을 사용하고 싶다면 `SpringBootTest.WebEnvironment.MOCK`를 사용하자.

## `flyway.clean()`

위 내용의 연장인데, 데이터 초기화 시 번거롭게 각 데이터를 삭제하는 로직을 넣지 않고 `flyway`의 `clean`함수를 이용하여 데이터를 초기화할 수 있다. (물론 flyway를 사용하는 경우에)

```kotlin
@TestComponent
@Transactional
internal class TestDatabase(private val flyway: Flyway) {
    fun clearAllSchema() {
        flyway.clean()
    }

    fun createAllSchema() {
        flyway.migrate()
    }
}
```

## JPA foreign key

JPA를 이용하여 영속성 데이터 관리 시 대부분의 경우 lazy loading을 사용한다. 그래서 아래와 같이 Entity가 정의되어 있는 경우 Child가 Parent를 조회하면 조회하는 시점에 부모 조회 쿼리가 실행된다. 하지만 만약 Prent의 ID만 조회하는 경우 Parent를 조회하는 쿼리는 실행되지 않는다. 그 이유는 어찌보면 당연할 수 있는데, Child가 `parent_id`를 가지고 있기 때문이다. `child.parent`를 호출하면 무조건 Parent를 조회하는 쿼리를 실행할 줄 알았는데 컬럼에 따라 조회여부를 판단하는 것 같다.

```kotlin
class Parent {
    @Id
    val id: Long = 0L

    @Column
    val name: String = ""

    @OneToMany(mappedBy = "paernt")
    val children: MutableList<Child> = muableListOf()
}

class Child {
    @Id
    val id: Long = 0L

    @Column
    val name: String = ""

    @ManyToOne(optional = false)
    @JoinColumn(name = "parent_id")
    val parent: Parent? = null
}
```

```kotlin
fun test() {
    val child = childRepository.findById(id)

    print(child.parent.name) # <-- parent 쿼리 실행
}

fun test2() {
    val child = childRepository.findById(id)

    print(child.parent.id) # <-- parent 쿼리 실행하지 않음
}
```

## DGS 코드젠 Class property 이슈

[DGS의 Codegen](https://github.com/Netflix/dgs-codegen){: target="\_blank" }을 이용하여 Schema 데이터 클래스들을 생성하는 경우 kotlin 클래스를 사용하면 DataClass가 만들어지기 때문에 생성자에 모든 프로퍼티값을 넣어주어야 한다. 이때 특정 필드를 조회 시에만 데이터를 조 할 수 있도록 하는 필드 리졸버를 구현할 수 없다는 문제점이 생긴다. 왜냐하면 root의 Schema의 생성자에서 모든 값을 넣어줘야 하기 때문이다. 그래서 선택할 수 있는 옵션이 `kotlinAllFieldsOptional`이다.

해당 옵션은 코드젠에서 생성되는 모든 DataClass의 모든 프로퍼티가 nullable하게 생성하도록 한다. 모든 프로퍼티가 nullable하기 때문에 필드 리졸버로 구현할 프로퍼티는 null값으로 넣어주고 추후 필드 리졸버에서 바인딩 될 수 있도록 할 수 있다. 다만, 아쉬운 점은 모든 프로퍼티가 nullable하기 때문에 필수값의 여부는 스키마를 통해서 확인할 수 있어 개발 시 스키마를 꼼꼼하게 챙겨서 봐야한다.

아래 코드는 설정 예시이다.

```kotlin
tasks.withType<GenerateJavaTask> {
    packageName = "com.spoqa.cart.generated"
    language = "kotlin"
    kotlinAllFieldsOptional = true
}
```

## Kotest nested test transaction

Kotest의 `4.2.4` 이전 버전에서는 Spring의 `@Transactional`이 nested test에서는 정상 동작하지 않았다. 그래서 `@DataJpaTest`와 같이 `@Trasactional`을 이용하는 테스트는 부득이하게 nested test가 아닌 단일 레벨의 테스트를 사용했어야 하는데 `4.2.4`버전 부터는 netsted test에서도 `@Transactional`이 동작하도록 옵션을 줄 수 있게 되었다.

```kotlin
class FooTest : FunSpec() {
    override fun extensions() = listOf(SpringTestExtension(SpringTestLifecycleMode.Root))

    // 이하 생략...
}
```

Github Issue : [https://github.com/kotest/kotest/issues/1643#issuecomment-686549912](https://github.com/kotest/kotest/issues/1643#issuecomment-686549912){: target="\_blank" }

## Infix notation

Kotlin에 [Infix notation](https://kotlinlang.org/docs/functions.html#infix-notation){: target="\_blank" }이 있다. kotest에서 단언시 항상 사용했던 문법인데 직접 사용해보진 않고 있다가 이번에 동료가 사용하는 것을 보고 좀 더 자세히 알아보게 되었다.

익숙해진다면 코드량을 줄이고 가독성을 향상시킬 수 있다는 점에서 장점을 가진다 생각한다. 다만 매개변수가 반드시 하나여야한다는 제약조건이 있으므로 매개변수의 수가 변경 가능성이 크거나 하나 이상인 경우에는 사용하는 것을 고민해봐야 될 듯 하다.

## `@Language`

Intellij에서 제공해주는 annotation 중 [@Language](https://www.jetbrains.com/help/idea/using-language-injections.html#language_annotation){: target="\_blank" }라는 것이 있다.

해당 어노테이션을 사용하면 문자열 입력 시 Intellij에서 해당 문법에 대한 코드지원을 받을 수 있다.

```kotlin
@Language("HTML")
val html = "<body><h1>Hello, World</h1></body>"
```

## Test/Compile Task Memory Setting

테스트나 컴파일 시 Memory 설정을 하려면 아래와 같이 `Task`내에서 메모리 설정을 해주어야 한다. `gradle.properties`의 `JAVA_OPTS=-Xmx2g` 옵션은 테스트와 컴파일과는 별개로 Client VM의 힙메모리의 설정이다.

```kotlin
tasks.withType<KotlinCompile> {
    kotlinOptions {
        kotlin
        freeCompilerArgs = listOf("-Xjsr305=strict -Xmx2g") // <--- 여기
        jvmTarget = "17"
    }
    dependsOn(tasks.withType<GenerateJavaTask>())
}

tasks.withType<Test> {
    this.maxHeapSize = "2g" // <--- 여기
    useJUnitPlatform()
}
```

## JPA Converter autoApply

JPA의 Entity 프로퍼티를 공통적으로 타입 변환해줄 때 Converter를 주로 사용하곤 한다. 이때 아래와 같이 `autoApply` 옵션을 사용하면 Entity 마다 컬럼에 Converter를 명시해주지 않아도 된다.

```kotlin
@Converter(autoApply = true)
class PhoneNumberConverter : AttributeConverter<Phonenumber.PhoneNumber, String> {
    override fun convertToDatabaseColumn(attribute: Phonenumber.PhoneNumber?): String? {
        return attribute?.toE164String()
    }

    override fun convertToEntityAttribute(dbData: String?): Phonenumber.PhoneNumber? {
        return dbData?.toKoreaPhoneNumber()
    }
}

class Foo(
    @Id
    val id: Long,

    @Column
    val phoneNumber: PhoneNumber, # 해당 컬럼은 DB 저장 시 문자열로 저장된다.
)
```

## JPA Entity에서 PK를 UUID로 사용하고 Entity 생성 시 null이 아닌 random 값으로 할당 할 때 발생할 수 있는 이슈

사내 프로젝트를 진행하면서 `isNew`를 재정의하여 사용하면서 겪었던 이슈를 정리해본다. 간단하게만 정리하고 추후 자세하게 별도의 글로 적어보겟다.

### 기본적으로 `isNew`가 `false`로 판단한다.

`AbstractEntityInformation#isNew`함수를 보면 아래와 같이 `null` 또는 `0`이면 `isNew`를 `true`로 판단한다.

```java
public boolean isNew(T entity) {
    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
	    return id == null;
    }

    if (id instanceof Number) {
	    return ((Number) id).longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```

### `isNew`를 `false`로 판단하면 저장 시 채번을 하기 때문에 성능이 좋지 않다.

`SimpleJpaRepository#save`함수를 보면 `isNew`가 `false`면 `merge`를 실행한다.

```java
// SimpleJpaRepository.java
@Transactional
    @Override
    public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) {
	    em.persist(entity);
	    return entity;
    } else {
	    return em.merge(entity);
    }
}
```

### `isNew`를 `false`로 판단하면 CASCADE 시 PERSIST가 동작하지 않는다.

연관 entity를 root에서 생성해주는 방법으로 CASCADE를 사용하곤하는데 새로운 연관 entity를 생성해서 root entity에서 함께 저장을하면 CASCADE가 되지 않는 현상을 발견했다. 즉, ID값이 이미 존재하도록 entity를 생성하면 persist event가 아니라 merge event가 발생되고 cascade도 `cascadeOnMerge` 함수가 실행된다.

```java
// SessionImpl.java
private Object fireMerge(MergeEvent event) {
    try {
	    // 생략 ...
	    fastSessionServices.eventListenerGroup_MERGE.fireEventOnEachListener( event, MergeEventListener::onMerge );
	    // 생략 ...
    }
    // 이하 생략 ...

    return event.getResult();
}
```

```java
// DefaultMergeEventListener.java
protected void cascadeOnMerge(
	final EventSource source,
	final EntityPersister persister,
	final Object entity,
	final Map copyCache
) {
    final PersistenceContext persistenceContext = source.getPersistenceContextInternal();
    persistenceContext.incrementCascadeLevel();
    try {
	    Cascade.cascade(
			    getCascadeAction(),
			    CascadePoint.BEFORE_MERGE,
			    source,
			    persister,
			    entity,
			    copyCache
	    );
    }
    finally {
	    persistenceContext.decrementCascadeLevel();
    }
}
```

### 그렇다고 `isNew`를 `true`로 강제하면 delete가 동작하지 않는다.

`CrudRepository`에서 `delete`함수를 호출할때 `isNew`가 `true`면 리턴해버리는 코드가 있다. 이는 아마 같은 Transaction 내에서 entity를 생성했다가 삭제하는 경우 flush 할 필요가 없기 때문에 최적화 or DB오류방지를 위한 동작이라 생각이 든다. 하지만 등록하자마자 삭제하는 사람이 있을까...싶기는 하다

```java
// SimpleJpaRepository.java
@Override
@Transactional
@SuppressWarnings("unchecked")
public void delete(T entity) {

    Assert.notNull(entity, "Entity must not be null!");

    if (entityInformation.isNew(entity)) {
	    return;
    }

    Class<?> type = ProxyUtils.getUserClass(entity);

    T existing = (T) em.find(type, entityInformation.getId(entity));

    // if the entity to be deleted doesn't exist, delete is a NOOP
    if (existing == null) {
	    return;
    }

    em.remove(em.contains(entity) ? entity : em.merge(entity));
}
```

### 그럼 기본 ID를 가지면서 persist가 잘 동작하게 하려면?

`delete`를 재정의 하는 방법은 `cascade`문제 때문에 수행할 수 없다. 그렇다고 `ID`를 `nullable`한 값으로 주는 것은 Entity의 가장 중요한 값을 더럽히는 근시안적인 방법이다. 그리고 ID의 생성을 Generated에 맡기는 것은 DB에 의존적인 코드를 양상할 뿐만 아니라 Test 시 flush를 염두해 두고 해야하기 때문에 좋은 방법이 아니다.

그래서 Jpa Entity Lifecycle Events를 활용해보기로 했다.

참고: [https://www.baeldung.com/jpa-entity-lifecycle-events](https://www.baeldung.com/jpa-entity-lifecycle-events){: target="\_blank" }

등록시에는 `isNew`가 `true`이고 조회된 `Entity`는 `isNew`가 `false`가 되어서 삭제가 정상동작하게 하려면 `@PostLoad`를 사용하면 된다.

```kotlin
@MappedSuperclass
abstract class PrimaryKeyEntity(
    @Id
    private val id: UUID = UlidCreator.getMonotonicUlid().toUuid(),
) : Persistable<UUID>, Cloneable {
    @Transient
    private var _isNew = true

    override fun getId(): UUID = id

    override fun isNew(): Boolean = _isNew

    @PostLoad
    protected fun load() {
        _isNew = false
    }
}
```

## Apache tika

아파치 티카는 PPT, CSV, PDF등 다양한 형태의 파일 메타 데이터와 텍스트를 감지하고 추출하는 라이브러리이다. 자세한 내용은 아래 글을 참고하자.

[https://kingname.tistory.com/214](https://kingname.tistory.com/214){: target="\_blank" }
