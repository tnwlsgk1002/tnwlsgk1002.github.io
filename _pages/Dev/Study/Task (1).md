--- 
title: "Tasks and the back stack (1) : Task"
date: "2024-11-27" 
tags: 
    - android, task
---

앱 내에서 화면 이동을 구현하다보면 Intent에서 Flag를 세팅해줄 일이 많이 생기는 것 같다.
아무래도 사이드 프로젝트에선 화면 진입이 대부분 한 군데인 것만 경험했는데, 실무에서는 푸시나 링크 등 진입점이 다양해지고, 중간에 쌓인 화면을 모두 꺼버리는 등 스택 관리가 중요시 하게 되어 이런 일들이 발생하는 듯하다.
작년 이맘때 쯤 이부분 헷갈려서 적어두었다가 이번 기회에 조금 더 정리해서 올려본당 

# Task
---
## task ?
> A **task** is a collection of activities that users interact with when trying to do something in your app. ([링크](https://developer.android.com/guide/components/activities/tasks-and-back-stack))

**📌 Task는 사용자 관점에서 Activity의 모음이다.**
예를 들어 이메일 앱에는 새 메시지 목록을 표시하는 Activity가 있으며,
사용자가 메시지를 선택하면 해당 메시지를 볼 수 있는 새 Activity가 열린다.
새로운 Activity는 **백 스택**에 추가되며, 뒤로가기를 하거나 종료하면 스택에서 제거가 된다.

![](https://velog.velcdn.com/images/bibbidi1819/post/a7cc4232-20e3-43f9-a4d7-5b015f8e4ff6/image.png)

## root launcher activities
앱에서 처음 실행할 Activity의 경우 AndroidManifest.xml에 아래처럼 인텐트 필터를 `ACTION_MAIN`, `CATEGORY_LAUNCHER`로 선언한다. 

```XML
<activity
          android:name=".feature.main.MainActivity"
          android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />

    <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
</activity>
```

이렇게 선언된 Activity는 곧 **root launcher activity**가 된다.
task의 root가 된다는 것이며, 
이러한 root activity에서 백 버튼을 눌렀을 때 동작 과정은 os 별로 상이하다.
* android 11 이하 : 시스템이 activity를 종료 시킨다.
* android 12 이상 : activity를 종료하는 대신, task를 background로 이동시킨다.

## background 및 foreground task

![](https://velog.velcdn.com/images/bibbidi1819/post/124d8ca1-4010-4f3c-9199-71df1b7b5a32/image.png)

Task는 하나의 응집된 작업 단위다.
만약 새 작업을 시작하거나, 새로운 앱을 열면 현재 Task는 Background로 이동하고, 현재 백스택을 그대로 유지하고 있다.

그러다 다시 Background에서 Foreground로 돌아오면, 스택에서 가장 위에 있던 Activity가 활성화 되어 사용자는 마지막으로 보고 있던 화면에서 이어서 진행할 수 있는 것.

## 😎 tip. ADB로 stack 확인하는 명령어 
```bash
adb shell dumpsys activity activities
```


# 궁금증 풀기
---
## 1. 앱마다 Task가 다른가? 앱 안에서도 Task가 다르게 선언될 수 있나?
기본적으로는 앱마다 task는 다르게 선언된다.
앱이 열리면, 기본적으로는 root launcher activity를 새로운 task에 추가하게 된다.
그러나 내가 임의로 어떤 Activity를 taskAffinity 속성을 사용해서 다른 앱의 task에 올라가게끔 선언이 가능하다. 
-> 이 부분은 직접 테스트 해보고 넣어볼 것.

앱 안에서 task 또한 위 속성을 사용하거나, Intent Flags 혹은 launchMode를 사용해서 제어가 가능하다. 

## 2. 멀티 윈도우로 있을 때에는 task는 어떻게 되는가?
시스템은 각 창별로 task를 따로 관리하게 된다. ([참고](https://developer.android.com/guide/components/activities/tasks-and-back-stack#multi-window))


