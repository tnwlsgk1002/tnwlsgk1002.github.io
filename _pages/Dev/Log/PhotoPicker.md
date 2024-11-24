--- 
title: "targetSDK34 대응하면서 정리한 PhotoPicker"
date: "2024-11-24" 
tags: 
    - android
---

전에 targetSDK 34 대응할 때 PhotoPicker 관련 히스토리 정리용.

# Photo Picker
---
>The photo picker provides a browsable interface that presents the user with their media library [(공식 문서)](https://developer.android.com/training/data-storage/shared/photopicker?hl=ko)

<div style="width: 50%;">
    <img src="/assets/img/post/photopicker/kotlin-picker.gif" alt="PhotoPicker" style="width: 100%; height: auto;">
</div>
* 안드로이드에서 제공하는 사진 선택기.
* 단일 / 멀티 and 이미지 / 동영상 제공 하며, 권한 없이 미디어에 접근 가능
* Photo Picker의 사용 조건
    *  Android 13 이상 이거나 혹은 Android 11 이상이면서 [SDK R Extensions](https://developer.android.com/guide/sdk-extensions?hl=ko)이 최소 2일 때
    * 그 외에 `ACTION_OPEN_DOCUMENT` 등의 방식으로 내부에서 분기 처리

![내부코드1](/assets/img/post/photopicker/create_intent.png "PhotoPicker 내부 createIntent()")
![내부코드2](/assets/img/post/photopicker/is_system_picker_available.png "PhotoPicker 내부 isSystemPickerAvailable()")

* ✅ 나중에 알게된 사실, PhotoPicker가 일부 앨범만 표시되는 경우가 있다. [관련 이슈 트래커 링크](https://issuetracker.google.com/issues/340582178?pli=1)


# PhotoPicker 외의 미디어 선택 방법
---
1. 단순 미디어 선택이 필요한 경우 권한 없이 `ACTION_OPEN_DOCUMENT`이나 `ACTION_PICK` 를 사용하면 충분히 가능하다.
    * 이 경우 이미지, 동영상만이 아닌 pdf와 같은 파일에도 접근이 된다.
    ```kotlin
        val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
            addCategory(Intent.CATEGORY_OPENABLE)
            type = "application/pdf"

            // Optionally, specify a URI for the file that should appear in the
            // system file picker when it loads.
            putExtra(DocumentsContract.EXTRA_INITIAL_URI, pickerInitialUri)
        }
    ```

2. 만약 PhotoPicker 만으로 원하는 기능을 구현할 수 없거나 혹은 앱 디자인과 통일하고 싶은 경우 MediaStore에 직접 접근하여 **커스텀 갤러리** 구현이 필요하다.
        * targetSDK34 부터 '선택한 사진에 대한 접근 권한' 추가 -> 접근 가능한 사진이 추가 가능하도록 기능 제공 필요
        * 또한 추가로 유저가 얼마든지 권한을 '모두 허용'으로 바꾸게끔 UI를 제공하는 것도 좋다.
    * [관련 블로그 - Android 14에서 추가된 ‘사진/동영상의 일부 접근 권한’ 제대로 대응하는 방법](https://medium.com/%EB%B0%95%EC%83%81%EA%B6%8C%EC%9D%98-%EC%82%BD%EC%A7%88%EB%B8%94%EB%A1%9C%EA%B7%B8/android-14%EC%97%90%EC%84%9C-%EC%B6%94%EA%B0%80%EB%90%9C-%EC%82%AC%EC%A7%84-%EB%8F%99%EC%98%81%EC%83%81%EC%9D%98-%EC%9D%BC%EB%B6%80-%EC%A0%91%EA%B7%BC-%EA%B6%8C%ED%95%9C-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%8C%80%EC%9D%91%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-0557db6c46d9)

![커스텀갤러리](/assets/img/post/photopicker/gallery_flow.png "갤러리 제공 플로우")
