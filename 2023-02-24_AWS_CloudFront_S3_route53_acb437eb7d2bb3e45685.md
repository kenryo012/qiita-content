<!--
title:   AWS Hands-on for Beginners　AWS上で静的な Web サイトを公開しよう！：学習メモ
tags:    AWS,CloudFront,S3,route53,ハンズオン
id:      acb437eb7d2bb3e45685
private: false
-->
## はじめに

AWSハンズオン for Beginnersシリーズとして提供されている「[AWS上で静的な Web サイトを公開しよう！](https://pages.awscloud.com/JAPAN-event-OE-Hands-on-for-Beginners-StaticWebsiteHosting-2022-reg-event.html?trk=aws_introduction_page)」を実施した際のメモです。
Amazon S3には静的なコンテンツをホスティングする機能があり、簡単にWebサイトをインターネット上に公開することができます。
本ハンズオンでは、その機能を使用したり、CDNサービスであるAmazon CloudFrontを使用して、コンテンツのキャッシュを実施します。また、DNSサービスであるAmazon Route53で独自ドメインの取得およびドメインから静的サイトにアクセスしてみます。AWS Certificate Managerで証明書を発行し、HTTPSでアクセスさせるなどセキュリティ関連の設定も行います。

## アジェンダ

1. 今回のハンズオンので構築する構成の紹介 + S3 と Cloud9 の紹介
2. S3 の静的ホスティング機能を使ってみる
3. Cloud9 環境を立ち上げて静的コンテンツを開発する + AWS CLI で S3 にファイルアップロードする
4. 続・静的コンテンツの開発 + AWS CLI で S3 に複数のファイルを一括アップロードする
5. このあとのハンズオンの流れ + CloudFront, Route 53, ACM の紹介
6. loudFront を使って、画像をキャッシュさせる
7. [Option / Demo] Route 53 で独自ドメインを取得し、S3 に HTTP アクセスする
8. [Option / Demo] ACM を使い、Route 53 - CloudFront - S3 で HTTPS アクセスする
9. 削除手順の紹介、本シリーズのまとめ、Next Step のご案内

## メモ

### S3バケットのパブリックアクセス

S3の「パブリックアクセスは全てブロック」はチェックを外すと全世界にバケットの中身が公開される可能性があり、注意が必要です。
今回のハンズオンの静的ホスティングのような用途で使用しない場合はチェックし、パブリックアクセスをブロックしておく必要があります。
なお、チェックを外してからすぐにオブジェクトが公開されるわけではなく、バケットポリシーで公開する設定を行なった後に実際に公開されます。

### 独自ドメインを使用する場合のバケット名

Route53で独自ドメインを使用し、S3バケットの静的ホスティングをする場合、S3バケット名に独自ドメインと同じ文字列を指定する必要があります。
例えば、**kenryo_s.net**というドメインであれば**kenryo_s.net**というバケット名にします。

### ローカルのファイルをS3バケットにアップロードするAWS CLIコマンド

#### 1ファイルアップロード

```
aws s3 cp <ファイルパス> s3://<S3バケット名>
```

#### 複数ファイルアップロード（再起的。フォルダもアップロードされる。）

```
aws s3 cp <フォルダ名> s3://<S3バケット名> --recursive
```

### faviconが表示されない

faviconを設定しても表示されない場合がありました。
Chromeの開発者ツールでfaviconファイルへのアクセスを確認したところ、faviconファイルへのリクエストがされていないことがわかりました。
一方、styles.cssやlambda.pngへはリクエストしていました。
Chromeのキャッシュの問題であったようで、スーパーリロード（Ctrl + Shift + R）をしたら表示されました。

### CloudFrontで指定するオリジンドメイン名

オリジンドメイン名にはS3静的ホスティングで発行されたドメイン名を指定します。

### コンテンツのキャッシュがCloudFrontから取得されたか確認する

CloudFrontのディストリビューションを作成すると、オリジンドメインとは別にドメイン名が作成されます。
ここにアクセスするとCloudFrontにキャッシュされているコンテンツにアクセスができます。
ただ、場合によってはブラウザにキャッシュされているコンテンツが画面に表示されている場合があります。
CloudFrontからコンテンツのキャッシュが取得されたかどうかは、当該コンテンツのHTTPレスポンスヘッダーのX-Cacheの値から判断できます。
値が「Hit from cloudfront」の場合、CloudFrontから取得したことを表します。
一方「Miss from cloudfront」の場合は、ブラウザのキャッシュを使用していることを表します。

### Route53でドメインの購入、Aレコードの追加

登録済みドメイン->ドメインの登録から任意のドメインを購入することができます。
登録したドメインのホストゾーンからAレコードを追加する。Aレコードのエイリアスとして、静的ホスティングをしているS3バケット名を指定します。

https://engineer-ninaritai.com/route53-s3-alias/

### AWS Certificate ManagerでCloudFrontが使用する証明書を作成する場合の注意点

証明書は**バージニア北部**で作成する必要があります。

### ACMのDNS検証

ACMが証明書に紐づくドメインのDNS検証を行う際、Route53にCNAMEレコードを作成する必要があります。

https://docs.aws.amazon.com/ja_jp/acm/latest/userguide/dns-validation.html

### ACMの証明書適用

発行された証明書を使用するためには、CloudFrontとRoute53で設定が必要です。

#### CloudFront

CloudFrontの画面のBehaviorsで「Viewer Protocol Policy」を「Redirect HTTP to HTTPS」に変更します。
これでCloudFrontへのHTTPへのアクセスはHTTPSにリダイレクトされます。
次に、Generalで「Alternate Domain Names(CNAMEs)」に購入したドメイン名を指定し、SSL Certificateで「Custom SSL Certificate」を選択し、表示された選択欄にACMで作成した証明書を選択します。

#### Route53

Aレコードのエイリアスの向き先をS3からCloudFrontのディストリビューションに変更します。

### オリジン（S3）の保護

現在の設定だとCloudFrontのオリジンであるS3バケットにインターネットからアクセスでき、セキュリティ的に問題があります。
S3とCloudFrontの設定を変更し、オリジンを保護します。
長年、CloudFrontからS3へのアクセス制限としてOAIが使用されてきましたが、最近OACという制御が追加されています。今回はこちらで制御してみました。

https://dev.classmethod.jp/articles/amazon-cloudfront-origin-access-control/

#### S3

アクセス制御からバケットポリシーを削除します。次に、静的ウェブサイトホスティングを無効化にします。


#### CloudFront

Origins and Origin Groupsで、オリジンドメイン名を変更します。
静的ホスティングで発行されたドメインからS3バケットのドメインに変更します。
次に、S3バケットアクセスをPublicからOrigin access control settings (recommended)に変更します。
Origin access controlで「コントロール設定を作成」し、作成したコントロール設定を選択します。
バケットポリシーで「ポリシーのコピー」をしておきます。S3のバケットポリシーに先ほどコピーしたポリシーを貼り付けます。
ハンズオンではOAIでオリジン保護しており、自動的にポリシーに反映されるよう選択できましたが、OACでは手動みたいです。

https://aws.amazon.com/jp/blogs/news/amazon-cloudfront-introduces-origin-access-control-oac/

## ハンズオンの感想

静的なサイトであれば簡単にインターネットに公開できるので非常に便利だと感じました。
Route53、CloudFront、ACMが絡むと多少設定が複雑になりますが、セキュアで高耐久、高可用性を求めるのであれば、理解しておく必要があります。