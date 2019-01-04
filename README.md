# Push-Notification-Study

## Chapter 1: Introduction
Push Notification 이 사용자에게 전달되는 여부는 보장되지 않으므로, 사용자가 Push Notification을 못받을 수도 있다고 생각해야한다.

## Chapter 2: Push Notifications
Notification은 특정조건에 따라 로컬기반으로 동작할 수 도 있고, 원격서비스가 디바이스로 보낼수도 있다.

#### What are they good for?
유저 경험에 대해 항상생각해야된다.

#### Remote notifications
Remote notification은 가장 흔한 타입으로, 서버나 다른서비스에 의해 디바이스로 보내지는 형태
애플은 TransportLayerSecurity(TLS)를 이용해 Apple Push Notification Service(APNs)을 제공하고, 앱 notification을 이용해서 APN을 사용해야한다.

#### Security
iOS 디바이스들은 APN 인증을 통해 Device token을 받고, Provider에게 알려주어, notification을 받을 수 있게 된다.
사용자가 다른 디바이스에 앱을 설치 할 수도 있기 때문에 Device token은 캐싱하면 안된다.
Device token은 서버입장에서 특정 유저를 식별하는 주소인 셈.

#### Notification message flow
Device에서 APN에 토큰 요청하고 받은 뒤, Provider에게 보내고, Provider가 DeviceToken을 APN에 보내 APN에서 노해당 디바이스에 Notification을 보낸다.  

#### Local notifications
Local notification은 remote notification과 달리 디바이스에 의해 생성된다.

#### Location aware
위치서비스와 연동해서 특정지역에 있을때 notification을 보내는 등의 처리가 가능하다.

## Chapter3: Remote Notification Payload
Notification payload는 JSON으로 되어있고, 최대크기는 4KB이다.
몇몇 키는 필수이나 대부분은 옵셔널이고, 이 챕터에서는 predefined key에 대해 다룰예정이다.

#### The app dictionary key
Aps key는 notification payload의 중요 키이며, 보여질 메시지, 뱃지 수, 노티 도착시 소리, 유저 인터랙션여부에 따른 알림 발생여부, 액션에 따른트리거여부 등을 설정 할 수 있다.

#### Alert
alert키는 사용자에게 보여질 메시지를 정의한다.
alert키 하위의 title, body를 이용해서 메시지를 정의 할수 있지만, localization고려를 위해 title-loc-key(+title-loc-args), loc-key(+loc-args)를 이용한다.
iOS12 부터 alert하위의 thread_identifer 키를 이용해서 같은 identifer값을 갖는 노티피케이션들을 그룹으로 묶을수 있다.

#### Badge
Aps 하위 badge키를 통해 앱아이콘에 표현될 뱃지 수를 지정한다.
이때 badge number는 수식이 아닌 absolute Value이다
Badge 제거하고싶으면 0으로 설정

#### Sound
Aps 하위 sound키를 통해 알림의 소리를 설정 할 수 있다.
커스텀 사운드를 설정하는 것은 사용자들이 싫어할수도 있으니 신중해라..
Critical alert sounds를 따로 지정할 수도 있다.

#### Other predefined keys
이외에 3개의 키 가 더있는데, 백그라운드 알림 업데이트, 커스텀타입, 그룹핑 할때 사용되고, 이후 챕터에서 자세하게 다룰예정이다.

#### Your custom data
Aps 키 밖의 것들은 개인목적으로 커스텀하게 사용가능하다.

#### Collapsing notifications
서버에서 APN으로 보낼때 키값이 apns-collapse-id 헤더필드를 보내는데, 해당 키값이 같다면 이전에 전달받았던 notification은 삭제된다.

## Chapter 4: Xcode Project Setup
Push Notification에 대한 Custom Profile설정을 하지 않았어도, Xcode - Target - Capability의 기능을 ON시키면 자동으로 추가된다.

#### Adding Capabilities
Xcode - Target - Capability에서 PushNotification 기능을 ON 시킨다.

#### Registering for notifications
앱이 시작될때 푸시 권한체크를 해주어야한다.
권한체크를 위해 UNUserNotificationCenter.current().requestAuthorization 메서드를 이용한다.
이때 notification closure는 메인쓰레드에서 동작하지 않으므로 디바이스 등록 메서드는 메인쓰레드에서 실행되도록 DispatchQueue를 이용한다.
RemoteNotification등록을 위해 UIApplication.registerForRemoteNotifications() 를 이용한다.

#### Provisional authorization
UNAuthorizationOptions enum의 .provisional 옵션을 사용하면 Notification이 Permission 없이 사운드와 얼럿을 일으키지 않고 사용자의 알림센터에 전달된다.

#### Critical alerts
Critical alerts는 .criticalAlert 옵션을 사용해서 생성할 수 있고, Critical alerts는 방해금지/유저의 알림 거부여부와 상관없이 alert sound를 재생한다.
사용하기 위해서는 애플로부터 승인을 받아야한다.

#### Getting the device token
디바이스토큰은 APNs에 등록된 앱 디바이스를 식별하는 Globally Unique한 ID이다.
디바이스 등록이 완료되면 
```application(_:didRegisterForRemoteNotificationsWithDeviceToken)```
메서드가 호출되며 디바이스토큰을 얻을수 있는데, Data타입이므로 ```토큰.reduce(“”) { $0 + String(format: “%02x”, $1) }```를 이용해서 String으로 변환해준다.
디바이스 토큰의 길이는 애플 정책에 의해 바뀔수 있기 떄문에 하드코딩해서 사용하지 말아야한다.
디바이스 토큰은 앱을 새로 설치하거나, 백업 복구 등에 의해 바뀔수 있는 점을 기억해야한다.

## Chapter5: Apple Push Notification Servers
Authentication Token은 애플서버에서 앱을 신뢰하기 위해 사용된다. Authentication Token을 통해 서버 유효성을 체크하고, 서버와 APNs 간 secure connection을 보장한다.

#### Token types
예전에는 확장자가 .p12인 PKCS #12(PFX) 포맷을 썼는데, 1년마다 갱신해야되고, development와 distributions 인증서가 분리되어 있으며 앱별로 분리 되어있어야 하는 불편함이 있었다.
2016년 .p8 확장자를 사용하는 JSON Web Tokens(JWT)가 나오며 p12의 불편함을 모두 개선했다.

#### Getting your Authentication Token
Authentication Token 생성을 위해 Member Center의 Keys 섹션의 APNs Key Services타입의 키를 생성한다.
키를 생성하고 다운받으면 파일명이 AuthKey_KeyID.p8 인 파일이 다운로드 되고, KeyID 자리(_와 .사이)에 있는 String이 KeyID이다.

#### Sending a push
푸시를 보낼때 Authentication Token, Device Token, BundleID가 필요하며, 이를 이용한 실습(굳)

## Chapter6: Server Side Pushes

#### SETTING UP A SQL SERVER
PostgreSQL 이용해서 실습할 예정이므로 설치하고 설정해준다.

#### SETTING UP VAPOR
웹커넥션을 처리하기 위해 Vapor를 사용할 건데, Vapor는 Swift를 이용해 서버사이드개발 하는것을 도와준다.
설치후 vapor xcode -y 명령어를 실행하면 프로젝트가 생성되고 확인해보면 Dependencies폴더에 서버사이드 개발을 위해 필요한 라이브러리들이 들어있다.

#### CREATING THE MODEL
Vapor를 통해 생성된 라이브러리를 이용해 DB에 저장될 토큰 모델을 만든다

#### CREATING THE CONTROLLER
토큰저장/삭제 API REST형태로 구현

#### SENDING PUSHES
Apple은 앱에서 푸시를 보내는것을 허가하지 않는다.

#### BUT THEY DISABLED PUSH!
푸시 기능을 꺼도, 애플에서 보내는 푸시는 여전히 유효하며, 디바이스에서 푸시를 무시해버리는 방식이다.
그러므로 디바이스가 푸시를 ON/OFF했는지 여부에 따라 토큰을 관리하지 마라

## Chapter 7: Expanding the Application

#### UPDATING THE SERVER
ATS때문에 Apple이 URL을 블록하므로 Info.plist에 App Transport Security Settings - Allow Arbitrary Loads 키의 값을 YES로 설정해야한다.

#### EXTENDING APPDELEGATE
AppDelegate Extension으로 푸시관련 로직을 빼서 푸시 노티피케이션 사용할때 사용할 수있도록 정리하면 편하다.

## Chapter8: Handling Common Scenarios

foreground에서도 notification을 처리하려면 UNUserNotificationCenterDelegate의 userNotificationCenter(_:willPresent:withCOmpletionHandler:) 메서드를 구현해야한다.
completionHandler에 표시할 파라미터(UNNotificationPresentationOptions타입)를 array형태로 넘기면 Notification이 백그라운드 상태와 같이 표시된다.
UNNotificationRequest가 APNS에서 보낸정보
notification.request.content.userInfo에 payload담김

#### Tapping the notification
좋은 notification은 인터렉션없이 사용자가 한눈에 필요한정보를 알아볼수 있도록 해야한다.
메서드를 빠져나가기 전 completionHandler를 수행해주어야하는데, defer를 이용하면 편하다.
defer { completionHandler() }
defer의 좋은 유스케이스다.

UNNotificationDefaultActionIdentifier
UNNotificationDismissActionIdentifier

#### Handle user interaction
기본적으로 notification을 탭했을때 앱이 실행되고 초기화면으로 이동한다.
payload의 정보를 이용해 userNotificationCenter(_:didReceive:withCompletionHandler:) 메서드에서 화면이동에 대한 처리를 해주면 특정화면으로 이동시킬 수 있다.

#### Slient notifications
알림과, 소리없이 전달되는 notification이 silent notification

#### Updating the payload
Aps 하위의 content-available 키의 값을 1로 설정하면 silent notification으로 전달됨
Alert, sound, badge필드와 함께 할경우 silent notification 적용안됨

#### Adding background modes capability
앱타겟 - Capabilities 탭의 Background Modes를 켜고 Remote notifications 옵션을 활성화한다.

#### App delegate updates
UIApplicationDelegate의 application(_:,didReceiveRemoteNotification:,fetchCompletionHandler) 메서드에서 필요한 처리를 구현해준다.

#### Method routing
Foreground여부, silent push 여부에 따라 action을 수행하는 델리게이트 메서드들 정리해놓음 페이지 저장해두고 참고해야겠다

##### Foreground
userNotificationCenter(_:willPresent:withCompletionHandler:)
##### User Action
userNotificationCenter(_:didReceive:withCompletionHandler:)
##### Silent Push
application(_:didReceiveRemoteNotification:fetchCompletionHandler) 

## Chapter 9: Custom Actions

#### Categories
Notification categories는 특정한 커스텀 액션(actionSheet와 유사한 것)을 사용할수있게함
application(_:didRegisterForRemoteNotificationWithDeviceToken:) 메서드 안에서 카테고리 액션을 등록한다.
UNUserNotificationCenter의 setNotificationCategories를 통해 카테고리를 등록( alert, actionsheet와 유사함)
카테고리는 UNNotificationCategory, 액션은 UNNotificationAction으로 생성
카테고리생성시 categoryIdentifier는 페이로드 category 키의 값과 일치해야함
액션이 눌렸을때 처리는 다른 액션처리와 마찬가지로userNotificationCenter(_:didReceive:withCompletionHandler:)에서 함

#### Extending Foundation’s notification
UNNotificationAction에서 받은 Accept, Reject 처리를 로컬 노티피케이션으로 발행시켜 옵저빙하는형태로 처리함.
로컬 노티피케이션 발행은 NotificationCenter의 post()메서드, 옵저버 추가및 처리는 addObserver()를 이용한다.

#### Responding to the action

## Chapter 10: Modifying the payload. 

#### Configuring Xcode for a service extension
Notification Service Extension을 타겟에서 추가하면 NotificationService.swift 파일이 생기고 didReceive(_:withContentHandler:), serviceExtensionTimeWillExpire 메서드 템플릿이 제공된다.
페이로드를 전부 수정할 수 있지만 alert text는 수정할 수 없다.

#### Decrypting the payload
Push Notification i페이로드는 ROT13(13번째 다음 알파벳 사용)을 이용해서 암호화된다.

#### Downloading a video
페이로드에 있는 URL을 로컬파일이 저장될 위치를 설정하고, 다운로드 받은 후 UNNotificationAttachment를 생성해서 Contents.attachments에 담으면 푸시 내용에 표현된다.
Async 다운로드 사용은 불가능하다.

#### Service extension payloads
서비스 익스텐션을 사용하기위해 aps하위의 mutable-content 키를 1로 설정해야한다. 해당 키가 없을 경우 익스텐션호출X

#### Sharing data with main target
일반적으로 앱타겟과 익스텐션은 분리된 다른 프로세스여서 데이터를 공유할 수 없는데, App Groups Capabilities를 켜게 되면 프로세스간 데이터 접근이 가능해진다.

#### Badging the app icon
서비스익스텐션을 이용해 뱃지를 로컬에 저장하는 형태로 관리하는 것이 좋은 유스케이스라고 한다.
뱃지를 로컬에서 관리해야하는지는 의문이 든다. 서비스 상황에 따라 로컬저장형태가 필요하다면 참고해서 처리해야겠다

#### Accessing Core Data

#### Localization
페이로드 수정할때 로컬라이징 신경쓰기

#### Debugging
Debug - Attach to Process by PID or Name 으로 다른타겟을 추가해서 디버깅 할 수 있다.

## Chapter11: Custom Interfaces

#### Configuring Xcode for custom UI
커스텀 UI 구현을 위해 Notification Content Extension 타겟을 추가해야한다.
커스텀 UI는 Unique한 category identifier를 가져야한다.
NSExtension - NSExtensionAttributes - UNNotificationExtensionCategory 의 value를 해당 커스텀 UI category identifier와 일치시켜주면 익스텐션 타겟이 동작한다.

#### Designing the interface
타겟을 추가하면 NotificationViewController와 함께 스토리보드가 생성되는데, 해당 스토리보드의 뷰가 사용자에게 노출된다.

#### Resizing the initial view
Notification Content Extension의 Info.plist - NSExtension - NSExtensionAttributes - UNNotificationExtensionInitialContentSizeRatio 값을 설정(<=1)해서 표현되는 가로세로 비율을 설정할 수 있다.(세로/가로)

#### Accepting text input
UNTextInputNotificationAction 을 카테고리에 추가해서 텍스트입력을 받을 수 있다.
텍스트입력에 대한 처리는 컨텐츠익스텐션의 didReceive(_:completionHandler:)에서 response를 UNTextInputNotificationResponse로 캐스팅 한 뒤 userText 속성에서 뽑을 수 있다.
didReceive(_:)는 Notification이 표시될때 호출되고, didReceive(_:completionHandler:)는 action에 의해 response를 받았을때 호출된다.

#### Changing action
extensionContext의 notificationActions 프로퍼티를 이용해서 로직을 구현해주면 Notification에 노출될 Action을 동적으로 처리할 수있다.

#### Attachments
ServiceExtension에서 content에 추가한 attachments를
didReceive(_:) 메서드에서 notification.request.content.attachments로 접근해서 커스텀UI에 사용할 수 있다.

#### Video attachments
ContentExtension의 optional 프로퍼티인 mediaPlayPauseButtonType: UNNotificationContentExtensionMediaPlayPauseButtonType 를 구현하고, frame을 설정하면 시스템에서 play, pause메서드를 수행하게 해준다.(play, pause 메서드에서 비디오 플레이어 관련 처리를 해야하는듯)

#### Custom user input
커스텀 인풋을 사용 할경우 키보드를 사용할 수 없다.

#### Adding a payment action
#### The first responder
canBecomeFristResponder true로 해주어야함.
BecomeFirstResponder를 didReceive(response:)에 구현해주면 inputView를 찾아 띄운다.
#### The user input
ContentExtension 에서 inputView 프로퍼티를 오버라이드해서 커스텀 뷰를 통해 커스텀인풋을 받도록 처리한다.

#### Hiding default content
ContentExtension의 Info.plist - NSExtension - NSExtensionAttributes - UNNotificationExtensionDefaultContentHidden 키의 값을 통해 노출되는 title과 body 를 숨김처리 할 수 있다.

#### Interactive UI
ContentExtension의 Info.plist - NSExtension - NSExtensionAttributes - UNNotificationExtensionUserInteractionEnabled 
키의 값을 YES로 설정해주면 탭했을때 앱 실행과 같은 시스템에서 제공하는 기능을 끄고, 개발자가 구현한대로 동작하게 하므로 해당옵션을 킬때는 액션처리에 대한 구현을 꼭 해주어야한다.

#### Debugging

## Chapter12: Putting It All Together

#### Setting up the Xcode project
#### AppDelegate code
application(:didFinishLaunchingWithOptions:) 메서드에서 디바이스토큰 등록 관련 메서드 호출.
application(:didRegisterRemoteNotificationsWithDeviceToken)에서 커스텀 액션 등록

#### Requesting calendar permissions
캘린더 정보를 가져오기위해 EventKit을 이용해 접근권한 요청

#### The payload
#### Validate calendar permissions
#### App badging
서비스 익스텐션의 didReceive(request:…)에서 노티피케이션 컨텐츠와 로컬저장용 뱃지 카운트 처리를 담당
applicationDidBecomeActive 메서드에서 로컬에 들고 있는 뱃지 카운트 초기화시킨다.

#### Notification body
#### Updating the Info.plist
이전장에서 설정했던 것처럼 컨텐츠 익스텐션의 info.plist에 categoryIdentifier 설정하고, 알림 기본 타이틀/바디를 감추기 위해 UNNotificationExtensionDefaultContentHidden 키값을 추가한다.

#### Adding information to CalendarKit
#### Getting nearby calendar items
컨텐츠 익스텐션 didReceive(notification:) 에서 UI에 값을 설정함

#### Accepting and declining
 컨텐츠 익스텐션 didReceive(response:…)에서 유저액션을 처리

#### Commenting on the invitation
#### Final cleanups

## Chapter 13. Local Notifications

#### You still need permission!
로컬 노티피케이션도 UNUserNotificationCenter 의 requestAuthorization을 통해 권한을 받아야 함.
Remote와 다른점은 registerForRemoteNotifications 
메서드를 호출하지 않아도 된다.

#### Objects versus payloads
로컬 노티피케이션 전달을 위해 클래스(UNNotificationContent)를 사용한다.

#### Creating a trigger
UNCalendarNotificationTrigger - 특정 일시가 되었을때 동작
UNTimeIntervalNotificationTrigger - 특정시간 이후 동작
UNLocationNotificationTrigger - 특정 위치좌표의 정의한 반경안에 들어오거나 나갈때 동작

#### Defining content
UNMutableNotificationContent 를 이용해 Notification Contents 내용을 생성해서, 트리거가 작동했을때 payload를 정의할수 있다.
트리거에 의해 로컬라이징 설정했을때, 설정한 시점기준이 아닌 Notification이 실제로 보여질때 기준으로 처리하기 위해 localizedUserNotificationString (forKey:arguments:) 메서드를 사용해라.

#### Scheduling
UNNotificationRequest를 생성하고 UNUserNotificationCenter 에 add(request:completionHandler:) 메서드를 이용해서 로컬 노티피케이션을 등록한다.

#### Foreground notifications
Remote Notification과 동일하게 UNUserNotificationCenterDelegate의 userNotificationCenter(:willPresent:..) 메서드를 구현해서 completionHandler에 사용할 NotificationOption(.badge 등)을 넘긴다.
 
#### The sample Platter

#### Configuring the main UITableView
UNUserNotificationCenter의 getPendingNotificationRequests(completionHandler:), getDeliveredNotifications(completionHandler:) 메서드를 이용해 발송대기중인 Notification과 유저에게 전달된 Notification 리스트를 얻을수 있고, 해당 Request의 Identifier를 이용해서 remove메서드도 지원한다.

#### Scheduling

#### Time Interval Notifications
UNTimeIntervalNotificationTrigger 인스턴스를 이용해서 특정 시간 뒤에 Notification을 보낼 수 있다.

#### Location Notifications
UNLocationNotificationTrigger 인스턴스를 이용해서 해당 지역에 진입 또는 떠날때 Notification을 보낼 수 있다.
파라미터 타입인 CLCircularRegion은 센터좌표와 반경을 이용해 생성함.

#### Calendar Notifications
UNCalendarNotificationTrigger 인스턴스를 이용해서 특정일시에 Notification을 보낼 수 있다.
