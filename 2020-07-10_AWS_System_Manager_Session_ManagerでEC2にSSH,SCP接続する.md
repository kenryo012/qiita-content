<!--
title:   AWS System Manager Session ManagerでEC2にSSH,SCP接続する
tags:    AWS,EC2,SSH,SystemManager,scp
id:      7dda13adb373ac40e984
private: false
-->
# はじめに

Session Managerというものを今更知った。
セキュリティグループでsshを開けたり、踏み台サーバを経由してEC2にアクセスしたり、といったことがより楽に、安全になるようだ。

本記事では

- Session Managerの基本的な設定手順
- 監査ログの出力設定手順
- Session Manager + ssh,scpコマンドでEC2にアクセスする手順
- Session Managerでのアクセスを無効にする手順

を示す。

## Session Managerとは

[公式ユーザガイドより](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html)

>Session Manager はフルマネージド型 AWS Systems Manager 機能であり、インタラクティブなワンクリックブラウザベースのシェルや AWS CLI を介して、EC2 インスタンス、オンプレミスインスタンス、仮想マシン (VM) を管理できます。Session Manager を使用すると、インバウンドポートを開いたり、踏み台ホストを維持したり、SSH キーを管理したりすることなく、監査可能なインスタンスを安全に管理できます。また、Session Manager を使用すると、マネージドインスタンスへの簡単なワンクリックのクロスプラットフォームアクセスをエンドユーザーに提供しつつ、インスタンスへの制御されたアクセス、厳格なセキュリティプラクティス、完全に監査可能なログ (インスタンスアクセスの詳細を含む) が要求される企業ポリシーに簡単に準拠できます。

上記をまとめるとSession Managerのメリットは大きく分けると以下の2つである。

- SSH鍵、アクセス制御、踏み台サーバの管理・設定が不要になる
- 監査ログを残すことができる

## Session Managerの基本的な設定手順

EC2インスタンス、ロール作成などの基本的な部分は掻い摘んで説明させていただく。

### EC2インスタンスの作成

セキュリティグループでSSHを許可する必要はない。

<img width="1438" alt="1-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/dce76e7c-e18e-aba4-589f-3bc5cb7522fb.png">

キーペアの作成も不要。

![1-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/5e977ef1-1910-cf63-2ee4-32cfdaca4159.png)

### ロール作成

IAMでロールの作成を行う。`ユースケースの選択`で`EC2`を選択する。

<img width="1439" alt="2-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7c8dbc40-dc74-fd9d-b3f1-db44d17ab084.png">

`AmazonSSMFullAccess`を選択する。

<img width="1440" alt="2-2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3c1f8063-59da-0716-b251-db9561878675.png">

適当に名前を入力し、ロールを作成する。

<img width="1440" alt="2-4.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/1e8da821-26ca-578f-729d-17c27bed281e.png">

### ロールをEC2へアタッチ

[EC2インスタンスの作成](#ec2インスタンスの作成)で作成したインスタンスを選択し、`IAM ロールの割り当て/置換`を選択する。

![3-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c21c1eb4-e2cc-cb2f-ff4c-51db27815298.png)

[ロール作成](#ロール作成)で作成したロールを選択し、適用する。

![3-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9cba0e1c-75d1-1e78-5a53-af61e937e09e.png)

### Session Managerを起動し、EC2にログイン

インスタンスをチェックし、`接続`ボタンを押下する。

![4-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d840c808-48ba-8b68-3678-bbe8f27f8d52.png)

`セッションマネージャー`を選択し、`接続`ボタンを押下する。

![4-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/18489e73-d7d8-6a33-2995-b16839096664.png)

別タブでターミナルが表示される。
以下のコマンドで`ec2-user`にスイッチすれば、いつものように扱える。

```
sudo su --login ec2-user
```

![4-3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/2009b7a0-5455-4e37-bcdf-4bb2e3ed89cf.png)

## 監査ログの出力設定手順

### CloudWatch Logsの有効化

AWS Systems Managerでナビゲーションペインの`セッションマネージャー`を選択する。

<img width="1438" alt="5-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/a834f53d-a339-d97c-8aa1-b18c8fabfe35.png">

`設定`タブを選択し、`編集`ボタンを押下する。

<img width="1433" alt="5-2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/042f6e59-ada9-594b-08fa-8eb5b63b5996.png">

`CloudWatch Logs`をチェック。`ログデータを暗号化する`は今回はチェックしない。`CloudWatchのロググループ`は、事前に作成したものを指定する。

<img width="1148" alt="5-3.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/19834bd4-e755-4e2c-6ad2-94e87a6c53c8.png">

### ログ出力の確認

ログの出力を確認するため、もう一度EC2インスタンスにログインし、コマンドを実行してみる。
Session ManagerのTOP画面からログインする。`セッションの開始`を押下する。

![5-4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/1b50177d-a8d0-b4fa-3a96-1f240368a419.png)

インスタンスを選択し、`セッションを開始する`を押下する。

![5-5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4476017c-b4c3-1a7a-7d0d-62324ddbf54b.png)

無事ログインできた。
適当にコマンドを入力した後、右上の`終了`ボタンを押下し、セッションを終了する。

![5-6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d2acb350-015c-5c50-5e50-3ac1f041864a.png)

CloudWatch Logsに移動し、[監査ログの出力設定手順](#監査ログの出力設定手順)で指定したロググループにログストリームが作成されていること、入力したコマンドが出力されていることを確認する。

![5-7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/1c3f2e86-8462-2a7b-7d98-64af7d29fad7.png)

## Session Manager + ssh,scpコマンドでEC2にアクセスする手順

Session ManagerでEC2インスタンスにSSH, SCPで接続するためにはEC2の秘密鍵が必要だ。
これを試す際はEC2インスタンスを再作成して、秘密鍵をダウンロードしておくこと。

### SSMエージェントのインストール

EC2インスタンスにSSMエージェントのバージョン2.3.672.0以降がインストールされている必要がある。
Amazon Linuxにはデフォルトでインストールされているため、インストール作業は不要。
その他の場合は[こちらの目次を参考にインストールしていただきたい](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/ssm-agent.html)

### SSM許可ポリシーの作成し、接続を有効化

本手順はユーザが管理者権限を持つ場合、実施不要。

Session Managerのセッション開始を許可するIAMポリシーを作成する。
IAMコンソールから`ポリシー`を選択し、`ポリシーの作成`ボタンを押下する。

`JSON`タブを選択し、以下のJSONを貼り付け、ポリシーを作成する。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ssm:StartSession",
            "Resource": [
                "arn:aws:ec2:*:*:instance/instance-id",
                "arn:aws:ssm:*:*:document/AWS-StartSSHSession"
            ]
        }
    ]
}
```

<img width="1426" alt="9-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/bcb58edf-6989-3920-1f19-43dbc68ceb89.png">

IAMコンソールに戻り、ユーザを選択し、先ほど作成したポリシーをアタッチする。

![9-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/395d9157-dcfb-8177-3811-670ff6d7ba89.png)

### AWS CLIのインストール

以下にしたがってAWS CLIをインストールする。

https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2.html
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-configure.html

<img width="581" alt="6-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/45aef21b-44c4-a33e-cf06-3c2e0ed160e1.png">

### Session Managerプラグインのインストール

以下にしたがってSession Managerプラグインをインストールする。

https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html

<img width="587" alt="6-2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/517f95ba-f7a1-0736-97bd-a375a6b3f0cd.png">

### SSH設定ファイルを更新

`~/.ssh/config`に以下を追記する。なかったら新規作成。

```shell
# SSH over Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```

### 接続

```shell
ssh -i /path/my-key-pair.pem username@instance-id
```

![7-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d03f92e2-e106-1eff-1aa1-b5507e4ae9f3.png)

無事接続することができた。
scpコマンドも同様に接続することができる。

### Session Managerでのアクセスを無効にする手順

Session Managerのセッション開始を拒否するIAMポリシーを作成する。
IAMコンソールから`ポリシー`を選択し、`ポリシーの作成`ボタンを押下する。

<img width="1440" alt="8-1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d81f1f2b-c90b-82bd-5448-53af70a7d3b6.png">

`JSON`タブを選択し、以下のJSONを貼り付け、ポリシーを作成する。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor1",
            "Effect": "Deny",
            "Action": "ssm:StartSession",
            "Resource": "arn:aws:ssm:*:*:document/AWS-StartSSHSession"
        }
    ]
}
```

<img width="1426" alt="8-2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/367a0469-b4d5-ba41-25c8-4e4f2da125e9.png">

IAMコンソールに戻り、ユーザを選択し、先ほど作成したポリシーをアタッチする。

![8-3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7211ab68-2774-fbb1-4763-8e3024ae7873.png)

SSHコマンドを実行し、EC2への接続が拒否されることを確認する。

![8-4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/96e44206-b356-5225-b566-89a4943aa273.png)

# おわりに

System Managerについて基本を学んだ。より、手軽に安全にEC2にアクセスできるので積極的に活用したい。
余談だが、SSH接続の際、パーミッション問題などでなかなか接続するのに苦労した。
デバッグログ出力させるとトラブルシューティングしやすかった。

```shell
ssh -vT 〜
```

# 参考

https://qiita.com/e__koma/items/009565384efbecb8a46e
https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html
https://dev.classmethod.jp/articles/session-manager-launches-tunneling-support-for-ssh-and-scp/
https://www.bioerrorlog.work/entry/session-manager-ec2-user