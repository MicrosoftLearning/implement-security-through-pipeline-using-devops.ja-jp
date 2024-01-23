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

#### タスク 1:マルチステージのメイン YAML パイプラインを作成する

1. Azure DevOps ポータル (`https://dev.azure.com`) に移動し、自分の組織を開きます。

1. **eShopOnWeb** プロジェクトを開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **[Create Pipeline]** を選択します。

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
     location: 'southcentralus' #name of the Azure region you want to deploy your resources
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. 変数の値を実際の環境の値に置き換えます。

   - **YOUR-RESOURCE-GROUP-NAME** を、このラボで使用するリソース グループの名前に置き換えます (たとえば、**rg-eshoponweb-multi**)。
   - **location** 変数の値を、リソースをデプロイする Azure リージョンの名前に設定します (たとえば、**southcentralus**)。
   - **YOUR-SUBSCRIPTION-ID** を、使用している Azure サブスクリプション ID に置き換えます。
   - **YOUR-AZURE-SERVICE-CONNECTION-NAME** を、**azure subs** に置き換えます
   - **YOUR-WEB-APP-NAME** を、デプロイする Web アプリのグローバルに一意な名前に置き換えます (たとえば、文字列 **eshoponweb-lab-multi-** に続けてランダムな 6 桁の数字)。  

1. **[コミット]** を選択し、コミットの [コメント] テキスト ボックスに「`[skip ci]`」と入力し、**[コミット]** を選択します。

   > [!NOTE]
   > `[skip ci]` コメントをコミットに追加すると、パイプラインが自動実行されなくなります。この時点では、パイプラインは既定でリポジトリに対する変更ごとに実行されています。 

#### タスク 3:テンプレートを使用するようにパイプラインを準備する

1. Azure DevOps ポータルの **eShopOnWeb** プロジェクト ページで、**[リポジトリ]** に移動します。 

1. リポジトリのルート ディレクトリで、**azure-pipelines.yml** を選択します。これには **eShopOnWeb-MultiStage-Main** パイプラインの定義が含まれています。

1. **編集**を選択します。

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

1. **jobs** セクションの上にあるものすべてを削除します。

   ```yaml
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

1. パイプラインが **Test** 環境の **Deploy** ステージに到達したら、パイプラインを開き、"この実行でテストを続行する前に、リソースにアクセスするためのアクセス許可がこのパイプラインに必要です" というメッセージを確認します。 **[表示]** を選び、**[許可]** を選んでパイプラインの実行を許可します。

   > [!NOTE]
   > Deploy ステージのジョブが失敗した場合は、パイプラインの実行ページに移動し、**[失敗したジョブの再実行]** を選択します。

1. パイプラインが **Production** 環境の **Deploy** ステージに到達したら、パイプラインを開き、"この実行で運用を続行する前に、リソースにアクセスするためのアクセス許可がこのパイプラインに必要です" というメッセージを確認します。 **[表示]** を選び、**[許可]** を選んでパイプラインの実行を許可します。

1. パイプラインが完了するまで待ってから、結果を確認します。

   ![3 つのステージで実行されているパイプラインと、対応するジョブのスクリーンショット](media/multi-stage-completed.png)

### 演習 2:Azure と Azure DevOps リソースのクリーンアップを実行する

この演習では、このラボで作成した Azure と Azure DevOps のリソースを削除します。

#### タスク 1:Azure リソースを削除する

1. Azure portal で、デプロイされたリソースを含むリソース グループ **rg-eshoponweb-multi** に移動し、**[リソース グループの削除]** を選んで、このラボで作成されたすべてのリソースを削除します。

#### タスク 2:Azure DevOps パイプラインを削除する

1. Azure DevOps ポータル (`https://dev.azure.com`) に移動し、自分の組織を開きます。

1. **eShopOnWeb** プロジェクトを開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **[パイプライン] > [パイプライン]** に移動し、既存のパイプラインを削除します。

#### タスク 3:Azure DevOps リポジトリを再作成する

1. Azure DevOps ポータルの **eShopOnWeb** プロジェクトで、左下隅にある **[プロジェクトの設定]** を選びます。

1. 左側の **[プロジェクトの設定]** 縦型メニューの **[リポジトリ]** セクションで、**[リポジトリ]** を選びます。

1. **[すべてのリポジトリ]** ペインで、**eShopOnWeb** リポジトリ エントリの右端にマウス ポインターを置き、**[その他のオプション]** の [...] アイコンが表示されたら、それを選びます。**[その他のオプション]** メニューで **[名前の変更]** を選びます。  

1. **[eShopOnWeb リポジトリの名前を変更する]** ウィンドウの **[リポジトリ名]** テキスト ボックスに「**eShopOnWeb_old**」と入力し、**[名前の変更]** を選びます。

1. **[すべてのリポジトリ]** ペインに戻り、**[+ 作成]** を選びます。

1. **[リポジトリの作成]** ペインの **[リポジトリ名]** テキスト ボックスに「**eShopOnWeb**」と入力し、**[Readme の追加]** チェックボックスをオフにして、**[作成]** を選びます。

1. **[すべてのリポジトリ]** ペインに戻り、**eShopOnWeb_old** リポジトリ エントリの右端にマウス ポインターを置き、**[その他のオプション]** の [...] アイコンが表示されたら、それを選びます。**[その他のオプション]** メニューで **[削除]** を選びます。  

1. **[eShopOnWeb_old リポジトリの削除]** ウィンドウで「**eShopOnWeb_old**」と入力し、**[削除]** を選びます。

1. Azure DevOps ポータルの左側のナビゲーション メニューで **[リポジトリ]** を選びます。

1. **[eShopOnWeb は空です。]** をクリックします。 ペインで **[リポジトリのインポート]** を選びます。

1. **[Git リポジトリをインポートする]** ウィンドウで、URL `https://github.com/MicrosoftLearning/eShopOnWeb` を貼り付けて、**[インポート]** を選びます。

## 確認

このラボでは、Azure DevOps を使用してパイプラインを複数のテンプレートに拡張する方法を学びました。 このラボでは、マルチステージ パイプラインの作成、変数テンプレートの作成、ジョブ テンプレートの作成、ステージ テンプレートの作成に関する基本的な概念とベスト プラクティスについて説明しました。