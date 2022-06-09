---
layout: post
title: Maven Central에 JAVA 라이브러리 등록하기
subtitle:
categories: 개발
tags: [Spring, Maven]

---

# Maven Central에 JAVA 라이브러리 등록하기

Controller를 만들면서, requestDto에 validation을 넣어 처리하는 경우가 많다. 

`spring-boot-starter-validation`에서 제공하는 어노테이션들로 대부분 커버가 가능했지만, 가끔 필요하지만 없어서 아쉬운 validation 이 있었다. 예를들면 파일업로드관련?

지금까지는 request를 받아서 `checkParam()` 같은 메소드를 만들어 처리를 했었는데, 뭔가 코드가 마음에 들지 않았다. 그래서 프로젝트 내부에 annotation을 만들어 spring에서 제공하는 validation과 같이 사용하였는데... 

매 프로젝트마다 이걸 복붙해서 사용한다고 생각하니 정말 별로라는 생각이 들었고.. 그런김에 겸사겸사.. 라이브러리를 만들어보자 하는 생각하여 도전해보게되었다!



**참고 블로그 및 사이트**

- [maven central 공식](https://central.sonatype.org/ ) 
- [참고 블로그](https://codeac.tistory.com/110) (프로젝트 초기 설정 및 gpg 설정 등)
- [기타참고문서](https://www.sosconhistory.net/soscon2019/content/data/session/Day%202_1730_1.pdf )



## pom.xml 설정

[참고 블로그](https://codeac.tistory.com/110)를 참고하여  maven 프로젝트 생성과 기타 설정을 완료한 후, [프로젝트](https://github.com/doohee94/common)를 만들어주었다. 

현재는 MultipartFile과 Enum으로 들어오는 Request를 validation하는 어노테이션을  만들었다.

### dependencies

Request를 Validation하는 어노테이션을 만들기 위해서는 `ConstraintValidator`를 상속받아 사용해야 하는데([관련참조블로그](https://cheese10yun.github.io/ConstraintValidator/)) 이는 `javax.validation.validation-api` 를 포함시켜주어야했다. 보통 spring 프로젝트 내부에서 사용한다면  `spring-boot-starter-validation` 의존성을 주입받아 사용했겠지만, 이 프로젝트에서는 스타터 전체보다는 관련 부분만을 끌어오고싶어 아래와 같은 dependency를 추가해주었다. 

하나 더,  MultipartFile 은 `org.springframework.spring-web`이 필요했다. `spring-web`안에는`spring-beans`, `spring-core`, `spring-jcl`의 의존성이 포함되어있는데 이 세개의 의존성 모두 필요했기 때문에 spring-web 전체를 끌어왔다. 

전체 dependencies는 아래와 같다.

```xml
<dependencies>
    <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>2.0.0.Final</version>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.2.4.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

여기서 optional 태그와 관련된 사항은 [여기](https://medium.com/@danismaz.furkan/difference-between-optional-true-optional-and-scope-provided-scope-7404ec24fb59)의 글을 참고하여 설정해주었다.

common 라이브러리를 사용하기 위해서는 상위 프로젝트에서 해당 의존성들을 모두 주입해줘야 사용할 수 있도록 하는 설정이다. (*지금 생각난것이지만, validation-api의존성은 optional 설정이 아닌 provide 설정이 더 맞는것 같다..*)

만약, `spring-web`처럼 여러개의 의존성 묶음에서 특정  의존성을 배제하고 싶다면 아래와 같이 설정해주면 제외할 수 있다. 

```xml
 <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.2.4.RELEASE</version>
        <exclusions>
        	<exclusion>
        		<groupId>org.springframework</groupId>
        		<artifactId>spring-core</artifactId>
        	</exclusion>
        </exclusions>
 </dependency>
```



### 그 외 설정

참고한 블로그 글과는 버전이 달라, 달라진 설정이 있어 포함시켜주었다. plugins 태그에 아래와 같은 내용을 포함시켜주는 것이다. (업데이트마다 달라질 수 있으니 공식문서를 보는게 좋다.)

```xml
<plugin>
	<groupId>org.sonatype.plugins</groupId>
	<artifactId>nexus-staging-maven-plugin</artifactId>
	<version>1.6.7</version>
	<extensions>true</extensions>
	<configuration>
		<serverId>ossrh</serverId>
		<nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
		<autoReleaseAfterClose>true</autoReleaseAfterClose>
	</configuration>
</plugin>
```

*완성된 pom.xml은 [github](https://github.com/doohee94/common/blob/master/pom.xml)에서 확인할 수 있습니다.* 



## sonatype 등록

라이브러리를 만들어졌다면 **sonatype**에 등록을 해야한다. 우선 프로젝트를 git에 올려둔다. 

(*~~repository만 생성해도 되는 것 같지만 올려두는게 마음이 편한것같습니다..~~* )

 이부분도 첨부한 블로그 글을 참고하면 쉽게 해결할 수있다. 🔥**하나 주의점**🔥 은 Group Id를 등록할 때, github주소를 사용한다면, `io.github.{유저네임}` 으로 등록을 해야한다는 것이다. com으로 시작할 경우 bot이 수정하라고 바로 댓글을 단다..

![com_error_img](https://user-images.githubusercontent.com/15356113/172785001-3ab44760-9903-4063-a53e-10c6e336ea8d.PNG)

아니면 본인 git에 `OSSRH-..` repository를 생성해서 본인인증을 하라고 하는데.. 난 혹시 몰라서 두개 다 해주었다. 

설정을 제대로 해준 후 시간이 지나면 <u>Congratulations! Welcome to the Central Repository!</u> 로 시작하는 댓글이 달린다. 그 후, 티켓 상태가 resolved로 바뀐다면 성공한 것이다. 



## 기타 설정

내 컴퓨터는 윈도우기 때문에 윈도우 기반으로 설정을 하였는데, 

🔥**<u>명령프롬프트는 꼭 관리자모드로 실행🔥</u>**시켜야한다.

### setting.xml

maven 프로젝트를 올리기 위해서는 [maven](https://allonsyit.tistory.com/12)을 다운받고 setting.xml을 설정해주어야한다. 

윈도우에서는 .m2 폴더 밑에 존재한다고 하였는데, 난 없었다. 그래서 생성하여 사용했다. 

`profiles`태그의 내용은 [기타참고문서](https://www.sosconhistory.net/soscon2019/content/data/session/Day%202_1730_1.pdf )를 참고하여 설정해주었다. 

**C:/Users/{usename}/.m2/setting.xml**

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>  
        <server>
            <id>ossrh</id>
            <username>{sonatype id}</username>
            <password>{sonatype password}</password>
        </server>
    </servers>
    <profiles>
        <profile>
          <properties>
            <gpg.executable>gpg</gpg.executable>
            <gpg.passphrase>${env.MAVEN_GPG_PASSPHRASE}</gpg.passphrase>
          </properties>
        </profile>
    </profiles>
</settings>
```

### GPG

 [gpg](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=phrack&logNo=80106480583)를 다운받고, 키를 생성하여 asc 파일을 받아놓는다.

받은 key를 등록해야하는데 블로그에 나와있는 사이트에서는 제대로 등록이 되는것 같지 않아 [새로운 사이트](https://keys.openpgp.org/)를 찾아냈다. 

형광표시된 링크로 들어가 첨부파일에 받아놓은 asc 파일을 올려준다. 조금 후에 key의 public id를 입력하고 찾기를 눌러준다면 나의 key 정보가 뜨게 된다. 

![key](https://user-images.githubusercontent.com/15356113/172785008-e7a1b004-5dfd-4afd-897e-5741fac0a0ac.PNG)



## Deploy

이제 만든 프로젝트를 deploy해야한다. 

명령프롬프트를 관리자 모드로 실행한 후, 프로젝트의 경로로 이동해준다. 

그 후, `mvn clean deploy -e` 명령어를 입력해준다. (-e 와 -X 옵션으로 로그를 확인할 수 있는데, -e가 더 작은 범위이다. )

중간에 `GPG Passphrase`가 뜬다면, GPG에서 생성한 key의 비밀번호를 입력해준다. 

![캡처](https://user-images.githubusercontent.com/15356113/172785028-6a06835d-51ba-41a7-8b3f-27b3a51ff88d.PNG)

이렇게 BUILD SUCCESS 가 나타나고 빌드가 끝난다면 모두 끝났다. 



### 만약 에러가 난다면?

이것 때문에 한창 고생을 했는데.. 

빌드에 성공후, 코드를 고치고 싶어 코드를 고치고 다시 빌드를 하면 에러가 발생한다. 

![error](https://user-images.githubusercontent.com/15356113/172785627-2b43bd29-069d-4942-b350-adb78825335e.PNG)

만약 마지막에 이런 에러가 발생한다면, pom.xml에서 버전을 업데이트 후 다시 deploy 명령어를 실행시켜주도록 한다. 

```xml
<groupId>io.github.doohee94</groupId>
<artifactId>common</artifactId>
<version>1.1</version>
<packaging>jar</packaging>
```





## 확인

https://s01.oss.sonatype.org/ 여기로 들어가 로그인을 해준다. (오른쪽 최상단에 로그인버튼.. sonatype에서 가입했던 아이디와 비밀번호를 사용한다.)

뭔가 업데이트가 되면서 nexus에는 따로 설정을 해주지않아도 자동으로 릴리즈가 되는것 같다. 

로그인 후, 왼쪽 artifact search에 본인의 github 아이디를 입력해주면, 이렇게 내가 올린 라이브러리가 검색된다.

![ar](https://user-images.githubusercontent.com/15356113/172785034-7bddba7b-ca00-484d-94ae-abbae30bdabe.PNG)

![finish](https://user-images.githubusercontent.com/15356113/172784736-464486c6-073f-4416-a0e7-078ac26c533f.PNG)


조금 더 시간이 지나면 외부 사이트에서도 검색이 가능해진다. 

나는 주로 https://mvnrepository.com/ 이 사이트에서 라이브러리를 검색하는데, 내가 올린 라이브러리도 검색이 잘 되는 것을 확인할 수 있다. 



## 후기

🎉생각만 했던 라이브러리 올리기를 어찌저찌 끝냈다!!🎉

사실 기능을 더 만들어서 올려야 조금더 보람찰 것 같지만.. 일단 올렸다는 그 자체만으로 뿌듯하다 :) 물론 이걸 사용하는 사람은 나밖에 없을 거지만 ㅎㅎ

개발하다가 문득 귀찮은 일이 발생할 것같다면 기능을 만들어 라이브러리에 올려서 사골처럼 우려먹고싶다 😁

