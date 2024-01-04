---
lab:
  title: 複数のテンプレートを使用するようにパイプラインを拡張する
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# 複数のテンプレートを使用するようにパイプラインを拡張する

このラボでは、パイプラインを複数のテンプレートに拡張することの重要性と、Azure DevOps を使ってそれを行う方法を確認します。 このラボでは、マルチステージ パイプラインの作成、変数テンプレートの作成、ジョブ テンプレートの作成、ステージ テンプレートの作成に関する基本的な概念とベスト プラクティスについて説明します。

この演習は約 **30** 分かかります。

## 開始する前に

ラボに従うには、Azure サブスクリプション、Azure DevOps 組織、eShopOnWeb アプリケーションが必要です。

- 手順に [従ってラボ環境](APL2001_M00_Validate_Lab_Environment.md)を検証します。

## 手順

### 演習 1: 複数ステージの YAML パイプラインを作成する

#### タスク 1: YAML パイプラインメインマルチステージを作成する

1. Azure DevOps ポータル `https://dev.azure.com` に移動し、組織を開きます。

1. eShopOnWeb** プロジェクトを**開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. [新しいパイプライン **] ボタンを選択**します。

1. **[Azure Repos Git (Yaml)]** を選びます。

1. **eShopOnWeb** リポジトリを選びます。

1. **[スタート パイプライン]** を選択します。

1. **azure-pipelines.yml** の内容を次のコードに置き換えます。

    ```YAML
    trigger:
    - main

    pool:
      vmImage: 'windows-latest'

    stages:
    - stage: Dev
      jobs:
      - job: Build
        steps:
        - script: echo Build
    - stage: Test
      jobs:
      - job: Test
        steps:
        - script: echo Test
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy

    ```

1. **[保存および実行]** を選択します。 メイン ブランチに直接コミットするか、新しいブランチを作成するかを選択します。 [保存して実行 **] ボタンを選択**します。

   > [!NOTE]
   > 新しいブランチを作成する場合は、変更を メイン ブランチにマージするプル要求を作成する必要があります。

1. 3 つのステージ (開発、テスト、運用) と対応するジョブでパイプラインが実行されていることがわかります。 パイプラインが完了するまで待ってから、[パイプライン **] ページに**戻ります。

    ![3 つのステージと対応するジョブで実行されているパイプラインのスクリーンショット](media/eshoponweb-pipeline-multi-stage.png)

1. **先ほど作成したパイプラインの右側にある [...** (その他のオプション)] を選択し、[名前の変更/移動 **] を選択します**。

1. パイプラインの名前を eShopOnWeb-MultiStage-Main** に**変更し、[保存] を選択**します**。

#### タスク 2: 変数テンプレートを作成する

1. リポジトリ > ファイルに **移動します**。

1. .ado フォルダーを**展開し、[新しいファイル **] を**クリック**します。

1. ファイル**に eshoponweb-variables.yml** という名前を付け、[作成 **] を**クリックします。

1. 次のコードをファイルに追加します。

    ```YAML
    variables:
      resource-group: 'YOUR-RESOURCE-GROUP-NAME'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'YOUR-WEB-APP-NAME'

    ```

    > [!IMPORTANT]
    > 変数の値を環境の値 (リソース グループ、場所、サブスクリプション ID、Azure サービス接続、Web アプリ名) に置き換えます。

1. **[コミット]** を選択し、コメントを入力してから **[コミット]** をもう一度選択します。

#### タスク 3: テンプレートを使用するようにパイプラインを準備する

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eShopOnWeb-MultiStage-Main** パイプラインを開きます。

1. **編集** を選択します。

1. **azure-pipelines.yml** の内容を次のコードに置き換えます。

    ```YAML
    trigger:
    - main
    variables:
    - template: .ado/eshoponweb-variables.yml
    
    stages:
    - stage: Dev
      jobs:
      - template: .ado/eshoponweb-ci.yml
    - stage: Test
      jobs:
      - template: .ado/eshoponweb-cd-webapp-code.yml
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy to Production or Swap

    ```

1. パイプラインを保存します。

1. メイン ブランチに直接コミットするか、新しいブランチを作成するかを選択します。 **[保存]** ボタンを選択します。

   > [!NOTE]
   > 新しいブランチを作成する場合は、変更を メイン ブランチにマージするプル要求を作成する必要があります。

#### タスク 4: CI/CD テンプレートの更新

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eshoponweb-ci** パイプラインを編集します。

1. ジョブ セクションの **上にあるすべてのものを** 削除します。

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    # trigger:
    # - main
    
    resources:
      repositories:
        - repository: self
          trigger: none
    
    stages:
    - stage: Build
      displayName: Build .Net Core Solution

    ```

1. パイプラインを保存します。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eshoponweb-cd-webapp-code** パイプラインを編集します。

1. ジョブ セクションの **上にあるすべてのものを** 削除します。

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    
    # Trigger CD when CI executed successfully
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true

    variables:
      resource-group: 'rg-eshoponweb'
      location: 'southcentralus'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: ''
      azureserviceconnection: 'azure subs'
      webappname: 'eshoponweb-lab'
      # webappname: 'webapp-windows-eshop'
    
    stages:
    - stage: Deploy
      displayName: Deploy to WebApp`

    ```

1. ダウンロード**手順を**次の内容に更新します。

    ```YAML
    - download: current
      artifact: Website
    - download: current
      artifact: Bicep
    ```

1. パイプラインを保存します。

1. (省略可能)運用環境の手順を更新して、アプリケーションを別の環境にデプロイするか、デプロイ スロットをスワップします。

#### タスク 5: パイプラインの実行

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eShopOnWeb-MultiStage-Main** パイプラインを開きます。

1. **[パイプラインの実行]** を選択します。

1. パイプラインが完了するまで待ち、結果をチェックします。

    ![3 つのステージと対応するジョブで実行されているパイプラインのスクリーンショット](media/multi-stage-completed.png)

### 演習 2:Azure ラボ リソースを削除する

1. Azure portal で作成したリソース グループを開き、このラボで作成されたすべてのリソースのリソース グループ**の削除を選択**します。

    ![[リソース グループ削除] ボタンのスクリーンショット。](media/delete-resource-group.png)

    > [!WARNING]
    > 新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

## 確認

このモジュールでは、パイプラインを複数のテンプレートに拡張することの重要性と、Azure DevOps を使ってそれを行う方法を学習しました。 このラボでは、マルチステージ パイプラインの作成、変数テンプレートの作成、ジョブ テンプレートの作成、ステージ テンプレートの作成に関する基本的な概念とベスト プラクティスについて説明します。
