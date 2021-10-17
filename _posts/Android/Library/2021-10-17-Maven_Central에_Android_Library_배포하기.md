---
title: Maven Central에 Android Library 배포하기
author: Yun-Jae Na
date: 2021-10-17 22:10:00 +0900
categories: [Android, Library]
tags: [Android, MavenCentral, Maven, Library]
image : https://drive.google.com/uc?export=view&id=1rkOZRaF9eInrMrKWw8UFVpRplH8gK62y
---

## Sonatype Jira Account 생성

[https://issues.sonatype.org/](https://issues.sonatype.org/)

## Sonatype Jira에서 새로운 이슈 생성

[참고 이슈](https://issues.sonatype.org/browse/OSSRH-70109?jql=text%20~%20%22yunjaena%22)

![making issue](https://drive.google.com/uc?export=view&id=1H5NuehiQJmKAYiRmud2tLgdzQF1t02dS)

1. 신규 프로젝트 생성
2. 제목 입력
  - ex) Create repository for io.github.yunjaena:slackwebhook
3. Group id
  - ex) io.github.yunjaena
4. Project URL
  - ex) https://github.com/yunjaena/SlackWebhookAndroid
5. SCM URL
  - ex) https://github.com/yunjaena/SlackWebhookAndroid.git

##  이슈 등록후 Group ID를 통한 인증

이슈를 등록하면 봇이 자동으로 답글을 단다.

### Github 주소로 등록할시

```
Please create a public repo called https://github.com/yunjaena/OSSRH-70109 to verify github account ownership.
If you do not own this github account, please read:
https://central.sonatype.org/publish/requirements/coordinates/
```

OSSH-70109 라는 레포를 만들고 생성 되었다는 답글을 달면

```
Done create https://github.com/yunjaena/OSSRH-70109 Thanks!
```

maven central에 등록할 준비가 되었다고 답글을 달아준다.

### 개인 도메인으로 등록할시

- ex) dev.yunzai

- DNS 관리자에서 새로운 Record로 등록해줘야 한다. => Type: TXT / Host : @ / TXT VALUE : OSSRH-70109 (지라이슈 번호) / TTL : 1 Hour

- 등록이 잘 되었는지 확인한다.

```$shell
$ host -t txt yunzai.dev

=> yunzai.dev descriptive text "OSSRH-70109"
```

- DNS Record에 추가했다는 것을 답글로 단다.

```
The DNS record has been added to yunzai.dev.
```

## Nexus repository manager에 접속

- [https://s01.oss.sonatype.org/](https://s01.oss.sonatype.org/)
- 로그인 후 (Jira와 아이디 비번 동일) Staging Profile을 선택한다.
- 리스트에서 등록한 Group Id를 선택
- 주소창을 보면 Profile Id가 보인다.
- ex) https://s01.oss.sonatype.org/#stagingProfiles;11111ec1cc1b1 => 11111ec1cc1b1 가 sonatypeStagingProfileId로 사용된다.

## GPG Signing (Mac 기준)

1. Homebrew를 통해 gpg 패키지를 설치한다.

```shell
$ brew install gpg
```

2. 터미널에 들어가서 키 생성을 한다.

```shell
$ gpg --full-generate-key
```

- 암호화 방식 선택 => 1
- 암호화 키 크기 선택 => 4096
- 키 유효기간 설정 => 0
- 이름, 이메일, 코멘트(공란 가능) 입력
- 이후 보안 암호 문구 작성 창에서 암호 입력 => 꼭 메모해 주세요

3. 생성된 키 확인

```shell
$ gpg --list-secret-keys --keyid-format LONG
```

```
sec   rsa4096/ABCDE12345678900 2021-10-13 [SC]
      844D192FB6761C91718A49CAED4AF8684FCBC4C9
uid                 [ultimate] YunJae Na <yunjaena@gmail.com>
ssb   rsa4096/10A8ECEC7DD30252 2021-10-13 [E]
```

여기서 `ABCDE12345678900` => 끝에 8자리 `45678900`가 사용된다.

4. 공개키를 업로드를 해줍니다. (끝에 8자리)

```shell
$ gpg --keyserver keyserver.ubuntu.com --sned-keys 45678900
```

5. secring.gpg 파일 생성

```shell
gpg --export-secret-keys 45678900 > signing.gpg
```  

## Signing parameter 설정

프로젝트 local.properties 에 아래 항목을 추가한다.

```groovy
signing.keyId=[KEY_ID] // GPG Signing을 한후 나온 마지막 8자리 입력 ex) 45678900
signing.password=[KEY_PASSPHRASE] // GPG Signing을 할때 사용한 암호 입력
signing.secretKeyRingFile=[ABSOLUTE_PATH] // secring.gpg 파일 경로 입력 ex) /Users/user/keys/secring.gpg
ossrhUsername=[SONATYPE_USERNAME] // SonarType 아이디
ossrhPassword=[SONATYPE_PASSWORD] // SonarType 비밀번호
sonatypeStagingProfileId=[SONATYPE_STAGING_PROFILE_ID] // SonarType Profile Id ex) 11111ec1cc1b1
```

전체 항목을 채우면 아래와 같이 나온다.

```groovy
signing.keyId=45678900
signing.password=asdfg1234
signing.secretKeyRingFile=/Users/nayunjae/project/SlackWebhookAndroid/SecretRingKey.gpg
ossrhUsername=yunjaena
ossrhPassword=1q2w3e4r!
sonatypeStagingProfileId=11111ec1cc1b1
```

## 프로젝트 정보 설정

### publish.gradle

```groovy
ext {
    PUBLISH_GROUP_ID = 'io.github.yunjaena'
    PUBLISH_VERSION = '0.0.3'
    PUBLISH_ARTIFACT_ID = 'slackwebhookandroid'
    PUBLISH_DESCRIPTION = 'SlackWebHook Android Library'
    PUBLISH_URL = 'https://github.com/yunjaena/SlackWebhookAndroid'
    PUBLISH_LICENSE_NAME = 'Apache License'
    PUBLISH_LICENSE_URL =
            'https://github.com/yunjaena/SlackWebhookAndroid/blob/main/LICENSE'
    PUBLISH_DEVELOPER_ID = 'yunjaena'
    PUBLISH_DEVELOPER_NAME = 'Yun-Jae Na'
    PUBLISH_DEVELOPER_EMAIL = 'yunjaena@gmail.com'
    PUBLISH_SCM_CONNECTION =
            'scm:git:github.com/yunjaena/SlackWebhookAndroid.git'
    PUBLISH_SCM_DEVELOPER_CONNECTION =
            'scm:git:ssh://github.com:yunjaena/SlackWebhookAndroid.git'
    PUBLISH_SCM_URL =
            'https://github.com/yunjaena/SlackWebhookAndroid/tree/main'
}
```

## 배포 스크립트 추가

프로젝트 폴더에 srcipt 폴더를 추가하고 아래 2개 파일을 추가해줍니다.

### publish-module.gradle

```groovy
apply plugin: 'maven-publish'
apply plugin: 'signing'

task androidSourcesJar(type: Jar) {
    archiveClassifier.set('sources')
    if (project.plugins.findPlugin("com.android.library")) {
        // For Android libraries
        from android.sourceSets.main.java.srcDirs
        from android.sourceSets.main.kotlin.srcDirs
    } else {
        // For pure Kotlin libraries, in case you have them
        from sourceSets.main.java.srcDirs
        from sourceSets.main.kotlin.srcDirs
    }
}


artifacts {
    archives androidSourcesJar
}

group = PUBLISH_GROUP_ID
version = PUBLISH_VERSION

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                // The coordinates of the library, being set from variables that
                // we'll set up later
                groupId PUBLISH_GROUP_ID
                artifactId PUBLISH_ARTIFACT_ID
                version PUBLISH_VERSION

                // Two artifacts, the `aar` (or `jar`) and the sources
                if (project.plugins.findPlugin("com.android.library")) {
                    from components.release
                } else {
                    artifact("$buildDir/libs/${project.getName()}-${version}.jar")
                }

                artifact androidSourcesJar

                // Mostly self-explanatory metadata
                pom {
                    name = PUBLISH_ARTIFACT_ID
                    description = PUBLISH_DESCRIPTION
                    url = PUBLISH_URL
                    licenses {
                        license {
                            name = PUBLISH_LICENSE_NAME
                            url = PUBLISH_LICENSE_URL
                        }
                    }
                    developers {
                        developer {
                            id = PUBLISH_DEVELOPER_ID
                            name = PUBLISH_DEVELOPER_NAME
                            email = PUBLISH_DEVELOPER_EMAIL
                        }
                    }

                    // Version control info - if you're using GitHub, follow the
                    // format as seen here
                    scm {
                        connection = PUBLISH_SCM_CONNECTION
                        developerConnection = PUBLISH_SCM_DEVELOPER_CONNECTION
                        url = PUBLISH_SCM_URL
                    }
                }
            }
        }
    }
}

signing {
    sign publishing.publications
}
```

### publish-root.gradle

```groovy
// Create variables with empty default values
ext["signing.keyId"] = ''
ext["signing.password"] = ''
ext["signing.secretKeyRingFile"] = ''
ext["ossrhUsername"] = ''
ext["ossrhPassword"] = ''
ext["sonatypeStagingProfileId"] = ''

File secretPropsFile = project.rootProject.file('local.properties')
if (secretPropsFile.exists()) {
    // Read local.properties file first if it exists
    Properties p = new Properties()
    new FileInputStream(secretPropsFile).withCloseable { is -> p.load(is) }
    p.each { name, value -> ext[name] = value }
} else {
    // Use system environment variables
    ext["ossrhUsername"] = System.getenv('OSSRH_USERNAME')
    ext["ossrhPassword"] = System.getenv('OSSRH_PASSWORD')
    ext["sonatypeStagingProfileId"] = System.getenv('SONATYPE_STAGING_PROFILE_ID')
    ext["signing.keyId"] = System.getenv('SIGNING_KEY_ID')
    ext["signing.password"] = System.getenv('SIGNING_PASSWORD')
    ext["signing.secretKeyRingFile"] = System.getenv('SIGNING_SECRET_KEY_RING_FILE')
}

// Set up Sonatype repository
nexusPublishing {
    repositories {
        sonatype {
            stagingProfileId = sonatypeStagingProfileId
            username = ossrhUsername
            password = ossrhPassword
            nexusUrl.set(uri("https://s01.oss.sonatype.org/service/local/"))
            snapshotRepositoryUrl.set(uri("https://s01.oss.sonatype.org/content/repositories/snapshots/"))
        }
    }
}
```


## 배포 플러그인 적용

```groovy
// project build.gradle 에 추가

buildscript {
    dependencies {
        classpath 'io.github.gradle-nexus:publish-plugin:1.1.0'
    }
}

apply plugin: 'io.github.gradle-nexus.publish-plugin'
apply from: "${rootDir}/scripts/publish-root.gradle"
apply from: 'publish.gradle'
```

## 라이브러리 업로드

```shell
$ ./gradlew module명:publishReleasePublicationToSonatypeRepository
```

## 라이브러리 배포

1. [https://s01.oss.sonatype.org/](https://s01.oss.sonatype.org/) 에 접속한다.
2. Staging Repositories 에 들어가면 업로드한 라이브러리가 보인다.
3. close 메뉴를 누른 후 confirm을 누른다.
4. refresh 후에 release메뉴를 눌러 배포를 진행한다.


## 배포 완료~!

2시간 정도 후에 [https://search.maven.org](https://search.maven.org)에서 업로드 된것을 확인할 수 있다.

![maven](https://drive.google.com/uc?export=view&id=1rkOZRaF9eInrMrKWw8UFVpRplH8gK62y)

## 참고 링크

- [https://www.waseefakhtar.com/android/publishing-your-first-android-library-to-mavencentral/](https://www.waseefakhtar.com/android/publishing-your-first-android-library-to-mavencentral/)
- [https://betterprogramming.pub/how-to-migrate-an-open-source-android-library-to-maven-central-432c37159bcd](https://betterprogramming.pub/how-to-migrate-an-open-source-android-library-to-maven-central-432c37159bcd)
- [https://www.waseefakhtar.com/android/publishing-your-first-android-library-to-mavencentral/](https://www.waseefakhtar.com/android/publishing-your-first-android-library-to-mavencentral/)
- [https://proandroiddev.com/publishing-android-libraries-to-mavencentral-in-2021-8ac9975c3e52](https://proandroiddev.com/publishing-android-libraries-to-mavencentral-in-2021-8ac9975c3e52)
