# ABAPerになろう～SAP CALを利用してAS ABAP Developer Edition環境構築～

**この記事は [SAP Advent Calendar 2020](https://adventar.org/calendars/5047) の12月14日分の記事として執筆しています**

アドベントカレンダーな記事だけあり、もう少しするとクリスマスです。
クリスマスには全世界のSAP開発者・運用者がニヤリ？　とするイースターエッグがあったりするので
興味がある方は下記の記事を参照してクリスマスを楽しみましょう。

[SAP QUITE_MERRY（イースターエッグ：メリークリスマス）](https://github.com/senkousya/SAP_QUITE_MERRY)

閑話休題

本記事ではABAPerになろうと題して、ABAP開発環境の構築について説明します。
具体的には`SAP CAL`を用いて`AWS`に`AS ABAP Developer Edition`をデプロイしてさわってみます。

## はじめに

SAPで利用される開発言語 ABAP。

エンタープライズど真ん中のSAPで利用されるプログラミング言語だけあり？　他のプログラミング言語に比べて試してみたいと思いついたとしても。

開発実行環境を構築するハードルは高い方かと思います。

本記事では、なんかSAPってのがあってその開発言語がABAPっていうものらしいぞ！　ちょっとさわってみたい！

そんな方を対象にお試しでさわれる実行環境の構築について説明します。
（AWSにデプロイするので少しばかりのお金はかかりますが）

## SAPから提供されている環境について

ABAPをトライアルとしてさわれる環境としては、下記がSAPから提供されているかと思います。

1. SAP NetWeaver AS ABAP Developer Edition
2. SAP Cloud Platform ABAP Environment Trial

それぞれ下記のようになっています。

1. `SAP NetWeaver AS ABAP Developer Edition`はSAPが提供しているトライアル用の製品で、環境構築することにより開発者用のABAP環境が利用できるようになります。
2. SAP Cloud Platform ABAP EnvironmentはSAPがPaaSで提供しているABAP実行基盤で、旧来のABAPerからするとクラウドでABAP！？　みたいな割と衝撃的な環境。　ただし旧来のABAPとはそれはそれで異なる。

今回は`SAP NetWeaver AS ABAP Developer Edition`を`SAP Cloud Appliance Library (CAL)`と呼ばれる仕組みを使い、AWSへデプロイしてみます。

`SAP Cloud Appliance Library (CAL)`は登録されているSAPの様々な製品を所有するパブリッククラウドアカウント（AWS,Azure、GCP）に対して手軽にデプロイできるサービスです。

そこにはもちろん`SAP NetWeaver AS ABAP Developer Edition`も登録されているのでこちらをAWSアカウントへデプロイします。

なお`CAL`を利用しないで、自前で用意した端末にインストールする場合は下記を参照して実施します。

- [AS ABAP 752 SP04, developer edition: NOW AVAILABLE](https://blogs.sap.com/2019/07/01/as-abap-752-sp04-developer-edition-to-download/)
- [AS ABAP 7.52 SP04, Developer Edition: Concise Installation Guide](https://blogs.sap.com/2019/10/01/as-abap-7.52-sp04-developer-edition-concise-installation-guide/)

SAPをさわったことない人がいきなりSAP製品のインストールをするのはハードルがあるような気もするので、まずは`SAP CAL`を利用してみましょう。

## 今回の作業に必要なもの

- P-User（このユーザについての説明と取得方法については後述します）
- Developer editionをデプロイするAWSアカウント
- 対象AWSアカウントを操作できるアクセスキーとシークレットキー

本記事では対象にしていませんが、`Azure`や`Google Cloud Platform`も`SAP CAL`の接続先アカウント設定の所を読み変えるだけでデプロイできるのでAWS以外にデプロイする人は適宜読み替えて下さい。

## SAP Community Userの用意

SAPの世界ではS-Userといえば誰にでも通じるようなユーザIDがあります。
このS-UserでSAPのオンラインリソースにアクセスするのですが、S-UserはSAPを導入した企業やSAPパートナー企業などの組織が発行するユーザIDとなり、いずれかの組織に属さないと発行されません。

SAPを利用している企業、SAPパートナー企業以外でS-Userは利用できないので、完全個人でSAPのオンラインリソースにアクセスは無理なのか……といえば、そんなことはなくて。

P-User（Public SAP users）が用意されています。

OpenSAPのページにちょっとだけこのIDの種類について書いてあったりします。

[Welcome SAP Employees, Partners and Customers](https://open.sap.com/pages/ecosystem)

- D-User: Germany based SAP Employees
- I-User: International based SAP Employees
- C-User: SAP Contractors
- S-User: Licensed Customers or SAP Partners
- P-User: Public SAP users, for example SCN users

今回利用する`SAP CAL`についてはP-Userがあれば利用できるため取得します。

[SAP Community](https://community.sap.com/)　にアクセスして`Sign-up`から必要事項を記入してIDを発行します。

Login/Sign-upを選択  
![](image/sapcommunity.png)

登録を選択  
![](image/register.png)

必要事項を入力  
![](image/register-info.png)

登録完了  
![](image/complete.png)

メールアドレスに確認用のメールが届くのでリンクをクリック  
![](image/notification.png)

アカウント有効化が完了しました  
![](image/validcomplete.png)

続行すると、ポリシーをどうするか聞かれたので適宜選択  
![](image/policy.png)

## SAP CALにログインする

下記URLから`SAP CAL`にアクセスを行い、取得したP-Userでログインして下さい。

[SAP Cloud Appliance Library](https://cal.sap.com/)

初回ログイン時に追加情報を求められたので入力。

適宜入力して`保存して続行`を選択  
![](image/add-info.png)

![](image/term.png)

必要事項の確認が終わると、`CAL`が表示  
![](image/cal-toppage.png)

## Accountを設定する

まずは左メニューからAccountを設定します。

ここでは`SAP CAL`からデプロイ先クラウドプラットフォーム接続用の資格情報を設定します。

AWSへデプロイする場合は`アクセスキー`と`シークレットキー`が必要となるので、事前にAWS側で発行して下さい。

AWS側で発行するIAMに必要なポリシーは下記の記載を参照して下さい。

[How to configure your IAM user?](https://wiki.scn.sap.com/wiki/display/SAPCAL/FAQ+-+Specific+questions+for+Amazon+Web+Services#FAQSpecificquestionsforAmazonWebServices-HowtoconfigureyourIAMuser?)

`Crete Account`を選択  
![](image/create-account.png)

`任意の名前`と`Cloud ProviderはAmazon Web Services`を選択  
![](image/account-setting.png)

アクセスキーとシークレットキーの入力ボックスが出てくるので入力  
![](image/account-setting-aws.png)

アカウントユーザの設定画面が出てきますが、変更せずに`Review`を選択  
![](image/account-setting-step2.png)

確認画面が出てくるので確認してから`Create`を選択  
![](image/cal-account-create.png)

作成したアカウントのステータスがActiveになっている事を確認  
![](image/cal-account-info.png)

## Solutionからデプロイしてみる

Solutionページで`SAP CAL`からデプロイできる製品一覧が参照できます。

`Solutions`  
![](image/solution-top.png)

`traial`や`dev`と記載がありますが詳細は下記URLを参照して下さい。

[FAQ Solution Types and Charging Model](https://blogs.sap.com/2016/08/23/faq-solution-types-and-charging-model/)

2020年12月現在、Solutionの一覧を確認した所、`CAL`で用意されているDeveloper Editionとしては`SAP NetWeaver AS ABAP 7.51 SP02 on ASE`が最新のようですので、今回はこちらを利用します。

`SAP NetWeaver AS ABAP 7.51 SP02 on HANA`もありますが、今回は別にHANAをさわりたいわけではないのでASEを選択。

Developer Editionとしては`SAP NetWeaver AS ABAP Developer Edition 7.52 SP04`が現在最新なので、`CAL`ではこのバージョンが用意されていないのは少し残念。

Solutionから`SAP NetWeaver AS ABAP 7.51 SP02 on ASE`の`Create Instance`を選択  
![](image/751sp2onase.png)

各種情報が表示されます、AWS、Azure、Google Cloud Platformにデプロイする時の推奨インスタンス情報やそのコスト計算等が確認できます。

`Create Instance`を選択  
![](image/751sp2onase-info.png)

確認して`I Accept`を選択  
![](image/751sp2onase-term.png)

登録画面が出てきました。
`basic`モードで設定してもいいのですが、今回は`Advanced Mode`を選択

`Acvanced Mode`を選択  
![](image/crateinstance-basic.png)

さきほど作成したデプロイ先のAccountを入力  
![](image/advance-step1.png)

デプロイ先のネットワークを選択します。
今回はデフォルトのまま変更なしでRegionは`us-east-1`を選択します。

`Public Staic IP Adrees`を選択すると、AWSの場合は[Elastic IP Address](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)を取得してデプロイ先のインスタンスにアタッチします。
※EIPはアタッチ先のインスタンスが稼働していない時に料金が発生してしまうので注意。

またせっかくだったら東京リージョン（ap-northeast-1）を選択したい所ですが、現状は下記のようです。

[When will other AWS regions be supported?
](https://wiki.scn.sap.com/wiki/display/SAPCAL/FAQ+-+Specific+questions+for+Amazon+Web+Services#FAQSpecificquestionsforAmazonWebServices-WhenwillotherAWSregionsbesupported?)

適宜入力して`Step 3`を選択  
![](image/advance-step2.png)

ここでは対象となるインタンス、ストレージ、エンドポイント設定をできます。

`SAP Frontend`については必須ではないためチェックを外してデプロイ対象から除外する事もできますが。
今回は`SAP GUI`や`Eclipse`などのツール類が入っている環境が欲しいのでデプロイにチェック（デフォルトのまま）

またエンドポイントについてデフォルトだと

`SAP Frontend`サーバのRDP（3389）を`0.0.0.0/0`で全許可にしているのと、
SAP LinuxサーバのSSH（22）を`0.0.0.0/0`で全許可にしていたりするので。

ここらへんは状況に応じて適宜調整した方が望ましい感じです。　これはあとからCALの画面で変更もできます。

適宜入力して`Step 4`を選択  
![](image/advance-step3.png)

指定できるマスターパスワードには結構制限がありますが、条件に沿うマスターパスワードを入力します。

マスターパスワードを入力して`Setp 5`を選択  
![](image/advance-step4.png)

インスタンスの起動停止スケジュールを設定できますが、`Manual Activate and Suspend`を選択

この場合、起動停止は完全マニュアルなのでインスタンス停止忘れで課金されてしまうので注意して下さい。

適宜入力して`Review`を選択  
![](image/advance-step5.png)

入力内容を確認して、`Create`を選択  
![](image/advance-summary.png)

プライベートキーをCALに保存しておくかの選択と、プライベートキーのダウンロード画面が出てきます。
プライベートキーについては後でLinuxへ接続する際に必要となるのでダウンロードしておいて下さい。

適宜入力とpemファイルのダウンロードをして`Close`を選択  
![](image/private-key.png)

しばらく待つように表示されるので、`Close`を選択  
![](image/create-info.png)

Instancesのページでデプロイ状況のステータスが確認できるので、時間をおいてたまに確認します。

`prepared`  
![](image/status-prepared.png)

`activating`  
![](image/status-activating.png)

`active`になったのでデプロイ完了  
![](image/status-active.png)

## SAP Frontendに接続してみる

デプロイされたSAPシステムに接続するためには、Windows環境ならば`SAP GUI for Windows`と呼ばれるGUIソフトで接続を行うのですが。

今回利用しているP-Userでは該当の`SAP GUI for Windows`をダウンロードする事はできません。

今回、必須ではない`SAP Frontend`サーバもデプロイしましたが、これは`SAP Frontend`サーバに`SAP GUI`がインストールされているためです。

なのでまずはRDPで`SAP Frontend`サーバにログインします。

Instance画面から`connect`を選択  
![](image/status-active.png)

この画面では下記が取得できます

- スタートガイド
- RDPの設定ファイル
- SAPシステムにつなぐためのGUIであるSAP GUI用の設定ファイル

スタートガイドについては、デプロイした環境について用意されている各種ユーザや初期パスワードについて記載があるため。一度目を通す方がよいです。

RDPの`Connect`を選択してrdpファイルをダウンロード  
![](image/connect.png)

ダウンロードしたrdpファイルを実行すると接続先が指定された状態でリモートデスクトップ接続が起動するのでログインします。

ログインする資格情報については、スタートガイドを見れば書いてありますが下記でログインできます。

- `Administrator`
- デプロイ時に設定したマスターパスワード

資格情報を入力して`OK`を選択  
![](image/rdp-credential.png)

`SAP Frontend`サーバに接続  
![](image/rdp-frontend.png)

`SAP Frontend`サーバ（Windows Server 2012 R2）に接続できました。
確認してみると、`SAP GUI`だったり、`Eclipse`だったりSAPなツールが`SAP Frontend`サーバにインストールされている事が確認できます。

デスクトップに`SAP Logon`のアイコンが有るため、こちらをクリック。

`SAP Logonpad`が起動してくるので、接続したいSAPシステムを選択します。

今回はCALからデプロイした`SAP Frontend`サーバなので、予め接続先が設定されいるので、設定済みの`SAP Netweaver AS ABAP and SAP BW 7.51 on SAP ASE`をダブルクリック。

`SAP Netweaver AS ABAP and SAP BW 7.51 on SAP ASE`をダブルクリック  
![](image/sap-logonpad.png)

`SAPシステム`のログイン画面が出てきました。

`SAPシステム`に用意されているユーザについても、スタートガイドに記載があります。
ここではログイン確認として下記ユーザでログインしてみます。

- CLIENT001
- DEVELOPER
- デプロイ時に設定したマスターパスワード

ログイン画面  
![](image/sap-login.png)

ログインできました。ログイン確認はできたのでとりあえずこの画面は閉じておきます。

CLIENT001のDEVELOPERでログイン  
![](image/sap-easy-access.png)

## ライセンスの登録

スタートガイドにある`Licenses: Developer edition`の項目を実施します。
何をするかといえば、Deveoper用のラインセスを　[Minisap](http://www.sap.com/minisap)　で発行して、`SAPシステム`にインポートします。

さきほどと同様に`SAP Logon`から下記ユーザでSAPにログインします。

- CLIENT000
- SAP*
- デプロイ時に設定したマスターパスワード

CLIENT000のSAP*でログイン  
![](image/cli000-sapask-login.png)

SAPでは各機能画面へアクセスするには左ペインにあるメニューから行く方法もありますが。
左上の入力ボックス（コマンドフィールド）にトランザクションコードと呼ばれる、機能に紐づくコードを入力する事により画面遷移ができます。
（大文字小文字の考慮はなし）

ここではSAPライセンス管理のトランザクションコードである、`SLICENSE`を入力してライセンス画面を表示します。

コマンドフィールドに`slicense`を入力してENTER  
![](image/call-slicense.png)

`SLICENSE`の画面が開きました、`Active Hardware Key`は後で使うので控えておきます。

後で利用するので`Active Hardware Key`を控える  
![](image/slicense-temp.png)

[Minisap](http://www.sap.com/minisap)のサイトにアクセス

今回利用している、`NPL - SAP NetWeaver 7.x(Sybase ASE)`を選択。

`NPL - SAP NetWeaver 7.x(Sybase ASE)`を選択  
![](image/minisap-generate.png)

先程控えておいて`Active Hardware Key`や必須項目を入力して`Generate`を選択

必要事項を入力して`Generate`を選択  
![](image/minisap-generate-info.png)

`NPL.txt`がダウンロードできるのでこのファイルを`SAP GUI`を実行している`SAP Frontend`サーバーへ移動する。
今回はデスクトップに配置してみました。

`SAP Frontend`サーバのデスクトップに`NPL.txt`を配置  
![](image/npltxt-on-desktop.png)

開いていた`SLICENSE`の画面から既存のTempライセンスの行を選択してから`Edit -> Delete License`を選択
※行を選択してからでないと、`choose a lisence`とインフォメーションが出ます。

削除対象の行を選択して`Delete Lisense`を選択  
![](image/delete-temp-lisence.png)

`チェックボタン`を選択  
![](image/delete-temp-lisence-check.png)

Tempライセンスが削除できました。

Tempライセンスが削除された事を確認  
![](image/delete-temp-lisence3.png)

`Edit -> Install License`を選択  
![](image/install-lisence.png)

ダイアログが立ち上がるので、先程デスクトップに配置した`NPL.txt`を選択してOpenを選択  
![](image/upload-lisence.png)

`Allow`を選択  
![](image/upload-allow.png)

`チェックボタン`を選択  
![](image/install-success.png)

ライセンスタイプが`Perm`のライセンスがインストールされました。  
![](image/slicense-perm.png)

この`Perm`ライセンスもpermanentとか書いてますが、Develop版なのでValidToの日付を見て分かる通り3か月で切れます。

ですのでライセンス期限が近くなったら、同様の手順で[Minisap](http://www.sap.com/minisap)からライセンスを発行してインストールしてください。

## Hello Worldをしてみる

ここではABAPで基本となるHell Worldを実行してみます。

下記でSAPにログインしてください。

- CLIENT001
- DEVELOPER
- デプロイ時に設定したマスターパスワード

左上の入力ボックス（コマンドフィールド）に`SE38`と入力して`ABAP Editor`へ移動します。

`se38`と入力してエンター  
![](image/call-se38.png)

Programに`ZHELLOWORLD`と入力して`Create`を選択  
![](image/create-helloworld.png)

下記を入力して`SAVE`を選択

- Title `Hello World`
- Type `Executable program`
- Status `Test Program`

必要事項入力して`SAVE`を選択  
![](image/create-helloworld2.png)

`Local Object`を選択  
![](image/create-helloworld3.png)

プログラムが保存され、コーディング画面が表示されます。

`ABAP Editor Edit Mode`  
![](image/se38-editor.png)

下記のように入力します。

```ABAP
REPORT ZHELLOWORLD.

WRITE 'Hello World'.
```

![](image/se38-helloworld.png)

保存ボタンで変更を保存します。

`Save`を選択  
![](image/se38-helloworld-save.png)

チェックボタンで構文チェックを実施します。

`Check`を選択  
![](image/se38-helloworld-check.png)

有効化します

`Activate`を選択  
![](image/se38-helloworld-activate.png)

実行を選択します。

`Direct Processing`を選択  
![](image/se38-helloworld-execute.png)

Hello Worldと表示されました

`Hello World`  
![](image/write-helloworld.png)

ABAPの世界へようこそ！

## コマンドフィールドでの画面移動について

コマンドフィールドにフォーカスを当てた状態で、`F1`キーを押すと。
コマンドフィールドの使い方ヘルプが表示されるのでこちらを参照して下さい。

![](image/command-field-help.png)

SAPにログインした最初の`SAP Easy Access`から移動する際は、トランザクションコードのみで移動できましたが。
それ以外の画面ではヘルプのように`/n`や`/o`をトランザクションコードの先頭につけて移動します。

違いはヘルプを参照。

## インストールされているソフトウェアコンポーネントを確認してみる

SAPはERPパッケージなので様々な業務トランザクションを処理する機能を提供します。
機能ごとにコンポーネントと呼ばれる単位で分かれており、必要な物を選択してSAPシステムへインストールする事ができます。

`AS ABAP Developer Edition`では下記のようなSAPシステムのベースとなるコンポーネントのみがインストールされており。
業務系の機能を持つコンポーネントはインストールされていません。

なので、この環境はABAPを書いて色々と遊べますが。業務系の画面だったり、それらのテーブルだったりはインストールされていません。
そこらへんさわってを遊んでみたい場合は、`SAP IDES`とかを利用する必要がありますが本記事では対象外。

インストールされているソフトウェアコンポーネントは下記から確認できます。

`System -> Status -> SAP System dataの虫眼鏡ボタン（Component）`

`System -> Status`を選択  
![](image/status.png)

`虫眼鏡ボタン（Component）`を選択  
![](image/status-component.png)

`Installed Softwre`が表示  
![](image/component.png)

`SAP_APPL`だったり、業務なコンポーネントがインストールされていないので本当に最小限という感じですね。

## ABAP keyword Documentation

トランザクションコード `abapdocu` で`ABAP keyword Documentation`を参照する事ができます。

`ABAP keyword Documentation`はABAPの概要や構文やサンプル等が記載されているドキュメントで。

ここにはABAPの知識が詰まっています。

なので、ここを読み込めば非常によい勉強になります。

## ABAP Examples

`SAP_BASIS`には各種ABAPのサンプルやデモが含まれており。
`BC*`や`DEMO*`はじまりでプログラムが登録されています。

また`SE38`画面の`Environment -> Examples -> ABAP Examples`

で別ウィンドウで`ABAP Examples`が表示されるので色々とサンプルが確認できます。
（さきほど開いたABAP keyword Documentationの一部分ですが）
)

`ABAP Examples`  
![](image/abap-examples.png)

`2048 Game`の項目を参照してみる。`Execute`でプログラムの実行  
![](image/2048-game.png)

実行するとポップアップが出てくるので`Enter`を選択  
![](image/2048-game-input.png)

`2048_Game!`  
![](image/2048-game-play.png)

## ABAP Development Tools in Eclipseで接続してみる

`SAP GUI`でSAPシステムに接続してABAP開発する以外にも。
`ABAP Development Tools（ADT） in Eclipse`というToolを利用してABAPをコーディングする事ができます。

`SAP Frontend`サーバにはこのツールもすでにインストール済みで用意されているためこちらを利用してSAPシステムに接続してみます。

デスクトップの`SAP Dev Tools for Eclipse`を起動  

![](image/execute-eclipse.png)

Eclipseが起動すると、すでにSAPの接続先が設定されている状態で起動してきます。
本来だったらeclipseのインストール後に、ADTインストールや接続設定をする必要がありますが、`SAP CAL`がここらへん全部調整してくれて、ユーザはクリックするだけです。

今回接続したい、`NPL_001_developer_en`をダブルクリックする  
![](image/login-npl001-developer-on-eclipse.png)

パスワードを入力して`OK`を選択  
![](image/login-npl001-developer-on-eclipt-input-password.png)

接続できました。

![](image/eclipse-login-success.png)

先程、`ZHELLOWORLD`を作成する時に、`Local Object`を選択しました。
パケージやら`Local Object`、`＄TMP`みたいな話はすべて飛ばしてしまいますが。
下記の場所に先程作成した`ZHELLOWORLD`があります。

`Favorite Packages -> $TMP -> Source Code Library -> Programs -> ZHELLOWORLD`

`ZHELLOWORLD`を表示  
![](image/eclipse-view-zhelloworld.png)

表示したプログラム`ZHELLOWORLD`を実行してみます。

`Run`を選択  
![](image/eclipse-execute-zhelloworld.png)

実行結果が表示されます。

`Hello World`と表示  
![](image/eclipse-execute-zhelloworld2.png)

プログラムを変更してみる。

`ZHELLOWORLD`を変更  
![](image/eclipse-change-zhelloworld.png)

`ZHELLOWRLD`を有効化  
![](image/eclipse-active-zhelloworld.png)

変更したプログラムを実行してみる。

`Run`を選択  
![](image/eclipse-execute-zhelloworld3.png)

変更が反映された実行結果を確認できます。

実行結果が表示  
![](image/eclipse-execute-zhelloworld4.png)

このように`SAP GUI`だけではなくEclipseで`ADT`をつかって開発が出来ます。

## SAP Serverにログイン

ここでは`SAP NetWeaver AS ABAP 7.51 SP02 on ASE`がインストールされているLinuxサーバにログインしてみます。

`SAP CAL`でデプロイしたInstanceのINFOにIPアドレスが記載されているので確認します。

`Linux External IP Address`を確認  
![](image/Linux-external-ip.png)

デプロイ時に取得してpemファイルを利用してsshで接続します。

ちなみに接続ユーザはスタートガイドに記載があります。

```PowerShell
ssh root@<<ipアドレス>> -i <<pemファイルのパス>>
```

`ssh`で接続  
![](image/ssh-connect-linux.png)

sshで接続できました、接続できない場合は`Access Points`の設定が適切かどうかや、そもそもインスタンスが起動しているか等々確認して下さい。

`SAP Frontend`は`Windows Server 2012 R2`でしたが、SAPサーバは`SUSE Linux Enterprise Server 12 SP3 x86_64 (64-bit)`ですね。

## SAPインスタンスを停止

`SAP CAL`から起動停止をかけると、OS起動時に自動でSAPインスタンスも起動されるようですが。
OSにログインしてマニュアルで起動停止する手順をここでは説明します。
（スタートガイドの手順にもあります）

```sh
su - npladm
sapcontrol -nr 00 -function GetProcessList
```

まずは`root`ユーザからsidadmユーザ`npladm`にスイッチして`sapcontrol`コマンドを実行して現在動いているプロセスの確認をします。
なお`00`はInstance Numberを指定しています。

Instance NumberはSAPのインスタンスを識別するための一意な番号になりますが、スタートガイドにこの開発環境はInstanceNoが`00`でインストールされていることが記載されています。

`GetProcessList`を実行  
![](image/get-process-list.png)

`disp+work`プロセスやらなにそれみたいな感じですが、GREENでRunningな事はわかります。

スタートガイドでは`stopsap`コマンドと`startsap`コマンドを利用してSAPを停止起動していますが。今現在、これらのコマンドは非推奨なので代わりにここでは`sapcontrol`を停止してみます。

```sh
sapcontrol -nr 00 -function StopSystem
```

`StopSystem`で止めた後に、`GetProcessList`で止まったことを確認  
![](image/stop-system.png)

画像だと、`GetProcessList`をインスタンスが落ちる前に実行して一度目は`Running`と表示されていますが。
2回目は少し待ってから実行したので`Stopped`になっている事を確認できます。

またSAPが停止しているので、`SAP Frontend`から`SAP GUI`で接続しようとしてもエラーになることが確認できます。

`SAP GUI`で接続エラーになることを確認  
![](image/connection-refused.png)

## SAPインスタンスを起動

```sh
sapcontrol -nr 00 -function StartSystem
```

今回はしばらくまってから`GetProcessList`を実行したので一度目で`Running`の状態になっていました。

![](image/start-system.png)

`sapcontrol`コマンドには起動停止時にwaitをするような機能も用意されているので、そちらを使って起動停止をまってみるのもよいでしょう。

`sapcontrol --help`でヘルプを読めるのでヘルプ参照。

## SAP CALからインスタンスの停止

利用が終わったら必ずインスタンスの停止をします。
停止しないと起動中はずっとAWSのEC2利用料が発生してしまいます。

`SAP CAL`のInstancesからsuspendを選択します。

`suspend`を選択  
![](image/sap-cal-suspend.png)

`OK`を選択  
![](image/suspend-narouabaper.png)

ステータスの所から、各インスタンスが停止している事が確認できます。

ステータスが`Suspended`になっている事を確認  
![](image/status-suspended.png)

## SAP CALからインスタンスの起動

利用する際は`SAP CAL`からインスタンスを起動します。

`Activate`を選択  
![](image/sap-cal-activate.png)

`OK`を選択  
![](image/activate-narouabaper.png)

しばらく待つとステータスが`Active`になります。

なお今回はデプロイ時に`Public Staic IP Adrees`のチェックをいれなかったので、`External IP`が固定されず都度AWSから払い出されて変更になります。

## ABAPについて

ABAPは1983年に誕生したそこそこ歴史が長い言語なので、もともとはCOBOLに由来するような所から出発して。
時代時代で拡張されて、オブジェクト指向（ABAP Object）だったり、自動テスト（ABAP Unit）な機能が拡張されたりと、今現在は言語としてモダンな書き方も出来たりします。

なので時代とコーディングする人によっておなじABAP言語のはずなのですが、書き方にだいぶ差が出てきます。
（標準機能でも古い機能だと古い書き方ですし、新しいOOPな書き方で作られている標準機能もあります）

最近は[styleguides - Clean ABAP](https://github.com/SAP/styleguides)と呼ばれるモダンなコーディングが推奨されているようですが。

現場現場では昔ながらの書き方で規約が統一されていたり色々です。

個人的には、その現場の規約に沿いつつ`ABAP Keyword Document`の`Obsolete Language Elements`にある現在廃止された命令（互換性の問題があるので利用はできる）については、極力避けるよう調整していければいいのかな？　と思います。

`Obsolete Language Elements`  
![](image/obsolete--language-elements.png)

## AWS利用料について

2020年12月現在、`CAL`の推奨設定で`SAP NetWeaver AS ABAP 7.51 SP02 on ASE`をデプロイすると`us-east-1`に下記のリソースがデプロイされました。

SAP

- ec2 us-east-1 Linux r4.xlarge（$0.266 per Hour）
- ebs us-east-1 magnetic 190GiB(0.05 USD/GiB)

SAP Frontend

- ec2 us-east-1 Windows m4.large（$0.192 per Hour）
- ebs us-east-1 magnetic 50GiB(0.05 USD/GiB)

両方合わせると

- EBS利用料としては月額 $12
- EC2利用料としては1時間 $0.458

になるかと思います。

一ヶ月間起動しっぱなしだと$350くらいかかる計算になるで、インスタンスの落とし忘れには注意しましょう。

なお`SAP Frontend`が必要ない場合は

- EBS利用料としては月額 $9.5
- EC2利用料としては1時間 $0.266

となります。

EC2利用料とEBS利用料は上記で、あとはデータ通信量が少し発生しますが。
RDPやSSHでつなぐだけったら気にするような金額は発生しないように思います。

余談ですが、`SAP CAL`の`Cost Calsulator`とは微妙に数値がちがいますね。
`CAL`に該当製品が登録された時のAWS単価なのかな？　という気が少しします。

`Cost Calsulator`  
![](image/cost-calculator.png)

## おまけ

`SAP Community`にABAP初心者向けの記事があるので紹介。

[ABAP for Newbies](https://community.sap.com/topics/abap/abap-for-newbies)

## 総評

昔から`AS ABAP Developer Edition`のインストール記事でも書こうかとは思いつつも手が動いてませんでした。

SAP Advent Calendar 2020に登録したことですし、一念発起して書こうかとも思いましたが。
最近、ネット上に日本語な`AS ABAP Developer Edition`のインストール記事をみかけるので。
せっかくSAP Advent Calendarの一枠を埋めるのだったら、もうひと味欲しいと考えて。

SAPに縁もゆかりもない人がABAPになれしたしむためにはどうするか？

というシナリオを想定して、`P-User`を取得し、`SAP CAL`を利用して`AS ABAP Developer Edition`で開発環境構築を書いてみました。

SAPに縁もゆかりもなかった人達が、SAPの存在を認識し、ABAPを知り、さわってみたいなんて事はどれくらいあるの？　とは思いつつ。

クラウド利用料がかかりますが、ABAPさわりはじめの人だったり、ABAPよくわからないので自分専用の開発環境が手軽に欲しい人にとってはこれはこれで少しは有用かと思います。

なお`P-User`で`CAL`から`SAP IDES`もデプロイできそうに見ますが、試してないです。　できるようだったらすごい！

これでSAP環境に慣れ親しんで、ドキュメント読みながら自分で`AS ABAP Developer Edition`をインストールしても良いですし。
`P-User`以外のIDを持ってて権限があるなら`SAP IDES`をインストールして、遊んでみてもいいでしょう。

ただ、ABAPと戯れるのは個人でもなんとかなるでしょうが、業務系知識となると個人レベルで太刀打ちするようなものでもないので。
あくまでABAPの経験がつめるっている感じですね。

あと`P-User`は今回初めて取得してみました。

ソフトウェアのダウンロードが出来ないので`SAP GUI`が入手できないという事実に、`P-User`を取得してみて初めて気づきました。

`P-User`でも`SAP GUI`は何かしら取得する方法があるのでは？　と勝手に考えていましたが。　現状方法はなさそうです？

`SAProuter`とかも`P-User`でダウンロード出来るようだったら、プライベートサブネットに`SAP`を配置して`SAProuter`経由でGUI接続とかやっても面白いかもと思いましたが、`P-User`だけだと多分出来ない気がします。

ともかく、`SAP CAL`から環境構築すると少額のクラウド利用料と、少ない手間でABAPの環境が手に入るので。

今まで個人利用のABAP環境が気になっていた人は試してみるのもいいかもしれません。
