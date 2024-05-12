---
title: Amazon WorkspacesをEntraIDでSAML2.0認証
tags:
- AWS
- WorkSpaces
- EntraID
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: true
---
## WorkspacesでMFAの設定は面倒
代表的なDaaSであるAWSのWorkspacesとAzureのAVD。
主な違いは
| Workspaces | AVD |
|:-----------|:-----------|
| Windows Serverのみ | Windows10/11利用可 |
| 1台=1人の占有 | 1台=複数人の共有もOK |
| 認証は内部 or AD | 認証はEntra ID利用可脳 |
| MFAはRadius必要 | MFAはEntra ID依存 |

と挙げだしたらキリがないのだけど、  
WorkspaceでMFAを利用するためにはRadius環境が必要。  
そのRadisuサーバを個別に立てるのは面倒なので、  
別の方法として、  
SAML2.0の連携を行い、その連携先でMFAを設定すれば  
WorkspacesでもMFAを利用可能するのでは、と思い  
設定することにしてみた。  

## 構成図
![workspaces](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces-saml2.png)

同じアカウントに作ったら面白みがないので、  
ADサーバは別アカウントに作る。  
加えて、ADとEntra IDはEntra ID Connectでユーザーを同期。  
これで同じID/PassでWorkspacesにもログインができるようになる。  

## Workspaces構築
ADの構築やEntra ID Connectの設定等は省略。  
VPCやPeeringも省略。
Workspacesの構築から。

### 1.Directory Service設定
Workspacesで使うディレクトリをセットアップする。  
ここではAD Connectorにする。  
![AD Connector](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces01.png)

サイズは当然スモール。  
![small](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces02.png)

設置先のVPC、サブネットを設定。  
![vpc](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces03.png)

ADサーバへの接続情報を入力。  
![connectad](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces04.png)

作成完了。  
![active](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces05.png)

### 2.Workspaces作成
次にWorkspacesを作成。  
先ほど作成したディレクトリを登録。  
![adcon](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces06.png)

ユーザー作成は不要。  
![skipuser](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces07.png)

対象のユーザーを選択。  
![selectuser](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces08.png)

バンドル(スペック)を選択。  
![selectbundle](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces09.png)

OSイメージも選択。  
![osimage](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces10.png)

実行モードを選択。  
![autostop](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces11.png)

必要に応じてカスタマイズ。  
![customerize](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces12.png)

Workspaces作成完了。  
![finish](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces13.png)

### 3.エンタープライズアプリケーション作成、SAML設定
実質的にここから本題。  
Entra IDにログインし、SAML2.0用のエンタープライズアプリケーションを作成する。  

[独自アプリケーションの作成]をクリック。  
![create](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra01.png)

任意の名前を入力して、アプリ作成。  
![create apps](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra02.png)

[シングルサインオンの設定 - 作業の開始]をクリック。  
![sso-saml](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra03.png)

[SAML]をクリック。  
![saml](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra04.png)

AWSが提供されているWorkspacesのSAMLメタデータを保存。  

https://signin.aws.amazon.com/static/saml-metadata.xml


DLしたSAMLメタデータをアップロードする。  
![upload-saml](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra05.png)

[保存]をクリック。  
![save](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra06.png)

[フェデレーションメタデータXML]をダウンロードする。  
![federation](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra07.png)

### 4.IAM Identity Provider作成
AWSのIAMコンソールに移動。  
[IDプロバイダ]をクリック。  
![IAM](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces20.png)

[プロバイダを追加]をクリック。  
![add provider](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces21.png)

任意のプロバイダ名を入力し、先ほどDLしたフェデレーションメタデータXMLファイルをアップロードする。  
![id provider](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces22.png)

IDプロバイダを作成できたので、[ロールの割り当て]をクリック。  
![attache role](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces23.png)

### 5.IAMロール作成
[新しいロールを作成]を選択して[次へ]をクリック。  
![create iam role](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces24.png)

信頼されたエンティティタイプ等はそのまま。  
![entity](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces25.png)

属性:**SAML:sub_type**  
値:**persistent**  
をそれぞれ設定する。  
![persistent](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces26.png)

許可ポリシーは未設定のまま次へ。  
![policy](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces27.png)

任意のロール名を入力して、IAMロール作成。  
![iamrole](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces28.png)

作成したIAMロールを開き、[信頼関係]タブをクリックして、[信頼ポリシーを編集]をクリック。  
![trust policy](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces29.png)

Actionに**"sts:TagSession"**を追記する。  
![tagsession](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces30.png)

次にインラインポリシーを作成する。  
![inlinepolicy](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces31.png)

次のポリシーを入力する。  
自分の環境に合わせて書き換える。  
* リージョン名 : *ap-northeast-1*
* アカウントID : *111111111111*
* ディレクトリID : *d-XXXXXXXX*
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "workspaces:Stream",
            "Resource": "arn:aws:workspaces:ap-northeast-1:111111111111:directory/d-XXXXXXXX",
            "Condition": {
                "StringEquals": {
                    "workspaces:userId": "${saml:sub}"
                }
            }
        }
    ]
}
```
![json](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces32.png)

任意のポリシー名を入力して、ポリシー作成。  
![policyname](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces33.png)

### 6.SAML認証応答のアサーション設定
再びEntra IDに戻って、属性とクレームを編集する。  
![claim](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra10.png)

デフォルトで設定されている4つの項目を削除する。  
![delete](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra11.png)

追加の要求は以下の3つを設定する。  
| 名前 | 値 |
|:-----------|:-----------|
| https://aws.amazon.com/SAML/Attributes/Role | *<ロールARN>,<プロバイダARN>*(1) |
| https://aws.amazon.com/SAML/Attributes/RoleSessionName | user.mail |
| https://aws.amazon.com/SAML/Attributes/PrincipalTag:Email | user.mail |

(1)の部分は具体的にはこんな感じ。  
```
arn:aws:iam::111111111111:role/workspaces_saml2.0_role,arn:aws:iam::111111111111:saml-provider/workspaces_saml2.0
```

追加の要求の入力が終わったら、必要な要求のクレームをクリックする。  
![claim](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra12.png)

名前識別子の形式を[永続的]に変更する。  
![persistent](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra13.png)

最後にユーザーを追加する。  
ここに登録したユーザーがSAMLログイン可能になる。  
![user](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra16.png)

### 7.フェデレーションのリレーステート設定
続けて、[基本的なSAML構成]を編集する。  
![relaystate](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra14.png)

リレー状態を入力する。  
```
https://workspaces.euc-sso.ap-northeast-1.aws.amazon.com/sso-idp?registrationCode=registration-code
```
"regitration-code"はWorkspacesの登録コード。  

こんな感じ。  
![relay](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra15.png)

### 8.Entra IDアプリリンクのコピー

https://myapps.microsoft.com/

にアクセス。

先ほど作成したエンタープライズアプリケーションが表示されているので、[リンクのコピー]をクリックする。  
![myapps](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/entra17.png)

こんなURLがコピーされるので、  
```
https://launcher.myapps.microsoft.com/api/signin/cc6f9bde-441c-4890-9772-XXXXXX?tenantId=07795552-9618-42cd-ad64-XXXXXXXX
```

[https://launcher.myapps.microsoft.com/api]の部分を  
[https://myapps.microsoft.com]に置き換える。  

置き換え後↓
```
https://myapps.microsoft.com/signin/cc6f9bde-441c-4890-9772-XXXXXX?tenantId=07795552-9618-42cd-ad64-XXXXXXXX
```

### 9.WorkspacesのSAML2.0設定
最後の設定。  
Workspacesのディレクトリをクリック。  
![directory](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces40.png)

[認証を編集]をクリック。  
![edit](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces41.png)

[SAML2.0アイデンティティプロバイダーの編集]をクリック。  
![saml2.0](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces42.png)

[SAML2.0認証の有効化]にチェックを入れ、  
[ユーザーアクセスURL]は先ほど確認したURLを入力する。  
![useraccess](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces43.png)

以上で設定完了。  
![finish](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces44.png)

### 10.動作確認
早速動作確認を行なってみる。  
Workspacesクライアントで登録コードを入力すると、ログイン画面が以下のようになっている。  
![workspaces](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces45.png)

サインインボタンをクリックすると、ブラウザが立ち上がり  
Entra IDのサインインを求められる。  
ここでMFAの設定を行っておけば、MFAの入力が求められる。  
サインインが完了すると、Safariの場合は以下のようにアプリに戻るように促される。  
![workspaces](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces46.png)

その後はWorkspacesにログインするためのID/Passを入力する。  
![login](https://raw.githubusercontent.com/https://github.com/pipiron/blog-article.git/main/images/workspaces-saml2/workspaces47.png)


## SAML2.0連携でセキュリティレベル向上
WorkspacesだとPCにログインするID/Passだけだと心許ない。  
それを解消するための方法としてSAML2.0連携を行い、MFA対応可能なIdpに認証させることが可能。  
設定はかなりクセがあるが、この方法で一定のセキュリティレベルに向上させることができる。  

SAML2.0なのでさまざまなIdpに対応していると思うが、  
IAM Identity Centerで試したところうまくいかずに断念。。  
SAMLの属性マッピング（アサーション）のところがよくわからなかったので、  
同じAWSサービス同士なのでそのうち対応してくれるようになることを期待。  

## 参考URL

https://docs.aws.amazon.com/ja_jp/workspaces/latest/adminguide/setting-up-saml.html



https://qiita.com/sugimount-a/items/abad85c04448d087303f#azure-ad--アプリのリンクをコピー
