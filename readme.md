
# 必要插件:
```
dependencies:
  flutter:
    sdk: flutter
    
  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  http: ^1.0.0
  flutter_secure_storage: ^8.0.0
  ...
  firebase_auth: ^4.6.2
  google_sign_in: ^6.1.4

```
- [ ] http 用於前後端分離
- [ ] secure_storage 是用於storage 但更安全
- [ ] firebase_auth 只借用firebase 驗證，windows 需安裝 全局 firebase
- [ ] google_sign_in


推薦插件:
```
dependencies:
  flutter:
    sdk: flutter
    
  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  ...
  flutter_signin_button: ^2.0.0
```

# Sign in With Google 沒有同意畫面 GSA 無法打印任何東西(#需要設定一個SHA key)
```
      GoogleSignInAccount? _currentUser = await _googleSignIn.signIn();
      GoogleSignInAuthentication? gSA = await _currentUser?.authentication;
      print(gSA);

```
## SHA generate(#keytool)
win 11 將java 的環境變量位置另設，需要找尋java 的安裝位置，才能運用java 的keytool。

親身試過 兩個java環境變量不會有衝突(#檔案位置截然不同)

1. 先將keytool 加進環境變量(C:\Program Files\Java\jre-{$yourJavaVersion}\bin)
2. cmd 測試keytool
3. 尋找金鑰:```keytool -list -v -keystore C:\Users\Ryan-Engineer\.android\debug.keystore -storepass android -keypass android```
4. ``不需要自己產生金鑰，android在安裝的時候已經將debug.keystore安裝在 該有的地點了 keypass 和 keystore 都是 android``
5. ``這裡需要注意的是，此金鑰為預設金鑰是專門做debug使用，未來上線安全起見，還會再給一組金鑰。``
6. 就可得到 憑證指紋，也就是尚未加密過的原始資料，將其copy 並放到 firebase上
### 此處注意 假設win 11是中文版 可能會有使用者與user，建議SHA key兩個檔案夾都找找 才找得到正確的金鑰儲存位置。

Flutter Google Sign In
來源來自:https://blog.lifetaiwan.net/2020/03/flutter-google-sign-in.html
2020-03-18
#Flutter 
整理一下，用 Flutter 想做 google 帳號登入，但是並沒有要使用 firebase 來做使用者的帳號管理，單純只想實做 google 帳號登入

網路看到很多介紹，但是很多都是過時的資訊，或是不夠完整

自己整理一下筆記，也給也有需要的人做參考

基本上，需要 firebase 的專案，及 gcp console 的設定權限，這裡的例子是以 flutter 的 android 的範例做說明，實際 iOS 的說明，可以參照 firebase 整合登入的文件，以下步驟，沒有需要按照順序

步驟1
建立 Flutter 專案，目前新版的 flutter 工作，新建立的專案，預設會採用 支援 androidx 的方式建立專案，但是後續要在 firebase console 中設定對應的 package name 時，要對應 app 的 package name ，所以這裡用 com.yoursite 當例子

flutter create –org com.yoursite gaccount

接著，設定安裝 flutter 需求套件

firebase_auth 及 google_sign_in

請於 Flutter 專案下的 pubspec.yaml 檔案裡，新增到 dependencies: 區段下，用命令列的朋友，用 flutter pub get 指令安裝，用 vscode 的 朋友加入後，會自動安裝

dependencies:
  flutter:
    sdk: flutter
  firebase_auth: ^0.15.5+2
  google_sign_in: ^4.1.4
  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
步驟2
設定 android 及 ios 的目錄檔案

以 android 做說明 更新 android/build.gradle

buildscript {
    ext.kotlin_version = '1.3.50'
    repositories {
        google()
        jcenter()
    }

    dependencies {
        .... 略
        classpath 'com.google.gms:google-services:4.3.3'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

rootProject.buildDir = '../build'
subprojects {
    project.buildDir = "${rootProject.buildDir}/${project.name}"
}
subprojects {
    project.evaluationDependsOn(':app')
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
更新 android/app/build.gradle

dependencies {
    .... 略
    implementation 'com.google.firebase:firebase-analytics:17.2.2'
}
apply plugin: 'com.google.gms.google-services'
放置 android/app/google-services.json 的認證檔案，我們下個步驟會取得

步驟 3
設定您的 firebase 專案，點選 Authentication 的 Sign-in method ，裡面有 Google 選項，請啟用



到專案總覽裡的 Settings 一般設定的位置，填寫，必填的公開資訊，新增應用程式，可以選擇平台，套件名稱記得要填對，以這個例子就是

com.yoursit.gaccount



重點來了， SHA 憑證指紋，我們一般會有兩組，一個由是正式 release 的 keystore 得到，一個則是預設安裝 android studio 時， 建立的在家目錄隱藏資料夾中的檔案，我的平台是 Mac 他放在 ~/.android/debug.keystore 可以用指令去取得，如果您有設定不同的 keystore alias 及密碼，請自己組指令，也可以參考 firebase 頁面上的說明，點擊問號的圖示，有文件連結可以參考

keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android


這裡設定好了後，可以下載，google-services.json 檔案，放在上一步驟說明的位置

步驟 4
設定 gcp console 服務說明資訊 https://console.developers.google.com/apis/credentials/consent

必須對應您的 firebas 專案，提供服務的聯絡 email 及授權的網域，這裡要加上您，firebase 對應的專案主機，一般 會自動加好，到這裡，設定的部分，結束了 像我們只有用認證 google 帳號的登入方式，帳號資訊要另外存 自己後端資料庫的方式，記得也準備一把，給後端 server 用的 API key ，可以用來驗證， google 帳號的 token 正確性， 完整的串完整個流程喔



步驟 5
終於開始寫程式了，Flutter 的部分，大概是這樣，下面範例 (直接貼不會動，我還有自己狀態管理，存 token 的程式)， 就是點擊 google 帳號登入後要做的事情，基本上就是拿到 accessToken 後，就丟給後端去驗證 accessToken 的正確性，後端驗證沒問題後，就可以直接用 google 的 api 去抓取 登入使用者的個人資訊了，這裡處理完帳號後，就是自己後端發自己的 token

// 前面不要忘了 import 需要的 library
... 略
import "package:http/http.dart" as http;
import 'package:flutter/material.dart';
import 'package:google_sign_in/google_sign_in.dart';

// 這個是自己後端要驗證 token 的網址
var _callBackUrl = API_BASE + "/auth/google?access_token=";

// 要求的 google token scope 
GoogleSignIn _googleSignIn = GoogleSignIn(
  scopes: <String>[
    'email',
    'profile',
  ],
);
    
 // 這段是在 login widget 裡面，點擊 google 登入按鈕後做的事情
  Future _handleSignIn() async {
    try {
      GoogleSignInAccount _currentUser = await _googleSignIn.signIn();
      GoogleSignInAuthentication gSA = await _currentUser.authentication;

      var getURL = _callBackUrl + gSA.accessToken;
      final http.Response response = await http.get(getURL);

      if (response.statusCode != 200) {
        print('People API ${response.statusCode} response: ${response.body}');
      }

      final Map<String, dynamic> data = json.decode(response.body);
      final String token = data['token'];
      print('Token: ' + token);
      state.setToken(token);
      Navigator.of(_scaffoldKey.currentContext)
          .pushReplacementNamed(HomeScreen.routeName);
    } catch (error) {
      print('ERROR: ${error}');
      _scaffoldKey.currentState.showSnackBar(SnackBar(
        content: Text("登入失敗，請檢查您的網路連線" ?? '您尚未登入 !!!'),
      ));
    }

  }
所需要的函式庫

https://pub.dev/packages/firebase_auth

https://pub.dev/packages/google_sign_in

參考資料

https://blog.codemagic.io/firebase-authentication-google-sign-in-using-flutter/


