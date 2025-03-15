---
title: Debounce와 Throttle
categories:
- study
tags:
- android
- cs
---

- 사용자가 버튼을 연속으로 클릭해서 연속으로 API가 호출하는 걸 어떻게 대처해야 할까?
- 검색창에 단어 입력할 때마다 검색 결과를 보여줘야 하는데 API를 자주 호출하게 되면 어쩌지?

개발하면서 이런 고민 하게 되는데, 이때 알아보게 된 방식이 Debounce와 Throttle.
둘이 근데 개념이 비슷한데 정확하게 차이가 뭘까? 싶어서 작성하는 글.

# Debounce
---

![image](https://gist.github.com/user-attachments/assets/36733b58-e228-4af8-ace9-875b5d2ce130)
> https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleFirst.png

> 🔥 debounce는 일정 시간동안 이벤트가 발생하지 않으면 마지막 이벤트를 방출한다. 
{: .prompt-info }

* 그 이전의 이벤트는 취소한다.
* 아래와 같이 flow로 구현하면 특정 시간 (1초) 전에 발생한 1과 2은 수집되지 않는 것을 확인할 수 있다. 

```kotlin
flow {  
    emit(1)  
    delay(90)  
    emit(2)  
    delay(90)  
    emit(3)  
    delay(1010)  
    emit(4)  
    delay(1010)  
    emit(5)  
}.debounce(1000).collectLatest {  
    println("debounce $it")
}
```

✅ 출력

```
debounce 3
debounce 4
debounce 5
```


# Throttle

> 🔥 Throttle은 일정 주기마다 이벤트를 캐치해서 전달한다.
{: .prompt-info } 

* 일정 주기마다 어느 시점의 이벤트를 처리하냐에 따라 throttleFirst, throttleLast로 나뉜다.
	* throttleFirst는 첫 번째 이벤트만 실행하고, 이후 이벤트는 무시한다.
	* throttleLast는 일정 시간 동안 발생한 이벤트 중 가장 마지막 이벤트를 실행한다.

### throttleFirst
* 첫 번째 이벤트만 실행만 실행하고, 이후 이벤트는 무시한다.
* flow에서는 직접 지원하지 않아서, 직접 구현해주어야 한다.
![image](https://gist.github.com/user-attachments/assets/b2d7af3d-33a6-426a-a123-079204aaf928)
> https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleFirst.png

```kotlin
fun <T> Flow<T>.throttleFirst(duration: Long): Flow<T> = flow {
	var lastTime = 0L
	collect { value ->
		val currentTime = System.currentTimeMillis()
		if (currentTime - lastTime >= duration) {
			lastTime = currentTime
			emit(value)
		}
	}
}
```

```kotlin
flow {  
    emit(1)  
    delay(90)  
    emit(2)  
    delay(90)  
    emit(3)  
    delay(1010)  
    emit(4)  
    delay(1010)  
    emit(5)  
}.throttleFirst(1000).collectLatest {  
    println("throttleFirst $it")
}
```

✅ 출력
```
throttleFirst 1
throttleFirst 4
throttleFirst 5
```

1초 간격으로 가장 먼저 발생한 이벤트 1, 4, 5가 호출된다. 

### throttleLast
* 일정 시간 동안 발생한 이벤트 중 가장 마지막 이벤트를 실행한다.
* flow에서는 sample()이 throttleLast를 지원한다. 

![image](https://gist.github.com/user-attachments/assets/382f2374-169f-4627-b386-be1582428f5b)
> https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleLast.png

```kotlin
flow {  
    emit(1)  
    delay(90)  
    emit(2)  
    delay(90)  
    emit(3)  
    delay(1010)  
    emit(4)  
    delay(1010)  
    emit(5)  
}.sample(1000).collectLatest {  
    println("sample $it")
}
```

✅ 출력
```
sample 3
sample 4
```

1초 간격으로 확인했을 때, 가장 마지막 이벤트였던 3, 4를 호출한다. 
마지막 방출된 5는 일정 주기(1초)가 돌기 전에 끝났기 때문에 방출되지 않는다.

# Debounce와 Throttle의 차이

✅ Debounce는 마지막 이벤트 이후 (이벤트 호출이 멈춘 상태로) **일정 시간이 지나야 실행**이 되는 반면, Throttle은 **일정 시간마다 이벤트를 실행**한다.

위 그림을 보면 조금 더 이해가 편한데, Debounce는 이벤트가 없는 채로 일정 주기가 지난 후에야 방출하고, Throttle은 일정 주기마다 한 번 체크를 한다는 점이다. 

실제로 버튼을 클릭했을 때 이벤트를 호출하고, debounce, throttleFirst, throttleLast를 1초로 설정하여 받았을 때 차이가 발생하는 것을 볼 수 있었다. 

이 때 구현은 flow로 진행했다. 

Debounce(1초):

{% include embed/video.html src='/path/video/soolab-debounce.mp4' %}

* 버튼을 연속으로 클릭했을 때 1초 정도 아무런 이벤트가 들어오지 않았을 때 처리했음을 볼 수 있다.

throttleFirst(1초):

{% include embed/video.html src='/path/video/soolab-throttle-first.mp4' %}

* 버튼을 연속으로 클릭했을 때 처음 이벤트가 실행되고 이후 1초 동안의 이벤트는 무시되었다.

throttleLast(1초):

{% include embed/video.html src='/path/video/soolab-throttle-last.mp4' %}

* 버튼을 연속으로 클릭했을 때 1초 동안 호출한 이벤트 중 마지막 이벤트만을 처리했다.
# 사용 예시

📌 debounce : 검색 시 자동 완성, 아이디 등 유효성 검사

단어를 타이핑할 때 사람마다 타이핑 하는 속도는 다르기 때문에 일정 기간마다 이벤트를 쏘는 Throttle은 같은 단어를 여러 번 쏘게 될 수 있다. 
debounce를 쓰면 일정 기간이 지난 후 호출하게 되어 이벤트가 과도하게 호출되는 것을 막을 수 있다. 

📌 throttle : 버튼 중복 클릭 방지, 무한 스크롤

버튼을 연속으로 클릭하는 경우 이벤트는 총 한 번만 호출되어야 한다.
이때 throttleFirst()를 사용하면 가장 처음 이벤트만 호출되어 효율적이다.

그 외에도 안드로이드에서는 직접 구현하진 않지만, 프론트에서는 무한 스크롤 구현 시 최적화를 위해서 throttle 기법을 사용하기도 한다.
(~~android 사례보다 react 관련 자료가 훨씬 많았다고 느낌~~)
(~~Paging3이 없었다면 직접 구현해야 했을 수도 있겠다는 생각도 든다.~~)

# 추가: ThrottleLatest

위 내용은 별도로 분리했다. 
flow로 구현하고자 해도 복잡하고, 인터넷 예제에 있는 대로 해봐도 동작 확인도 제대로 못했다. 

RxJava에서는 지원하는 함수인데, 기존 throttleFirst + throttleLast를 합쳤다고 보면 된다. 
처음 이벤트를 즉시 실행하며, 그 이후에는 최신 이벤트만을 실행한다.

![image](https://gist.github.com/user-attachments/assets/327121f9-c9d2-45ee-a065-926496e841b8)
> https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/throttleLatest.png

어떤 차이가 있는지 궁금해 이 부분은 함수를 제공하는 RxJava로 테스트해봤다.
처음 이벤트를 받는 부분은 throttleFirst처럼 동작하다가, 이후에는 throttleLast처럼 동작한다. 

{% include embed/video.html src='/path/video/soolab-throttle.mp4' %}
# 요약

 🚀 `debounce()` vs `throttle()`

|             | `debounce()`            | `throttle()`        |
| ----------- | ----------------------- | ------------------- |
| **동작 방식**   | 마지막 이벤트 후 일정 시간이 지나야 실행 | 일정 시간마다 실행          |
| **사용 예시**   | 검색 입력, 자동 완성            | 버튼 중복 클릭 방지, 무한 스크롤 |

---


🚀 `throttleFirst()`, `throttleLast()`, `throttleLatest()`

|        | `throttleFirst()` | `throttleLast()` | `throttleLatest()`         |
| ------ | ----------------- | ---------------- | -------------------------- |
| **설명** | 첫 번째 이벤트만 실행      | 가장 마지막 이벤트 실행    | 처음 이벤트 실행 이후 가장 마지막 이벤트 실행 |

# 참고 자료 

* [브랜디 랩스 > 안드로이드 이벤트 핸들링 적용하기](https://labs.brandi.co.kr/2021/05/13/leesy5.html)
* [(Android) Debounce와 Throttle 이해하기](https://medium.com/jaesung-dev/android-debounce%EC%99%80-throttle-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-e6da12d18d26)
* [(영문) Throttle vs Debounce in RxSwift](https://medium.com/fantageek/throttle-vs-debounce-in-rxswift-86f8b303d5d4)
* [(StackOverFlow) What difference between throttlelatest and throttlelast in rxjava]( https://stackoverflow.com/questions/56323709/what-difference-between-throttlelatest-and-throttlelast-in-rxjava)
* [From RxJava to Kotlin Flow: Throttling](https://proandroiddev.com/from-rxjava-to-kotlin-flow-throttling-ed1778847619)
