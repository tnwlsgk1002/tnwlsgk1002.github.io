---
title: Github Action으로 Android Firebase App Distribution 배포 자동화 하기
categories:
- log
tags:
- android
- ci/cd
- git
---

## 목표 : 테스트앱 배포 자동화
---

매번 테스트 앱 배포할 때마다 배포하고 노티하는 번거로움을 CD로 해결해보자.

1. 트리거가 발생하면 -> Firebase App Distribution 배포
2. 성공 시 Slack 채널에 알람

```yaml
name : App Build

on : 
    # 트리거 정의
    
job :
    # APK 추출
    # App Distrubution 업로드
    # Slack 채널에 노티
```

## 테스트
---
### 최종 코드
#### action.yaml

```yaml
name: App-Distribution

on:
  push:
    branches:
      - release/** 
      - hotfix/**
    paths:
      - docs/release-note.txt

env:
  RELEASE_NOTE_FILE: docs/release-note.txt
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout branch
        uses: actions/checkout@v2

      - name: Cache NDK
        uses: actions/cache@v2
        with:
          path: |
            ~/ndk
          key: ${{ runner.os }}-ndk-20.0.5594570-${{ hashFiles('**/build.gradle') }}
          restore-keys: |
            ${{ runner.os }}-ndk-

      - name: Install NDK
        run: |
          mkdir -p $HOME/ndk
          wget -q https://dl.google.com/android/repository/android-ndk-r20-linux-x86_64.zip -O $HOME/ndk/ndk.zip
          unzip -qn $HOME/ndk/ndk.zip -d $HOME/ndk
          echo "ndk.dir=$HOME/ndk/android-ndk-r20" >> $GITHUB_WORKSPACE/local.properties

      - name: Gradle cache
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Grant Permission for gradlew # gradlew 에 대한 퍼미션을 허용한다.
        run: chmod +x ./gradlew
        shell: bash

      - name: Create google-services.json
        run: echo '${{ secrets.GOOGLE_SERVICES_JSON }}' > ./app/google-services.json

      - name: Build APK
        run: ./gradlew assembleDebug
				
      - name: Find APK in folder
        id: find_apk
        run: |
          apk_file=$(find app/build/outputs/apk/debug -name '*.apk' -type f -printf '%T@ %p\n' | sort -n | tail -1 | cut -d' ' -f2-)
          echo "Found APK file: $apk_file"
          echo "apk_file=$apk_file" >> $GITHUB_ENV

      # Deploy to Firebase App Distribution
      - name: Deploy to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
          file: ${{ env.apk_file }}
          groups: ${{ vars.FIREBASE_GROUPS }}
          releaseNotesFile: ${{ env.RELEASE_NOTE_FILE }}
      
      # 파일 내용 읽어 환경 변수로 설정
      - name: Read Release Notes and Set Environment Variable
        run: |
          if [ -f "$RELEASE_NOTE_FILE" ]; then
              echo "Converting release notes to a JSON-friendly format..."
              RELEASE_NOTES=$(cat "$RELEASE_NOTE_FILE" | sed ':a;N;$!ba;s/\n/\\n/g')
              echo "FIREBASE_RELEASE_NOTES=$RELEASE_NOTES" >> "$GITHUB_ENV"
          else
              echo "FIREBASE_RELEASE_NOTES=No release notes found" >> "$GITHUB_ENV"
          fi

      - name: Debug Specific Environment Variable
        run: |
          echo "FIREBASE_RELEASE_NOTES: $FIREBASE_RELEASE_NOTES"
    
      # Notify
      - name: Send success notification to Slack
        if: success()
        uses: slackapi/slack-github-action@v2.0.0
        env:
          RELEASE_NOTES: ${{ env.FIREBASE_RELEASE_NOTES }}
          GROUP: ${{ vars.FIREBASE_GROUPS }}
        with:
          payload-file-path: "./artifacts.json"
          payload-templated: true
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: webhook-trigger
```

#### ./artifacts.json

```json
{
  "text": "Firebase App Deployment Notification",
  "attachments": [
    {
      "color": "#74d990",
      "blocks": [
        {
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": ":rocket: *Firebase App Distribution Successful*"
          }
        },
        {
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": "${{ env.RELEASE_NOTES }}"
          }
        },
        {
          "type": "divider"
        },
        {
          "type": "context",
          "elements": [
            {
              "type": "mrkdwn",
              "text": "Group: ${{ env.GROUP }}"
            }
          ]
        }
      ]
    }
  ]
}
```

#### release/**, hotfix/** 브랜치에서 docs/release-note.txt 변경 시 배포:

- 배포 노트까지 로컬에서 써서 push만 하면 바로 배포가 되도록 설정
- 처음에는 트리거를 이슈로 하고 이슈 내용을 배포 노트로 해도 좋을 것 같아서 해봤는데, 배포할 브랜치 설정도 필요하고, 이슈 오픈하는 과정이나 이런 게 귀찮아서 최대한 단순하게 하는 게 좋을 것 같아 pass
    - 이슈 트리거로 했던 코드는 아래에 메모

```yaml
on:
# trigger : 이슈 생성 시
 issues:
   types: [ opened ]

    # 중간 설정
    # ...
    
    # 이슈 설명 가져오기
    - name: Get issue description
      id: fetch-issue
      uses: actions/github-script@v6
      with:
        script: |
          const description = context.payload.issue?.body || "";
          core.setOutput("issue_body", description);

    # Firebase 배포
    - name: Deploy to Firebase App Distribution
      uses: wzieba/Firebase-Distribution-Github-Action@v1
      with:
        appId: ${{ secrets.FIREBASE_APP_ID }}
        serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
        file: ${{ env.apk_file }}
        groups: ${{ vars.FIREBASE_GROUPS }}
        releaseNotes: ${{ steps.fetch-issue.outputs.issue_body }}
```

#### 변수 설정:

![image](https://gist.github.com/user-attachments/assets/c72c89da-21fc-4f7d-b189-918aeaa922fc)
![image](https://gist.github.com/user-attachments/assets/583e0964-883e-4fa9-a645-fb2bd53f4a40)

- 기본적으로 위와 같은 설정 해주었고 추가로 keystore 등 추가하면 된다.
- FIREBASE_GROUPS는 파이어베이스 배포 그룹이었는데 간단히 바꿀 수 있게끔 하려고 var 로 뺐다.
	- 이 부분 그룹 ID 대충 해보려고 하다가 고생함...
	- 그룹은 들어가서 파이어베이스 그룹 이름이 아니고 이름 옆에 표기된 걸로 해야 한다.
	- 여러 개 그룹 넣을 경우에는 띄워쓰기 없이 아래처럼 해야한다.
		- "그룹1,그룹2"
		- 그룹 ID가 제대로 설정하지 않으면 아래와 같은 오류를 마주한다..
			- `HTTP Error: 404, Requested entity was not found.`

=> 단 요건 github action 버전 마다 다를 수 있어서 주의 필요
	
	![image](https://user-images.githubusercontent.com/17172841/173846092-89ae68b6-96f8-4f66-826f-47962efecd7f.png
	)

* CREDENTIAL_FILE_CONTENT의 경우는 [해당 사이트](https://github.com/wzieba/Firebase-Distribution-Github-Action/wiki/FIREBASE_TOKEN-migration) 참고해서 발급받으면 된다. 
 
#### ./artifacts.json 으로 Slack 에 노티 시 템플릿 설정: 

* [Slack Kit Builder](https://app.slack.com/block-kit-builder/T03V2A4FCLA) 에서 만들었음
* payload-templated을 true로 해주고, payload-file-path를 위로 설정해주어야 되며, 처음에 json 파일에서 바로 변수 참조했다가 `???' 로 뜨는 경우가 있었다... 요 [이슈](https://github.com/slackapi/slack-github-action/issues/203) 참고해서 env로 설정해서 해결
* 추가로 release-note.txt 파일을 그냥 cat으로 읽으면 --> 멀티 라인은 변수로 저장할 수 없어서 변수 저장할 때부터 오류, EOF로 바꾸면 형식을 artifacts.json에서 읽을 수 없다고 하기 때문에 json 이 읽을 수 있는 형식에 맞게 바꿔주었다.
