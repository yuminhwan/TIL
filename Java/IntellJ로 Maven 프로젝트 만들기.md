# IntellJ로 Maven 프로젝트 만들기

> IntellJ 2021.3 버전 기준으로 작성하였습니다.
> 

이때까지의 프로젝트는 Gradle을 사용했지만 데브코스에서는 Maven을 사용한다하여 과제를 실시할 때 Maven으로 프로젝트를 생성해보았다. 

자세한 설정은 없고 그저 과제를 하기 위한 프로젝트이니 간단하게만 만들었다.

## 프로젝트 생성

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0j0z3tipnj216b0u0762.jpg)

인텔리제이 시작화면에서 **NEW PROJECT**를 클릭한다.

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0j0z4ei2jj216b0u0762.jpg)

Maven 프로젝트를 만들것이니 왼쪽에서 Maven 프로젝트를 클릭하고 Project SDK에서 자바 버전 선택 

나는 8버전으로 선택하였다. 

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0j0z4xls2j20u00vg78a.jpg)

자신이 원하는 프로젝트 이름을 Name 칸에 적어준다. ( Name에 해당 하는 폴더 생성 )

나머지 칸도 알맞은 내용으로 적어준다.

- **GroupId**
    - 프로젝트를 고유하게 식별할 수 있게 해주는 ID
    - 따라서, 네이밍 스키마를 적용한다.
        - package 명명 규칙을 따른다.
        - 즉, 최소한 프로젝트 안에 있는 도메인 이름 이여야 한다.
        - 하위 그룹은 얼마든지 추가할 수 있다.
        - `com.apache.maven`
- **ArtifactID**
    - 버전 정보를 생략한 jar 파일 이름
    - 기본적으로 프로젝트 이름과 같다.
- **Version**
    - 프로젝트의 버전을 나타낸다.

이후 FINISH를 눌러주면 프로젝트가 생성된다.

![스크린샷 2022-03-22 오후 11.01.23.png](https://tva1.sinaimg.cn/large/e6c9d24egy1h0j0z7fb8hj20ga0bg3yv.jpg)

![스크린샷 2022-03-22 오후 11.00.41.png](https://tva1.sinaimg.cn/large/e6c9d24egy1h0j0z903qej20ia0fs751.jpg)

## 의존성 추가

pom.xml 파일에 아래와 같이 추가해준다. 

테스트 코드 작성을 위해 Junit5, AssertJ를 추가해주었다.

```xml
<dependencies>
    <!-- Junit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.21.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

만약 다른 것도 추가하고 싶다면 [Maven Repository](https://mvnrepository.com)에 검색한 뒤 추가해주면 된다.

### .gitignore

맥, 인텔리제이, 메이븐의 경우 아래와 같이 설정해주면 된다.

```java
# Intellij
.idea/
*.iml
*.iws

# Mac
.DS_Store

# Maven
log/
target/
```

이후 깃허브에 푸쉬하게 되면 과제를 위한 Maven 프로젝트 세팅이 완료된다.