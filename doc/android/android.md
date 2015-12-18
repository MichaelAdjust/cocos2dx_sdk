---
layout: page
title: adjust Android Cocos2dx SDK
permalink: /ja/sdk/cocos2dx/android/
locale: ja
type: guide
description: "こちらは、adjust™のCocos2d-x用SDKです。"
---

こちらは、adjust™のCocos2d-x用SDKです。adjust™についての詳細は
adjust.comをご覧ください。

<section id='toc-section'></section>

### Cocos2d-x Androidプロジェクトへの基本的な統合

adjust のSDK をCocos2d-xのAndroidプロジェクトに統合するための手順を説明します。

#### SDKの入手

最新バージョンのSDKを[リリースページ][releases]からダウンロードし、アーカイブを任意のダイレクトリーに
解凍してください。

#### adjustソースファイルのプロジェクトへの追加

`Adjust`フォルダのファイルを保存し、Android プロジェクトに追加してください。

![][add_android_files]

#### adjust C++ソースファイルの`Android.mk`への追加

`Android.mk`ファイルの`LOCAL_SRC_FILES`セクションに、adjust C++ファイルのパスを必ず追加してください。

![][add_to_android_mk]

#### adjustライブラリのプロジェクトへの追加

`adjust-android.jar`ライブラリを保存し、プロジェクトの`libs`フォルダにコピーしてください。

![][add_android_jar]

#### Google Play開発者サービスの追加

2014年8月1日よりGoogle Playストアのアプリには、デバイスを一意に識別するための
[Google 広告 ID][google_ad_id] の使用が義務付けられました。
adjust SDKでGoogle 広告 IDを使用するには、[Google
Play開発者サービス][google_play_services]を統合する必要があります。統合済みでない場合は、
以下の手順に沿って設定して下さい。

1. 以下のパスのライブラリプロジェクトを、
	
	<pre>
	&lt;android-sdk&gt;/extras/google/google_play_services/libproject/google-play-services_lib/
	</pre>

	Androidアプリのプロジェクトの保存場所にコピーしてください。

2. Eclipseワークスペースにライブラリプロジェクトをインポートします。`File >
	Import`, select `Android > Existing Android Code into Workspace`の順にクリックし、
	コピーしたライブラリプロジェクトを指定してインポートしてください。

3. アプリのプロジェクトで、Google Play開発者サービスのライブラリプロジェクトを参照してください。詳しい設定方法は、[Eclipseでのライブラリプロジェクトの参照について][eclipse_library] をご覧ください。
	
	参照先には、開発中のワークスペースへコピーしたライブラリを指定し、Android SDKダイレクトリー内のライブラリを直接参照しないようにしてください。
     

4. アプリのプロジェクトが依存するライブラリとしてGoogle Play開発者サービス ライブラリを追加したら、
アプリのマニフェスト ファイルを開き、[<application>][application] エレメントの子要素として次のタグを追加してください。

	<pre>
	&lt;meta-data android:name="com.google.android.gms.version"
	      android:value="@integer/google_play_services_version" /&gt;
	</pre>

#### パーミッションの追加

Package ExplorerでAndroidプロジェクトの`AndroidManifest.xml`を開いて下さい。
`INTERNET`の`uses-permission` タグが記述されていない場合は、追加してください。

<pre>
&lt;uses-permission android:name="android.permission.INTERNET" /&gt;
</pre>

Google Playストアでの配信を想定して*いない*場合は、代わりに以下のパーミッションを両方追加してください。

<pre>
&lt;uses-permission android:name="android.permission.INTERNET" /&gt;
&lt;uses-permission android:name="android.permission.ACCESS_WIFI_STATE" /&gt;
</pre>

![][manifest_permissions]

#### ブロードキャスト レシーバの追加

`AndroidManifest.xml`ファイルの`application`タグの中に、以下の`receiver`タグを追加してください。

<pre>
&lt;receiver
    android:name="com.adjust.sdk.AdjustReferrerReceiver"
    android:exported="true" &gt;
    &lt;intent-filter&gt;
        &lt;action android:name="com.android.vending.INSTALL_REFERRER" /&gt;
    &lt;/intent-filter&gt;
&lt;/receiver>
</pre>

![][receiver]

adjustはインストール リファラを取得するためにこのブロードキャスト レシーバを使用して
コンバージョン トラッキングを改善しています。

`INSTALL_REFERRER`インテントに対して、既に他のブロードキャスト レシーバを使用している場合は、
[こちらの説明][referrer] の通りに
adjustのレシーバを追加してください。

#### アプリへのAdjustの統合

はじめに、基本的なセッション トラッキングの設定を行います。

##### 基本設定

パッケージエクスプローラーで、アプリのデレゲートを開きます。ファイルの先頭にインポートステートメントを追加し、アプリのデレゲートに下記のadjustコールを`applicationDidFinishLaunching`に追加してください。   

```cpp
#include "Adjust/Adjust2dx.h"
// ...
std::string appToken = "{YourAppToken}";
std::string environment = AdjustEnvironmentSandbox2dx;

AdjustConfig2dx adjustConfig = AdjustConfig2dx(appToken, environment);

Adjust2dx::start(adjustConfig);
```

![][add_adjust2dx]

`{YourAppToken}`とうい文字列はアプリのトークンと置き換えて下さい。トークンは
[ダッシュボード]で確認することができます。

アプリのビルドがテスト版か製品版かにより、
`environment`に以下のいずれかの値を適宜設定してください。

<pre>
std::string environment = AdjustEnvironmentSandbox2dx;
std::string environment = AdjustEnvironmentProduction2dx;
</pre>

**重要:** アプリがテスト版の場合のみ、値を
`AdjustEnvironmentSandbox2dx`に設定し、
アプリの公開前に必ずこの値を`AdjustEnvironmentProduction2dx`に変更してください。
再度開発やテストを行う際は、設定を`AdjustEnvironmentSandbox2dx`
に戻してください。

adjust ではこの設定により、トラフィックが実際のものなのか、テスト機から生じたものなのかを判別しています。
この値を常に正しく設定することは非常に大切で、
売上のトラッキングを行う際には特に重要となります。

##### Adjustのログ設定

`AdjustConfig`インスタンスで、以下のパラメータのいずれかを設定して`setLogLevel`関数を呼び出せば、
テスト時に表示されるログの量を
増減させることができます。

<pre>
config.setLogLevel(LogLevel.VERBOSE);   // enable all logging
config.setLogLevel(LogLevel.DEBUG);     // enable more logging
config.setLogLevel(LogLevel.INFO);      // the default
config.setLogLevel(LogLevel.WARN);      // disable info logging
config.setLogLevel(LogLevel.ERROR);     // disable warnings as well
config.setLogLevel(LogLevel.ASSERT);    // disable errors as well
</pre>

#### セッション計測

セッションの トラッキングを適切に行うには、アクティビティが再開、または一時停止する度に必ず
特定のadjustメソッドを呼び出す必要があります。
そうしないと、SDKはセッションの開始とセッションの終了を取りこぼすことがあります。これを防ぐため、以下の手順で設定を行って下さい。
**各アクティビティ**ごとに、各のステップを従ってください。

1. アクティビティのソースファイルを開いてください。
2. `applicationWillEnterForeground`のメソッドにある`onResume`のメソッドにコールを追加します。
3. `applicationDidEnterBackground`のメソッドにある`onPause`のメソッドにコールを追加します。

以上の手順で設定したら、アクティビティのソースは以下のようになっているはずです。

<pre>
#include "Adjust/Adjust2dx.h"
// ...

void AppDelegate::applicationDidEnterBackground() {
    // ...

#if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
    Adjust2dx::onPause();
#endif
}

void AppDelegate::applicationWillEnterForeground() {
    // ...

#if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
    Adjust2dx::onResume();
#endif
}

// ...
</pre>

![][activity]

#### アプリのビルド

Androidアプリをビルドし、実行します。LogCatビューアで
`tag:Adjust`フィルタをセットすれば、他のログをすべて非表示にできます。アプリが起動すると、
`Install tracked`というadjust のログを確認できるようになります。

![][log_message]

### 追加機能

プロジェクトにadjust のSDKを統合すると、以下の
機能を利用できるようになります。

#### カスタムイベントのトラッキングの追加

adjustを使用してイベントをトラッキングすることができます。[ダッシュボード]で新しいイベントトークンを作成し、
そこに`abc123`のようなイベントトークンを関連付けてください。
そしてトラッキングを行いたいイベントに、次の文字列を追加してください。

<pre>
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");
Adjust2dx::trackEvent(adjustEvent);
</pre>

このイベントインスタンスを使って事前に設定を行えば、更に詳細なトラッキングを行うことも可能です。

#### 売上のトラッキングの追加

広告のタップや
アプリ内購入で売上が発生する場合は、売上情報をイベントと一緒にトラッキングすることができます。
例えば、タップ1回で0.01ユーロの収入が発生する場合、次のように設定して売上イベントをトラッキングすることができます。

<pre>
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");
adjustEvent.setRevenue(0.01, "EUR");
Adjust2dx::trackEvent(adjustEvent);
</pre>

もちろん、このコードはコールバック パラメータと組み合わせることもできます。

通貨トークンを設定した場合、adjustは売上高を自動的に為替計算し、任意の通貨で表示します。 
詳細は[為替レートの計算][currency-conversion]ページをご覧ください。

売上とイベントのトラッキングの詳細は、[イベントトラッキングについて][event-tracking]をご覧ください。


##### コールバックパラメータの追加

[ダッシュボード]でイベント追跡用のコールバックURLを設定することが可能で、
イベントがトラッキングされる度に、そのURLへGETリクエストが送信されます。
トラッキング前に、イベントインスタンスで`addCallbackParameter`　を呼び出して、そのイベント用のコールバックパラメーターを設定することができます。
その後、設定されたパラメーターがコールバックURLに付け足されます。


例えば、
`http://www.adjust.com/callback`をコールバックURLとして登録し、イベントを次のようにトラッキングする場合、

<pre>
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");
adjustEvent.addCallbackParameter("key", "value");
adjustEvent.addCallbackParameter("foo", "bar");
Adjust2dx::trackEvent(adjustEvent);
</pre>

イベントがトラッキングされると、次のURLにリクエストが送信されます。

<pre>
http://www.adjust.com/callback?key=value&foo=bar
</pre>

注目すべき機能として、adjustでは`{android_id}`といった、パラメータ値として利用できる様々なプレースホルダに対応しています。
実際に生成されるコールバックでは、この
プレースホルダはデバイスのAndroidIDで置き換えられることになります。
また、adjust ではカスタムパラメータはコールバックに付加されるだけで、
一切保存されませんのでご注意下さい。イベントにコールバックを登録していなければ、
これらのパラメータは読み込まれることすらありません。

URLコールバックの使い方については、
[コールバックについて][callbacks-guide]のページで、使用可能な値を網羅したリストと一緒に、詳しく紹介しています。

##### パートナー パラメータ

ダッシュボードでトラッキングデータを共有するよう設定したネットワーク パートナーに対し、送信されるパラメータを追加することもできます。


これは、上記のコールバック パラメータと同様の働きを持つもので、
`AdjustEvent2dx`インスタンスで`addPartnerParameter`メソッドを呼び出せば追加することができます。

<pre>
AdjustEvent2dx adjustEvent = AdjustEvent2dx("abc123");
adjustEvent.addPartnerParameter("foo", "bar");
Adjust2dx::trackEvent(adjustEvent);
</pre>

スペシャルパートナーとその統合については、
[スペシャルパートナーについて][special-partners]のページをご覧ください。

#### ディープリンク　リアトリビューションの設定

adjust のSDKをディープリンクに対応するよう設定して、カスタムURLからアプリが開かれた際にトラッキングを行うことができます。
adjustでは、adjust専用のパラメータだけを読み込みます。
ディープリンクを使ったリターゲッティングやリエンゲージメントキャンペーンを計画している場合、この設定は必須となります。

ディープリンクに対応しているアクティビティごとに、`onCreate`メソッドまたは`onNewIntent`メソッドに以下の通りadjustへの呼び出しを設定します。

##### `Standard`起動モードの場合

<pre>
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Intent intent = getIntent();
    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    // ...
}
</pre>

##### `singleTop`起動モードの場合

<pre>
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);

    Uri data = intent.getData();
    Adjust.appWillOpenUrl(data);
    // ...
}
</pre>

詳細は[ディープリンクについて][deeplinking]の説明を参照してください。

#### イベントのバッファリングの有効化

アプリでイベント トラッキングが頻繁に発生する場合、
一部のHTTPリクエストを遅らせておいて、1分ごとに一括送信した方がよいことがあります。
その場合は、`AdjustConfig`インスタンスでイベント バッファリングを有効にしてください。

<pre>
config.setEventBufferingEnabled(true);
</pre>

#### アトリビューションの変化を通知するリスナの設定

トラッカーがアトリビューションの変化を検出した際に通知を受け取れるよう、リスナを登録することができます。
アトリビューションのソースを比較しなければいけないため、
変化の発生時に通知を行うことは出来ません。最も簡単な方法で設定するには、単一の匿名リスナを作成してください。


また、adjustの[アトリビューションデータの適切な利用に関する
方針][attribution-data]を必ず参照してください。

SDKを開始する前に、`AdjustConfig`インスタンスで以下の通り匿名リスナを追加してください。



1. `AppDelegate.cpp`を開き、下記のデレゲートコールバック昨日をアプリデレゲートに追加します。

<pre>
cpp
void attributionCallbackMethod(AdjustAttribution2dx attribution) {
}
</pre>

2. `AdjustConfig2dx`のインスタンスにてデレゲートを設定：

<pre>
cpp
adjustConfig.setAttributionCallback(attributionCallbackMethod);
</pre>
    
デレゲートコールバックが`AdjustConfig2dx`のインスタンスで設定しているため、`Adjust2dx::start(adjustConfig)`を呼び出す前に、`setAttributionCallback`を呼び出してください。

このリスナ関数は、SDKが最後のアトリビューション情報を取得した際に呼び出されます。
呼び出されたリスナ関数内には`attribution`パラメータが含まれており、アトリビューションの種類を確認することができます。
以下に、このパラメータのプロパティを簡単にまとめました。

- `String trackerToken` 発生したインストールのトラッカー トークン。
- `String trackerName` 発生したインストールのトラッカー名。
- `String network` 発生したインストールのネットワーク階層のグループ名。
- `String campaign` 発生したインストールのキャンペーン階層のグループ名。
- `String adgroup` 発生したインストールの広告グループ階層のグループ名。
- `String creative` 発生したインストールのクリエイティブ階層のグループ名。
- `String clickLabel` 発生したインストールのクリック ラベル。

#### トラッキングの無効化

adjustSDKが行っているデバイスのアクティビティのトラッキングをすべて無効化することができます。
そのためには、`false`パラメータをセットして`setEnabled`関数を呼び出します。この設定はセッションが進んでも
維持されます。

### オフラインモード

一旦adjustサーバまでの送信を停止スルことができます。すると、すべての数値がファイルで保存されますので、たくさんのイベントを発生しないようにご注意ください。

`true`というパラメータで`setOfflineMode`を呼び出すことでオフラインモードを利用できます。

```objc
Adjust2dx::setOfflineMode(true);
```

また、`false`パラメータで`setOfflineMode`を呼び出すとオンラインになります。すると、すべてのデータが正しいタイムスタンプと一緒に弊社のサーバへ送信されます。

トラッキングの無効化と違って、こちらの設定は起動旅にリセットしますので、オフラインモードの状態でアプリを終了しても、再起動するとオンラインモードとしてアプリが起動します。

<pre>
Adjust.setEnabled(false);
</pre>

`isEnabled`関数を呼び出せば、adjust のSDKが現在有効かどうかを確認できます。
また、`true`パラメータをセットして`setEnabled`関数を呼び出せば、いつでもadjust のSDKを
有効化することができます。

[dashboard]:     http://adjust.com
[releases]:      https://github.com/adjust/cocos2dx_sdk/releases
[add_android_files]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/add_android_files.png
[add_android_jar]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/add_android_jar.png
[add_to_android_mk]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/add_to_android_mk.png
[manifest_permissions]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/manifest_permissions.png
[receiver]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/receiver.png
[application_class]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/application_class.pngmanifest_application]: 
[manifest_application]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/manifest_application.png
[application_config]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/application_config.png
[activity]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/activity.png
[log_message]: https://raw.github.com/adjust/sdks/master/Resources/cocos2dx/android/log_message.png

[referrer]:             https://github.com/adjust/android_sdk/blob/master/doc/referrer.md
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[google_play_services]: http://developer.android.com/google/play-services/setup.html
[android_application]:  http://developer.android.com/reference/android/app/Application.html
[application_name]:http://developer.android.com/guide/topics/manifest/application-element.htmlnm
[google_ad_id]:         https://developer.android.com/google/play-services/id.html
[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[special-partners]:     https://docs.adjust.com/en/special-partners
[multidex]:             https://developer.android.com/tools/building/multidex.html
[maven]:                http://maven.org
[example]:              https://github.com/adjust/android_sdk/tree/master/Adjust/example
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/tracking-purchases-in-different-currencies
[deeplinking]: https://docs.adjust.com/en/tracker-generation/deeplinking
[eclipse_library]:	http://developer.android.com/tools/projects/projects-eclipse.html#ReferencingLibraryProject
