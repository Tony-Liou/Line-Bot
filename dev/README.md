# Line Bot

本篇主要介紹藉由Line Developers中提供的Messaging API達成能夠收發訊息的Line Bot。
Bot的伺服器端則由Google App Script來擔此重任。

## 製作Line Bot

### 創立一個官方帳號

1. 註冊Line Developers帳號，使用你平常在用的一般Line帳號登入即可
2. 創建一個Provider(之後會顯示此帳號是由誰提供)
3. 於該Provider的Channels下創立一個Messaging API Channel
   ![Messaging API Channel](https://i.imgur.com/bpaGgt7.png)
5. 填寫完相關資料後就完成囉～

### 設定Channel

- Basic Settings
  - 確認有無**Channel Secret**，如無則按下**Issue**再發一個 (非必須，如需加強安全性才會使用此字串)
- Messaging API
  - **Webhook URL**欄位先留空，等完成GAS的時候才能產生URL
  - 如果之後想將此bot加入群組，則啟動**Allow bot to join group chats**
  - 建議可以關閉**Auto-reply messages**，避免傳送不必要的訊息
  - 檢查**Channel Access Token**，如無則按下**Issue**申請一個

如果想要更改此app的名稱或大頭貼等，需前往[Line Official Account Manager](https://manager.line.biz/)

# Google App Script

## Google App Script (以下簡稱GAS) 簡介

它為其他G Suite的軟體提供了可程式的(Programmable)擴充功能。像是Google的Doc, Sheet, Gmail, Analytics, Drive, Hangouts...等，皆能整合GAS。

重要的是，它也能製作簡單的Web Apps。

其編寫語言為Javascript，早期使用Mozilla提供的[Rhino JavaScript interpreter](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino)作為執行環境，目前為[V8](https://v8.dev/) Runtime，支援較多現代語法像是：`let`,`const`之類等。

## 製作Line bot server

我們需要能夠處理Line Platform傳送的訊息，因此我們要利用GAS建立一個Web App來接收、處理並發送訊息。
![Messaging API Architecture](https://developers.line.biz/assets/img/messaging-api-architecture.f40bffbb.png "architecture")

1. 首先進入雲端硬碟新增GAS專案 ![Create a Google App Script project](https://i.imgur.com/wTrmHXe.png)
2. 進入編輯頁面後，有兩個function是Web app需要實作的，一個是`doGet()`，另一個是`doPost()`。顧名思義就是實現HTTP的Request-method：使用者是利用**Get**或是**Post**方法來向我們傳送請求訊息。
   由於Line Platform只會向我們發送Post請求，因此我們可以只實作`doPost()` funciton。
3. 以下為自製的code，是利用push message的API傳送訊息到指定群組
   ```Javascript
   const CHANNEL_ACCESS_TOKEN = "Enter your channel access token here";
   const PUSH_API_URL = "https://api.line.me/v2/bot/message/push";
   const GROUP_ID = "Ccfc3c76624b68ec16994ed9e9da00d93";
   
   function doPost(e) {
       let jsonObj = JSON.parse(e.postData.contents);
       let userMsg = jsonObj.events[0].message.text;

       UrlFetchApp.fetch(PUSH_API_URL, {
           "headers": {
               "Content-Type": "application/json;",
               "Authorization": "Bearer " + CHANNEL_ACCESS_TOKEN,
           },
           "method": "post",
           "payload": JSON.stringify({
               "to": GROUP_ID,
               "messages": [{
                   type: "text",
                   text: msg
               }]
           }),
       });
   }
   ```
   `Channel Access Token`換成一開始設定Line帳號時取得的
   `Group ID`可以換成特定使用者的ID或是某個群組的ID
   想要簡單一點可以使用[reply message](https://developers.line.biz/en/reference/messaging-api/#send-reply-message)方法，修改JSON內容(`to`改成`replyToekn`)及API的URL(`push`改成`reply`)就行了
4. 如果以上都修改完成，那麼就可以發布程式了。按下上方**發布**按鈕，選擇**部署為網路應用程式**
5. 專案版本(Project version)選擇**新增**，誰能存取此應用程式(Who has access to the app)選擇**任何人，甚至是匿名的(Anyone, even anonymous)**，以誰的身分執行此應用程式(Execute the app as)建議選擇自己(me)，如果要選擇**存取此網路應用的使用者**不確定需不需要額外的設定。
6. 發佈後，會得到一串URL，將這網址放回上面說過的Line帳號設定的Webhook URL欄位按下更新，並開啟**Use Webhook**後就完成啦~
   可以先按下Verify試試看一切是否正常
7. 最後進入Line bot的聊天室，傳訊息給他，如果它也回傳一模一樣的文字就代表成功囉~

> [Google App Script參考文件](https://developers.google.com/apps-script/guides/web)
> [Line Messaging API參考文件](https://developers.line.biz/en/reference/messaging-api/#messages)
> 可參考以上文件進行修改

## 17 Live API

### Request

[Streamlink reference](https://github.com/streamlink/streamlink/blob/master/src/streamlink/plugins/app17.py)

- API URL: `https://api-dsa.17app.co/api/v1/liveStreams/getLiveStreamInfo`
- Contnet-Type: `application/json`
- JSON Payload:
   ```json
   {"liveStreamID": "7266278"}
   ```

Using HTTP POST method to send the request.

### Response

Checking the `status` code from response to get the streaming status.
The status code will be 2 if the streaming is live. *(Mostly)*
If not, it will be 0.

- Headers
  | Name              | Value                            |
  | ----------------- | -------------------------------- |
  | Content-Type      | application/json                 |
  | Date              | Mon, 30 Mar 2020 15:26:00 GMT    |
  | Trace-Id          | 90fcf71c3a2fb00d0afafc0b0670bf7e |
  | Transfer-Encoding | chunked                          |
  | Via               | 1.1 google                       |
  | Alt-Svc           | clear                            |
- Body
  ```json
  {
    "key": "",
    "data": "{\"userID\":\"d3e7118a-85cc-410b-9de4-279974df458f\",\"streamerType\":0,\"status\":0,\"caption\":\"\",\"restreamOpenID\":\"\",\"allowCallin\":0,\"closeBy\":\"normalEnd\",\"reason\":\"\",\"restreamerOpenID\":\"\",\"streamType\":\"\",\"liveStreamID\":7266278,\"streamID\":\"7266278\",\"endTime\":0,\"beginTime\":0,\"publishSec\":0,\"receivedLikeCount\":0,\"duration\":0,\"viewerCount\":0,\"totalViewTime\":0,\"liveViewerCount\":0,\"audioOnly\":0,\"locationName\":\"\",\"coverPhoto\":\"\",\"latitude\":0,\"longitude\":0,\"shareLocation\":0,\"followerOnlyChat\":0,\"chatAvailable\":0,\"replayCount\":0,\"replayAvailable\":0,\"numberOfChunks\":0,\"canSendGift\":0,\"userInfo\":{\"pushLiveStream\":1,\"userID\":\"d3e7118a-85cc-410b-9de4-279974df458f\",\"openID\":\"醬醬兒__\",\"displayName\":\"醬醬兒__\",\"name\":\"\",\"bio\":\"四月開播時間\\n週一～週四晚上8:00～10:00❤️\",\"picture\":\"B6F18965-56FF-4A94-8C03-972AA8F64F53.jpg\",\"website\":\"\",\"followerCount\":23,\"followingCount\":1,\"receivedLikeCount\":0,\"likeCount\":0,\"isFollowing\":0,\"isBlocked\":0,\"isAdmin\":0,\"isRemoved\":0,\"isVerified\":0,\"isFreezed\":0,\"isBanned\":0,\"unLockUser\":0,\"followTime\":0,\"blockTime\":0,\"followRequestTime\":0,\"roomID\":7266278,\"privacyMode\":\"open\",\"ballerLevel\":0,\"postCount\":0,\"lastLogin\":1583298820,\"coverPhoto\":\"\",\"age\":20,\"gender\":\"female\",\"pushLike\":\"\",\"pushPost\":0,\"pushFollow\":0,\"pushComment\":\"\",\"pushTag\":\"\",\"pushSystemNotif\":0,\"totalGiftRevenueEarned\":-1e-45,\"isOpenIDChangable\":false,\"deviceModel\":\"12.3.1 - Unknown iPhone\",\"isCelebrity\":0,\"isChoice\":0,\"isInternational\":0,\"adsOn\":0,\"subscribeExpireTime\":0,\"baller\":0,\"enterAnimation\":0,\"level\":3,\"giftModuleState\":1,\"experience\":190,\"version\":\"3.91.3\",\"deviceType\":\"IOS\",\"followPrivacyMode\":0,\"revenueShareIndicator\":\"\",\"clanStatus\":0,\"createClanID\":\"\",\"clanInfo\":{\"joinCount\":0},\"chatMuteDuration\":0,\"language\":\"TW\",\"livePass\":0,\"experienceToNext\":210,\"newbieThreshold\":30,\"newbieIapCheapPromotion\":0,\"region\":\"TW\",\"registerRegion\":\"TW\",\"registerTime\":1583298820,\"paypalVerifyState\":0,\"pkWinRate\":-1e-45,\"enterNotifState\":1,\"enterAnimationState\":1,\"hideAllPointToLeaderboard\":2,\"unreadTerm\":\"\",\"monthlyVIPBadges\":{},\"vipGroupType\":0,\"sportsCarAccumulatedPoint\":0,\"sportsCarThresholdTip\":0,\"lastActiveTime\":1585502180,\"maxStreamDuration\":0,\"lastLiveTimestamp\":0,\"mithHasAgreed\":false,\"mithEmailVerifyState\":0,\"mithSmsVerifyState\":0,\"mithServiceOpen\":false,\"invisibleInfo\":{\"enable\":false,\"startTime\":0,\"endTime\":0},\"hasCommodity\":false,\"stealthLeaderboardInfo\":{\"enable\":false,\"startTime\":0,\"endTime\":0},\"buyMarqueeCommentInfo\":{\"enable\":false,\"startTime\":0,\"endTime\":0},\"streamerRecapEnable\":true,\"followReminder\":1,\"gloryroadMode\":0,\"leagueInfo\":{\"shouldShowEntrance\":false},\"hasVipPurchase\":false,\"deviceID\":\"466b562c88f1490b825218fdae6759e2\",\"ageVerificationStatus\":0,\"ageVerificationLastUpdatedTimestamp\":0,\"referralCode\":\"\",\"isNewbieHintPopped\":1,\"newbieDisplayAllGiftTabsToast\":false,\"disableMakeLiveHotToast\":false},\"canSendGiftWithTextAndVoice\":0,\"videoCodec\":\"\",\"hiddenFromTimeline\":0,\"privateLiveStream\":0,\"landscape\":false,\"mute\":false,\"birthdayState\":0,\"dayBeforeBirthday\":0,\"hotLiveStatus\":0,\"achievementValue\":0,\"position\":0,\"topPosition\":0,\"mediaMessageReadState\":0,\"region\":\"\",\"specialTag\":0,\"hashtag\":\"\",\"internalInfo\":\"\",\"guardianUserID\":\"\",\"guardianPicture\":\"\",\"campaignIcon\":\"\",\"campaignURL\":\"\",\"campaignEndTime\":0,\"campaignShowTimer\":0,\"campaignSize\":0,\"campaignTitle\":\"\",\"filterMode\":0,\"revenueUserID\":\"\",\"commodityState\":0,\"commodityInfo\":{\"type\":0,\"price\":0,\"amount\":0,\"desc\":\"\",\"endTimeMS\":0},\"canSellCommodity\":false,\"gridStyle\":0,\"device\":\"\",\"redEnvelopeAvailable\":false,\"armyConfigInfo\":{\"enable\":false},\"deviceInfo\":{\"type\":\"\",\"version\":\"\",\"hardware\":\"\",\"OSVersion\":\"\",\"Customization\":\"\",\"publicIP\":\"\",\"packageName\":\"\",\"isViaMobile\":false,\"app\":\"\",\"deviceID\":\"\",\"ipRegion\":\"\"},\"vipFrameURL\":\"\",\"iosFrameURL\":\"\",\"vipFrameID\":\"\",\"subtabDisplayName\":\"\",\"sportsCarAccumulatedPoint\":0,\"sportsCarThresholdTip\":0,\"pmInfo\":{\"enable\":false,\"pmThreshold\":0,\"pmHours\":0,\"totalPoint\":0,\"pmStatus\":0},\"superstarAvailable\":false,\"messageProvider\":0,\"cdnProvider\":0,\"purchaseEventStickerList\":[],\"debugLevel\":0,\"entryTitleType\":0,\"entryIconURL\":\"\",\"verifiedStatus\":0,\"landscapeBarrage\":false,\"tradeID\":\"\",\"defaultGiftTab\":\"\",\"receivedLikeLayout\":0,\"cellTab\":0,\"pkID\":\"\"}"
  }
  ```