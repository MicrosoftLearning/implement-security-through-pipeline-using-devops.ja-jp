---
lab:
  title: 複数のテンプレートを使用するようにパイプラインを拡張する
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# 複数のテンプレートを使用するようにパイプラインを拡張する

このラボでは、パイプラインを複数のテンプレートに拡張することの重要性と、Azure DevOps を使ってそれを行う方法を確認します。 このラボでは、マルチステージ パイプラインの作成、変数テンプレートの作成、ジョブ テンプレートの作成、ステージ テンプレートの作成に関する基本的な概念とベスト プラクティスについて説明します。

これらの演習の所要時間は約 **20** 分です。

## 開始する前に

ラボの演習を行うには、Azure サブスクリプション、Azure DevOps 組織、eShopOnWeb アプリケーションが必要です。

- 手順に従って[ラボ環境を検証](APL2001_M00_Validate_Lab_Environment.md)します。

## 手順

### 演習 1:マルチステージの YAML パイプラインを作成する

この演習では、Azure DevOps でマルチステージの YAML パイプラインを作成します。

#### タスク 1:マルチステージのメイン YAML パイプラインを作成する

1. Azure DevOps ポータル (`https://aex.dev.azure.com`) に移動し、自分の組織を開きます。

1. **eShopOnWeb** プロジェクトを開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **[新しいパイプライン]** ボタンをクリックします。

1. **[Azure Repos Git (Yaml)]** を選びます。

1. **eShopOnWeb** リポジトリを選びます。

1. **[スタート パイプライン]** を選択します。

1. **azure-pipelines.yml** ファイルの内容を次のコードに置き換えます。

   ```yaml
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

1. **[保存および実行]** を選択します。 メイン ブランチに直接コミットすることを選び、**[保存および実行]** を選択します。

1. 3 つのステージ (Dev、Test、Production) で実行されているパイプラインと、対応するジョブが表示されます。 パイプラインが完了するまで待ってから、**[パイプライン]** ページに戻ります。

   ![3 つのステージで実行されているパイプラインと、対応するジョブのスクリーンショット](media/eshoponweb-pipeline-multi-stage.png)

1. **[...]** (その他のオプション) を選択し (先ほど作成したパイプラインの右側)、**[名前の変更/移動]** を選択します。

1. パイプラインの名前を **eShopOnWeb-MultiStage-Main** に変更し、**[保存]** を選択します。

#### タスク 2:変数テンプレートを作成する

1. **[リポジトリ] > [ファイル]** に移動します。

1. **.ado** フォルダーを展開し、**[新しいファイル]** をクリックします。

1. ファイルに **eshoponweb-variables.yml** という名前を付け、**[作成]** をクリックします。

1. 次のコードをファイルに追加します。

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'centralus'
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. 変数の値を実際の環境の値に置き換えます。

   - **YOUR-RESOURCE-GROUP-NAME** を、このラボで使用するリソース グループの名前 (たとえば、**rg-eshoponweb-secure**) に置き換えます。
   - **location** 変数の値を、リソースをデプロイする Azure リージョンの名前 (たとえば、**centralus**) に設定します。
   - **YOUR-SUBSCRIPTION-ID** を、お使いの Azure サブスクリプション ID に置き換えます。
   - **YOUR-AZURE-SERVICE-CONNECTION-NAME** を、**azure subs** に置き換えます
   - **YOUR-WEB-APP-NAME** を、デプロイする Web アプリのグローバル一意識別子 (たとえば、文字列 **eshoponweb-lab-multi-123456** の後にランダムな 6 桁の数字) に置き換えます。  

1. **[コミット]** を選択し、コミットの [コメント] テキスト ボックスに「`[skip ci]`」と入力し、**[コミット]** を選択します。

   > **注**: `[skip ci]` コメントをコミットに追加すると、パイプラインが自動実行されなくなります。この時点では、パイプラインは既定でリポジトリへの変更が発生するたびに実行されています。 

#### タスク 3:テンプレートを使用するようにパイプラインを準備する

1. Azure DevOps ポータルの **eShopOnWeb** プロジェクト ページで、**[リポジトリ]** に移動します。

1. リポジトリのルート ディレクトリで、**azure-pipelines.yml** を選択します。これには **eShopOnWeb-MultiStage-Main** パイプラインの定義が含まれています。

1. **[編集]** ボタンをクリックします。

1. **azure-pipelines.yml** ファイルの内容を次のコードに置き換えます。

   ```yaml
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

1. **[コミット]** を選択し、コミットの [コメント] テキスト ボックスに「`[skip ci]`」と入力し、**[コミット]** を選択します。

#### タスク 4:CI/CD テンプレートを更新する

1. **eShopOnWeb** プロジェクトの **[リポジトリ]** で、**.ado** ディレクトリを選択し、**eshoponweb-ci.yml** ファイルを選択します。

1. **[編集]** ボタンをクリックします。

1. **jobs** セクションの上にあるものすべてを削除します。

   ```yaml
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

1. **[コミット]** を選択し、コミットの [コメント] テキスト ボックスに「`[skip ci]`」と入力し、**[コミット]** を選択します。

1. **eShopOnWeb** プロジェクトの **[リポジトリ]** で、**.ado** ディレクトリを選択し、**eshoponweb-cd-webapp-code.yml** ファイルを選択します。

1. **[編集]** ボタンをクリックします。

1. **jobs** セクションの上にあるものすべてを削除します。

   ```yaml
    # NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml") #
    # Trigger CD when CI executed successfully
    
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true
    
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity # name of the project and repository
    
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity # name of the template and repository
    
    stages:
      - stage: Test
        displayName: Testing WebApp
        jobs:
          - deployment: Test
            pool: eShopOnWebSelfPool
            environment: Test
            strategy:
              runOnce:
                deploy:
                  steps:
                    - script: echo Hello world! Testing environments!
    
      - stage: Deploy
        displayName: Deploy to WebApp
   ```

1. **#download artifacts** ステップの既存の内容を次のように置き換えます。

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. **[コミット]** を選択し、コミットの [コメント] テキスト ボックスに「`[skip ci]`」と入力し、**[コミット]** を選択します。

#### タスク 5:メイン パイプラインを実行する

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eShopOnWeb-MultiStage-Main** パイプラインを開きます。

1. **[パイプラインの実行]** を選択します。

   > **注**: この実行を続ける前に、リソースにアクセスするためのアクセス許可がパイプラインに必要であるというメッセージが表示された場合は、**[表示]**、**[許可]**、さらにもう一度 **[許可]** を選択して、パイプラインの実行を許可します。

   > **注**: デプロイ ステージのジョブが失敗した場合は、パイプライン実行ページに移動し、[失敗したジョブの再実行] を選択します。

1. パイプラインが完了するまで待ってから、結果を確認します。

   ![3 つのステージで実行されているパイプラインと、対応するジョブのスクリーンショット](media/multi-stage-completed.png)

> [!IMPORTANT]
> 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。

## 確認

このラボでは、Azure DevOps を使用してパイプラインを複数のテンプレートに拡張する方法を学びました。 このラボでは、マルチステージ パイプラインの作成、変数テンプレートの作成、ジョブ テンプレートの作成、ステージ テンプレートの作成に関する基本的な概念とベスト プラクティスについて説明しました。
