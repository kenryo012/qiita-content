<!--
title:   AWS Amplify Android を試してみる(Mac) 
tags:    AWS,Android,Mac,amplify
id:      008c4605eb9bebd841aa
private: false
-->
[AWS Amplify iOSを試したい方はこちらを参照ください](https://qiita.com/ksato2032/items/f38830b8b6decc0cc936)

# はじめに
AWS Amplify Android を[Getting Started](https://aws-amplify.github.io/docs/android/start)にしたがってMacで試してみる。
以降の各章はGetting Startedに合わせている。

# Prerequisites

以下の通り、Getting Started に記載されている手順は Mac でしか動作しない。Windows の場合、下の説明にあるリンク先の手順を実施する必要がある。
https://aws-amplify.github.io/docs/android/start#prerequisites

```
These steps currently only work on Mac. If you have a Windows machine, follow the steps on one of our categories such as API here.
```

### node.js のインストール

バージョン 12.14.0をインストールした。
![nodeインストール.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/fd552b4a-5475-627a-2d8f-2dcd38fdf605.png)

https://nodejs.org/en/

### Android Studio のインストール

以下からインストール。バージョン 3.1 以上である必要がある。
現在最新のやつをインストールすれば問題なし。

https://developer.android.com/studio/index.html#downloads

### プロジェクトの作成

以下にしたがってプロジェクトを作成する。
https://developer.android.com/training/basics/firstapp/creating-project

Minimum API levelは15(Ice Cream Sandwich)以上を選択する必要がある。

![プロジェクト作成1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8ab5787d-2470-19ec-a63c-17a3441a5299.png)

### Android SDK のインストール

上で選択したAPI levelのSDKをイントールする。

Android Studio -> Preference をクリック
![SDKインストール1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/638530ec-37c9-7a68-ddc7-af8295490f12.png)


Appearance & Behavior -> System Settings -> Android SDK をクリックし、必要なSDKがインストールされていなければインストールする。
![SDKインストール2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8ea2e743-1a2f-e839-163f-598fd428cf52.png)

### Amplify CLI のインストール
`
npm install -g @aws-amplify/cli
`
# Step 1: Configure your app


### プロジェクト用build.gradleの修正

`build.gradle(Project:My First App)`を開き、いろいろ追加する。追加した箇所にコメントをした。

```gradle:build.gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        // mavenCentral()を追加
        mavenCentral()
        google()
        jcenter()

    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.3'

        // amplify-tools-gradle-pluginを追加
        classpath 'com.amplifyframework:amplify-tools-gradle-plugin:0.2.0'
    }
}

// amplifytoolsを追加
apply plugin: 'com.amplifyframework.amplifytools'

allprojects {
    repositories {
        google()
        jcenter()

    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

### アプリ用のbuild.gradleの修正

`build.gradle(Module:App)`を開き、いろいろ追加する。追加した箇所にコメントをした。

```gradle:build.gradle
apply plugin: 'com.android.application'

android {
    // compileOptionsを追加
    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }
    compileSdkVersion 28
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "com.example.myfirstapp"
        minSdkVersion 15
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.0.2'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.1'

    // amplifyframeworkを追加
    implementation 'com.amplifyframework:core:0.9.0'
    implementation 'com.amplifyframework:aws-api:0.9.0'
}
```
### Make Projectの実行

Build メニュー -> Make Project をクリック

![MakeProject結果1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ce5fb2a7-67fd-e39e-82a0-81238ce8843e.png)


Make Project に成功すると Gradle Task として`modelgen`と`amplifyPush`が追加されます。

![MakeProject結果2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d05ee44c-9e5d-7381-dc2d-cfeb78532366.png)

# Step 2: Generate your Model files

`amplify/backend/api/amplifyDatasource/schema.graphql`を確認する

```graphql:amplify/backend/api/amplifyDatasource/schema.graphql
type Task @model {
  id: ID!
  title: String!
  description: String
  status: String
}
type Note @model {
  id: ID!
  content: String!
}
```

Gradle Task `modelgen`を実行する

![modelgen実行結果1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/a9ccf02a-a365-4dba-6aa0-f3ebe295f7b4.png)

`app/src/main/java/com/amplifyframework.datastore.generated.model`の下に `schema.graphql`に記述したmodelクラスが生成されたことを確認する。

![modelgen実行結果2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/13a4293b-6122-20ef-96a0-12474c0e9ad0.png)

# Step 3: Add API and Database

### amplify configureの実行
コマンドプロンプトでプロジェクトのルートディレクトリに移動し、以下を実行する。

```
amplify configure
```

AWSログイン画面が出るため、ログインしておく。
コマンドプロンプトに戻り、リージョンやら新規ユーザ名やらを入力すると、ブラウザにIAMユーザ登録画面が出るため、新規ユーザを登録する。

![IAM登録完了画面.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4598c0de-79fb-aaeb-6a99-4b88880df4d6.png)

新規ユーザ登録後、アクセスキーIDとシークレットアクセスキーが発行される。
コマンドプロンプトでそれを聞かれるため、入力する。

![AmplifyConfigFinish.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/cf4dbade-3402-ec5b-9bc5-8a2f2e80160e.png)

### amplifyPushの実行

AndroidStudioに戻り、Gradle Task `amplifyPush`を実行する

![amplifypush実行結果.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7a665ffc-c4fc-9cba-16e3-aafa21b08170.png)

Task完了後、以下の2ファイルが生成されていることを確認する

`src/main/res/raw/amplifyconfiguration.json`
各サービスにアクセスするための情報を記載したファイル
![amplifyconfiguration.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/051aa2b8-25ed-03d9-8567-673a13d994ae.png)
`src/main/res/raw/awsconfiguration.json`
通信対象の全てのリージョンとエンドポイントを記載したファイル
![awsconfiguration.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/73522841-b0d1-3560-87e6-928975d77faf.png)

# Step 4: Integrate into your app

Amplifyの準備ができたので、Androidアプリに組み込んでいく、`com.example.myfirstapp.MainActivity#onCreate`を以下の通り修正する

```java:MainActivity.java
package com.example.myfirstapp;

import android.os.Bundle;
import android.util.Log;

import androidx.appcompat.app.AppCompatActivity;

import com.amplifyframework.AmplifyException;
import com.amplifyframework.api.aws.AWSApiPlugin;
import com.amplifyframework.api.graphql.GraphQLResponse;
import com.amplifyframework.api.graphql.MutationType;
import com.amplifyframework.api.graphql.SubscriptionType;
import com.amplifyframework.core.Amplify;
import com.amplifyframework.core.ResultListener;
import com.amplifyframework.core.StreamListener;
import com.amplifyframework.datastore.generated.model.Task;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        try {
            // Amplifyの初期化処理
            Amplify.addPlugin(new AWSApiPlugin());
            Amplify.configure(getApplicationContext());
            Log.i("AmplifyGetStarted", "Amplify is all setup and ready to go!");


            // テスト用Taskを生成し、mutate APIを実行。DynamoDBに登録
            Task task = Task.builder().title("My first task").description("Get started with Amplify").build();

            Amplify.API.mutate(task, MutationType.CREATE, new ResultListener<GraphQLResponse<Task>>() {
                @Override
                public void onResult(GraphQLResponse<Task> taskGraphQLResponse) {
                    Log.i("AmplifyGetStarted", "Added task with id: " + taskGraphQLResponse.getData().getId());
                }

                @Override
                public void onError(Throwable throwable) {
                    Log.e("AmplifyGetStarted", throwable.toString());
                }
            });

            // 登録したTaskの取得。Dynamoはデフォルトでは「結果的に整合性のある読み込み」のため、
            // 上記で登録したばかりのデータは取得できない場合がある。
            Amplify.API.query(Task.class, new ResultListener<GraphQLResponse<Iterable<Task>>>() {
                @Override
                public void onResult(GraphQLResponse<Iterable<Task>> iterableGraphQLResponse) {
                    for(Task task : iterableGraphQLResponse.getData()) {
                        Log.i("AmplifyGetStarted", "Task : " + task.getTitle());
                    }
                }

                @Override
                public void onError(Throwable throwable) {
                    Log.e("AmplifyGetStarted", throwable.toString());
                }
            });

            // Taskが登録された時に実行するメソッドをsubscribe
            Amplify.API.subscribe(
                    Task.class,
                    SubscriptionType.ON_CREATE,
                    new StreamListener<GraphQLResponse<Task>>() {
                        @Override
                        public void onNext(GraphQLResponse<Task> taskGraphQLResponse) {
                            Log.i("AmplifyGetStarted", "Subscription detected a create: " +
                                    taskGraphQLResponse.getData().getTitle());
                        }

                        @Override
                        public void onComplete() {
                            // Whatever you want it to do on completing
                        }

                        @Override
                        public void onError(Throwable throwable) {
                            Log.e("AmplifyGetStarted", throwable.toString());
                        }
                    }
            );

        } catch (AmplifyException exception) {
            Log.e("AmplifyGetStarted", exception.getMessage());
        }
        setContentView(R.layout.activity_main);
    }
}
```

### AWS AppSync

以下のコマンドを実行し、`GraphQL`を選択すると、ブラウザにAWS AppSyncが表示される

```
amplify console api
```

この画面からこれまで生成したデータソースを確認したり、GraphQLクエリを発行して、APIのテストができる

![データソース.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c2385cc7-1a36-bf6b-44ac-17dd0c1f63de.png)

クエリ画面で作成されたAPIの仕様が確認できる。

![QueryDocuments.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3172f6a9-410e-f10a-7d3d-25ab53dcf845.png)

Java に記載されているソースが実行されると、GraphQL クエリが発行されるイメージ

# アプリを起動してみる

作成したAndroidアプリをシミュレーターやら実機で起動してみる。

### mutateの確認
`Amplify.API.mutate`でDynamoDBにtitleが`My first task`のレコードが登録されたことを確認できた。

![Mutate結果.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/558d14d9-ed21-2824-3828-fc4743eb4fca.png)

### queryの確認
登録後、`Amplify.API.query`で上記レコードを取得したいが、DynamoDBはデフォルトでは`結果的に整合性のある読み込み`のため、取得できない場合がある。
アプリを2回起動して、1回目のレコードを取得できるか確認する。

- 1回目
![1回目.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ffde0499-6818-588a-b375-89d524c9dc34.png)

- 2回目
![2回目.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7ed85a66-94b8-d683-b521-f3837115cf9b.png)

ログから1回目に登録したレコードの取得を確認できた。

### subscribeの確認
`Amplify.API.subscribe`で指定したテーブルへの登録やレコードの削除などをトリガーにして、リスナーのメソッドを実行することができる。
試しに、アプリを実行した状態で、AWS AppSyncからTaskを登録してみる。
![mutation.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/7b6a2bdf-76f1-9677-f455-b297acf6f1c7.png)

Taskの登録は成功しているが、`onError()`が実行され、エラーログが出力されてしまった。
問題なく`onNext()`のログが表示された方がいたら教えていただきたい。
![エラー.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ae3512c8-9c99-222a-df66-50343515b95d.png)

# 終わりに
最後の方、少し消化不良となってしまったが、チュートリアルにしたがって Amplify SDKを使用してAndroidアプリを作成することができた。
次はiosアプリの方を試してみたい。iosの場合、Prediction（予測機能）が使えるようだ。

# 参考
https://aws-amplify.github.io/docs/android/start#step-1-configure-your-app
https://dev.classmethod.jp/cloud/aws/amplify-ios-identify-celebrities/
https://qiita.com/uenohara/items/44d2334c597dc631bc60