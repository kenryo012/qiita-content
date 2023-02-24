<!--
title:   AWS CodeStarを試してみる
tags:    AWS,CodeStar,lambda
id:      f750a7551d2cf4e10154
private: false
-->
# はじめに
あるプロジェクトで初めてLambdaの開発を行った。
Eclipseで開発していたが、AWSと連携するわけでもなく、ローカルでの修正のたびに、AWSにログインして手動で上書きするという流れでやっていた。(非常に非効率的)
CodeStarを活用することで上記が解消されるらしいので試してみる。

本資料は大まかに以下を行っています

- CodeStarでのプロジェクトの作成とローカルでの開発準備
- コードを編集し、アプリケーションの自動ビルド、自動デプロイを実行
- ユニットテストを追加して、自動テストを実行

# 前提
ルートユーザ、「AWSCodeStarFullAccess」管理ポリシーを持つユーザで実施してください。

# プロジェクトの作成
- 「プロジェクトを開始する」を押下
![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/bb42d1b2-b834-33fd-0b7f-10572bd3ee69.png)


- CodeStar用のサービスロールを作成することを求められるので、「はい、ロールを作成します」を押下する。
![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/cfc9afec-227c-d532-ee6d-92c185392678.png)


- プロジェクトのテンプレートとして、Python + AWS lambdaを選択する。
![5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/25692cde-ca20-9ba9-187e-529249b88db7.png)


- 名前は「CodeStarTest」に、リポジトリはCodeCommitを使ってみる。
![6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4cdd013a-ebb1-f6e7-fdfa-c47d4b874123.png)


- 「プロジェクトを作成する」を押下
![7.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/63d6ada3-a3d1-3c21-3665-b47f23b68632.png)


- CodeStarのチーム画面などで使用する名前とメールアドレスを入力する
![8-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/47fb7268-b162-667e-1f06-b480f944edd0.png)
![8-3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/0fb9095e-7550-2e55-d619-7d5b0814ab70.png)


- 後で確認できるため選択しなくてもいい。「スキップ」を押下
![9.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9c1437ce-9091-5f4c-b33b-276ebcc7203d.png)


- 以上でプロジェクトの作成が完了。
![13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/f7e5b7be-3081-977a-2deb-04758685c699.png)
アプリケーションのエンドポイントに記載れているURLをクリックするとAPI Gatewayに登録されているAPIが実行される。
![12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/87deb855-0ddb-a2b2-d9af-591c472b44cb.png)



# 作成されたリソースの確認
AWS lambdaプロジェクト作成により、以下のようにAWSリソースが生成される。各サービス画面から確認する。
![CodeStarLambda.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c4dcfded-9097-1fb3-cdad-ad4f921b8ed8.png)


- CodePipeline
![22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/e64b72ce-1358-17d9-95cf-ed4745b0f762.png)


- CodeCommit
![21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/204c8b66-8386-ef4f-90be-51792eb28b8f.png)

ちなみに以下が生成されたコード群
![27.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/66488112-9d3d-7da5-c23d-c458e29021ce.png)


- CodeBuild
![25.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d7708e44-dcfe-ac13-df2c-c46d45111b8c.png)


- CodeDeploy
![26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/17605f75-a6d5-3a90-6acf-c4eeb2bc59d1.png)


- Lambda
![23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/2da6baa7-8bed-0640-d67f-9caec7451aba.png)


- API Gateway
![24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4346da0a-8464-121c-9cd6-d4cf93dd5589.png)


# ローカルツールに接続
Eclipseにつなげて、CodeCommitで管理されているソースをCloneしてみる。

- Eclipseマーケットプレースで「AWS Toolkit for Eclipse」を検索し、インストールし、Eclipseの再起動を行う。
![31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/84111a53-b64f-2ca9-8bf8-b653325eab4a.png)

- 上部にAWSツールバーが表示されているので押下、「AWS CodeStarプロジェクトのインポート」を選択。
![32.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/990c1919-f3e0-d7df-c5ae-877d0b6f3fa7.png)


- 次工程ではGitの認証情報が必要。以下の手順で生成する
  - IAMでCodeStarプロジェクト作成時に使用したユーザの概要画面に行く
  - 認証情報タブを選択し、「AWS CodeCommit の HTTPS Git 認証情報」の「認証情報を生成」ボタンを押下する
![34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4aab5cca-541d-311f-0e84-f171c9314e0e.png)


- CodeStarプロジェクトの選択画面。スクショを参考に上記手順で作成したプロジェクト情報を入力する。
![35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3929b7a1-b9cc-1370-863d-75e2a11558e8.png)


- ブランチ選択。masterしかないので、そのまま「次へ」を押下
![36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/1857085d-4b99-3c7f-9f69-5b5972bab816.png)


- Cloneするパスを入力し、「完了」ボタン押下
![37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/5061ce14-be84-0a89-3111-aaa93b02a48f.png)


- パッケージエクスプローラを確認すると、CodeCommitで管理しているコードをCloneできた。
![38.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7642972d-ca76-cadd-405a-ef1a95a1d5fd.png)


# ローカルツールでコードを編集してみる
- 新規lambda関数の追加
Cloneしたプロジェクトに適当に新規ファイル(hello.py)を追加する。追加したファイルの中身は「参考」のチュートリアルを参考にしてほしい。
![40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9859c515-b282-07c4-8c96-a7f25342e9bc.png)

- template.ymlの編集
Resourcesの子要素として以下を追加

```yml:template.yml
  Hello:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub 'awscodestar-${ProjectId}-lambda-Hello'
      Handler: hello.handler
      Runtime: python3.7
      Role:
        Fn::GetAtt:
        - LambdaExecutionRole
        - Arn
      Events:
        GetEvent:
          Type: Api
          Properties:
            Path: /hello/{name}
            Method: get
```

- 上記変更をコミットし、masterにpushする
![41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9cfcd6c5-60ee-2bfa-b1fa-791be2d3473f.png)
![42.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8c03ba7b-f6eb-5ca9-382a-e68bde19997d.png)


- コミット結果の確認
  - CodePipeline
  CodeCommit,CodeBuild,CodeDeployが順に実行され、全て成功していることを確認
![43.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4934fe67-8614-0024-c479-b0ec81cd1a8a.png)

  - Lambda
  lambda関数「awscodestar-codestartest-lambda-Hello」が追加されていることを確認
![46.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/84ade2d1-3244-143d-5b45-2d227dde579b.png)

  - API Gateway
  GET /hello/{name}が追加されていることを確認
![44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/359cf284-10f4-2a60-c0e9-f2295d85c6d8.png)
  追加したAPIを実行してみる
![47.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9f66f437-6323-fe95-7dbb-c48d97b1dd90.png)


# ユニットテストを追加し、自動テストしてみる

- ローカルツールにCloneしたプロジェクトにhello_test.pyを追加する。内容は以下の通り。
GET /hello/{name}　を実行し、そのレスポンスが期待値通りかを確認する。

```python:hello_test.py

from hello import handler

def test_hello_handler():

  event = {
    'pathParameters': {
      'name': 'testname'
    }
  }

  context = {}

  expected = {
    'body': '{"output": "Hello testname"}',
    'headers': {
      'Content-Type': 'application/json'
    },
    'statusCode': 200
  }

  assert handler(event, context) == expected
```

- CodeBuildのビルド仕様を記載しているbuildspec.ymlを変更する。変更部分は日本語でコメントしている

`yml:buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.8
    commands:
      # (追加)ユニットテストツールのインストール
      - pip install pytest
      # Upgrade AWS CLI to the latest version
      - pip install --upgrade awscli

  pre_build:
    commands:
      # (追加)pytestの実行
      - pytest

      # Discover and run unit tests in the 'tests' directory. For more information, see <https://docs.python.org/3/library/unittest.html#test-discovery>
      - python -m unittest discover tests

  build:
    commands:

      # Use AWS SAM to package the application by using AWS CloudFormation
      - aws cloudformation package --template template.yml --s3-bucket $S3_BUCKET --output-template template-export.yml

      # Do not remove this statement. This command is required for AWS CodeStar projects.
      # Update the AWS Partition, AWS Region, account ID and project ID in the project ARN on template-configuration.json file so AWS CloudFormation can tag project resources.
      - sed -i.bak 's/\$PARTITION\$/'${PARTITION}'/g;s/\$AWS_REGION\$/'${AWS_REGION}'/g;s/\$ACCOUNT_ID\$/'${ACCOUNT_ID}'/g;s/\$PROJECT_ID\$/'${PROJECT_ID}'/g' template-configuration.json

artifacts:
  files:
    - template-export.yml
    - template-configuration.json
`
- 上記変更をpushする

- コミット結果の確認
  - CodeStarのダッシュボード画面を表示し、継続的デプロイメントが成功していることを確認する。続いて、BuildのCodeBuildのリンクを押下する。
![50.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/905d3e9e-e220-c2dd-4e1d-14af4c273b8b.png)

  - ビルド履歴の一番上に先ほど行ったビルド結果が表示されている。「ビルドの実行」列のリンクを押下する。
![51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/cc7c94d1-3a98-83f4-3414-fc2cc5db7bec.png)

  - ビルド結果の詳細が表示される。「ビルドログ」の「ログ全体の表示」を押下する。
![52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/855a25fb-032e-2e38-5cd8-6fcdea03fa72.png)

  - CloudWatchが表示される。「test session starts」で囲まれているログをみるとhello_test.pyで記述したユニットテストが自動実行され、成功していることが確認できる。
![53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/5771ab12-a67c-072d-047f-4c773e26ebad.png)


# 終わりに
CodeStarを使用することで、Eclipseなどのローカルツールと連携してサーバレスアプリケーションを非常に簡単に素早く構築することができた。
不満なのは、CodeStarで選択できるリポジトリがCodeCommitかGithubしかないこと。(Gitlabでできたらな)

# 参考
https://docs.aws.amazon.com/ja_jp/codestar/latest/userguide/sam-tutorial.html