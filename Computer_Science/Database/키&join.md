# Relation
- table 중 데이터베이스에서 사용되기 위한 조건을 갖춘 것이 relation
- 제약조건
  - table의 cell은 단일 값을 갖는다
  - 어떤 두개의 row도 동일하지 않다
- 근데 relation과 table을 구분하지 않고 사용하기도 함

![image](https://github.com/ChaesongYun/ssafy_CS_study/assets/139418987/2f10c1f5-c9ca-4779-802a-c49541779356)

# Super key
- Super Key(슈퍼키)는 각 row를 유일하게 식별할 수 있는 하나 or 그 이상의 속성들의 집합
- 예를 들면 이름이라는 속성은 동명이인이 있을수도 있자나?
  그럼 (이름+학번)이라는 key를 통해서 row를 식별하고 이 (이름+학번)은 슈퍼키 중 하나가 되는 거ㅇㅇ
  
```plain text
유일성: 하나의 key값으로 특정 row만을 유일하게 찾아낼 수 있어야 한다
```

- 예시
  - (학번)
  - (학번, 이름)
  - (학번, 이름, 학과)
  - (주민등록번호)
  - (주민등록번호, 학과, 성별)


# Candidate key
- 후보키는 Super key 중에서 더이상 쪼개질 수 없는 super key를 말한다
- 각 row를 유일하게 실별할 수 있는 최소한의 속성
  
```plain text
최소성: 모든 row를 유일하게 식별하는데 꼭 필요한 속성만으로 구성되어야 한다
```

- 예시
  - (학번)
  - (주민등록번호)

# Primary key
- candidate key 중에서 선택한 main key로써 각 row를 구분하는 유일한 열을 말한다
- Null값 가지면 x, 중복된 값 x
- 기본키는 table 당 1개만 지정

# Alternative key
- 대체키는 후보키가 두 개 이상일 경우 기본키로 지정되지 못하고 남은 후보키들을 말한다
- 위의 예시에서 학번이 기본키라면 주민등록번호가 대체키가 되는거

![image](https://github.com/ChaesongYun/ssafy_CS_study/assets/139418987/c0dd8474-b0d9-452d-bc84-0447a5c10119)


---

<div>Q. composite key에 대해 설명해주세요<div>
<div>A. composite key란 table에서 각 row를 식별할 수 있는 두 개 이상의 column으로 구성된 candidate key를 말합니다.</div>


---

# Join
- Join이란 두 개 이상의 테이블을 서로 연결하여 하나의 결과를 만들어 보여주는 것

## Inner Join
- 두 테이블 모두 있는 내용만 join됨

![image](https://github.com/ChaesongYun/ssafy_CS_study/assets/139418987/0145800a-62d9-46bd-b6cc-606098a88225)


## Left outer join
- 혹은 left join: 왼쪽 table의 모든 행에 대해서 join을 진행

![image](https://github.com/ChaesongYun/ssafy_CS_study/assets/139418987/d2f7b3c0-fd61-49ef-b864-ccc2a5ab8479)

