---
title: MacでMicrosoft Enterprise SSOを使う
tags: 
 - Mac
 - Entra ID
 - mdm
 - Jamf
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---
## MacでもEntra IDのSSOを実現させたい
WindowsではWindows Hello for Businessによって、
Entra ID認証をシームレスにSSOすることができます。

Macはまだ残念ながら完全に対応していない。
が、ブラウザとWordやExcelなどのOfficeアプリであればSSOが利用できるようになっている。
詳しくは以下参照。

https://learn.microsoft.com/ja-jp/entra/identity-platform/apple-sso-plugin


このプラグインを使うことで、少しばかりEntra IDのSSOができるようになる。
というわけで手持ちのMacで試してみました。

## 事前準備
MacをMDMで管理できるようにしておくこと。
MSのサイトにはIntuneとJamf Proのやり方が載っているが、
MDMならなんでもOK。

自分はJamf Nowを使ってみた。

https://qiita.com/pipiron3/items/eee1130ff9440dfa8d2f


MDMじゃなくても構成プロファイルをインストールしていたらいいかも。

## SSOカスタムプロファイル作成
SSO設定可能な構成プロファイルを作成する。
IntuneやJamf Proは専用のGUIがあるが、Jamf Nowは残念ながらGUIがない。
なので、手動で作る。
Jamf Nowのカスタムプロファイルの項目を確認すると、
次の2つのツールで作成したプロファイルを読み込むことが可能。
1. Apple Configurator
2. iMazing Profile Editor
![app00.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/30268016-c7a0-35f8-d644-87902572fb7a.png)

Apple ConfiguratorではSSO設定ができなかったため、
2のツールを使う。
以下のURLからインストール。

https://imazing.com/profile-editor


Profile Editorを開いたら、
GeneralのNameに任意のプロファイル名を入力する。
![app01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/6e3f31d7-4729-393a-983e-03cb5364b580.png)

左ペインに"*Single Sign-On Extention*"があるので、そこからペイロードを新規に作成する。
![app02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/fa636182-dd26-ec8a-fd68-403babb2b225.png)

MSのサイト通りに入力する。
| 項目 | 入力値 | 
|:-----|:----|
| Extention Identifier | com.microsoft.CompanyPortalMac.ssoextension |
| Type | Redirect |
| Team Identifier | UBF8T346G9 |

![app03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/349437b3-9e7b-91cb-1771-2a2382791443.png)


"*URLs*"には以下を入力する。
- https://login.microsoftonline.com
- https://login.microsoft.com
- https://sts.windows.net
- https://login.partner.microsoftonline.cn
- https://login.chinacloudapi.cn
- https://login.microsoftonline.us
- https://login-us.microsoftonline.com

![app04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/b428a4ee-da6e-f489-3316-e0e33ea66f49.png)

少し下の方にスライドして、
"*App Allow List*"には**com.microsoft.,com.apple.**を入力する。
また、"*Allow Users to Sign in ...*"にチェックを入れる。
![app05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/1333f48e-4ced-f3ac-acbc-3a49e5d42cdc.png)

以上でプロファイルの作成は完了。
PCに保存する。

## Jamf Nowにプロファイル追加
次にJamf Nowに移動して、プロファイルの追加を行う。
![jamf20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/571b6423-3b7b-1cfe-b247-39ebe0c4a51a.png)

先ほど作成したプロファイルをアップロードする。
![jamf21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/fc686266-1288-107a-e8a0-38282ba554ff.png)

プロファイルをJamfに追加。
![jamf22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/7d09b023-a7d5-000f-4207-3ca69bf4b6c4.png)

プロファイルの追加完了。
![jamf23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/2376715d-65d4-1dfc-7da8-960ee99db2eb.png)

しばらくすると、Macの方にもプロファイルが追加されている。
![jamf24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/46d50933-911d-ecc7-a15d-ddd1e3671aa2.png)

## ポータルサイトアプリをMacにインストール
Microsoftポータルサイトアプリをインストールする。
以下のURLをクリックしてインストーラーをDL。

https://go.microsoft.com/fwlink/?linkid=853070


インストーラーを実行して、指示に従ってインストールする。
![portal01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/29ab50b7-1e9c-8dad-a23f-be894cfa6b9f.png)

インストール完了。
![portal02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/78523f21-b792-4dbe-600f-431b81538bc9.png)

## 動作確認
以上で設定が終わったので、SSOのテストをやってみる。
Safariをプライベートモードに開くか、Entra IDでサインアウト。
その後、Officeポータル(https://portal.office.com )にログインする。

すると、別画面が開いてEntra IDのサインインを求められる。
![portal03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/f3b8e075-5a1b-916b-a8a1-fc3e92a21c97.png)

しかしながら以降はサインインが不要になり、自動的にEntra IDにログインが行われる。

Officeアプリでも同様にサインインが不要となった。
（勝手にログインができているイメージ）

## 明らかにサインインの頻度が減った
Macの場合、Officeアプリを使う場合はWordやExcelなどのアプリを開くたびに
わざわざサインインを行なっていたが、
このEnterprise SSOプラグインを使うことでサインイン処理の回数が激減。

非常にストレスが減るように。

WindowsみたいにMacOSのサインイン自体もSSOできるようになったらいいのだろうけど、
それはまだ未実装かな？
今後のアップデートに期待。