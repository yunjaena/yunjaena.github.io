---
title: Slack Webhook from Android
author: Yun-Jae Na
date: 2021-06-15 02:15:00 +0900
categories: [Android, Advance]
tags: [Android, Slack]
image: https://drive.google.com/uc?export=view&id=1Sf11rU-CDmQo1BF92Fq-8ZKFzWi1YJXI
---

Android 앱에서 크리티컬한 이슈가 발생하였을때(ex) 인앱 결제 실패, SNS 로그인 실패 등..) 빠른 대응을 하고 싶은 경우가 생겨서 슬랙으로 메시지를 보내는 방법을 생각하게 되었다.


## 사용 라이브러리

- [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager?hl=ko) : 앱이 종료되거나 기기가 시작될때 또는 네트워크가 연결되어있을때 메시지를 보내기 위해 사용

- [Retrofit](https://square.github.io/retrofit/) : REST 기반의 웹 서비스를 통해 JSON 구조의 데이터를 쉽게 가져오고 업로드 가능하게 도와주는 라이브러리

## 설정 및 구현 과정

### Slack 설정

- Slack 에서 Webhook을 받을 채널을 추가한다.

![slack_add_channel](https://drive.google.com/uc?export=view&id=1_OjOJ1sKOUZ8kRRspqIw0hfC_JA5DCrh){: width="400"}

- [Incoming-webhook](https://slack-webhook-test-hq.slack.com/apps/A0F7XDUAZ--) App을 설치하는 페이지로 이동한다.

- `Slack에 추가` 버튼을 누른 후 Webhook을 받을 채널을 추가 해준다.

![webhook_add](https://drive.google.com/uc?export=view&id=1UfHhotOsTlBR9yM9bVIOYhHtZJEiIlEy){: width="400"}

- 설정 완료 후에 아래와 같은 화면이 나오는데 웹후크 URL을 통해서 메시지를 보내게 된다.

![webhook_app_setting_complete](https://drive.google.com/uc?export=view&id=1c-GUIezmWmkQW4bavF_L6yB6VNt_HGL1){: width="400"}

### 구현

- incoming webhook document 를 참고해서 request body가 아래와 같다는 것을 알 수 있다.

```json
{
	"fallback": "첨부 파일이 있음을 알고 있지만 이를 표시하지 않는 클라이언트가 표시하는 첨부 파일에 대한 필수 텍스트 요약입니다.",
	"text": "첨부 파일 내에 표시되어야 하는 옵션 텍스트",
	"pretext": "서식이 지정된 데이터 위에 표시되어야 하는 옵션 텍스트",
	"color": "#36a64f", // 'good', 'warning', 'danger' 또는 16진수 색상 코드 중 하나일 수 있습니다.
	// 메시지의 테이블에 필드가 표시됩니다.
	"fields": [
		{
			"title": "필수 필드 제목", // 제목에 태그가 포함되어 있지 않을 수 있으며 자동으로 이스케이프됩니다,
			"value": "필드의 텍스트 값입니다. 표준 메시지 태그를 포함할 수 있으며 정상적으로 이스케이프되어야 합니다. 복수 행일 수 있습니다.",
			"short": false // `value`가 다른 값과 나란히 표시될 정도로 짧은지를 나타내는 옵션 플래그
		}
	]
}
```

- 이를 토대로 POJO 클래스를 구현해준다. 구현 코드는 아래 링크에서 확인할 수 있다.
  - [Slack webhook request body](https://github.com/yunjaena/SlackWebhookAndroid/blob/main/slackwebhook/src/main/java/dev/yunzai/slackwebhook/SlackWebHook.kt)

- WorkManager를 통해서 네트워크 연결 시 해당 Request를 보낼 수 있게 한다. 여기서 setInputData를 통해서 데이터를 Worker class에 전달해 줄 수 있다.

```kotlin
internal fun SlackWebHook.sendMessage(context: Context) {
    val requestBodyString = Gson().toJson(this)

    val webHookWork = OneTimeWorkRequestBuilder<SlackWebHookWorker>()
            .setInputData(workDataOf(BODY to requestBodyString))
            .setConstraints(getWorkerConstraints())
            .build()

    WorkManager.getInstance(context.applicationContext).enqueue(webHookWork)
}

private fun getWorkerConstraints(): Constraints {
    return Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
}
```

- 마지막으로 Worker 클래스에 전송하는 로직을 추가하면 끝난다!
  - [Slack webhook send worker](https://github.com/yunjaena/SlackWebhookAndroid/blob/main/slackwebhook/src/main/java/dev/yunzai/slackwebhook/SlackWebHookWorker.kt)

## How To Use?

- 쉽게 사용해보고 싶으신 분들을 위해 Maven central에 해당 라이브러리를 올려놨습니다.

<a href="https://search.maven.org/artifact/io.github.yunjaena/slackwebhookandroid/">
<img src="https://img.shields.io/maven-central/v/io.github.yunjaena/slackwebhookandroid" alt="License: maven-central"></a><br><br>


### Gradle
- 사용하실 application 모듈에 라이브러리를 implement 해줍니다. (X.Y.Z => 최신 버전)

```groovy
dependencies {
    implementation 'io.github.yunjaena:slackwebhookandroid:X.Y.X'
}
```

### Android Manifest

- AndroidManifest.xml에 meta data로 웹후크 URL에서 "https://hooks.slack.com/" 를 제외한 나머지 path를 복사해주세요. (string resource에 선언하시는걸 추천합니다.)

```xml
<meta-data
          android:name="slack_webhook_path"
          android:value="services/12345/6789"
/>
```

### Simple Example

- Builder를 사용하여서 보내고 싶은 내용을 세팅하고 send(context)를 통해서 메시지를 보낼 수 있습니다.

```kotlin
SlackWebHook.builder()
             .pretext("hello")
             .title("this is webhook")
             .text("test")
             .color("#FF0000")
             .timeStampEnabled(true)
             .fields("name" to "jason", "github" to "https://github.com/yunjaena")
             .build()
             .send(this)
```

## Github Link

- [https://github.com/yunjaena/SlackWebhookAndroid](https://github.com/yunjaena/SlackWebhookAndroid)
