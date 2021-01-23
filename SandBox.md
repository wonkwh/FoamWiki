안녕하세요 어제 인사드렸던 개발팀 iOS 개발자 원광현입니다.  
다름이 아니라 점심식사하면서 아이폰에서`QR코드`를 뒷탭두번으로 하는걸 보여드렸는데 아주 유용한듯 하여 혹시 아이폰 유저중 안쓰시는분들게  공유드립니다. 
https://github.com/jeyraof/covid-19-qr-ios14-back-tap
위 링크 가보시면 캡쳐와 함께 자세히 나와 있어 쉽게 따라하시면 됩니다.  
정리하면 
```
Shortcuts 준비

1. Shortcuts 앱을 실행합니다.

2. + 버튼을 눌러 새 Shortcuts 등록을 시작합니다.

3. \+ Add Action 버튼을 누릅니다.

4. Apps 를 선택합니다.

5. Safari 를 선택합니다.

6. Open URLs 를 선택합니다.

7. URL 부분에 주소를 입력합니다.

– 내가 만약 네이버 체크인을 사용한다면:

◦ [nid.naver.com/login/privacyQR](https://nid.naver.com/login/privacyQR)

– 내가 만약 카카오 체크인을 사용한다면:

◦ [kakaotalk://con/web?url=https://accounts.kakao.com/qr\_check\_in](kakaotalk://con/web?url=https://accounts.kakao.com/qr_check_in)

1. Covid-19 QR Shortcut 이 등록된 것을 확인하고 종료합니다.

  

Back tap 등록

1. Settings 앱을 실행합니다.

2. Accessibility 를 선택합니다.

3. Touch 를 선택합니다.

4. Back Tap 을 선택합니다.

5. 선호에 따라 Double Tap 혹은 Triple Tap 을 선택합니다.

6. 목록의 하단에서 위에서 생성한 Covid-19 QR 을 찾아 선택합니다.

7. 선택한 메뉴에 Covid-19 QR 이 설정되었는지 확인합니다.

  

사용

아무곳에서나 단말기의 뒷부분을 천천히 더블탭 혹은 트리플탭 하면 실행됩니다.
```
앞으로 가끔 아이폰, 맥 유용한 팁들 있으면 공유드리겠습니다. ^^  그럼.. 

- 기존 `open`, `inProgress`, `Done` 으로 되어 있던 board의 statuses를 `Open`, `InProgress`, `Finish`, `QA/DQA`, `Deliver` , `ReadyForRelease` 로 6개로 변경하였습니다. 
	- open, inprogress 는 기존과 같습니다. 
	- Finish는 개발자가 이슈를 수정하여 repository에 develop branch에 merge한 상태
	- Deliver는 TestFlight등으로 내부배포가 완료된 상태 
	- QA/DQA : 이슈를 만든 사람이 QA를 실시하여 완료되고 Design 검증이 필요하면 DQA 를 진행하는 상태 , 여기서 이슈가 수정이 완료되면 ReadyForRelease로 변경하고 이슈가 수정되지 않았거나 디자인이 변경되지 않았으면 reject하여 open상태로 만듭니다. 
	- ReadyForRelease는 AppStore에 릴리즈할 사항들


```
kwanghyun won
Research & Development Division
iOS Product Director
```



----
