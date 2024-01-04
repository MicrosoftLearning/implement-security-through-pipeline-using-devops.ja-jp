---
lab:
  title: ラボ環境を検証する
  module: 'Module 0: Welcome'
---

# ラボ環境を検証する

ラボの準備として、環境を正しく設定することが重要です。 このページでは、すべての前提条件が満たされていることを確かめるセットアップ プロセスについて説明します。

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)が必要です。

- **Azure サブスクリプションを設定する:** Azure サブスクリプションをまだお持ちでない場合は、このページの手順に従って作成するか、[https://azure.microsoft.com/free](https://azure.microsoft.com/free) にアクセスして無料でサインアップしてください。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。
  
- [Git for Windows ダウンロード ページ](https://gitforwindows.org/)。 このラボでは前提条件の一部としてインストールされます。

- [Visual Studio Code](https://code.visualstudio.com/)。 このラボでは前提条件の一部としてインストールされます。

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)。 セルフホステッド エージェントのマシンに Azure CLI をインストールします。

## Azure DevOps 組織を作成する手順 (これを行う必要があるのは 1 回だけです)

> **注**: **個人の Microsoft アカウント**のセットアップとアクティブな Azure サブスクリプションがそのアカウントにリンクされている場合、手順 3 から開始します。

1. プライベート ブラウザー セッションを使用し、`https://account.microsoft.com` で新しい **個人 Microsoft アカウント (MSA)** を取得します。

1. 同じブラウザー セッションを使用して、`https://azure.microsoft.com/free` で無料の Azure サブスクリプションにサインアップします。

1. ブラウザーを開き、`https://portal.azure.com` の Azure portal に移動し、Azure portal 画面の上部で **Azure DevOps** を検索します。 表示されたページで、**[Azure DevOps 組織]** をクリックします。

1. 次に、**My Azure DevOps Organizations** というラベルの付いたリンクをクリックするか、`https://aex.dev.azure.com` に直接移動します。

1. **[We need a few more details](詳細情報をいくつか入力する必要があります)** ページで、 **[続行]** を選びます。

1. 左側のドロップダウン ボックスで、**Microsoft アカウント**の代わりに **[既定のディレクトリ]** を選びます。

1. プロンプト ( *[We need a few more details](詳細情報をいくつか入力する必要があります)* ) が表示されたら、名前、メールアドレス、場所を入力して、 **[続行]** をクリックします。

1. **[既定のディレクトリ]** を選択した状態で `https://aex.dev.azure.com` に戻り、青いボタン **[新しい組織の作成]** をクリックします。

1. **[続行]** をクリックして*利用規約*に同意します。

1. プロンプト ( *[Almost done](ほぼ完了)* ) が表示されたら、Azure DevOps 組織の名前を既定のままにし (グローバルに一意の名前である必要があります)、一覧から最寄りのホスティング場所を選びます。

1. 新しく作成した組織が **Azure DevOps** で開いたら、左下隅にある **[組織設定]** を選びます。

1. **[組織設定]** 画面で、**[課金情報]** を選びます (この画面を開くには数秒かかります)。

1. **[課金の設定]** を選び、画面の右側で **[Azure サブスクリプション]**、**[保存]** の順に選んでサブスクリプションを組織にリンクします。

1. 画面の上部にリンクされた Azure サブスクリプション ID が表示されたら、**MS Hosted CI/CD** の**有料並列ジョブ**の数を 0 から **1** に変更します。 次に、下部にある **[保存]** を選びます。

1. 新しい設定がバックエンドに反映されるように、**CI/CD 機能を使用する前に少なくとも 3 時間待ちます**。 それができなかった場合、 *"ホストされた並列処理は購入も許可もされていません"* というメッセージが依然として表示されます。

## サンプルの Azure DevOps プロジェクトを作成する手順 (これを行う必要があるのは 1 回だけです)

> **注**: これらの手順を続ける前に、Azure DevOps 組織を作成する手順を完了していることを確認してください。

すべてのラボの手順に従うには、新しい Azure DevOps プロジェクトを設定し、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) アプリケーションに基づくリポジトリを作成する必要があります。

### チーム プロジェクトを作成して構成する

最初に、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ブラウザーを開き、Azure DevOps 組織に移動します。

1. **[新しいプロジェクト]** オプションを選び、次の設定を使用します。
   - 名前: **eShopOnWeb**
   - 可視性: **プライベート**
   - 詳細設定: バージョン コントロール: **Git**
   - 詳細設定: 作業項目プロセス: **スクラム**

1. **［作成］** を選択します

    ![プロジェクトの作成](media/create-project.png)

### eShopOnWeb Git リポジトリをインポートする

次に、eShopOnWeb を Git リポジトリにインポートします。

1. ブラウザーを開き、Azure DevOps 組織に移動します。

1. 前に作成した **eShopOnWeb** プロジェクトを開きます。

1. **[Repos] > [ファイル]** を選び、**[リポジトリをインポートする]**、**[インポート]** の順に選びます。

1. **[Git リポジトリをインポートする]** ウィンドウで、URL `https://github.com/MicrosoftLearning/eShopOnWeb` を貼り付けて、**[インポート]** を選びます。

    ![インポートリポジトリ](media/import-repo.png)

1. リポジトリは次のように編成されています。
    
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
    - **.azure** フォルダーには、Bicep & ARM インフラストラクチャがコード テンプレートとして含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 6 Web サイトが含まれています。

これで、ラボを続けるために必要な前提条件の手順が完了しました。
