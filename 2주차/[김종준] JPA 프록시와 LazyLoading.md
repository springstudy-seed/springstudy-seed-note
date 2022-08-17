## JPA 프록시와 LazyLoading

---

JPA 프록시는 구현체들이 연관된 객체를 처음부터 DB에서 조회하는 것이 아니라, 실제 사용하는 시점에 DB에서 조회할 수 있도록 하는 기술이다.

이를 통해 LazyLoading을 할 수 있게된다.



### 프록시

---

우선 프록시는 실제 클래스를 상속 받아서 만들어 진다.

이는 하이버네이트가 내부적으로 상속 받아서 만들기 때문에 실제 클래스와 겉모양이 동일하다.

이때 객체는 실제 객체의 참조(target)을 보관하고 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출 한다.

그렇기에 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용할 수 있는 것이다.



![프록시01](https://raw.githubusercontent.com/belljun3395/typoraImage/main/image/프록시01.png)

#### em.getReference()

---

em.find()는 DB를 통해서 실제 엔티티 객체를 조회하는 메서드라면 em.getReference()는 프록시 엔티티 객체를 조회하는 메서드이다.



예제 코드를 통해 이해해보자.

```java
Member member = new Member();
member.setUsername("hello1");

em.persist(member);

em.flush();
em.clear();

Member m = em.getReference(Member.class, member.getId());
```

`em.getReference(Member.class, member.getId())`를 통해 실제 객체의 참조와 그 객체의 PK 정보를 가진 프록시 객체가 만들어진다.

![프록시02](https://raw.githubusercontent.com/belljun3395/typoraImage/main/image/프록시02.png)





#### 프록시 객체의 초기화

---

`m.getUsername()` 과 같이 프록시 객체에 존재하지 않는 정보를 조회하면 프록시 객체의 초기화가 일어난다.

이때 초기화는 프록시의 실제 객체 참조와 PK 정보를 가지고 영속성 컨텍스트에 초기화를 요청하고 DB 조회를 한후  실제 Entity를 생성한다는 것이다.

즉 조회가 필요한 시점에서야 DB에 쿼리가 나가는 것이다.

이렇게 초기화를 통해 프록시 객체에 참조가 할당되면 더 이상 프록시 객체의 초기화는 일어나지 않는다.



![프록시03](https://raw.githubusercontent.com/belljun3395/typoraImage/main/image/%ED%94%84%EB%A1%9D%EC%8B%9C03.png)



이번에도 예제 코드를 통해 알아보자.

```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("hello1");
member.setTeam(team);

em.persist(member);

em.flush();
em.clear();

Member m = em.getReference(Member.class, member.getUsername());
System.out.println("m.getId() = " + m.getId());
System.out.println("===================================================");
System.out.println("m.getUsername() = " + m.getUsername());
System.out.println("===================================================");
System.out.println("m.getTeam() = " + m.getTeam().getId());
System.out.println("===================================================");
```



아래는 위의 결과 코드이다.

```
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    /* insert hellojpa.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (INSERT_MEMBER, createdDate, lastModifiedBy, lastModifiedDate, TEAM_ID, USERNAME, MEMBER_ID) 
        values
            (?, ?, ?, ?, ?, ?, ?)
m.getId() = 2
===================================================
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_3_0_,
        member0_.INSERT_MEMBER as insert_m2_3_0_,
        member0_.createdDate as createdd3_3_0_,
        member0_.lastModifiedBy as lastmodi4_3_0_,
        member0_.lastModifiedDate as lastmodi5_3_0_,
        member0_.TEAM_ID as team_id7_3_0_,
        member0_.USERNAME as username6_3_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
m.getUsername() = hello1
===================================================
m.getTeam() = 1
===================================================
```

`m.getUsername()`의 username이 프록시 객체에 존재하지 않는 정보이기 때문에 DB에 조회하여 프록시 초기화를 하였다.

그렇기에 `m.getTeam()`을 할 때는 DB에 쿼리가 나가지 않고 프록시 객체에서 정보를 찾아오는 것을 볼 수 있다.



#### 프록시 갞체 주의점

---

프록시 객체는 원본 엔티티를 상속 받는다고 하지만 프록시 객체와 원본 객체와 타입은 다르다.

그렇기에 타입 체크시에는 `==` 이 아닌 `instanceOf` 를 사용해야 한다.



또 영속성 컨텍스트에 찾는 엔티티가 이미 있으면, `em.getRefernce()`를 호출해도 실제 엔티티를 반환한다.

반대로 프록시가 존재하면 `em.find()`를 호출해도 프록시를 반환한다.



`em.detach(), em.close(), em.clear()` 과 같은 코드 이후 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태이다.

프록시 객체는 영속성 컨텍스트의 도움을 받아 프록시 객체를 초기화 하는데 위의 코드는 준영속 상태를 만드는 코드이기 때문에 영속성 컨텍스트의 도움을 받지 못하여  `org.hibernate.LazyInitializationException` 와 같은 문제가 일어난다.



#### 프록시 관련 Utils

---

`emf.getPersistenceUnitUtil().isLoaded(referenceObject)` - 프록시 인스턴스의 초기화 여부를 확인하는 메서드이다.

`Hibernate.initialize(referenceObject)` - 프록시를 강제 초기화 해주는 메서드이다.



### LazyLoading

---

LazyLoading은 프록시를 활용하여 구현한다.

```java
@Entity
public class Member {
        @Id
        @GeneratedValue
        @Column(name = "MEMBER_ID")
        private Long id;

        @Column(name = "USERNAME")
        private String username;

        @ManyToOne
        @JoinColumn(name = "TEAM_ID")
        private Team team;
}
```

Member 객체를 다음과 같이 정의한다면 Member을 조회하는 쿼리를 DB에 보내면 Team까지 조인하여 같이 조회되는 문제가 발생한다.

이는 예상하지 못한 조인이 발생하는 문제와 조인 때문에 일어나는 성능의 문제가 발생한다.

(성능의 문제의 경우 이를 N+1 문제라고 한다. 이는 1개의 쿼리를 DB에 전송하여 N개의 값을 받는다면 테이블간 연관관계 때문에 N번의 쿼리가 더 필요한 문제이다.)

이는 `@ManyToOne`의 기본 값이 `fetch = FetchType.EAGER` 이기 때문에 일어나는 문제로 이를 `fetch = FetchType.LAZY` 로 설정해주면 해결 할 수 있다.



위의 경우를 예시로 들자면  `private Team team` 에서 Team이라는 클래스를 알 수 있고 `Member.setTeam(team)` 을 통해 team의 id를 알 수 있기 때문에 프록시 객체를 만들 수 있다.

이렇게 프록시 객체를 만들어두면 Member을 조회하면 Team까지 조회하는 것이 아닌 Member만 조회할 수 있게 된다.



이를 아래의 예시 코드를 통해 알아보자.

```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Member member = new Member();
member.setUsername("hello1");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

Member m = em.find(Member.class, member.getId());
System.out.println("===================================================");
System.out.println("m = " + m);
System.out.println("===================================================");
System.out.println("m.getTeam() = " + m.getTeam());
System.out.println("===================================================");
```

아래는 위 코드의 결과이다.

```
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    call next value for hibernate_sequence
Hibernate: 
    /* insert hellojpa.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (INSERT_MEMBER, createdDate, lastModifiedBy, lastModifiedDate, TEAM_ID, USERNAME, MEMBER_ID) 
        values
            (?, ?, ?, ?, ?, ?, ?)
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_3_0_,
        member0_.INSERT_MEMBER as insert_m2_3_0_,
        member0_.createdDate as createdd3_3_0_,
        member0_.lastModifiedBy as lastmodi4_3_0_,
        member0_.lastModifiedDate as lastmodi5_3_0_,
        member0_.TEAM_ID as team_id7_3_0_,
        member0_.USERNAME as username6_3_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
===================================================
m = hellojpa.Member@3bed3315
===================================================
Hibernate: 
    select
        team0_.TEAM_ID as team_id1_7_0_,
        team0_.name as name2_7_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
m.getTeam() = hellojpa.Team@2806d6da
===================================================
```

위의 결과를 보면 단순히 Member을 조회할 때는 DB에 쿼리가 안나간 것을 볼 수 있다.

다만 Member 조회 결과인 `m = hellojpa.Member@3bed3315` 객체 안에는 team에 관한 프록시 객체가 존재한다.

그렇기에 `m.getTeam()`을 실행하게 되면 DB에 쿼리가 나가고 Team의 정보를 가져오는 것을 볼 수 있다.



이는 위의 예제와 같이 연관관계가 복잡하지 않은 경우는 상관이 없지만 연관관계가 복잡해지면 질수록 조인되는 테이블이 많아지고 이에 따른 성능 문제가 발생하기 때문에 `fetch = FetchType.LAZY` 로 설정해 주는 것이 좋다.



또한 `xxxOne`의 경우 `fetch`의 기본값이 `FetchType.Eager`이기 때문에 조인을 필요로하는 조회가 더 많지 않는 이상 `FetchType.Lazy`로 설정하는 것이 좋다.