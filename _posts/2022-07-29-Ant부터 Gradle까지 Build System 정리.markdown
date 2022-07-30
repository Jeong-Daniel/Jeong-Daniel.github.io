---
layout: post
title:  "Ant부터 Gradle까지 Build System 정리"
date:   2022-07-29 12:38:46 +0900
categories: JAVA
tags: java, project, intellij
---
스프링부트 예제를 따라하다가 gradle build하는 구간에서 한동안 막혀서 고생을 했습니다. 결론적으로 원인은 쓰기 권한 문제였는데 관리자 권한으로 실행하자 정상적으로 빌드가 되었습니다. 그러던 와중에서 지금 내가 하고 있는 gradle build하는 것 자체가 무엇인지 궁금했습니다. 여기에서 사용할 라이브러리를 빌드를 하고 의존성을 주입하고 직감적으로 프로젝트 관리를 위한 툴인 것인 것을 알겠는데, 좀더 알았으면 하는 생각에서 찾기 시작했습니다.

![프로젝트](\assets\img\project.png)

Intellij에서 프로젝트를 생성하면 다음과 같이 나옵니다. Build system에서 intelliJ와 Maven, Gradle 3가지가 있습니다. Maven과 Gradle을 알아보고 왜 지금 프로젝트에서는 Gradle을 사용하는지 살펴보겠습니다.

![빌드시스템](\assets\img\build-system.png)

일단 Build system에 대한 이해가 필요합니다.
사전적인 정의는 소스코드 파일을 컴퓨터나 휴대폰에서 실행할 수 있는 독립 소프트웨어 가공물로 변환하는 과정과 결과물을 뜻합니다. 그리고 이 과정을 담당하는 빌드 도구에 의해서 전체 프로젝트가 관리를 합니다.

이전의 개발자는 라이브러리를 직접 주입했으나 여러가지 문제를 내포하고 있었습니다. 공통작업에서 개발자 간의 버전관리가 어려웠으며 다운받은 jar파일은 출처가 불분명하다면 보안상의 위험이 있습니다. 계속해서 빠른 시간 동안 필요하 라이브러리는 급속하게 증가하는데 이들의 버전을 싱크하기 위해서라도 빌드 도구는 필요했습니다.

그렇게 이런 문제를 한번에 해결해줄 수 있는 도구가 필요했으며 위 그림 처럼 아파치 Ant - Maven - Gradle로 이어지는데 어떤 형태로 따라가는지 살펴보겠습니다.

![Apache Ant](\assets\img\Apache_Ant.png)
## Apache Ant
아파치 앤트는 Java기반 빌드 도구로 2000년도에 발표했습니다. 앤트의 특징은 XML기반으로 버전을 관리를 했으며 자유도가 높았지만 정해진 형식이 없어 스크립트 재사용이 어려웠으며 복잡한 프로젝트일 수록 Build과정을 이해하기 어려웠습니다. 그리고 XML, Remote Repository를 가져올 수 없었기에 이를 해결하기 위해 ivy가 도입이 되었습니다.

![Maven](\assets\img\maven.jpg)
## Apache Maven
Maven은 프로젝트에 필요한 모든 'Dependency(종속성)'을 리스트 형태로 Maven에게 알려 관리할수 있도록 돕는 방식을 지원합니다.

Mavne의 특징은
- Dependency를 관리하고, 표준화된 프로젝트(Standardized project)를 제공
- XML, remote repository를 가져 올 수 있음 : 개발에 필요한 종속되는 'jar', 'class path'를 다운로드 할 필요 없이 선언만으로 사용 가능
- 상속형 : 하위 XML이 필요 없는 속성도 모두 표기
'POM.xml' Maven 파일에 필요한 'Jar', 'Class Path' 선언시 직접 다운로드 필요 없이 Repository에서 필요한 모든 파일들을 해당 프로젝트로 불러와 준다.

![Maven_deps](\assets\img\maven-deps.png)

한편 Maven의 단점으로는
- 라이브러리가 서로 종속할 경우 XML 복잡해짐
- 계층적인 데이터를 표현하기에 좋지만, 플로우나 조건부 상황을 표현하기엔 어려움
- 편리하나 맞춤화된 로직 실행이 어려움

 ```xml
<project>
  <!-- model version is always 4.0.0 for Maven 2.x POMs -->
  <modelVersion>4.0.0</modelVersion>
  <!-- project coordinates, i.e. a group of values which
       uniquely identify this project -->
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0</version>

  <!-- library dependencies -->
  <dependencies>
    <dependency>
      <!-- coordinates of the required library -->
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <!-- this dependency is only used for running and compiling tests -->
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
 ```

![Gradle](\assets\img\Gradle.png)
## Apache Gradle
하지만 Maven역시 빌드를 '자동화'하는 것에서는 부족함이 많았습니다. 이에 요구에 따라서 Gradle은 JVM 기반 빌드 도구로 Ant와 Maven의 장점을 보완하였습니다.

- 오픈소스기반의 build 자동화 시스템으로 Groovy기반 DSL(Domain-Specific Language)로 작성됨
여기서 Groovy는 JVM에서 실행 가능한 스크립트 언어로 JAVA와 달리 소스 코드 컴파일을 할 필요가 없이 소스코드 그대로 실행할 수 있으며 JAVA와 호환된다는 특징이 있습니다.

- Build-by-convention(빌드별 규약)을 바탕으로해서 스크립트 규모가 작고 읽기 쉬움
필요할 경우 Gradle에서 제공하는 플러그인을 사용하면 간단하게 빌드 스크립트를 작성할 수 있고, 필요하다면 사용자 정의 플러그인을 생성하는 것 또한 가능

- Multi 프로젝트의 빌드를 지원하기 위해 설계(관리자 서버와 사용자 서버를 분리)
- 초기 프로젝트 설정 시간 절약 가능(기존의 Maven과 Ivy같은 도구와 호완)
- Incremental Builds 점진적 빌드 제공(모든 상태에 대해서 빌드를 수행하는 것이 아니라 마지막 빌드 호출 이후 task의 입출력/구현이 변경됬을때만 수행)
- 빌드 캐시, 데몬 프로세스 지원, 확장성 용의

![Gradle_plugin](\assets\img\gradle-plugin-overview.png)
- 설정 주입 방식(Configuration Injection)
Maven에서 멀티 프로젝트에서 특정 설정을 다른 모듈에서 사용하려면 상속을 받아야하지만 Gradle는 설정 주입 방식으로 이를 해결했습니다. 프로젝트의 조건을 체크할 수 있어서 각 프로젝트별 주입되는 설정을 다르게 할 수 있습니다. 또한 빌드 속도가 Maven에 비해 10배 이상 빠르다고 합니다. 그래서 대규모일 수록 Gradle를 사용 하는 것이 바람직 합니다.

그리고 안드로이드 스튜디오의 공식 빌드 시스템이며 Java이외에도 C언어 Python등 여러 언어를 지원합니다. 

```groovy
plugins {
	id 'org.springframework.boot' version '2.7.2'
	id 'io.spring.dependency-management' version '1.0.12.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
	jcenter()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	implementation 'org.springframework.boot:spring-boot-starter-web'
  //스프링관련의존성은 컴파일과 런타임 모두에 사용

	compileOnly 'org.projectlombok:lombok'
  //lombok는 컴파일 시에만 사용
  
  runtimeOnly 'com.h2database:h2'
  //h2데이터베이스는 런타임에만 사용

	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'
	testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.2'
	testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.2'
  //junit와 spring-boot-start-test는 테스트에만 사용
}

tasks.named('test') {
	useJUnitPlatform()
}
```

다만 아직까지 현업에서 Maven을 많이 사용한다는 이야기가 있는데 이는 기존에 만들어진 빌드시스템은 특별한 이유가 있는 것이 아니라면 그대로 유지하는 것이 유지보수하는데 유리하다. 하지만 새로 프로젝트를 시작할 것이라면 Gradle이 속도면에서도 편의성 측면에서도 이점이 크기 사용할 것을 적극 권장