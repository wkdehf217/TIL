# JPA

## JPA(Java Persistence API)

자바에서 사용하는 ORM 기술 표준이다.

JPA는 자바 애플리케이션과 JDBC 사이에서 동작하며, 자바 인터페이스로 정의되어 있다.

JPA 인터페이스를 구현한 대표적인 오픈소스가 Hibernate 이다.

여기서 또 ORM에 대해서 먼저 알아야 한다.

<br/>

## ORM(Object-Relational Mapping)

우리가 일반적으로 알고 있는 애플리케이션 Class와 RDB(Relational DataBase)의 테이블을 매핑(연결)한다는 뜻이며, 

기술적으로는 어플리케이션의 객체를 RDB 테이블에 자동으로 영속화 해주는 것이다.

<br/>

## Entity의 상태

- 비영속과 영속 상태
비영속은 그냥 관리받기 전
persist를 통해 영속성 컨텍스트로 전환

- 준영속상태?
detached : 영속성 컨텍스트에 의해 관리되다가 분리된 상태
detach / clear / close 이 3개의 메서드에 의해 전환됨

- detach : 분리
clear : 껍데기만 남기고 내용만 비움 (비우고 재사용)
close : 완전히 종료

- merge : 다시 영속상태로 바꾸는 방법
비영속 상태의 Memo와 
merge를 통해서 받아온 mergedMemo는 서로 상태가 다르다
mergedMemo는 새로 받아온 메모니까 insert로 인해 영속 상태
가지고 오고 합치고, 없으면 새로 insert ... 깃허브와 비슷하다고 보면 된다

![](https://velog.velcdn.com/images/wkdehf217/post/5f288ffb-9b18-462c-9d12-7bca59a1cfd4/image.png)

<br/>

## SpringBoot 의 JPA

- Springboot에서는 EntityManeger와 EntityManegerFactory 자동 생성
jpa는 DB의 트랜잭션 개념을 가져와서 사용하고 있다.
그럼 Spring에서는 어떻게 하느냐?
-> @Transaction

- 수정하려면 Transaction 상태가 필요하지만 지금 적용이 안되어있기 때문에 (@Transaction이 없으니까) 수정 실패

 > 1차 캐시 저장, 변경감지 등등 다 영속성 컨텍스트에서 이루어지는데 그걸하려면 Transaction이 필요함 => 생명주기 일치!

- 트랜잭션 전파(propagation)

  - Repository를 @Autowired를 통해 주입받아옴

  - 자식의 Transaction이 끝나도 commit이 날라가지 않고, 부모의 Transaction이 끝나야 commit이 됨
    = 자식 메서드의 Transaction이 부모의 Transaction에 합쳐졌기 때문

  - 만약 부모가 Transaction이 없다면?
= 자식의 Transaction이 끝나자마자 commit


- Spring Data JPA

  + JPA를 쉽게 사용할 수 있게 만들어놓은 하나의 모듈
repository 인터페이스 제공
JPA를 간편하게 사용 가능해짐

![](https://velog.velcdn.com/images/wkdehf217/post/35527d95-8100-417c-a97c-2f36036da089/image.png)


+ 등록하는법 : extends JpaRepository<T, ID>를 통해 class -> interface로 만듦
제네릭스..
return memoRepository.findAll().stream().map(MemoResponseDto::new).toList();
: MemoResponseDto 생성자중에서 Memo를 파라미터로 가지고 있는 생성자를 찾아서 리스트로 변환

+ Optional 처리 : orElseThrow(null 체크) 
: 만약 메모가 null값이라면 Throw를 던진다

+ update / delete
memoRepository 내부에 이미 정의되어있음, 인터페이스로 상속받았기 때문에
들어가는 인자 변경함

+ 변경감지 <- 영속성 컨텍스트 <- Transaction이 걸림
updateMemo 부분 Transaction 주석 처리해보면 알 수 있음


## JPA Auditing 적용하기

  + 프로젝트에서 시간이 적용된다고 생각
원래 DateTime 넣으려면 변수 생성 -> 다른 클래스 로직마다 넣어줌
근데 JPA는 클래스 하나 만들어서 어노테이션(@) 달아주면 알아서 해준다

  + @MappedSuperclass + abstract
Timestamped를 상속받는 클래스는 createdAt이랑 modifiedAt을 가지게 된다

  + 메인에 @EnableJpaAuditing 달아주기!
  
![image](https://github.com/wkdehf217/TIL/assets/45251507/3f99fc59-75dd-4555-8ca9-d0e75bb1cd97)
