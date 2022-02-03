---
title: 초보자를 위한 스프링 부트 가이드: 엔티티 관계
categories:
  - Entity
feature_text: |
  ## ERD를 실제 데이터베이스 모델 구현으로 변환
---

#### Goal

> Spring Framework는 개발자가 다양한 모듈 선택을 지원하여 쉽게 서비스를 구축할 수 있도록 합니다. Spring의 일반적인 모듈 통합 중 하나는 Spring Data JPA입니다. 이 라이브러리는 많은 노력을 들이지 않고도 모델 엔터티와 그 관계를 생성하는 데 도움이 됩니다. ORM 라이브러리로서 Hibernate와의 조합은 말할 것도 없이 데이터베이스에서 선택, 생성, 필터링 및 기타 많은 기능을 수행하는 기본 기능을 수행하는 데 도움이 될 것입니다.

>이 기사에서는 주어진 엔티티 관계 다이어그램을 실제 데이터베이스 스키마 모델로 구성하는 방법을 안내 하고 모델 엔티티 간의 관계를 매핑하는 방법을 강조합니다. Spring Boot에는 배울 것이 너무 많으며 한 번에 한 가지만 배우라고 조언합니다.

#### 전제조건

이 기사에서는 스프링 부트 서버를 시작하는 방법을 다루지 않습니다. 다음 종속성이 있는 실행 중인 애플리케이션이 있는지 확인하십시오.

- 스프링 웹
- 스프링 데이터 JPA
- 모든 데이터베이스 드라이버(예: 선호하는 DB에 따라 PostgreSQL, H2 또는 DB2)

모든 요구 사항이 충족되면 기본 응용 프로그램 폴더로 이동하여 "모델"이라는 이름으로 새 패키지/폴더를 만듭니다. 이 폴더는 모델 엔터티가 저장되는 위치입니다. 유지보수가 가능하도록 코드를 구성하는 것이 중요합니다. 아래의 놀라운 기사에서 Spring Boot 프로젝트 구조 모범 사례에 대해 자세히 알아볼 수 있습니다.

#### 코딩 실습

Spring Data JPA 모델을 구현하기 위해 아래의 ERD(Entity Relationship Diagram)가 제공되었습니다. 이 ERD는 고객과 택배라는 두 종류의 사용자가 있는 시스템을 설명합니다. 고객은 서비스(Courier에서 수행)를 주문하고 해당 활동을 트랜잭션 모델에 기록합니다.

![이미지1](./assets/1_WeJrS0RbizKm6UtTqsdcKQ.png)

구현해야 할 5가지 가상 모델이 있지만 결국에는 그 이상이 될 수도 있습니다. 우리가 다룰 수 있는 관계에는 일대일, 다대일 및 다대다와 같이 세 가지 종류가 있습니다. 이 가이드에서는 Spring Data JPA에서 엔터티 관계 구현을 강조하므로 모델 간의 관계에 따라 각 모델을 세분화합니다.

#### 기본 엔터티 모델

모델 클래스를 간단히 변경하여 데이터베이스의 스키마/테이블에 엔터티를 손쉽게 구현할 수 있습니다. 아래의 샘플 엔터티에 대한 샘플 코드는 그렇게 하는 것이 얼마나 쉬운지 보여줍니다.

![이미지1](./assets/1_w650sePgH2q-2CnfF6sTmw.png)

수동으로 테이블을 생성하지 않고 데이터베이스에 테이블을 자동으로 생성하는 몇 가지 주석만 사용했습니다. 이러한 주석에 대한 자세한 설명은 다음과 같습니다.

- @Entity주석은 해당 클래스가 엔티티이고 데이터베이스 테이블에 매핑될 것임을 정의합니다.
- @Table주석은 매핑에 사용할 데이터베이스 테이블의 이름을 정의합니다.
- @Id주석은 엔터티의 기본 키를 정의합니다.

#### 일대일 관계

일대일 관계는 두 개의 엔터티 A와 B가 있고 요소 A는 하나의 요소 B에만 연결될 수 있으며 그 반대의 경우도 마찬가지입니다. 이 문제에는 Customer와 Courier라는 두 가지 모델 클래스를 상속하는 UserBase 모델이 있습니다. 이러한 클래스 간의 관계는 일대일 관계입니다. 즉, 하나의 Customer 인스턴스만 하나의 UserBase 인스턴스에 연결할 수 있으며 그 반대의 경우도 마찬가지입니다.

![이미지1](./assets/1_QoBVHloVxTVx1pRrHBwh_g.png)

먼저 UserBase 클래스인 슈퍼클래스를 구현해야 합니다. id, email, fullName및 와 같이 ERD에 따라 지정된 유형으로 필수 속성을 생성합니다 dob.
주석을 사용 @Inheritance하여 슈퍼클래스와 서브클래스 간에 엔티티를 매핑하는 데 사용하는 전략을 지정할 수 있습니다. 계층 구조의 모든 엔터티를 매핑해야 하므로 를 InheritanceType.JOINED전략으로 사용합니다. 여기 에서 이 기사의 다른 전략을 검토할 수 있습니다 .

![이미지1](./assets/1_pirkeCs1pvgBvGD8DWlYwQ.png)

![이미지1](./assets/1_sihBKOl7qhlFz5e6_S_f3g.png)

수퍼 클래스를 확장하면 추가 주석을 제공하지 않고도 모든 하위 클래스를 매핑할 수 있습니다.

![이미지1](./assets/1_GSWsFKX7UJw4lzoCbdjrpg.png)

위의 스크린샷에서 볼 수 있듯이 우리는 InheritanceType.JOINED 전략으로 사용하고 있기 때문에 엔티티 클래스에서 세 개의 테이블을 성공적으로 구현했습니다. from Courier 및 Customer 테이블 은 id기본 키인 동시에 UserBase 테이블의 외래 키입니다.

#### 일대다 관계

일대다 관계는 두 개의 엔터티 A와 B가 있고 요소 A가 B의 많은 요소에 연결될 수 있지만 그 반대는 아닌 경우입니다. ERD는 고객이 여러 트랜잭션 인스턴스를 가질 수 있는 동안 하나의 고객으로 트랜잭션 엔터티를 구현하도록 요구했습니다. 또한, 이 조건은 택배에도 적용됩니다.

![이미지1](./assets/1_VTGQ7NQGeKufJdbImPM6YA.png)

Transaction 클래스를 생성하지 않았으므로 먼저 생성해야 합니다. Transaction 엔터티가 Customer 및 Courier 엔터티와 다대일 관계를 갖기를 원했기 때문에 대상 클래스의 인스턴스를 @ManyToOne주석이 있는 Transaction 속성으로 생성해야 합니다.

![이미지1](./assets/1_KL8aX7JyAkUSg2m1OmlP_g.png)

![이미지1](./assets/1_JY9Z17N9EFZH3waMU0OZ8Q.png)

그런 다음 Customer/Courier에는 둘 이상의 트랜잭션 엔터티가 있을 수 있으므로 거래 목록을 Customer 엔터티 및 Courier 엔터티에 추가합니다.

![이미지1](./assets/1_YADg6v9ZqUZrXzM-lVGNOA.png)

위의 스크린샷에서 볼 수 있듯이 데이터베이스에 대한 트랜잭션 테이블을 성공적으로 구현했습니다. Spring Data JPA가 자동 생성하는 추가 테이블이 있음을 주목하십시오. 이러한 테이블은 모든 사용자의 트랜잭션(사용자당 둘 이상 존재할 수 있음)을 보유하기 때문에 예상된 동작입니다.

#### 다대다 관계: 1부

다대다 관계는 두 개의 엔터티 A와 B가 있고 요소 A가 B의 많은 요소에 연결될 수 있으며 그 반대의 경우도 마찬가지입니다. 문제 컨텍스트에서 트랜잭션 엔터티는 둘 이상의 서비스 엔터티를 가질 수 있으며 하나의 서비스 엔터티는 둘 이상의 트랜잭션 엔터티에 포함될 수 있습니다.

![이미지1](./assets/1_uLmYX-Rqh2Fwv8XueyRK3Q.png)

![이미지1](./assets/1_W3ZLFlMmFNrRtVrNhAl-1w.png)

먼저 다른 속성과 함께 서비스 클래스의 트랜잭션 목록을 정의했습니다. 그런 다음 @ManyToMany주석을 transactionList트랜잭션 목록 인스턴스로 추가합니다. Transaction 엔터티에서도 서비스 목록을 정의해야 합니다.

![이미지1](./assets/1_VEzuASWYYIRmZKJnk5VPHQ.png)

마지막으로 모든 엔터티와 해당 관계가 데이터베이스 테이블에 매핑되었습니다. 그러나 왜 우리는 두 개의 중복 테이블을 가지고 service_transaction_list있습니까 transaction_service_code? 걱정하지 마십시오. 곧 이 문제를 해결할 것입니다.

#### 다대다 관계: 2부

다음과 같이 필요한 주석을 추가하여 몇 가지 수정으로 서비스 클래스를 수정합니다.

![이미지1](./assets/1_ccdysg1rLEiwmUvMxZzPmw.png)

@JoinTableService 클래스에 주석을 추가 합니다. 상기 주석의 매개변수 이름은 임의적이지만 나머지는 따라야 할 규칙이 있습니다. joinColumn매개변수는 이 클래스 식별자를 이름 매개변수로 사용하는 것이어야 합니다 @JoinColumn. 반면에 inverseJoinColumn 매개변수는 @JoinColumn주석의 이름 매개변수로 대상 클래스 식별자를 가진 매개변수입니다.

![이미지1](./assets/1_raOIMh4KvS1GfRK30jeiQA.png)

그런 다음 대상 클래스에서 매개변수 mappedBy를 매개변수에 추가하기만 하면 @ManyToMany됩니다. 이 매개변수는 이전에 본 소스 인스턴스로 채워집니다.
참고: 소스와 대상은 임의적입니다. 바꿔서 사용하시면 됩니다.

![이미지1](./assets/1_G56MTpCLG8m-wFdUQq02Kg.png)

짜잔! 우리 데이터베이스가 훨씬 깨끗하지 않습니까? 데이터베이스 시스템이 완성되었으며 비즈니스 프로세스를 작성할 준비가 되었습니다.

#### Further Reading

이것은 Spring Boot에서 Model Relationship을 개발하기 위한 간단한 가이드입니다. 개선해야 할 사항이 많이 있습니다. Spring Boot의 다른 개선 사항 및 구성 요소를 배울 수 있도록 아래에 외부 링크를 추가했습니다.
Spring @Aleksandr Kozhenkov의 모든 사람이 한 트랜잭션 실수
Bhagya Rupasinghe 의 Spring 부트 자바에서 @EmbeddedId 주석을 사용하여 복합 키 처리
Tranh Tran이 가장 많이 사용하는 Spring Data MongoDB 주석

#### Acknowledgement

이 기사는 Kampus Merdeka x Gojek 인턴쉽 프로그램에서 나의 학습 진도를 나타냅니다. 내 학습 여정에 대한 업데이트를 받으려면 저를 팔로우하십시오!

References
