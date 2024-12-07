---
lab:
  title: 変数とパラメーターを安全に使用するようにパイプラインを構成する
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# 変数とパラメーターを安全に使用するようにパイプラインを構成する

このラボでは、変数とパラメーターを安全に使用するようにパイプラインを構成する方法について学習します。

これらの演習の所要時間は約 **20** 分です。

## 開始する前に

ラボの演習を行うには、Azure サブスクリプション、Azure DevOps 組織、eShopOnWeb アプリケーションが必要です。

- 手順に従って[ラボ環境を検証](APL2001_M00_Validate_Lab_Environment.md)します。

## 手順

### 演習 1:パラメーターと変数の型を確認する

#### タスク 1: (完了している場合はスキップしてください) CI パイプラインをインポートして実行する

まず、[eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) という CI パイプラインをインポートします。

1. Azure DevOps ポータル (`https://aex.dev.azure.com`) に移動し、自分の組織を開きます。

1. Azure DevOps で **eShopOnWeb** プロジェクトを開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **[パイプラインを作成]** ボタンを選択します。

1. **[Azure Repos Git (Yaml)]** を選びます。

1. **eShopOnWeb** リポジトリを選びます。

1. **[既存の Azure Pipelines YAML ファイル]** を選びます。

1. **/.ado/eshoponweb-ci.yml** ファイルを選び、 **[続行]** をクリックします。

1. **[実行]** ボタンを選んでパイプラインを実行します。

   > **注**: パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを特定しやすいように名前を変更します。

1. **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインを選択します。 省略記号を選択し、**[名前の変更/移動]** オプションを選択します。

1. **eshoponweb-ci** という名前を付けて、**[保存]** を選択します。

#### タスク 2:YAML パイプラインのパラメーターの型を確認する

このタスクでは、パイプラインのパラメーターとパラメーターの型を設定します。

1. **[パイプライン]、[パイプライン]** の順に移動し、**eshoponweb-ci** パイプラインを選択します。

1. **編集**を選択します。

1. YAML ファイルの先頭で、ジョブ セクションの上に次のパラメーターを追加します。

   ```yaml
   parameters:
   - name: dotNetProjects
     type: string
     default: '**/*.sln'
   - name: testProjects
     type: string
     default: 'tests/UnitTests/*.csproj'

   jobs:
   - job: Build
     pool: eShopOnWebSelfPool
     steps:

   ```

1. "Restore"、"Build"、および "Test" タスク内でハードコーディングされているパスを、先ほど作成したパラメーターに置き換えます。

   - **プロジェクトの置き換え**: `**/*.sln` プロジェクト: `Restore` および `Build` タスク内の `${{ parameters.dotNetProjects }}`。
   - **プロジェクトの置き換え**: `tests/UnitTests/*.csproj` プロジェクト: `Test` タスク内の `${{ parameters.testProjects }}`

   YAML ファイルの steps セクションの "Restore"、"Build"、および "Test" タスクは次のようになります。

    ```yaml
    steps:
    - task: DotNetCoreCLI@2
      displayName: Restore
      inputs:
        command: 'restore'
        projects: ${{ parameters.dotNetProjects }}
        feedsToUse: 'select'
    
    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: ${{ parameters.dotNetProjects }}
    
    - task: DotNetCoreCLI@2
      displayName: Test
      inputs:
        command: 'test'
        projects: ${{ parameters.testProjects }}
    
    ```

1. **[検証して保存]** をクリックして変更を保存し、**[保存]** をクリックします。

1. **[パイプライン]、[パイプライン]** の順に移動し、自動的にトリガーされて実行されている **eshoponweb-ci** パイプラインを開きます。

1. パイプラインの実行が正常に完了することを確認します。

   ![パラメーターを含むパイプライン実行のスクリーンショット。](media/pipeline-parameters-run.png)

#### タスク 3:変数とパラメーターを保護する

このタスクでは、パイプラインから変数グループを使用して、変数とパラメーターを保護します。

1. **[パイプライン] > [ライブラリ]** に移動します。

1. **[+ 変数グループ]** ボタンを選択して、`BuildConfigurations` という名前の新しい変数グループを作成します。

1. `buildConfiguration` という名前の変数を追加し、その値を `Release` に設定します。

1. 変数グループを保存します。

   ![BuildConfigurations を含む変数グループのスクリーンショット。](media/eshop-variable-group.png)

1. **[パイプラインのアクセス許可]** ボタンを選択し、新しいパイプラインを追加する **[+]** ボタンを選択します。

1. **eshoponweb-ci** パイプラインを選択して、パイプラインで変数グループを使用できるようにします。

   ![パイプラインのアクセス許可のスクリーンショット。](media/pipeline-permissions.png)

   > **注**: **[セキュリティ]** ボタンをクリックして、変数グループを編集できるように特定のユーザーまたはグループを設定することもできます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eshoponweb-ci** パイプラインを開き、**[編集]** を選択します。

1. yml ファイルの先頭にあるパラメーターのすぐ下で、以下を追加して変数グループを参照します。

   ```yaml
   variables:
     - group: BuildConfigurations
   ```

1. "Build" タスクで、構成パラメーターをタスクに追加して、変数グループのビルド構成を利用します。

    ```yaml
            command: 'build'
            projects: ${{ parameters.dotNetProjects }}
            configuration: $(buildConfiguration)
    ```

1. **[検証して保存]** をクリックして変更を保存し、**[保存]** をクリックします。

1. **eshoponweb-ci** パイプライン実行を開きます。 これは、ビルド構成が "Release" に設定された状態で、正常に実行されます。 これを確認するには、"Build" タスクのログを調べます。

> **注**: ログでビルド構成が "Release" に設定されていない場合は、システム診断を有効にし、パイプラインを再実行して構成値を確認します。

> **注**: このアプローチに従うと、YAML ファイルにハードコーディングしなくても、変数グループを使用して変数とパラメーターを保護することができます。

#### タスク 4:必須の変数とパラメーターを検証する

このタスクでは、パイプラインを実行する前に必須の変数を検証します。

1. **[パイプライン] > [パイプライン]** に移動します。

1. **eshoponweb-ci** パイプラインを開き、**[編集]** を選択します。

1. steps セクションの先頭 (**steps:** 行の後) に、パイプラインの実行前に必須の変数を検証する新しいスクリプト タスクを追加します。

    ```yaml
    - script: |
        IF NOT DEFINED buildConfiguration (
          ECHO Error: buildConfiguration variable is not set
          EXIT /B 1
        )
      displayName: 'Validate Variables'
     ```

    > **注**: これは、変数が設定されているかどうかを確認するための簡単な検証です。 変数が設定されていない場合、スクリプトは失敗し、パイプラインは停止します。 より複雑な検証を追加して、変数の値を確認したり、変数が特定の値に設定されているかどうかを確認したりできます。

1. **[検証して保存]** をクリックして変更を保存し、**[保存]** をクリックします。

1. **eshoponweb-ci** パイプライン実行を開きます。 buildConfiguration 変数が変数グループに設定されているため、正常に実行されます。

1. 検証をテストするには、変数グループから buildConfiguration 変数を削除するか、この変数の名前を変更して、パイプラインをもう一度実行します。 次のエラーにより失敗します。

    ```yaml
    Error: buildConfiguration variable is not set   
    ```

    ![検証が失敗したパイプライン実行のスクリーンショット。](media/pipeline-validation-fail.png)

1. 変数グループに buildConfiguration 変数を追加し直し、パイプラインをもう一度実行します。 これは正常に実行される必要があります。

> [!IMPORTANT]
> 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。

## 確認

このラボでは、変数とパラメーターを安全に使用するようにパイプラインを構成する方法と、必須の変数とパラメーターを検証する方法について学習しました。
