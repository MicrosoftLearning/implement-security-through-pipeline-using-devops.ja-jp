---
lab:
  title: 変数とパラメーターを安全に使用するようにパイプラインを構成する
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# 変数とパラメーターを安全に使用するようにパイプラインを構成する

このラボでは、変数とパラメーターを安全に使用するようにパイプラインを構成する方法について学習します。

この演習は約 **30** 分かかります。

## 開始する前に

ラボに従うには、Azure サブスクリプション、Azure DevOps 組織、eShopOnWeb アプリケーションが必要です。

- 手順に [従ってラボ環境](APL2001_M00_Validate_Lab_Environment.md)を検証します。

## 手順

### パラメーターと変数の型を確認する

#### タスク 1: CI パイプラインをインポートして実行する

まず、[eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) という CI パイプラインをインポートします。

1. Azure DevOps ポータル `https://dev.azure.com` に移動し、組織を開きます。

1. eShopOnWeb プロジェクトを開きます。

1. **[パイプライン] > [パイプライン]** に移動します。

1. [新しいパイプライン **] ボタンを選択**します。

1. **[Azure Repos Git (Yaml)]** を選びます。

1. **eShopOnWeb** リポジトリを選びます。

1. **[既存の Azure Pipelines YAML ファイル]** を選びます。

1. **/.ado/eshoponweb-ci.yml** ファイルを選び、 **[続行]** をクリックします。

1. **[実行]** ボタンをクリックしてパイプラインを実行します。

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、名前を変更しましょう。

1. **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをポイントします。 省略記号と **[名前の変更または移動]** オプションをクリックします。

1. eshoponweb-ci-parameters** という**名前を付け、[保存] を選択**します**。

#### タスク 2: YAML パイプラインのパラメーター型を確認する

このタスクでは、パイプラインのパラメーターとパラメーターの型を設定します。

1. Pipelines > Pipelines** に**移動し、eshoponweb-ci-parameters** パイプラインを選択**します。

1. **編集** を選択します。

1. パラメーター配列の先頭に  を追加します。

    ```YAML
    parameters:
    - name: dotNetProjects
      type: string
      default: '**/*.sln'
    - name: testProjects
      type: string
      default: 'tests/UnitTests/*.csproj'

    resources:
      repositories:
      - repository: self
        trigger: none

    stages:
    - stage: Build
      displayName: Build .Net Core Solution
    ```

1. 'Restore'、'Build'、'Test' タスクのハードコーディングされたパスを、先ほど作成したパラメーターに置き換えます。
   - **'Restore' タスクと 'Build' タスクのプロジェクト** '**/*.sln' をプロジェクト : ${{ parameters.dotNetProjects }} に置き換えます。
   - **'Test' タスクのプロジェクト** :'tests/UnitTests/*.csproj' をプロジェクト: ${{ parameters.testProjects }} に置き換えます。

    新しい YAML ファイルは次のようになります。

    ```YAML
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

1. パイプラインを保存して実行すると、正常に実行されます。

    ![パラメーターを含むパイプライン実行のスクリーンショット。](media/pipeline-parameters-run.png)

#### タスク 2: 変数とパラメーターのセキュリティ保護

このタスクでは、変数グループを使用して、パイプラインから変数とパラメーターをセキュリティで保護します。

1. パイプライン > ライブラリに **移動します**。

1. **[+ 変数グループ]** を選択して、新しい変数グループを作成します。 BuildConfigurations のような **名前を付けます**。

1. buildConfiguration** という名前**の変数を追加し、その値を `Release`.

1. 変数グループを保存します。

    ![BuildConfigurations を含む変数グループのスクリーンショット。](media/eshop-variable-group.png)

1. **[パイプラインのアクセス許可]** を選択してから、**+** 符号を選択してパイプラインを追加します。

1. **eshoponweb-ci-parameters** パイプラインを選択して、パイプラインで変数グループを使用できるようにします。

    ![パイプラインのアクセス許可のスクリーンショット](media/pipeline-permissions.png)

1. (省略可能)[セキュリティ **] ボタンをクリックして、変数グループを編集できるように特定のユーザーまたはグループを**設定することもできます。

1. YAML ファイルに戻り、ファイルの上部にあるパラメーターのすぐ下で、次を追加して変数グループを参照します。

    ```YAML
    variables:
      - group: BuildConfigurations
    
    ```

1. 'Build' タスクで、コマンド 'build' を次の行に置き換えて、変数グループのビルド構成を利用します。

    ```YAML
    command: 'build'
    projects: ${{ parameters.dotNetProjects }}
    configuration: $(buildConfiguration)
    
    ```

1. パイプラインを保存し、実行します。 ビルド構成が に設定された状態で正常に実行されます `Release`。 これを確認するには、'ビルド' タスクのログを確認します。

この方法に従うと、変数グループを使用して変数とパラメーターをセキュリティで保護できます。YAML ファイルにハードコーディングする必要はありません。

#### タスク 3: 必須の変数とパラメーターの検証

このタスクでは、パイプラインを実行する前に必須変数を検証します。

1. **[パイプライン] > [パイプライン]** に移動します。

1. eshoponweb-ci-parameters パイプラインを開き **、[編集] を選択します****。**

1. パイプラインを実行する前に必須変数を検証するために、Validate** という名前**の最初のステージとして新しいステージを追加します。

    ```YAML
    - stage: Validate
      displayName: Validate mandatory variables
      jobs:
      - job: ValidateVariables
        pool:
          vmImage: ubuntu-latest
        steps:
        - script: |
            if [ -z "$(buildConfiguration)" ]; then
              echo "Error: buildConfiguration variable is not set"
              exit 1
            fi
          displayName: 'Validate Variables'
    
    ```

    > [!NOTE]
    > このステージでは、buildConfiguration 変数を検証するスクリプトを実行します。 変数が設定されていない場合、スクリプトは失敗し、パイプラインは停止します。

1. **dependsOn を追加してビルド** ステージを**検証**ステージに依存させる: ビルド ステージの下で検証します。

    ```YAML
    - stage: Build
      displayName: Build .Net Core Solution
      dependsOn: Validate
    
    ```

1. パイプラインを保存し、実行します。 buildConfiguration 変数が変数グループに設定されているため、正常に実行されます。

1. 検証をテストするには、変数グループから buildConfiguration 変数を削除するか、変数グループを削除して、パイプラインをもう一度実行します。 次のエラーで失敗します:

    次のエラーが表示されます。

    ```YAML
    Error: buildConfiguration variable is not set
    
    ```

    ![検証が失敗したパイプライン実行のスクリーンショット。](media/pipeline-validation-fail.png)

1. 変数グループと buildConfiguration 変数を変数グループに追加し直し、パイプラインをもう一度実行します。 これは正常に実行される必要があります。

## 確認

このラボでは、変数とパラメーターを安全に使用するようにパイプラインを構成する方法について学習します。
