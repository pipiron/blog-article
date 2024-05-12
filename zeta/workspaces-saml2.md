---
title: Amazon WorkspacesをEntraIDでSAML2.0認証
emoji: 🪪
type: tech
topics: [AWS,WorkSpaces,EntraID]
published: true
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
![workspaces-saml2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/ea4a3e3e-aaf4-3971-66ec-5adec6783d16.png)

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
![workspaces01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/d7a7c77e-86f0-d6eb-f902-d8213f97e9df.png)

サイズは当然スモール。  
![workspaces02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/5acdfed9-7a8f-650e-c876-d85b97f9621a.png)

設置先のVPC、サブネットを設定。  
![workspaces03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/dbf64af0-becd-2ff5-0f28-9cf77dd1cb80.png)

ADサーバへの接続情報を入力。  
![workspaces04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/0b43a65c-d0b0-c4ac-0d51-bb9e2d94af48.png)

作成完了。  
![workspaces05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/4052ac52-8af9-4d16-a298-79bf2f2c29d5.png)

### 2.Workspaces作成
次にWorkspacesを作成。  
先ほど作成したディレクトリを登録。  
![workspaces06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/2998848a-8b60-74e8-efa8-976c9837ab87.png)

ユーザー作成は不要。  
![workspaces07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/d34dc835-d8f9-5164-3c5a-60d2638fb9a6.png)

対象のユーザーを選択。  
![workspaces08.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/4ef1e133-9c8e-4aa3-d7d0-ddd3e3a757b5.png)

バンドル(スペック)を選択。  
![workspaces09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/545e67f1-0e9c-39fe-8cef-7f3db307735f.png)

OSイメージも選択。  
![workspaces10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/837f41e4-969f-037d-f881-3191a8c16b11.png)

実行モードを選択。  
![workspaces11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/e061cff8-10c8-ae1c-0ac3-d8632fd5230e.png)

必要に応じてカスタマイズ。  
![workspaces12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/067f2ce3-309b-9d15-6113-10107c6a63a2.png)

Workspaces作成完了。  
![workspaces13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/6e4bcdc4-4826-540b-4ed6-45e5d13e4ffd.png)

### 3.エンタープライズアプリケーション作成、SAML設定
実質的にここから本題。  
Entra IDにログインし、SAML2.0用のエンタープライズアプリケーションを作成する。  

[独自アプリケーションの作成]をクリック。  
![entra01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/de92a9c2-8933-e2e8-5dd2-0c60c9154fba.png)

任意の名前を入力して、アプリ作成。  
![entra02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/3a0867ae-43c7-2d10-8715-62c40b4ca440.png)

[シングルサインオンの設定 - 作業の開始]をクリック。  
![entra03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/0eceb4b1-a6cb-8f52-c45e-626c02724a26.png)

[SAML]をクリック。  
![entra04.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/3ce75cb0-44cd-11b9-03a9-8b1d408d589f.png)

AWSが提供されているWorkspacesのSAMLメタデータを保存。  
https://signin.aws.amazon.com/static/saml-metadata.xml

DLしたSAMLメタデータをアップロードする。  
![entra05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/bdad0bd8-75e8-0ccc-d1a0-6d7ee3efd1f4.png)

[保存]をクリック。  
![entra06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/b1295d32-f8af-79e4-f8d6-a30514b105f5.png)

[フェデレーションメタデータXML]をダウンロードする。  
![entra07.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/d161ed1d-3298-c70a-2b24-df89fc2f3712.png)

### 4.IAM Identity Provider作成
AWSのIAMコンソールに移動。  
[IDプロバイダ]をクリック。  
![workspaces20.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/9927b5c2-37e1-257e-5e23-2ccc0122f9c5.png)

[プロバイダを追加]をクリック。  
![workspaces21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/532ae1a6-420a-5b27-00bd-5486b20e196b.png)

任意のプロバイダ名を入力し、先ほどDLしたフェデレーションメタデータXMLファイルをアップロードする。  
![workspaces22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/a9218647-855d-8a7e-f1b9-fb565bbbd9fb.png)

IDプロバイダを作成できたので、[ロールの割り当て]をクリック。  
![workspaces23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/9f0f3694-85ea-d64f-e634-090391fe7c9c.png)

### 5.IAMロール作成
[新しいロールを作成]を選択して[次へ]をクリック。  
![workspaces24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/a373f95b-2902-8059-a79d-31515c3c2d3a.png)

信頼されたエンティティタイプ等はそのまま。  
![workspaces25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/8431ac48-9162-e63e-4d04-57d5f50ebfde.png)

属性:**SAML:sub_type**  
値:**persistent**  
をそれぞれ設定する。  
![workspaces26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/2082a970-d95a-7d6b-fc75-2f1d43a3ddb9.png)

許可ポリシーは未設定のまま次へ。  
![workspaces27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/5736e438-845e-b30a-cfc6-ecd3b5a058e0.png)

任意のロール名を入力して、IAMロール作成。  
![workspaces28.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/4393f4dd-1172-2b9c-8c8d-276e1189f146.png)

作成したIAMロールを開き、[信頼関係]タブをクリックして、[信頼ポリシーを編集]をクリック。  
![workspaces29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/fc40c33f-0a2d-d7f5-00a9-0d8c2cf60ed1.png)

Actionに**"sts:TagSession"**を追記する。  
![workspaces30.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/94dc380e-95d5-11ae-0a8a-00684c095d2c.png)

次にインラインポリシーを作成する。  
![workspaces31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/9afba30a-83af-1ead-9a90-6392a20dff95.png)

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
![workspaces32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/ca865dbb-0453-bb9e-3b29-35c9a716eb19.png)

任意のポリシー名を入力して、ポリシー作成。  
![workspaces33.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/5a8621ab-d877-cde1-23de-be2347e7d099.png)

### 6.SAML認証応答のアサーション設定
再びEntra IDに戻って、属性とクレームを編集する。  
![entra10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/07dd1349-c203-6a4d-1a41-32aa51e2edfc.png)

デフォルトで設定されている4つの項目を削除する。  
![entra11.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/9866a6d4-8358-6687-c08c-043e48e9f91c.png)

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
![entra12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/44928591-73ff-fd47-401f-07b21432d8b0.png)

名前識別子の形式を[永続的]に変更する。  
![entra13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/902758a6-d2d4-bd97-4095-d06a29b9897b.png)

最後にユーザーを追加する。  
ここに登録したユーザーがSAMLログイン可能になる。  
![entra16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/c9a0f1c4-a6d5-eb3c-fe59-8c40fac7cc47.png)

### 7.フェデレーションのリレーステート設定
続けて、[基本的なSAML構成]を編集する。  
![entra14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/e4d5543f-6f63-82ac-99ef-5bbce8d32f06.png)

リレー状態を入力する。  
```
https://workspaces.euc-sso.ap-northeast-1.aws.amazon.com/sso-idp?registrationCode=registration-code
```
"regitration-code"はWorkspacesの登録コード。  

こんな感じ。  
![entra15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/f7e1b7c6-e2f2-eaca-994c-c0962964d21a.png)

### 8.Entra IDアプリリンクのコピー
https://myapps.microsoft.com/
にアクセス。

先ほど作成したエンタープライズアプリケーションが表示されているので、[リンクのコピー]をクリックする。  
![entra17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/10ee0d04-35b3-8c1f-fd71-32387fdb4267.png)

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
![workspaces40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/a363d051-ea9f-3bf3-6a63-5975fd6302fc.png)

[認証を編集]をクリック。  
![workspaces41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/d3742827-5233-e8b3-20cc-0745df1629f5.png)

[SAML2.0アイデンティティプロバイダーの編集]をクリック。  
![workspaces42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/0225d781-7df0-9419-cce1-f7de75e1b6f6.png)

[SAML2.0認証の有効化]にチェックを入れ、  
[ユーザーアクセスURL]は先ほど確認したURLを入力する。  
![workspaces43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/c72fd637-2532-b0d2-e813-329e9945fcfa.png)

以上で設定完了。  
![workspaces44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/cdb5350f-5cce-8118-4d4f-6223fa6b8ac8.png)

### 10.動作確認
早速動作確認を行なってみる。  
Workspacesクライアントで登録コードを入力すると、ログイン画面が以下のようになっている。  
![workspaces45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/f3b30b2e-6616-cd47-6b39-27a7cf455908.png)

サインインボタンをクリックすると、ブラウザが立ち上がり  
Entra IDのサインインを求められる。  
ここでMFAの設定を行っておけば、MFAの入力が求められる。  
サインインが完了すると、Safariの場合は以下のようにアプリに戻るように促される。  
![workspaces46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/028fc7be-210c-6778-774c-8d27c4bfe570.png)

その後はWorkspacesにログインするためのID/Passを入力する。  
![workspaces47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/329638/83463089-f15a-7338-1ec5-bb2f4829f671.png)


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