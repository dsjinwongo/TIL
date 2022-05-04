# JPA
### 컬럼의 기본값 설정
```
@Column(columnDefinition = "varchar(255) default 'John Snow'")
private String name;
 
@Column(columnDefinition = "integer default 25")
private Integer age;
 
@Column(columnDefinition = "boolean default false")
private Boolean locked;
```

### @Embedded, @Embeddable
여러개의 Column을 하나의 객체로 사용하기 위함
```
@Embeddable
public class Address {
  private String country;
  private String apartment;
  private String road;
}

@Entity
public class User {
  ...
  
  @Embedded(생략가능)
  private Address address;
}
```

### 영속성 컨텍스트
_(Namjun Kim 블로그 참고)_
- 엔티티를 **영구 저장하는 환경**이라는 뜻이다.
- 그치만 실제로는 DB에 저장하는 것이 아니라, **영속성 컨텍스트**를 통해서 엔티티를 영속화 한다는 것이다.
- 영속성 컨텍스트란 논리적인 개념으로 엔티티 매니저를 통해서 접근한다.
- 정확히 말하자면 persist() 시점에 영속성 컨택스트에 저장한다. DB 저장은 그 이후이다.
- 트랜잭션의 커밋 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 날라간다.
- 영속성 컨텍스트의 이점
  - 1차 캐시
    - 영속성 컨텍스트(엔티티 매니저)에는 1차 캐시가 존재한다.
    - 1차 캐시가 있으면 find()가 일어나면 1차 캐시에서 엔티티를 먼저 확인하고 존해하면 DB를 들리지 않고 바로 반환한다.
    - 다만, 1차 캐시는 해당 스레트가 끝나면 사라진다! 글로벌한 것은 2차 캐시이다.
    - **동일성을 보장**받을 수 있다: 1차 캐시 덕분에 객체를 두번 조회해도 같은 객체에 대한 레퍼런스가 된다.
    - 트랜잭션 커밋이 될 때가지 **쓰기 지연** SQL 저장소 라는 곳에 쿼리들을 생성해서 쌓아놓았다사 한번에 처리한다.
    
### 결과 조회
- ~.getResultList()
> 결과를 컬렉션으로 반환한다. 결과가 없으면 빈 컬렉션이 반환된다.
- ~.getSingleResult()
> 결과가 정확히 1건일 때 사용한다. 결과가 없으면 NoResultException이 발생한다.

### mappedBy
- 다대다 관계의 OneToMany어노테이션에서 쓰인다.
- 테이블의 pk를 fk로 갖고 있는 테이블이 주인이다.
- mappedBy에는 fk의 컬럼명이 들어가야한다.

### JPA CascadeType
Cascade란 영속성 전이를 뜻한다.
부모 엔티티가 영속화되거나 업데이트, 삭제될 때 자식 엔티티도 자동으로 수행하는 것을 뜻한다.
- ALL
상위 엔터티에서 하위 엔터티로 모든 작업을 전파
- PERSIST
하위 엔티티까지 영속성 전달
- MERGE
하위 엔티티까지 병합 작업을 지속
- REMOVE
연결된 하위 엔티티까지 엔티티 제거
- REFRESH
연결된 하위 엔티티까지 인스턴스 값 새로고침
- DETACH
연결된 하위 엔티티까지 영속성 제거
- PERSIST vs MERGE (ynoolee 블로그 참조)
  - merge는 영속성 컨텍스트에 의해 관리되는 instance를 리턴한다.
  - PersistenceContext 안에 존재하는 것을 리턴하거나, 그 관리되는 entity의 [ 새로운 instance를 생성 ]해서 리턴해 준다.
  ```java
  MyEntity e = new MyEntity();
  
  // scenario 1
  // tran starts
  em.persist(e); 
  e.setSomeField(someValue); 
  // 트랜잭션이 끝나면, db에, e.setSomeField(someValue)에서의 변화가 update된다. 

  // scenario 2
  // tran starts
  e = new MyEntity();
  em.merge(e);
  e.setSomeField(anotherValue); 
  // 트랜잭션이 끝나도, db에는  e.setSomeField(anotherValue); 로 인한 변화가 update되지 않는다. 
  // merge 이후에 change를 했기 때문임

  // scenario 3
  // tran starts
  e = new MyEntity();
  MyEntity e2 = em.merge(e);
  e2.setSomeField(anotherValue); 
  // 트랜잭션이 끝나면, db에는 e2.setSomeField(anotherValue);로 인한 변화가 update 된다. 
  //  영속상태인 e2에 대한 change가 일어난 거니까.
  ```

### CascadeType.REMOVE vs orphanRemoval = true
- 부모 엔티티 삭제
```CascadeType.REMOVE```와 ```orphanRemoval = true```는 부모 엔티티를 삭제하면 자식 엔티티도 삭제한다.

- 부모 엔티티에서 자식 엔티티 제거
```CascadeType.REMOVE```는 자식 엔티티가 그대로 남아있는 반면, ```orphanRemoval = true```는 자식 엔티티를 제거한다.

**orphanRemoval = true**
```java
team.addMember(member1);
team.addMember(member2);

teamRepository.save(team);

// when
team.getMembers().remove(0);
```
```sql
Hibernate: 
    delete 
    from
        member 
    where
        id=?
```

### spring.jpa.hibernate.ddl-auto: ~
_(축구하는 개발자 참고)_
- create: 기존테이블 삭제 후 다시 생성 (DROP + CREATE)
- create-drop: create와 같으나 종료시점에 테이블 DROP
- update: 변경분만 반영(운영DB에서는 사용하면 안됨)
- validate: 엔티티와 테이블이 정상 매핑되었는지만 확인
- none: 사용하지 않음(사실상 없는 값이지만 관례상 none이라고 한다.)


### 스칼라 타입 프로젝션
프로젝선이란 SELECT 절에 조회할 대상을 지정하는 것이다.
```java
em.createQuery("select count(l) from Like l where l.following = :id", Long.class)
                .setParameter("id", id)
                .getSingleResult();
```