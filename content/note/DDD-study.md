---
title: "DDD 핵심 정리"
author: ["조민준"]
date: 2022-12-30T17:10:50+09:00
draft: false
tags: ["DDD"]
ShowToc: true
---

## 0. 참고

- 도메인 주도 개발 시작하기

  - [도메인 주도 개발 시작하기 - YES24](http://www.yes24.com/Product/Goods/108431347)

- 핵사고날 아키텍처

  - [Hexagonal Architecture with Java and Spring](https://reflectoring.io/spring-hexagonal/)

  - [https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/)

- 예제 프로젝트

  - [https://github.com/jo-minjun/order-delivery-project](https://github.com/jo-minjun/order-delivery-project)

## 1. 도메인이란 무엇일까?

### wikipedia: Domain (software engineering)

> A domain is the targeted subject area of a [computer program](https://en.wikipedia.org/wiki/Computer_program). It is a term used in [software engineering](https://en.wikipedia.org/wiki/Software_engineering).
> Formally it represents the target subject of a specific programming project, whether narrowly or broadly defined.

1. 소프트웨어 엔지니어링에서 사용되는 용어
2. 컴퓨터 프로그램의 대상이 되는 영역

### Example

- 소프트웨어 프로젝트의 목표가 특정 병원을 위한 프로그램을 만드는 경우
- 범위를 확장하여 모든 병원을 대상으로 하는 프로그램을 만드는 경우
- 상점과 기사를 이어주고 고객에게 물품을 전달해주는 라스트마일 서비스

## 2. DDD란 무엇일까?

- Domain Driven Design이다.
- 프로그램을 도메인별로 나누어 설계하는 방법
- 모듈(도메인)간 응집도는 높이고, 결합도는 낮춰 준다.
- DDD의 목표를 달성하기 위해 전략적 설계 패턴과 전술적 설계 패턴을 사용한다.

## 3. 왜 DDD를 할까?

- 복잡도 관리
  - 시간 경과에 따라 코드 라인이 늘어나고, 변경 비용이 증가한다.
    ![1](../DDD-study/1.png)
    [https://dreamix.eu/blog/java/why-good-clean-software-architecture-matters](https://dreamix.eu/blog/java/why-good-clean-software-architecture-matters)
- 개발자는 특정 도메인의 전문가보다 도메인에 대한 전문성이 떨어진다.
  - 공인 중개사와 개발자
  - 변호사와 개발자
  - 인사팀과 개발자
  - …
- 전문가와 기획자, 개발자의 언어가 다르다.
  - 도저히 이해할 수 없는 말들…
  - 네트워크 광전송 장비: OTN, PTN, ROADM, SERVICE, TUNNEL…

→ 최소 문서를 읽거나 대화할 때 서로가 하는 말을 이해하고 context를 맞춰 나가야 한다.

## 4. 전략적 설계

### 유비쿼터스 언어

- 도메인 전문가, 기획자, 개발자 등 구성원들이 서로 다른 용어를 사용하면, 의사소통에 불편함이 있다.
  - 지번주소 vs 구주소

→ **유비쿼터스 언어**를 사용해야 한다.

- 구성원들 모두가 **보편적으로 사용하는 언어**
- 구성원들의 공통된 언어를 만들고 대화, 문서, 코드, 테스트 **모든 곳에서 같은 용어**를 사용한다.

### 도메인 모델과 경계

- 다시 도메인에 대해 짚어보자면
  - 소프트웨어 프로젝트에서 대상이 되고, 해결해야 할 영역
  - 온라인 쇼핑몰을 개발하는 프로젝트
- 도메인은 다시 하위 도메인으로 나뉘어 진다.
  - 회원, 혜택(쿠폰), 주문, 카탈로그, 배송, 결제…
- **도메인 모델**
  - 특정 도메인을 개념적으로 표현한 것
  - 도메인에 대한 이해도에 따라 도메인 모델도 변경된다.
    ![2](../DDD-study/2.png)
- 위와 같은 서브 도메인을 하나의 도메인으로 표현하기는 불가능에 가깝다.
- 서브 도메인마다 같은 대상이라도 지칭하는 용어가 다를 수 있다.

→ Problem Space가 된다.

- 상품
  - 카탈로그의 상품: 이미지, 상품명, 가격…
  - 배송의 상품: 무게, 수량…
- 회원
  - 회원 도메인의 회원: 회원
  - 주문 도메인의 회원: 주문자
  - 배송 도메인의 회원: 받는 사람
- 즉, 모델은 특정한 컨텍스트 하에서 완전한 의미를 갖는다.

→ 각 서브 도메인마다 **명시적으로 구분되는 경계**를 가져서 섞이지 않도록 해야 한다.

### 바운디드 컨텍스트

- 각 도메인 영역의 경계를 결정하는 **명시적인 구분**
- 각각의 도메인이 가진 모델을 정확하게 표현하기 위함이다.

→ 즉 문제를 해결하기 위한 공간, Solution Space이다.

- 바운디드 컨텍스트를 구분하는 조건
  - 같은 용어, 다른 의미
    - **계정**을 의미하는 Account
    - **계좌**를 의미하는 Account
      → 이런 경우 두 가지 의미를 하나의 도메인 모델에 포함해서는 안된다.
  - 같은 개념, 다른 용도
    - 회원 서비스의 **맴버**
    - 주문 서비스의 **맴버**
      → 맴버는 서로 다른 도메인에 집중하고 있고, 발전의 방향성도 다르다.
  - 팀 조직 구조
    - A팀의 관심사는 주문, B팀의 관심사는 결제.
      - 하나의 주문 도메인에서도 관심사에 따라 컨텍스트가 달라진다.
    - 한 팀이 하나의 시스템에서 온라인 쇼핑을 서비스한다.
      - 서브 도메인은 회원, 카탈로그, 재고, 구매, 결제 등이 있다.
      - 상품 컨텍스트에서 재고와 카탈로그를 구현한다.
- 이상적으로는 바운디드 컨텍스트와 하위 도메인이 1대1로 대응되는 것이 좋다.
- 하지만 팀 상황이나 유비쿼터스 언어가 명확하게 정의되지 않아 1대1로 대응되지 않는 경우도 있다.
  ![3](../DDD-study/3.png)

### 바운디드 컨텍스트 간 관계

- 바운디드 컨텍스트는 어떻게든 연결되기 때문에 다양한 방식으로 관계를 형성한다.
  - 고객/공급자
  - 공유 커널
  - 독립 방식
- **고객/공급자**
  - 가장 흔한 관계이다.
  - 한쪽에서 **API를 제공(상류)**하고 다른쪽에서 **API를 호출(하류)**한다.
    ![카탈로그 바운디드 컨텍스트는 추천 바운디드 컨텍스트에 의존한다.](../DDD-study/4.png)
    카탈로그 바운디드 컨텍스트는 추천 바운디드 컨텍스트에 의존한다.
- **공유 커널**
  - 여러 바운디드 컨텍스트가 **같은 모델을 공유**하는 관계이다.
  - 중복을 줄일 수 있지만 공유 모델을 사용하는 바운디드 컨텍스트가 서로 영향을 받을 수 있다.
- **독립 방식**
  - 여러 바운디드 컨텍스트가 **외부에 의해 관계**를 맺는다.
    - 수동으로 두 바운디드 컨텍스트 간 통합시킨다.
      - 사람에 의한 관계
    - 자동화 시스템을 개발해서 두 바운디드 컨텍스트를 통합시킨다.
      - 자동화 시스템에 의한 관계

### 바운디드 컨텍스트 맵

- 특정 바운디드 컨텍스트에 과도하게 집중하면 전체적인 바운디드 컨텍스트 간의 관계를 인식하지 못할 수 있다.
- 도메인을 더 잘 이해하거나 컨텍스트 간 관계가 바뀌면 컨텍스트 맵도 바뀐다.
  ![5](../DDD-study/5.png)

## 5. 핵사고날 아키텍처

- 예제 프로젝트에 핵사고날 아키텍처를 적용했다.
  ![6](../DDD-study/6.png)
  [https://reflectoring.io/spring-hexagonal/](https://reflectoring.io/spring-hexagonal/)

```java
└── xxx
    ├── adapter
    │   ├── in
    │   │   ├── xxxController.java
    │   │   └── EventHandler.java (or MessageHandler.java)
    │   └── out
    │       └── EventPublisher.java (or MessagePublisher.java)
    ├── application
    │   ├── xxxService.java
    │   └── port
    │       ├── in
    │       │   ├── xxxCommand.java
    │       │   ├── xxxDto.java
    │       │   └── xxxUsecase.java
    │       └── out
    │           ├── xxxEvent.java
    │           ├── xxxEventPublisher.java
    │           └── xxxRepository.java
    └── domain
        ├── AggregateRootEntity.java
        └── ValueObject.java
```

## 6. 전술적 설계

### 도메인 영역의 주요 구성 요소

| 요소                           | 설명                                                                                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 엔티티 (ENTITY)                | 고유의 식별자를 갖는 객체로 자신의 라이프 사이클을 갖는다. 도메인의 고유한 개념을 표현한다. 도메인 모델의 데이터를 포함하며 해당 데이터와 관련된 기능을 함께 제공한다. |
| 밸류 (VALUE)                   | 고유의 식별자를 갖지 않는 객체다. 엔티티의 속성으로 사용할 뿐만 아니라 다른 밸류 타입의 속성으로도 사용할 수 있다.                                                     |
| 애그리거트 (AGGREGATE)         | 애그리거트는 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다.                                                                                                |
| 리포지터리 (REPOSITORY)        | 도메인 모델의 영속성을 처리한다.                                                                                                                                       |
| 도메인 서비스 (DOMAIN SERVICE) | 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 도메인 로직이 여러 엔티티와 밸류를 필요로 하면 도메인 서비스에서 로직을 구현한다.                                    |

### 엔티티 & 밸류

- 도메인 모델을 표현할 때 이용한다.
- 도메인 모델의 엔티티는 기능을 함께 제공한다.

  - 도메인 관점에서 도메인 로직을 구현하고 캡슐화해서 데이터가 임의로 변경되는 것을 막는다.

  ```java
  package minjun.ddd.delivery.domain;

  public void changeDeliveryInfo(Address address, String phoneNumber) {
    canChangeDelivery();
    this.address = address;
    this.phoneNumber = phoneNumber;
  }

  private void canChangeDelivery() {
    if (!this.status.canChangeDelivery()) {
      throw new RuntimeException("배송 정보 수정 불가");
    }
  }
  ```

  - 외부에서 setter를 이용해 배송지 정보를 변경한다면 배송지 변경 가능 여부 검증이 누락될 수 있고, 같은 로직이 반복될 수도 있다.

  ```java
  final DeliveryStatus status = delivery.getStatus();

  // 배송지 변경 조건
  if (!this.status.canChangeDelivery()) {
    throw new RuntimeException("배송 정보 수정 불가");
  }
  delivery.setAddress(newAddress);
  delivery.setPhoneNumber(newPhoneNumber);
  ```

- 밸류는 도메인 모델에서 두 개 이상의 데이터가 개념적으로 하나인 경우 사용한다.

  ```java
  package minjun.ddd.order.domain;

  @Entity
  @Table(name = "orders")
  @Getter
  @NoArgsConstructor(access = AccessLevel.PROTECTED)
  @AllArgsConstructor
  @EqualsAndHashCode(of = {"id"})
  @ToString(of = {"id", "orderLine", "totalAmount", "deliveryId", "paymentId"})
  public class Order implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private OrderLine orderLine;

  	// 밸류 타입, AttributeConverter를 이용함.
    private Money totalAmount = Money.ZERO;

  	... // 다른 필드
  ```

  ```java
  package minjun.ddd.common;

  @Converter(autoApply = true)
  public class MoneyConverter implements AttributeConverter<Money, BigDecimal> {

    @Override
    public BigDecimal convertToDatabaseColumn(Money attribute) {
      return attribute.getValue();
    }

    @Override
    public Money convertToEntityAttribute(BigDecimal dbData) {
      return new Money(dbData);
    }
  }
  ```

- 엔티티는 @Entity 애너테이션을 사용한다.
- 밸류는 @Embeddable, @Embedded, @SecondaryTable, @ElementCollection, @CollectionTable을 사용한다.
  - @ElementCollection은 생명주기를 상위 엔티티에 종속시킨다.
  - 즉, cascade와 orphanRemoval 옵션을 제공하지 않는다.
  - @OneToMany(cascade = ALL, orphanRemoval = true)와 차이점
    - @ElementCollection은 식별자를 갖지 않는다.

```java
package minjun.ddd.order.domain;

@Embedded
private OrderLine orderLine;
```

```java
package minjun.ddd.order.domain;

@Embeddable
public class OrderLine {

  @ElementCollection
  @CollectionTable(name = "order_lines", joinColumns = @JoinColumn(name = "orders_id"))
  private Set<LineItem> lineItems = new HashSet<>();

  ... // 메서드
}
```

- @AttributeOverride를 이용해서 @Embeddable 밸류의 애트리뷰트를 override할 수도 있다.

### 애그리거트

- 도메인이 커지면 도메인 모델이 복잡해진다.
- 도메인 모델이 복잡해지면 전체 구조에 초점을 맞추지 못하게 되고, 모델 간에 관계를 이해하기 어렵게 된다.
- 애그리거트는 관련 객체를 묶어서 **상위 개념으로 표현**해준다.
  - Order 도메인은 주문, 주문 목록, 총 결제 금액 등 하위 모델로 구성된다.
  - 이를 하나로 묶어 **주문**이라는 상위 개념으로 표현해준다.
- 애그리거트는 루트 엔티티를 가지며, 이를 애그리거트 루트라 한다.
- 애그리거트 루트는 애그리거트에 속해 있는 엔티티와 밸류 객체를 이용해서 애그리거트가 구현해야 할 기능을 제공한다.

  - 애그리거트의 내부 구현을 숨겨서 애그리거트 단위로 구현을 캡슐화 한다.

  ```java
  package minjun.ddd.delivery.domain;

  @Entity
  ... // 애너테이션
  public class Delivery {

  	@Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

  	@Embedded
    private Address address;

    private String phoneNumber;
  	... // 다른 필드

  	public void changeDeliveryInfo(Address address, String phoneNumber) {
      canChangeDelivery();
      this.address = address;
      this.phoneNumber = phoneNumber;
    }
  ```

### 리포지터리

- 리포지터리는 애그리거트 단위로 도메인 객체를 저장하고 조회한다.

```java
package minjun.ddd.order.application.port.out;

import minjun.ddd.order.domain.Order;
import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderRepository extends JpaRepository<Order, Long> {

}
```

- 애그리거트의 **루트 엔티티만 리포지터리를 갖는다.**

  - 애그리거트의 밸류 등은 루트 엔티티와 생명 주기가 같다.
  - **생명 주기가 다르거나 데이터 변경 주체가 다르다면 다른 애그리거트일 가능성이 높다.**

  ```java
  package minjun.ddd.product.application.port.out;

  import minjun.ddd.product.domain.Product;
  import org.springframework.data.jpa.repository.JpaRepository;

  public interface ProductRepository extends JpaRepository<Product, Long> {

  }
  ```

  - Product 애그리거트는 Order나 Delivery와 연관이 있기 때문에 같은 애그리거트로 생각될 수 있다.
  - 하지만 Order나 Delivery는 변경 주체가 주문자와 기사이지만, Product는 상품 관리자가 관리한다.

### 도메인 서비스

- 도메인 영역을 개발하다 보면 **한 애그리거트로 기능을 구현하지 못할 때**가 있다.
- 결제 금액 계산 로직에 할인이 적용되는 경우
  - 할인 쿠폰 애그리거트: 쿠폰별로 지정한 금액이나 비율에 따라 총 금액을 할인한다.
  - 회원 애그리거트: 회원 등급에 따라 추가 할인이 가능하다.
- 주문 애그리거트에 할인 관련 로직을 적용하면 할인 정책 변경시 주문 애그리거트가 변경된다.

→ **도메인 서비스 사용**

```java
public class DiscountCalculationService {

	public Money calculateDiscountAmounts(List<OrderLine> orderLines,
																				List<Coupon> coupons,
																				MemberGrade grade) {
		Money couponDiscount = coupons.stream()
												.map(coupon -> calculateDiscount(coupon))
												.reduce(new Money(0), (v1, v2) -> v1.add(v2));

		Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());

		return couponDiscount.add(membershipDiscount);
	}

	private Money calculateDiscount(Coupon coupon) {
		...
	}

	private Money calculateDiscount(MemberGrade grade) {
		...
	}
}
```

- 외부 시스템이나 타 도메인과 **연동 기능**도 도메인 서비스가 될 수 있다.
- 상품 관리 시스템에서 사용자가 권한을 가졌는지 확인하기 위해 맴버 시스템을 연동하는 경우

  ```java
  public interface PermissionChecker {
  	boolean hasUserPermission(String userId);
  }
  ```

  - 여기서 인터페이스는 외부 시스템의 역할을 표현하기 위해 도메인 로직 관점에서 작성한다.

  ```java
  public class CreateProductService {
  	private PermissionChecker permissionChecker;

  	public Long createProduct(CreateProductRequest req) {
  		validate(req);
  		if (!permissionChecker.hasUserPermission(req.getUserId)) {
  			throw new NoPermissionException();
  		}
  		... // 생성
  	}
  }
  ```

### 이벤트

- API를 이용하는 방법은 **시스템간 결합 문제**를 발생시킨다.
  - 외부 서비스의 성능
  - 트랜잭션 처리 정책
  - 설계상 문제
- **외부 서비스의 성능**
- **트랜잭션 처리 정책**
  1. 환불 외부 서비스에서 익셉션이 발생하면 주문까지 트랜잭션을 롤백한다.
  2. 주문만 취소 상태로 변경하고 환불은 나중에 처리한다.
- **설계상 문제**

  ```java
  package minjun.ddd.order.application.service;

  @Service
  @RequiredArgsConstructor
  @Transactional
  public class OrderService implements OrderUsecase {

  	private final PaymentPort paymentPort;

  	@Override
    public void cancelOrder(Long orderId) {
      final Order order = findOrder(orderId);

  		// Payment 도메인 로직
      final Boolean responseFromPayment = paymentPort.cancelPayment(order.getPaymentId());
      if (!responseFromPayment) {
        throw new RuntimeException("결제 취소 실패");
      }

  		// Order 도메인 로직
      order.cancelOrder();
    }
  }
  ```

  - 다른 도메인의 로직이 섞이고, 트랜잭션 처리 정책 및 외부 서비스 영향이 증가한다.

**→ 시스템의 결합도를 낮추기 위해 이벤트를 사용한다.**

- 이벤트는 **과거에 벌어진 어떤 것**을 의미하며, **상태가 변경**됐다는 것을 의미한다.
- 이벤트는 다음과 같이 네 개의 구성요소를 가진다.
  ![7.png](../DDD-study/7.png)

  - 이벤트
    - 이벤트 종류, 발생 시간, 이벤트 관련 정보
  - 이벤트 생성 주체

    - 도메인 로직을 실행해서 **상태가 바뀌면** 관련 이벤트를 발생시킨다.

    ```java
    package minjun.ddd.delivery.application;

    @Override
    public void startDelivery(Long deliveryId) {
      final Delivery delivery = deliveryRepository.findById(deliveryId)
          .orElseThrow(NoSuchElementException::new);

      delivery.startDelivery();
      deliveryEventPublisher.publish(new DeliveryStartedEvent(delivery));
    }
    ```

  - 이벤트 디스패처

    - 핸들러에 **이벤트를 전파**한다.

    ```java
    package minjun.ddd.delivery.adapter.out;

    // org.springframework.context.ApplicationEventPublisher
    private final ApplicationEventPublisher publisher;

    @Override
    public void publish(DeliveryEvent event) {
      publisher.publishEvent(event);
      log.info("Order Event Published: {} {}", event.getDelivery(), event.getTimestamp());
    }
    ```

  - 이벤트 핸들러

    - 이벤트 생성 주체가 발생시킨 이벤트에 반응한다.

    ```java
    package minjun.ddd.order.adapter.in;

    @EventListener(DeliveryStartedEvent.class)
    public void handleDeliveryStartedEvent(DeliveryStartedEvent event) {
      orderUsecase.startDelivery(event.getDelivery().getOrderId());
    }
    ```

- 위와 같은 방법이 아닌, **메시징 시스템을 이용한 방법**도 가능하다.
  - 이벤트 저장소를 이용한 메시징 (**Transactional Outbox Pattern**)
  - 이벤트 발행 서비스
    ![transactional-outbox-pattern.png](../DDD-study/8.png)
  - 이벤트 소비 서비스
    ![idempotent-receiver.png](../DDD-study/9.png)
