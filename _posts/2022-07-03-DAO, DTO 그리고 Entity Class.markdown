---
layout: post
title:  "DAO, DTO 그리고 Entity Class"
date:   2022-07-03 09:38:46 +0900
categories: DB
tags: java, db, spring, jpa, DAO, DTO
---
JAVA에서 데이터를 핸들링하기 위한 방법을 먼저 정리할려고 했으나 먼저 객체가 어떻게 DB에 접근하는가를 정리하는게 맞겠다고 생각했습니다. 특히 Spring을 사용하다보면 공식문서를 비롯해서 DAO, DTO, Entity Class등의 개념이 나오게 되는데 해당 개념에 대해 정리를 해두면 좋을 듯 합니다.

## 1. DAO(Data Access Object)
DAO는 데이터베이스의 Data에 접근하기 위한 객체입니다. 데이터만 담당하는 클래스로 DB의 CRUD 기능을 수행합니다. 우리가 필요한 객체를 바로 DB에 연결하지 않는 이유는 크게 두가지인데 DB 접근 로직과 비즈니스 로직의 분리를 위함이며 나중에 DB가 변경되더라도 DB 접근로직만 변경하면 되기에 비즈니스 로직을 건드리지 않아도 됩니다. 주로 DAO로 뭉떵거리기 보다 DAOInterface/DAOImplement 로 구분지어 명세와 구현 분리하며 개발합니다.

![DAO image](\assets\img\DAO.png)

요약
* 실제로 DB에 접근하는 객체로 Persistence Layer(DB에 data를 CRUD하는 계층)
* Service와 DB를 연결하는 고리
* SQL을 사용하여 DB에 접근한 다음 적절한 CRUD API를 제공

*JPA의 Repository package사용을 통한 예시
```java
public interface QuestionRepository extends CrudRepository<Question, Long> {
}
```
문제는 여기서 개념이 한번더 나누어지게 되는데 Repository와 DAO의 차이다. 결론부터 말하면 DAO는 Data Peristence의 추상화이고, Repository는 객체 Collection의 추상화다. 데이터를 접근한다는 점에서 공통점을 가지고 있지만 Repository는 객체 중심, DAO는 데이터 저장소(DB 테이블)중심이 된다.


## 2. DTO(Data Transfer Object)
DTO는 계층간 데이터 교환을 위한 객체(Java Beans)입니다. Controller, Service, View 처럼 계층 간의 데이터 교환을 위해 사용합니다. 이 객체는 로직을 갖고 있지 않으며 순수한 데이터 객체이며 getter, setter 메소드만을 갖고 있습니다. 많은 경우 DB에서 꺼낸 값을 전달하는 것이 목적이지 변경할 이유는 없기에 DTO 클래스에는 setter이 없을 수도 있습니다.

![DAO image](\assets\img\data_tier.jpg)

즉 DB의 데이터가 Presentation Logic Tier로 넘어오게 될 때는 DTO의 모습으로 전달합니다.

```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class UserDto {
  @NotBlank
  @Pattern(regexp = "^([\\w-]+(?:\\.[\\w-]+)*)@((?:[\\w-]+\\.)*\\w[\\w-]{0,66})\\.([a-z]{2,6}(?:\\.[a-z]{2})?)$")
  private String email;

  @JsonIgnore
  @NotBlank
  @Size(min = 4, max = 15)
  private String password;

  @NotBlank
  @Size(min = 6, max = 10)
  private String name;

  public User toEntity() {
      return new User(email, password, name);
  }

  public User toEntityWithPasswordEncode(PasswordEncoder bCryptPasswordEncoder) {
      return new User(email, bCryptPasswordEncoder.encode(password), name);
  }
}
```
toEntity() 메서드를 통해서 DTO에서 필요한 부분을 이용하여 Entity로 만든 코드
* Request와 Response용 DTO는 View를 위한 클래스

### 추가 정리 VO(Value Object)와 BO(Business Object)
* VO는 DTO와 유사하게 보이고 read only 속성을 갖는다. 하지만 개념이 다르다.
VO는 특정한 비즈니스 값을 담는 객체이고, DTO는 Layer간의 통신 용도로 오고가는 객체를 말한다. VO는 값들에 대해 Read-Only를 보장해줘야 존재의 신뢰성이 확보되지만 DTO의 경우는 단지 데이터를 담는 그릇의 역할일 뿐 값은 그저 전달되어야 할 대상일 뿐이다. 값 자체에 의미가 있는 VO와 전달될 데이터를 보존해야하는 DTO의 특성상 개념이 다르다.

* BO는 VO와 마찬가지로 데이터 저장 담당 클래스이나 이름처럼 비즈니스 로직을 담고 있는 객체, 즉 비즈니스 관련 내용을 담은 VO가 BO

![DAO image](\assets\img\DTO.png)

유저가 입력한 데이터를 DB에 넣는 과정은 웹앱 기준으로 브라우저에서 데이터를 입력하여 form에 있는 데이터를 DTO에 넣어서 전송
그리고 DTO를 받은 서버가 DAO를 이용하여 DB로 데이터를 저장


## 3. Entity Class
실제 DB의 테이블과 매칭될 클래스다. DB 테이블에 존재하는 Column들을 필드로 가지는 객체를 말한다. 가장 Core한 클래스라고 부르기도하며 JPA에서 @Entity 어노테이션을 붙여서 Entity클래스임을 명시해야하고 내부 필드에는 @Column, @Id 어노테이션을 이용한다. DB 테이블과 1:1 대응관계에 있기 때문에 테이블이 가지지 않는 컬럼을 필드로 가지면 안되며 Entity클래스는 다른 클래스를 상속받거나 인터페이스의 구현체가 되서는 안된다. 그래서 외부에서 Entity 클래스의 getter method를 사용하지 않도록 해당 클래스 안에서 필요한 로직 method을 구현한다. 다시 말해 Domain 로직만 구현하고 Presentation 로직은 구현하지 않는다.

Entity 클래스에서 Setter를 만드는 것은 피하길 권장한다. Setter가 모든 클래스에 있거나 너무 많을 경우 인스턴스 값들이 어느 지점에서 변경이 이루어지는지 명확하게 알 수 없기에 다른 방법으로 필드에 값을 채우는 것이 좋다. 하지만 생성 시점에 생성자로 필드에 값을 넣어주는 방법 또한 그다지 좋지 않은 방법일 수 있다. 생성자에 현재 넣는 값이 어떤 필드인지 명확히 알 수 없고, 매개변수끼리의 순서가 바뀌더라도 코드가 모두 실행되기 전까지는 문제를 알 수 없다는 단점이 있기 때문이다.

아래 예시는 Builder 패턴을 사용하는데 멤버 변수가 많아지더라도 어떤 값을 어떤 필드에 넣는지 코드를 통해 확인할 수 있고, 필요한 값만 집어넣는 것이 가능하기 때문

![DAO image](\assets\img\Entity.png)

Entity Class와 DTO를 분리하는 이유는 View Layer와 DB Layer의 역할을 철저하게 분리하기 위함입니다. Entity Class는 기획단계에서 결정되기 때문에 한번 확정이 되면 변경하는 것이 어려워 여러 클래스에 영향을 끼치게 되는 반면 View와 통신하는 DTO 클래스(Request / Response 클래스)는 자주 변경되므로 분리해야 한다.

Entity 클래스 생성 및 Builder 패턴 예시
```java
@Getter
@Entity
@NoArgsConstructor
public class Membmer member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String name;
    private String email;
    @Column(length = 13, nullable = false)
    private String phoneNumber;
 
    @Builder
    public Member(long id, String name, String email, String phoneNumber) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.phoneNumber = phoneNumber;
    }
}

// 사용 방법
Member member = new member.builder()
        .name("홍길동")
        .email("test@gmail.com")
        .phoneNumber("010-1234-5678")
        .build();
```
