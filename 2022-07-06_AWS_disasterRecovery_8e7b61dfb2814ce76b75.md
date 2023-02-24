<!--
title:   AWS Elastic Disaster Recovery Workshopをやる
tags:    AWS,disasterRecovery
id:      8e7b61dfb2814ce76b75
private: false
-->
# はじめに

この記事は[AWS Elastic Disaster Recovery Workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/080af3a5-623d-4147-934d-c8d17daba346/en-US)をやった際のメモ書きです。
リンク先のWorkshopは英語で書かれています。もし、英語を読むことが大変である方はこの記事をぜひ参考にしてください。
ただし、リンク先の情報を全てこちらに記載しているわけではないため、詳細な情報を得たい方は本家Workshopを実施してみてください。

# Introduction

AWS Elastic Disaster Recovery（DRS）は、OS、データベースやアプリケーションなどのマシンを、低コストなステージングエリアに継続的にレプリケートするサービスです。
災害発生時にDRSに指示することで、レプリケートしていたマシンをリカバリーインスタンスとして、数分で自動的に起動することができます。

以下はDRSの基本的なネットワーク図です。
左手のオンプレデータセンターにあるサーバにはAWS Replication Agentがインストールされています。
そして、サーバから継続的にブロックレベルのレプリケーションが行われます。レプリケーションのデータは圧縮、暗号化されます。
右手のステージングエリアには軽量なEC2インスタンスがレプリケーションサーバとして起動されており、ここにレプリケーションデータが蓄積されます。
DRSへの指示により、リカバリーサブネットにリカバリーインスタンスを起動させることができます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/51d608f8-a70d-1977-cd8a-cc903e8f7d44.png)

# Workshop

このWorkshopでは次のことを行います。
- AWS Replication Agentのインストールとサーバのレプリケーション設定
- 起動テンプレートの設定
- レプリケーションのモニタリング
- フェールオーバーの実施とそれが成功したか確認

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8a7d2d54-12bf-0116-c545-0c65958d197c.png)

# 1. Account Setup

自己学習のためのCloudFormationテンプレートを[ここ](https://catalog.us-east-1.prod.workshops.aws/workshops/080af3a5-623d-4147-934d-c8d17daba346/en-US/workshop/prereqs)からダウンロードし、CloudFormationスタックを作成してください。(入力は全てデフォルト値で構いません)
使用するリージョンはバージニア北部（us-east-1）とします。
以下の認証情報をメモっておいてください。

```
Bastion Host
user = Administrator
password = Adm1nP@s

Linux Hosts
password = SshPass1
```
# 3. Connect to the Bastion Host

BastionにRDPで接続してみます。
当方Macを使用しているためMicrosoft Remote Desktopでアクセスしてみます。

BastionのIPアドレスはCloudFormationで作成したスタックの出力から確認します。
`ネストされていない`スタックを選択し、出力の`BastionRDP`の値を確認します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/b971c7ec-8a19-f5f1-bd75-6c132eaef0af.png)

続いて、EC2の画面に移動します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/42910512-2be8-3959-adcc-3ea9dbe1fae9.png)

Bastionインスタンスを選択し、セキュリティグループのインバウンドルールを確認します。
タイプがRDPのもののソースをマイIPに変更します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/dd25d5dc-a439-1be3-459c-2e385efef41c.png)

Microsoft Remote DesktopでBastionにRDPで接続します。
ユーザー名、パスワードは`1. Account Setup`でメモしたものを入力してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9b8e05bc-5d8f-7e0f-d48f-9390edb0f8a6.png)

# 4. Configure DRS Settings

AWS Elastic Disaster Recovery (DRS)のコンソールを表示します。
ここでは、ステージングエリアをどこにするかや、レプリケーションサーバのキャパシティなどの設定を行います。

1. 画面に表示されている`Set default replication settings`ボタンを押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/de91b71b-0340-78ca-3575-760aa019de4d.png)

2. `Stagin area subnet`で`CloudEndure Staging`を選択、`Replication server instance type`で`t3.small`を選択し、`Next`ボタンを押下します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/800f88cf-85c1-2cb0-9cd9-bae78d9367cd.png)


3. レプリケーションを保存するEBSのタイプや暗号化有無、また、ステージングエリアへのアクセスを制御するセキュリティグループを選択します。今回は全てデフォルト設定で構いません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4bc1568e-d0f5-fa90-ed22-6d04a737e046.png)

4. レプリケーショントラフィックのルーティング設定やポイントインタイムリカバリの設定を行います。今回は全てデフォルト設定で構いません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/bbb0324b-788b-de50-9bb7-1a4f4c95c6ac.png)

5. `Create default`ボタンを押下します。このデフォルト設定はAWS Replication Agentがインストールされたサーバのデフォルト設定となります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/06f000df-571a-69c4-7d75-fb9d06ef18c9.png)

6. AWS Replication Agentはまだどのサーバにもインストールされていないため、現時点で`Source servers`には1件も表示されません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ba34f09e-a95c-056f-8abd-bc2ed2904ca9.png)

# 5. Configure DRS Agent IAM User

AWS Replication AgentはAWSアカウントと対話するため、IAMクレデンシャルを必要とします。
ここでは、IAMユーザーとクレデンシャルを設定します。

IAMコンソールを表示し、ユーザー、`ユーザーを追加`ボタンを押下します。
スクショの`CloudEndureUser`はCloudFormationで自動的に作成されるようです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/caaede3a-4233-32d5-76e3-c14331610b87.png)


ユーザー名`drs-agent-user`、`アクセスキー-プログラムによるアクセス`をチェックし、`次のステップ：アクセス権限`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4ff6983d-e068-01d6-1e23-aa549c033e97.png)

`既存のポリシーを直接アタッチ`を選択、フィルタに`disaster`を入力し、ポリシー名`AWSElasticDisasterRecoveryAgentInstallationPolicy`をチェック。`次のステップ：タグ`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9cef0d7e-8626-54f2-704a-e0c35906415e.png)

`次のステップ：確認`、`ユーザーの作成`ボタンを押下し、ユーザーを作成します。
作成後、アクセスキーIDとシークレットアクセスキーがCSVでダウンロードできるので、ダウンロードしておきます。

# 6. Installing the Agent

各マシンにAWS Replication Agentをインストールし、各マシンからステージングエリアのインスタンスへのレプリケーションを開始します。

RDPでBastionインスタンスにログインします。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c87245d9-2f8f-1f62-1ded-9342651cc4cd.png)

デスクットップのPuttyアイコンをダブルクリックし、HostNameにAWS Replication AgentをインストールするサーバのFQDNを入力し、Openボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/6cfe96c6-6be6-a489-9230-2c5513ca178c.png)

__AWS Replication Agentインストール対象__

|Server Name	|FQDN|	OS|	Username|	Password|
|---|---|---|---|---|
|wordpress-web|	wordpress-web.onpremsim.env|	Linux|	user |`1. Account Setup`の`Linux Hosts`の`password`|
|wordpress-db|	wordpress-db.onpremsim.env|	Linux|	user	|`1. Account Setup`の`Linux Hosts`の`password`|

ユーザー名とパスワードが求められるので、上記の通り入力します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/5a1a4559-609e-e23f-e771-28cee520f725.png)

ログインできたら、以下のコマンドを実行します。

```
wget -O ./aws-replication-installer-init.py https://aws-elastic-disaster-recovery-us-west-2.s3.amazonaws.com/latest/linux/aws-replication-installer-init.py
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d88a8822-1959-6b48-d3c3-0a00c8be6a8a.png)

次に以下のコマンドを実行します。

```
sudo python3 aws-replication-installer-init.py
```

途中入力が求められます。
`AWS Region Name`はBastionインスタンスを起動しているリージョン(us-east-1など)を入力してください。
`AWS Access Key ID`は`5. Configure DRS Agent IAM User`の手順でダウンロードしたCSVに記載されているAccess Key IDを入力してください。
`AWS Secret Access Key`は`5. Configure DRS Agent IAM User`の手順でダウンロードしたCSVに記載されているSecret Access Keyを入力してください。
最後にディスクについて質問されますがそのままエンターしてください。
完了まで体感5分かかります。

以上の手順を2つのサーバ（wordpress-web, wordpress-db）に対して実施してください。

Agentインストール完了後、AWS Elastic Disaster Recovery（DRS）の`Source servers`に戻り、Source ServerにAgentをインストールした2サーバが表示されていることを確認します。

登録した直後は`Ready for recovery`が`Initial sync`になっており、初回同期を実行しています。
初回同期では、以下のタスクを実行しています。

- セキュリティグループの作成
- レプリケーションサーバーの起動
- レプリケーションサーバーのBoot
- サービスとの認証
- レプリケーションソフトウェアのダウンロード
- ステージングディスクの作成
- ステージングディスクのアタッチ
- レプリケーションサーバとAWS Replication Agentのペアリング
- AWS Replication Agentとレプリケーションサーバを接続
- データ転送の開始

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/71a0f1e5-52c5-fa03-630c-36fa0ba82c66.png)

初回同期が完了すると、`Data replication status`が`Healthy `に変わります。

# 7. Configure Launch Settings

リカバリーインスタンスを起動するため、EC2の起動テンプレートの設定を行います。

AWS Elastic Disaster Recovery（DRS）コンソールで、`Source servers`を表示し、Source Serverの1つをクリックします。
`Launch settings`タブを選択し、`General launch settings`の`Edit`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ef8966a3-0489-d96b-4dda-2d7b3b096664.png)

`Instance type right sizing`で`None`を選択し、`Save settings`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/31ef14e3-1318-7ef7-352b-932c6fb71de3.png)

続いて、`EC2 launch template`の`Edit`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8d2ad567-727a-362b-5f17-33769256f396.png)

EC2の起動テンプレートボックスで、`Edit`を押下します。
テンプレートバージョンの説明に`WorkshopTemplate`と入力します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/b3d484dd-f2dc-72db-e906-574cac585800.png)

インスタンスタイプに`t3.small`を選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/0a4abb78-f056-1a4b-7bfd-2fbacba58e97.png)

`ネットワーク設定`の`サブネット`で`TargetPrivate`を選択します。
また、ファイアウォール（セキュリティグループ）で、`既存のセキュリティグループを選択する`を選択します。
そして、`共通のセキュリティグループ`の欄で`TargetSecurityGroup`を名前に含んだセキュリティグループを選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/43d2f36d-ddbf-30cd-11ac-29a817b7aede.png)

`ストレージの設定`でボリュームの種類を`gp3`に変更します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/841e95e4-4076-ed1f-761b-74aef0ba3092.png)

それ以外はデフォルトのまま、`テンプレートのバージョンを作成`ボタンを押下します。

作成した起動テンプレートIDを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/afe3c01e-a1e9-9efd-c9bd-79baf3d29ddb.png)

`アクション`で`デフォルトバージョンを設定`を押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7bcd9ea4-6917-7c72-3ef1-8f065e87643a.png)

`テンプレートのバージョン`で`WorkshopTemplate`を選択し、`デフォルトバージョンとして設定`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d9baa66a-2661-342c-b60d-ea63e6e54615.png)

以上を2サーバに対して実行してください。

# 8. Failing Over

DRSコンソールで、Source Serverの1つを選択し、`Server info`を表示します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/1af07a1e-e99e-993a-bf89-53a227800a4c.png)

画面右上にある`Initiate recovery job`ボタンを押下し、`Initiate Drill`を選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c15f3bd8-d239-e817-b5cf-bd089b86ce64.png)

`Use most recent date`を選択し、`Initiate drill`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/08f68abe-0fdb-e10e-84db-67df9020faf9.png)

画面左の`Recovery job history`メニューを選択し、`Job ID`を選択します。
`Job log`でRecoveryが成功したことを確認します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/362b05da-6b13-4f69-11bf-65089628b3db.png)

以上を2サーバに対して実行してください。

# 9. Validating Fail Over

EC2コンソールでインスタンス`wordpress-web.onpremsim.env`、`wordpress-db.onpremsim.env`が新たに起動されていることを確認します。
`wordpress-web.onpremsim.env`の`プライベート IPv4 アドレス`をメモしておきます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/af36f4a2-2126-0609-f158-639b96e3fda4.png)

Bastionサーバにログインし、PuTTyします。
`Saved Sessions`で`wordpress-web`を選択し、`Load`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/302235fa-5c06-73da-8c28-b6019c64dd0d.png)

`Host Name(or IP address)`の`wordpress-web`の部分を先ほどメモした`プライベート IPv4 アドレス`に変更し、`Open`ボタンを押下します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/53492208-1880-80bc-ebe0-2661ad90e5d7.png)

以下のコマンドを実行し、インスタンスのDNSレコードを更新します。

```
ADDR=`hostname -I`
HOST=`hostname`
sudo touch /tmp/nsupdate.txt
sudo chmod 666 /tmp/nsupdate.txt
echo "server dns.onpremsim.env" > /tmp/nsupdate.txt
echo "update delete $HOST A" >> /tmp/nsupdate.txt
echo "update add $HOST 86400 A $ADDR" >> /tmp/nsupdate.txt
echo "update delete $HOST PTR" >> /tmp/nsupdate.txt
echo "update add $HOST 86400 PTR $ADDR" >> /tmp/nsupdate.txt
echo "send" >> /tmp/nsupdate.txt
sudo nsupdate /tmp/nsupdate.txt
```

以上を2サーバ実行してください。

最後にBastionサーバのChromeブラウザを起動し、`http://wordpress-web.onpremsim.env`にアクセスしてください。
以下のように表示されれば成功です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/80dd3804-ec59-cfd7-2041-985c445a6f64.png)

# 後片付け

CloudFormationで`ネストされた`スタック以外のスタックを選択し、削除を実行してください。
スタックの削除に失敗する場合があります。私の場合、再度削除を実施することで解決しました。
それでも削除に失敗するリソースがある場合、CloudFormationからではなく、手動で直接削除してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/6099a635-f62a-1515-912e-732a37432b4b.png)

EC2コンソールにを表示し、Recoveryしたサーバ2台を終了します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/5d377775-34f5-d7ae-8ee3-2074b6cca5cb.png)

EC2コンソールで`ボリューム`メニューを選択し、本ハンズオンで生成した2つのボリュームをチェックします。`アクション`の`ボリュームの削除`を選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/af39ea0d-75bb-0a29-07a7-653e337077bc.png)

EC2コンソールで`スナップショット`メニューを選択し、本ハンズオンで生成した1つのスナップショットをチェックします。`アクション`の`スナップショットの削除`を選択します。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/021618e2-fdd9-055b-7459-e5263a725c2b.png)

DRSコンソールで`Recovery instances`を選択し、表示されているサーバをチェックし、`Actions`の`Disconnect from AWS`を押下してください。
ダイアログで`Disconnect`ボタンを押下してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/6a8d5db6-1c3c-daf9-3d24-295b355d2bca.png)

続いて、再度サーバをチェックし、`Delete recovery instances`を押下してください。ダイアログで`Delete`ボタンを押下してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/82081d6a-267b-4aed-7cf0-547e5f13fecf.png)

DRSコンソールでSource serversを選択し、表示されているサーバをチェックし、`Actions`の`Disconnect from AWS`を押下してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/abeb213f-b37b-7221-b4f6-2d24264b6885.png)

続いて、再度サーバをチェックし、`Delete server`を押下してください。ダイアログで`Permanently delete`ボタンを押下してください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3e0dd844-d176-b383-8f54-80d74bdeb07f.png)