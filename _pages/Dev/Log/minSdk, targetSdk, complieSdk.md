---
title: minSdk, targetSdk, complieSdk
date: 2023-10-25
tags:
  - android
---
예전에 처음 회사에서 targetSdk34 대응할 때 적어두었던 글 백업!

### minSdk, targetSdk, complieSdk
----
> **minSdk** : 애플리케이션을 실행하는데 필요한 최소 API 수준
    - 만약 API 레벨이 이 속성에 지정된 값보다 낮은 경우 사용자는 애플리케이션을 설치할 수 없다.

> **targetSdk** : 애플리케이션이 대상으로 하는 API 레벨
    - 설정하지 않으면 기본 값은 minSdk 버전과 같다.
    - targetSdk를 올리게 되면, 최소 sdk(즉 minSdk)에서 정의된 요소만 사용하도록 제한 X, API 수준에 정의된 매니페스트 요소 혹은 동작을 사용할 수 있다.

> **complieSdk** : 컴파일 시 사용할 Android API 버전
    - targetSdkVersion <= complieSdkVersion
	    - complieSdk를 올리면 지정 API로 **컴파일** 시 사용되고, targetSdk를 올리면 **런타임** 때 사용된다.
	    - target API가 컴파일이 되지 않았는데 지원은 할 수 없기 때문에 그렇다.

 - API 레벨: Android 플랫폼 버전에서 제공하는 프레임워크 API 계정을 고유하게 식별하는 정수
 	- 각 플랫폼 버전은 정확히 하나의 api를 지원하지만, 모든 이전 API 레벨(API 레벨 1까지)에 대한 지원이 암시적으로 지원된다.

### targetSdk를 올릴 때에는 언제인가?
---
- Android 버전이 높은 기기에서 최소 targetSdk가 있을 때
	- 예를 들면 Android 15에서는 targetSdk 
- Google Play 요구사항
	- targetSdk34 때는 타겟팅이 되어야 신규 앱 등록 or 앱 업데이트가 가능했었다. [(링크)](https://support.google.com/googleplay/android-developer/answer/11926878?hl=ko)
* 왜 올려야 할까?
	* 새로운 버전의 Sdk가 나오면, 보안과 성능이 개선된다.
    - 일부는 이런 지원을 명시적으로 선언한 앱에서만 적용된다.
