---
title: JPA/Hibernate 관계 매핑을 위한 궁극적인 모범 사례 가이드
categories:
  - Entity
feature_text: |
  ## 그리고 양방향 관계의 위험성을 이해합니다.
---

#### Goal

> Spring의 Hibernate/JPA에 대한 시리즈의 네 번째 회차에 오신 것을 환영합니다! 마침내 몇 가지 모범 사례에 대해 이야기할 수 있게 되었습니다. 오늘은 JPA 관계 매핑에 대한 모범 사례를 다루겠습니다.

지금까지 우리는 다음을 다루었습니다.

Spring에서 JPA를 사용하는 다양한 방법 탐색
다양한 페치 전략 이해 및 N+1 방지
트랜잭션 대 JPA 세션 경계
(이 기사) 관계 매핑에 대한 모범 사례
(미래) 마지막 기사: 도메인 주도 개발을 사용하여 엔터티 설계 및 JPA 사용 간소화를 위한 기타 휴리스틱
이 글은 여러분이 이전에 다룬 주제에 이미 익숙하다고 가정합니다. 처음 세 개의 글은 여기에서 확인할 수 있습니다.

#### 소개

이 문서에서는 각 JPA 매핑 유형과 관련된 모범 사례를 다루며, 이를 따르면 일반적인 JPA 함정 중 일부를 줄일 수 있습니다. 최적화 또는 성능 튜닝에 대한 내용이 아닙니다. 이러한 경우 Vlad Mihalcea 와 Thorben Jenssen 의 리소스를 추천합니다.

JPA는 네 가지 주요 관계 유형을 지원합니다.

One To One
Many To One
One to Many
Many to Many
각각의 모범 사례를 하나씩 살펴보겠습니다.

#### OneToOne Association

![OneToOne](/assets/11.webp.png)

Example of OneToOne

```java
@Entity
@Data
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  Long id;

  @OneToOne
  OrderDetail orderDetail;
}

@Entity
@Data
public class OrderDetail {

  @Id
  @GeneratedValue
  Long id;

  String extraDetail;
  String extraDetail2;
  String extraDetail3;

}
```

가장 좋은 매핑은 매핑을 전혀 하지 않는 것입니다. OneToOne 관계의 경우, 처음부터 사용하지 않는 것이 가장 좋습니다. OneToOne (마치 ) 관계는 N+1 문제ManyToOne 로 이어질 가능성이 높습니다 . 따라서 처음부터 사용하지 않으면 많은 문제도 피할 수 있습니다 .

```java
@Entity
@Data
@Table(name = "ORDERS")
public  class  Order {

  @Id
  @GeneratedValue
   Long id;

  String extraDetail;
  String extraDetail2;
  String extraDetail3;
}
```

OneToOne 많은 경우, 사람들 이 비즈니스 로직에 맞는 그룹으로 열을 분할하고 싶을 때 관계를 사용하는 것을 보았습니다 . 효율성을 높이는 것으로 가정합니다. 또는 단순히 엔터티 클래스/테이블에서 많은 수의 필드/열을 처리하고 싶지 않을 수도 있습니다. 최신 RDBMS는 많은 수의 열을 처리할 수 있으므로 대부분의 경우 이러한 결정은 물리적 제한이 아니라 정신적 제한에 따라 이루어집니다.

가능하다면, 로 연결된 두 개의 테이블을 만드는 대신 OneToOne모든 열이 있는 단일 테이블을 만드세요. Entity 클래스에 열이 수백 개가 포함되지 않도록 열을 그룹화하려면 @Embeddable클래스를 사용하여 필드를 그룹화하세요.

```java
@Entity
@Data
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  Long id;

  @Embedded
  OrderDetail orderDetail;
}

@Embeddable
public class OrderDetail {

  String extraDetail;
  String extraDetail2;
  String extraDetail3;

}
```

물론, 이러한 경험적 방법이 효과가 없는 상황도 있습니다.

- OneToOne 동일한 테이블과 관계 가 있는 경우 (예: Person <-> Spouse)
- 관련 테이블의 열의 데이터 크기가 너무 커서(즉, blobs열) 지연 페치해야 합니다. 이 경우 실제로 LAZY 로딩을 사용하고 있는지 확인하세요.
  실제로 대부분의 관계는 단방향 관계로 사용됩니다. 양방향 관계가 정말 필요한 경우 관계의 소유자 측을 처리하지 않기 위해 를 OneToOne사용하여 두 엔터티가 동일한 기본 키를 공유할 수 있는지 확인하세요 .@PrimaryKeyJoinColumn

![OneToOne](/assets/12.webp.png)

관계의 소유자 측은 무엇을 의미합니까? 다음 섹션에서 이에 대해 논의합니다.

#### Bidirectional and Owner side of Relationship

다른 관계 유형을 살펴보기 전에 관계의 소유자 측이 무엇인지 알아봐야 합니다. JPA에서 관계의 소유자 측은 기본 데이터베이스에서 외래 키를 소유한 엔터티입니다. 이 측은 관계의 업데이트, 삽입 및 삭제를 관리할 책임이 있습니다.

OneToMany 예를 들어, <->를 사용하여 Order와 OrderItem 사이에 양방향 관계가 있다고 가정해 보겠습니다.ManyToOne

```java
@Entity
@Data
 public class Order {

  @Id
  @GeneratedValue
   Long id;

  @OneToMany (mappedBy = "order" )
  List<OrderItem> orderItems = new ArrayList<>();

}


@Entity
@Data
 public class OrderItem {

  @Id
  @GeneratedValue
   Long id;

  Long  Quantity ;

  @ ManyToOne
  Order  order ;

}
```

다음은 OrderItem 외래 키 열이 포함된 테이블이 있는 관계의 소유자 측입니다. order_id.비소유 측에는 관계의 소유자 측에 대한 위임을 정의하는 추가 Order속성이 있습니다 .mappedByOneToManyOrderItem

즉, Order 와 사이의 관계를 생성/수정할 때 OrderItem유일하게 관련된 관계는 의 관계입니다 OrderItem.

예를 들어,

```java
Order order = new Order();
... save(order);

OrderItem orderItem = new OrderItem();
... save(orderItem);

order.getOrderItems().add(orderItem);

...save(order);

// Does not create association

...

// LATER in a separate Transaction.

Order order = orderRepository.findById(..);
order.getOrderItem().size() // is 0
```

```java
Order order = new Order();
... save(order);

OrderItem orderItem = new OrderItem();
... save(orderItem);

orderItem.setOrder(order);

...save(order);

...

// LATER in a separate Transaction.

Order order = orderRepository.findById(..);
order.getOrderItem().size() // is 1
```

그렇다면 양방향 관계에서 연관 관계를 만들 때 관계의 소유 측 설정에만 집중해야 할까요? 답은 '아니요'입니다. 항상 양쪽을 설정해야 합니다.

#### Transactions and Flushing

양방향 관계의 양쪽을 모두 설정해야 하는 이유를 설명하려면 먼저 트랜잭션(Transaction)과 플러싱(Flushing)에 대해 설명해야 합니다.

save vs 의 차이점이 무엇 saveAndFlush 인지 궁금했을 것입니다 JpaRepository. 가장 큰 차이점은 트랜잭션 블록 내에서 save트랜잭션이 종료될 때까지 실제로 DB에 아무것도 쓰지 않는 반면, saveAndFlushDB에 즉시 삽입/업데이트를 한다는 것입니다.

무엇을 하는지 알지 못하는 한 항상 save(이 기사에서는 다루지 않음) 을 사용하려고 노력하십시오.

트랜잭션 내에 있는 경우 JPA 트랜잭션 컨텍스트는 플러시되지 않은 엔터티를 추적하고 사용자가 해당 엔터티를 쿼리하면 반환합니다.

```java
@Transaction  // within Tx block
...

Order order = new Order();
... save(order);

OrderItem orderItem = new OrderItem();
... save(orderItem);

orderItem.setOrder(order);

...save(order);

Long orderId = order.getId();

... // LATER STILL IN THE SAME Tx block (in separate part of the code)

Order order = orderRepository.findById(orderId); // retrieves the same unflushed Order object from tx context


order.getOrderItem().size() // still 0
```

```java
@Transaction  // within Tx block
...

Order order = new Order();
... saveAndFlush(order);

OrderItem orderItem = new OrderItem();
... saveAndFlush(orderItem);

orderItem.setOrder(order);

...saveAndFlush(order);

Long orderId = order.getId();

... // LATER STILL IN THE SAME Tx block (in separate part of the code)

Order order = orderRepository.findById(orderId); // retrieves from DB

order.getOrderItem().size() // 1
```

#### 도우미 메서드 사용

이러한 모호성은 도우미 메서드를 사용하여 연결을 설정하면 극복할 수 있습니다.

```java
@Entity
@Data
public  class  Order {

  @Id
  @GeneratedValue
   Long id;

  @OneToMany(mappedBy = "order")
   List<OrderItem> orderItems = new  ArrayList <>();

  public  void  addOrderItem (OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder( this );
  }

}

@Entity
@Data
public  class  OrderItem {

  @Id
  @GeneratedValue
   Long id;

  Long 수량;

  @ManyToOne
   Order order;

  public  void  setOrder (Order order) {
    this .order = order;
    order.getOrderItems().add( this );
  }

}
```

이제 어떤 엔터티 메서드를 사용하든 항상 관계의 양쪽을 설정하여 Transactional 블록 내에서 모호성을 피할 수 있습니다.

이 모든 것을 말한 뒤에도 가능하다면 양방향 관계는 피하는 것이 좋습니다.

#### ManyToOne Association

![ManyToOne](/assets/13.webp.png)

OneToOne 관계 매핑 과 유사하게 , ManyToOne 연관은 종종 N+1 문제로 이어질 수 있습니다. 따라서 Lazy 페칭을 사용하는 것이 가장 좋습니다. 또한, 결코 누락되지 않을 경우 속성을 다음 optional과 같이 설정합니다.false

#### OneToMany Assocation

![OneToMany](/assets/14.webp.png)

만약 OneToMany단순히 양방향 관계의 비소유 측에 있다면 ManyToOne, 실제로 필요한지 다시 한 번 확인하세요. 그렇지 않다면, 간단히 연관성을 제거하세요. 그러면 양방향 관계에서 발생할 수 있는 모든 문제가 방지됩니다.

OneToMany와 ManyToOne 간의 양방향 관계는 DDD의 집계 내에서만 고려되어야 합니다. (이것은 다음 기사에서 다룰 예정입니다)

OneToMany연결이 필요하지만 ManyToOne다른 엔터티의 연결이 필요하지 않은 경우 unidirectional 을 사용하면 됩니다 OneToMany. unidirectional OneToMany 관계는 다음과 같이 매핑할 수 있습니다.

```java
@Entity
@Data
public class Order {

  @Id
  @GeneratedValue
  Long id;

  @OneToMany
  @JoinColumn(name = "ORDER_ID")
  List<OrderItem> orderItems = new ArrayList<>();

}

@Entity
@Data
public class OrderItem {

  @Id
  @GeneratedValue
  Long id;

  Long quantity;

}
```

테이블 의 @JoinColumn외래 키를 정의한다는 점에 유의하세요 .ORDER_IDORDER_ITEM

또 다른 눈에 띄는 점은 필드가 empty로 초기화된다는 것입니다. 이렇게 하면 관계 ArrayList에 대한 연결이 없는 경우 Hibernate가 null 대신 항상 빈 컬렉션을 반환하므로 코드 전체에서 불필요한 null 확인이 방지됩니다 .\*ToMany

#### ManyToMany Assocation

![ManyToMany](/assets/15.webp.png)

```java
@Entity
@Data
public class Product {

  @Id
  @GeneratedValue
  Long id;

  @ManyToMany
  List<Category> categories = new ArrayList<>();

}

@Entity
@Data
public class Category{

  @Id
  @GeneratedValue
  Long id;

  @ManyToMany
  List<Product> product = new ArrayList<>();

}
```

ManyToMany가장 복잡한 관계인데, . 이 필요하기 때문입니다 JoinTable. 연결이 에 의해 관리되기 때문에 JoinTable관계의 명확한 소유자 측이 없습니다. 이 때문에 앞서 논의한 양방향 문제 중 일부는 연결로 인해 더 악화될 수 있습니다 . 그래서 저는 "양방향 관계를 사용하지 마세요 " ManyToMany라는 말을 반복할 것입니다. 어떤 이유로든 정말 필요하다면 실수로 추가 엔터티가 생성되는 것을 방지하기 위해 어떤 관계가 관계의 소유자 측 역할을 할지 염두에 두십시오.

할 수 있는 또 다른 최적화는 중복이 필요 없고 AND 순서가 필요 없는 경우 Set대신 을 사용하는 것입니다. Thorben Janssen은 최근 그의 강연 중 하나에서 이에 대해 논의했습니다.

References
