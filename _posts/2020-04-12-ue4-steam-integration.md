---
title: UE4 Steam 업적 연동하기
layout: post
date: '2020-04-13 01:30:00'
tags:
- UE4
- Steam
image: post/ue4-steam-integration/ue4-steam-integration.png
---

UE4에서 Steam 업적 연동을 하기 위해 여러 자료를 찾아봤지만, 예상치 못한 오류가 발생해서 애를 먹었었다. 이 글에선 UE 4.22에서 Steam SDK 최신 버전을 연동하는 법, 그중 특히 업적과 내가 겪었던 이슈들에 대해서 정리해본다.<br>
공식 문서를 따라가지만, 몇몇 이슈 해결책은 완전한 방법이 아니란 점을 감안했으면 좋겠다.

* *작성일 기준 Steam SDK 버전은 1.48이므로, 이를 기준으로 진행한다.*


<br>
1. 준비
2. Online Subsystem Steam 플러그인 설정
3. 버전 업그레이드
4. Steam 업적 연동
5. Shipping 빌드용 App ID 파일 배치

<br>
기본적으로 Unreal Engine 4에서 Steam 연동은 생각보다 간단한 절차로 진행된다.<br>
아래 공식 문서 내용을 바탕으로 작성했다.
https://docs.unrealengine.com/ko/Programming/Online/Steam/index.html


---
### 1. 준비
먼저 언리얼 엔진4에서 Steam 연동을 하기 위해선 기본적인 준비가 필요하다.
- Steamworks에 등록된 해당 게임 App ID (없을 시 Dev용인 480 사용 가능)
- Steam 클라이언트
- 최신 Steam SDK 버전 파일
- Github 소스코드로 빌드한 엔진 (런처에서 다운 받는 binary 엔진 불가)

<br>
Steam SDK 파일을 받았으면, 엔진에서 Steam을 연동할 수 있도록 몇몇 dll 파일들을 수동으로 옮겨주어야 한다. 
32비트와 64비트 모두 지원할 수 있도록 해당 경로에 직접 폴더를 만들어주어야 한다.
> - */YourUnrealEnginePath/Engine/Binaries/ThirdParty/Steamworks/Steam[Current Version]/Win64*
> - */YourUnrealEnginePath/Engine/Binaries/ThirdParty/Steamworks/Steam[Current Version]/Win32*

[Current Version]은 'v148'과 같은 형식으로, 이를 포함한 전체 파일 경로는 아래 형식과 같다.
*"/YourUnrealEnginePath/Engine/Binaries/ThirdParty/Steamworks/Steamv148/Win64"*

만약 엔진에서 이미 제공하는 기본 버전이 같다면, 새로 폴더를 만들 필요는 없다.

각각 폴더에 4 개의 파일을 넣어주면 되는데, 이는 Steam SDK 폴더 내 */redistributable_bin/* 경로에 있는 파일이거나 Steam 클라이언트, 즉 Steam 런처 프로그램 경로에 있는 파일들이다.

> - steam_api.dll / steam_api64.dll
> - steamclient.dll / steamclient64.dll
> - tier0_s.dll / tier0_s64.dll
>  - vstdlib_s.dll / vstdlib_s64.dll

각각 32비트와 64비트에 맞게 파일을 복사 및 옮겨준다. Steam SDK 폴더 내에는 steam_api(64).dll 파일만 있었다. 나머지 3 개의 파일들은 Steam 클라이언트가 설치 돼있는 폴더에서 찾아 옮겨주었다.

<br>
이렇게까지 했다면, 준비 과정이 끝났다.

---
### 2. Online Subsystem Steam 플러그인 설정
공식 문서에는 일일이 ```DefaultEngine.ini```를 수정했지만, UE 4.22처럼 비교적 최신 엔진들에선 에디터에서 볼 수 있는 플러그인 세팅에 OnlineSubsystemSteam 플러그인을 활성화 해주면 된다. 그러면 자동으로 ```DefaultEngine.ini``` 파일을 세팅해주기 때문에 여기서 더 수정할 것은 SteamDevAppId와 그 외 필요하다면 추가 설정이 있다.

<br>
* *이 글에선 네트워크 통신이 필요 없는 로컬 환경 게임 기준으로 다루기 때문에 SteamDevAppID 외 더 이상은 수정하지 않고 진행하겠다.*

---
### 3. 버전 업그레이드
이제 최신 버전 SDK를 실제 UE4에 적용할 차례다. 물론 버전 업그레이드를 가장 먼저 진행하고 위 1, 2번을 진행해도 상관 없다. 오히려 공식문서에서는 버전 업그레이드를 먼저 진행했다.
본론으로 들어가서, UE4.22에서 기본적으로 제공하는 Steam SDK 버전은 1.39이다. 최신 버전(이 글에선 1.48)을 사용하기 위해선 엔진에서 여러 값을 수정해줘야 한다.
원래 Epic Games 런처에서 다운 받은 엔진을 사용하고 있었지만, 최신 Steam SDK를 사용하기 위해선 결국 엔진을 수정해야 할 수 밖에 없었기 때문에 github에서 엔진 소스코드를 받고 빌드하는 과정을 거쳤다.

문서에 따르면 엔진 내 Steam SDK 버전을 업그레이드하는 방법은 다음과 같다.

> - *.../Engine/Source/ThirdParty/Steamworks/Steam[Current Version]/sdk* 에 위치한 ```Steamworks.build.cs``` 파일
> - *.../Engine/Plugins/Online/OnlineSubsystemSteam/Source/Private/* 에 위치한 ```OnlineSubsystemSteamPrivatePCH.h``` 파일

<br>
그러나 내 경우에는 좀 달랐다. 특히 이상하게도 "OnlineSubsystemSteamPrivatePCH.h" 파일이 보이지 않았다. 솔루션 탐색을 아무리 해봐도 찾을 수가 없었다... 그래서 해당 소스코드가 있는 다른 파일인 ```OnlineSubsystemSteamPrivate.h``` 파일 내에 코드를 다음과 같이 수정하려 했다.

{% highlight cpp %}
#define STEAM_SDK_ROOT_PATH TEXT("Binaries/ThirdParty/Steamworks")
{% endhighlight %}
이 코드를 아래와 같이 말이다.
{% highlight cpp %}
#define STEAM_SDK_VER TEXT("Steam[Current Version]")
{% endhighlight %}

<br>
##### 하지만 결과적으로 수정하지 않았다.
오히려 컴파일 과정에서 매크로 재정의 오류가 발생해서 가만히 두었다.
대신, 그 외에 다른 파일을 수정했는데, 바로 이 전 폴더 경로(../) 에 있는 ```OnlineSubsystemSteam.Build.cs``` 파일을 수정했다.
{% highlight cpp %}
- OnlineSubsystemSteam.Build.cs -
string SteamVersion = "Steamv148";
{% endhighlight %}
이렇게 "Steamv139"에서 "Steam[Current Version]"인 "Steamv148"로 바꿔주었다.

그리고 문서를 참고해 ```Steamworks.build.cs``` 파일도 아래와 같이 수정해주었다.
{% highlight cpp %}
- Steamworks.build.cs -
string SteamVersion = "v148"; (line: 12)
PublicDefinitions.Add("STEAM_SDK_VER=TEXT(\"1.48\")"); (line: 15)
{% endhighlight %}

<br>
처음에는 엔진 재빌드할 때 위와 같은 시행착오를 겪느라 오류를 만나 혹시 모르는 마음에 솔루션 전체를 재빌드 했지만, 그렇지 않아도 된다는 것을 깨달았다. 그냥 우클릭 후 빌드해도 정상적으로 빌드가 잘 진행됐다.

하지만, 이와 같은 과정을 거쳤을 때 가장 큰 문제가 발생했다. 컴파일 과정에서 다음과 같은 오류를 맞닥뜨렸다.

<br>
{% highlight text %}
C2664	'TSharedRef<FInternetAddr,0> ISocketSubsystem::CreateInternetAddr(uint32,uint32)': 인수 1을(를) 'SteamIPAddress_t'에서 'uint32'(으)로 변환할 수 없습니다.
{% endhighlight %}

<br>
??? 처음 보는 오류였다. 심지어 구글링해도 나오질 않아 한참 헤맸다. 결국 에러를 해결하기 위해 코드를 얉게 훑어보고 다음과 같이 바꿔봤다.

*./private/* 경로에 있는 ```OnlineSessionAsyncServerSteam.cpp``` 파일 내에
{% highlight cpp %}
NewSessionInfo->HostAddr = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateInternetAddr(SteamGameServerPtr->GetPublicIP(), Subsystem->GetGameServerGamePort()); (line: 409)
SteamUser()->AdvertiseGame(k_steamIDNonSteamGS, SteamGameServerPtr->GetPublicIP(), Subsystem->GetGameServerGamePort()); (line: 444)
{% endhighlight %}
이 두 줄에 있는 GetPublicIP() 함수 뒤에 .m_unIPv4를 추가했다.
{% highlight cpp %}
NewSessionInfo->HostAddr = ISocketSubsystem::Get(PLATFORM_SOCKETSUBSYSTEM)->CreateInternetAddr(SteamGameServerPtr->GetPublicIP().m_unIPv4, Subsystem->GetGameServerGamePort()); (line: 409)
SteamUser()->AdvertiseGame(k_steamIDNonSteamGS, SteamGameServerPtr->GetPublicIP().m_unIPv4, Subsystem->GetGameServerGamePort()); (line: 444)
{% endhighlight %}
그랬더니, 놀랍게도 컴파일이 성공적으로 됐다!

<br>
* *Note: 이 방법이 완벽한 방법인지는 의문이다. 원래 코드는 ```.m_unIPv4``` 없이 잘 빌드되는 코드이며, 이렇게 명시해줌으로써 오류를 야기할 수도 있다. 그러나 로컬 환경 게임에서 Steam 업적 연동은 잘 진행되었다. 오히려 ```Steamworks.build.cs``` 파일 내 버전을 원래 버전으로 바꾸는 순간 ```GetPublicIP()``` 함수 뒤에 있는 ```.m_unIPv4```에 대한 컴파일 오류가 발생했다. (구조체를 매개변수로 받아야 하는데, unit32형을 전달하였기 때문) <br><br>정리하면, 최초 "v148"과 같이 버전을 바꾸는 순간 컴파일 오류가 발생했고, 위처럼 임시적으로 ```GetPublicIP()``` 함수가 반환하는 구조체인 ```SteamIPAddress_t```의 ```uint32```형을 명시함으로써 해결한 방법이다. 이 임시 해결책이 과연 모든 경우에도 정상적으로 동작하는지는 의문이기 때문에, 혹여나 오류가 발생하지 않으면 무시해도 좋다.*

<br>
어찌됐든, 이러한 과정을 거쳐 성공적으로 엔진 재컴파일이 됐다면 본격적으로 Steam 연동을 테스트할 준비가 끝났다.

---
### 4. Steam 업적 연동
UE4에서 Steam 연동은 간단하다. 먼저, 다음과 같은 조건을 확인해야 한다.

> - Steam 클라이언트 로그인
> - 독립형 게임으로 실행
> - Steamworks에 있는 해당 게임의 stat 및 업적

이 세 가지 조건 중 위 두 가지를 충족한다면, 게임 실행 시 우측 하단에 Steam 플레이 중 안내 팝업이 생긴다.
성공적으로 세팅이 끝났다면, 다음과 같이 간단하게 업적을 연동할 수 있다.
1. 먼저 업적을 가져와 cache에 저장하고,
2. 해당 업적에 해당하는 stat을 업데이트하면 된다.

아래처럼 블루프린트 함수로 쉽게 사용할 수 있다.

<br>
<div style="text-align: center">
	<img src="{{site.baseurl}}/img/post/ue4-steam-integration/cache-achievement.PNG">
	<em>1. 업적 캐싱</em>
	<br>
	<img src="{{site.baseurl}}/img/post/ue4-steam-integration/write-leaderBoard-integer.PNG">
	<em>2. Stat 업데이트</em>
</div>
<br>
*참고) 이 예제에선 Stat 업데이트를 위해  ```SetSteamAchievementStat``` 이라는 BlueprintNativeEvent를 별도로 정의해주었다.*

<br>
* *C++ 코드로 연동하는 법?*<br>원래 업적 연동도 코드로 작성하려 했으나, 여러 문서를 찾아봐도 제대로 동작하는 걸 찾을 수가 없었다. 그래서 시간 절약을 위해 소스코드와 이미 편리하게 지원하는 블루프린트를 같이 사용하기로 결정하였다.<br>이 때 프로젝트 세팅에 BlueprintNativeEvent를 Inclusive가 아닌 Exclusive로 선택 후 해당 블루프린트 애셋들을 지정해주어야 패키징 후 crash가 발생하지 않았다. 이 에러도 처음 보는 에러였는데, *"Assertion failed: Ptr (중략) PersistentEvaluationData.h"* 대략 이런 에러였다. <br>이 프로젝트에서 기존에 BlueprintNativeEvent를 사용하지 않았기 때문에, 단순히 프로젝트의 BlueprintNativeEvent 세팅이 문제였는지 혹은 BlueprintNativeEvent에 Steam 업적 함수를 같이 사용하였기 때문에 문제가 생겼는지는 확실하지 않다.

<br>
사실 stat을 업데이트 한다는 것에 의문이 들었다. 왜 ```WriteAchievementProgress``` 함수는 사용하지 않을까?
로컬환경에서 테스트했을 때 이 함수를 사용하면 항상 업적이 클리어되는 현상이 있었다. 구글링을 해본 결과 ```WriteAchievementProgress``` 함수는 Progress가 0보다 크면 무조건 업적을 클리어시킨다고 한다. 그렇기 때문에 stat을 업데이트 시키고, 자동으로 stat이 해당 업적의 Max 카운트에 도달하면 업적이 클리어되도록 하는 방법을 선택했다. ```WriteLeaderboardInteger``` 함수는 해당 stat 값을 업데이트해준다.

<div style="text-align: center">
	<img src="{{site.baseurl}}/img/post/ue4-steam-integration/write_achievement_progress.PNG">
	<em>WriteAchievementProgress 함수</em>
</div>

<br>
##### * 여기서 주의할 점이 있다.
바로 stat 이름을 짓는 법인데, 이것도 커뮤니티 덕분에 오류를 빨리 해결할 수 있었다.

만약 내가 *'test'* 라는 이름의 stat의 값을 업데이트하고 싶다면, Steamworks에는 실제 stat 이름을 *'test_test'*와 같이 지정해줘야 한다. 이는 언리얼에서 자동으로 해당 stat이름을 ```_``` 마크를 통해 이어 붙이기 때문이라고...

그래서 실제 위 예제에서 AchStatName은 *'test'* 값이 넘어가고, 정상적으로 *'test_test'* stat의 값이 변경되는 것을 확인할 수 있다.

<br>
이렇게 업적 연동 테스트를 마쳤다면, 성공적으로 배포할 준비가 되었다!

---
### 5. Shipping 빌드용 App ID 파일 배치
모든 과정을 끝마치고 패키징할 단계가 되었다면, 마지막으로 shipping 빌드 시 유의할 점이 있다.<br>
바로 *steam_appid.txt*를 패키징된 실행파일이 있는 경로에 배치하는 것이다.

이를 해주지 않으면 해당 앱이 어떤 ID에 접근하는지 알 수 없기 때문에 정해진 이름인 *steam_appid.txt* 파일 내에 해당 Steam App ID를 만들어주면 된다. 이 때, 파일 내에는 어떠한 다른 텍스트도 없이 온전히 ID를 가리키는 숫자만 적혀 있어야 한다.<br>

ex) steam_appid.txt
{% highlight text %}
480
{% endhighlight %}

<br>
처음엔 이 앱의 위치라는 게 애매했다. exe 파일은 패키징된 폴더의 루트에도 있으니.<br>
결과적으로 *steam_appid.txt* 파일이 놓여질 경로의 예는 아래와 같다.

*YourPackagePath/YourProjectName/Binaries/Win64/*

위 경로에 shipping 빌드로 패키징된 exe 파일이 있을 것이고, 바로 이 경로에 *steam_appid.txt* 을 만들어주면 된다.

그리고 실제 Steam에 배포하고 다운을 받아도 정상적으로 잘 동작하는 것을 확인할 수 있다.


---
이렇게 UE4에서 Steam 업적을 연동하는 과정을 정리해봤다. 내 경우엔 좀 더 특별한 이슈도 있긴 했지만, Steam 업적 연동을 처음 시작했을 때 느꼈던 막막함에 비해 오히려 간단하게 해결할 수 있어서 다행이었다. 더 자세한 건 Steamworks와 UE4의 공식 문서를 참고하면 있으니, 다음에 또 기회가 되면 정리할 때가 오겠지.

다른 누군가에게도 도움이 되길 바라며, 이만 줄이겠다. :)

<br>
*참고자료*<br>
- https://docs.unrealengine.com/ko/Programming/Online/Steam/index.html
- https://allarsblog.com/2016/02/26/basicsteamintegration/
