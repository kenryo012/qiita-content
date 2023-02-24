<!--
title:   AWS Amplify iOS を試してみる
tags:    AWS,amplify,iOS
id:      f38830b8b6decc0cc936
private: false
-->
[AWS Amplify Androidを試したい方はこちらを参照ください](https://qiita.com/ksato2032/items/008c4605eb9bebd841aa)

# はじめに
AWS Amplify iOS を[Getting Started](https://aws-amplify.github.io/docs/sdk/ios/start)にしたがって試してみる。
以降の各章はGetting Startedに合わせている。

# Prerequisites

## Amplify CLIのインストールと設定

### node.js のインストール

バージョン 12.14.0
https://nodejs.org/en/

### Amplify CLI のインストール
`shell
npm install -g @aws-amplify/cli
`

### Amplify CLI の設定
`shell
amplify configure
`

ブラウザにAWSログイン画面が出るため、ログインしておく。
入力を続けると、IAM ユーザ登録画面が出るため、新規ユーザを登録する。

![IAM登録完了画面.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/877e92e4-26ae-2707-9531-358c954c098e.png)


ターミナルでアクセスキーIDとシークレットアクセスキーを聞かれるため、上記で登録した新規ユーザのを入力する。

![AmplifyConfigFinish.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/08a45f40-ad6b-8439-4f6f-11cb328540f3.png)

## Xcodeバージョン10.2以降をインストール

インストールしておいてください。

## プロジェクトの作成

以下にしたがってプロジェクトを作成する。
[Start Developing iOS Apps (Swift)](https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/BuildABasicUI.html)

簡単に説明する。

Create a new Xcode projectを選択
![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/a3905ab0-5d68-07dd-e4ac-4b73874c4eb6.png)


Single View Appを選択し、Nextを押下
![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/0c3bcf58-637a-7c69-13ea-1f9cdea0f42a.png)

以下のように適当に入力し、Nextを押下
![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/cdae9490-2624-3f05-a98c-f68fc131e9e8.png)

以上でプロジェクトの作成完了。なお、プロジェクト名は`FoodTracker`とした。
![4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/0a86c42b-2bc3-a138-38b4-265448659236.png)

# Step 1: Configure your app

ライブラリ管理ツールである`cocoapads`をインストールする

```shell
cd ./YOUR_PROJECT_FOLDER
sudo gem install cocoapods
pod init
```

上記コマンド実行後、プロジェクトフォルダ直下に`Podfile`が生成されていることを確認する。

![6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/fc1442f6-3f57-ac30-7574-25be6a5f8efc.png)

以下の通り、`Podfile`にAWS Mobile SDKのコアコンポーネントの依存関係を追記する。追記部分に「追加」とコメントしておく。

```:Podfile
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'FoodTracker' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # 追加
  pod 'AWSAppSync', '~> 3.1.0'

  # Pods for FoodTracker

  target 'FoodTrackerTests' do
    inherit! :search_paths
    # Pods for testing
  end

end
```

以下のコマンドで追加した依存関係をインストールする。

```shell
pod install --repo-update
```

コマンド実行後、Xcodeを再起動する必要がある。必ず`./YOUR-PROJECT-NAME.xcworkspace`を使用してプロジェクトを開くこと。
また、以降`./YOUR-PROJECT-NAME.xcworkspace`を使用してプロジェクトを起動する必要がある。

# Step 2: Initialize your project

以下のコマンドでAmplifyプロジェクトの初期化を行う。

```shell
cd ./YOUR_PROJECT_FOLDER
amplify init
```

![amplify-init入力.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8d06d94a-37de-8de8-bab1-0eb1d29ad066.png)


コマンド実行後、プロジェクトフォルダ直下に`awsconfiguration.json`が作成されていることを確認する。

![amplify-init結果.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/8eaa4f93-73da-2c7d-f0c8-f95435b0e308.png)


# Step 3: Add config
`awsconfiguration.json`とは、このアプリケーションが通信をするリージョンおよびサービスのエンドポイントが記載された設定ファイルのこと。
`amplify push`を実行することで自動的に更新される。

フォルダから`awsconfiguration.json`をXCodeの`/YOUR-PROJECT-NAME/YOUR-PROJECT-NAME`にドラックアンドドロップしてコピーする。

![Step3-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/47ee7a84-1800-9f1f-f1da-a486be54bfcd.png)
![Step3-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/210aa66d-aec7-a307-c6d2-efa374244ce2.png)


# Step 4: Add API and Database
以下のコマンドを実行し、「AWS AppSyncへのGraphQL API作成」と「APIと接続をするDynamoDBのプロビジョニング」を行う設定ファイルを作成する。

```shell

amplify add api
```

![add-api結果1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/a8436ca2-b3cd-3edb-56e8-83096ff9e5b0.png)

設定ファイルが`./YOUR-PROJECT-NAME/amplify/backend/api/PROVIDE_API_NAME/`下に作成されていることを確認する。

![add-api結果2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3b4a3679-b21f-d7ad-b03f-0aac4133037d.png)

# Step 5: Push changes
以下のコマンドで[Step4](#step-4-add-api-and-database)の設定をAWSに反映する。

```shell

amplify push
```

![amplify-push入力.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/4a05dfe9-7806-c6e1-cb45-b95fad88f948.png)


APIが追加されていることを確認する。

![amplify-push結果1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3b25df64-58cb-aceb-5ca7-d4913b0306b6.png)


DynamoDBテーブルが作成されていることを確認する。

![amplify-push結果2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/10f5634e-4634-e76e-eee0-5ba7283fab2d.png)
![amplify-push結果3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/d0927869-af26-22a4-eb6f-79757916da01.png)


プロジェクトフォルダ直下に`API.swift`が生成されていることを確認する。

![amplify-push結果4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/bddf4639-a517-dd34-665b-3720c418a038.png)


# Step 6: Add generated code
`API.swift`にはクエリやミューテーションなどのAPI(GraphQL)に関わるコードが記載されている。
フォルダから`API.swift`をXCodeの`/YOUR-PROJECT-NAME/YOUR-PROJECT-NAME`にドラックアンドドロップしてコピーする。

![Step6-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/77bc3b7f-1eee-d6a5-ab83-e04d882b15e1.png)
![Step6-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/b6b53e0b-14cb-af28-54ed-f73da24ff4ab.png)


# Step 7: Integrate into your app
`AppDelegate.swift`にAppSyncクライアントを初期化する処理を追記する。追記した箇所は「追加」とコメントしておいた。

```swift:AppDelegate.swift
// 追加
import AWSAppSync
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    // 追加
    var appSyncClient: AWSAppSyncClient?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // 追加
        do {
            // You can choose the directory in which AppSync stores its persistent cache databases
            let cacheConfiguration = try AWSAppSyncCacheConfiguration()

            // AppSync configuration & client initialization
            let appSyncServiceConfig = try AWSAppSyncServiceConfig()
            let appSyncConfig = try AWSAppSyncClientConfiguration(appSyncServiceConfig: appSyncServiceConfig,
                                                                  cacheConfiguration: cacheConfiguration)
            appSyncClient = try AWSAppSyncClient(appSyncConfig: appSyncConfig)
            print("Initialized appsync client.")
        } catch {
            print("Error initializing appsync client. \(error)")
        }
        // other methods
        return true
    }

    // MARK: UISceneSession Lifecycle

    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        // Called when a new scene session is being created.
        // Use this method to select a configuration to create the new scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
        // Called when the user discards a scene session.
        // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
        // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
    }


}
```

`ViewController.swift`にクエリ、ミューテーション、サブスクライブを実行するfunctionを追記する。追記した箇所は「追加」とコメントした。

```swift:ViewController.swift
// 追加
import AWSAppSync
import UIKit

class ViewController: UIViewController {
    // 追加
    var appSyncClient: AWSAppSyncClient?

    override func viewDidLoad() {
        super.viewDidLoad()
        // 追加
        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        appSyncClient = appDelegate.appSyncClient
        runMutation()
    }

    //　追加
    func runMutation(){
        let mutationInput = CreateTodoInput(name: "Use AppSync", description:"Realtime and Offline")
        appSyncClient?.perform(mutation: CreateTodoMutation(input: mutationInput)) { (result, error) in
            if let error = error as? AWSAppSyncClientError {
                print("Error occurred: \(error.localizedDescription )")
            }
            if let resultError = result?.errors {
                print("Error saving the item on server: \(resultError)")
                return
            }
            print("Mutation complete.")
            self.runQuery()
        }
    }

    // 追加
    func runQuery(){
        appSyncClient?.fetch(query: ListTodosQuery(), cachePolicy: .returnCacheDataAndFetch) {(result, error) in
            if error != nil {
                print(error?.localizedDescription ?? "")
                return
            }
            print("Query complete.")
            result?.data?.listTodos?.items!.forEach { print(($0?.name)! + " " + ($0?.description)!) }
        }
    }

    // 追加
    var discard: Cancellable?
    func subscribe() {
        do {
            discard = try appSyncClient?.subscribe(subscription: OnCreateTodoSubscription(), resultHandler: { (result, transaction, error) in
                if let result = result {
                    print("CreateTodo subscription data:"+result.data!.onCreateTodo!.name+" "
                        + result.data!.onCreateTodo!.description!)
                } else if let error = error {
                    print(error.localizedDescription)
                }
            })
            print("Subscribed to CreateTodo Mutations.")
            } catch {
                print("Error starting subscription.")
            }
    }
}
```

### ミューテーションとクエリの確認
Xcodeから上記コードを実行する。コンソールログからミューテーション、クエリの実行を確認した。

![コンソールログ.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/32cc2bc0-22de-9e80-2ec6-d570e772ecfc.png)

DynamoDBにミューテーションされることを確認した。

![ミューテーション確認.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/ba4838a7-89b6-1425-dcf2-7b201a23e85d.png)


### サブスクライブの確認
`ViewController.swift`の`viewDidLoad()`で`subscribe()`を実行するよう修正し、コードを実行する。
実行後、AppSyncで以下のようにクエリを発行し、ミューテーションイベントを発生させる。

![サブスクライブ確認1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/3388f601-5b54-a6e0-b29f-65ac09c1a559.png)


ミューテーションをトリガーとして`subscribe()`が実行されることを確認した。

![サブスクライブ確認2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/c1c66379-c3ad-b258-eb4b-e6ae42c76838.png)

# 終わりに
チュートリアルにしたがってiOSアプリにAmplifyを設定することができた。
[Amplify iOS版にはAndroidにはない機能](https://dev.classmethod.jp/articles/amplify-ios-identify-celebrities/)があるので今後機会があれば試したい。

# 参考
https://aws-amplify.github.io/docs/sdk/ios/start
https://qiita.com/ksato2032/items/008c4605eb9bebd841aa