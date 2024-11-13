<img src="./images/ms-cloud-workshop.png" />

Retrieval Augmented Generation (RAG) pattern for Azure AI Search  
Dec 2024

<br />

### Contents

- [Exercise 5: GitHub Actions から Azure への接続](#exercise-5-github-actions-から-azure-への接続)

  - [Task 1: Microsoft Entra ID へのアプリケーションの登録](#task-1-microsoft-entra-id-へのアプリケーションの登録)

  - [Task 2: フェデレーション資格情報の追加](#task-2-フェデレーション資格情報の追加)

  - [Task 3: ロールの割り当て](#task-3-ロールの割り当て)

  - [Task 4: GitHub シークレットの作成](#task-4-github-シークレットの作成)

  - [Task 5: OpenID Connect 認証を使用した Azure へのログイン](#task-5-openid-connect-認証を使用した-azure-へのログイン)

- [Exercise 6: GitHub Actions を用いた Azure リソースの展開](#exercise-6-github-actions-を用いた-azure-リソースの展開)

  - [Task 1: Bicep デプロイ用のパラメーター ファイルの作成](#task-1-bicep-デプロイ用のパラメーター-ファイルの作成)

  - [Task 2: ワークフローの実行](#task-2-ワークフローの実行)

  - [Task 3: Kay Vault へのアクセス権の付与](#task-3-kay-vault-へのアクセス権の付与)

- [Exercise 7: GitHub Actions を用いた API の展開](#exercise-7-github-actions-を用いた-api-の展開)

  - [Task 1: マネージド ID の作成](#task-1-マネージド-id-の作成)

  - [Task 2: ロールの割り当て](#task-2-ロールの割り当て)

  - [Task 3: コンテナー イメージのビルド、プッシュ](#task-3-コンテナー-イメージのビルドプッシュ)

  - [Task 4: シークレットの登録](#task-4-シークレットの登録)

  - [Task 5: Azure Container Apps へのイメージの展開](#task-5-azure-container-apps-へのイメージの展開)

  - [Task 6: API の動作確認](#task-6-api-の動作確認)

<br />

## Exercise 5: GitHub Actions から Azure への接続

<img src="./images/Ex5.png" />

<br />

### Task 1: Microsoft Entra ID へのアプリケーションの登録

- [Azure Portal](https://portal.azure.com/) から Microsoft Entra ID の **アプリの登録** を選択

- **＋ 新規登録** をクリック

  <img src="./images/create-microsoft-entra-application-02.png" />

- **アプリケーションの登録** で **名前**、**サポートされているアカウントの種類** を指定

  - **名前**: GitHubActions (任意)

  - **サポートされているアカウントの種類**: この組織ディレクトリのみに含まれるアカウント

    <img src="./images/create-microsoft-entra-application-03.png" />

- **登録** をクリックし、アプリケーションを登録

<br />

### Task 2: フェデレーション資格情報の追加

- **証明書とシークレット** を選択、**フェデレーション資格情報** > **＋ 資格情報の追加** をクリック

  <img src="./images/configure-federated-identity-02.png" />

- **資格情報の追加**

  - **フェデレーション資格情報のシナリオ** で **Azure リソースをデプロイする GitHub Actions** を選択

    <img src="./images/configure-federated-identity-03.png" />

  - 接続先の GitHub アカウント、資格情報の詳細を入力

    - **GitHub アカウントを接続する**

      - **組織**: GitHub アカウント

      - **リポジトリ**: RAG-on-Azure-Hands-on-Lab

      - **エンティティ型**: ブランチ

      - **サブジェクト識別子**: main

    - **資格情報の詳細**

      - **名前**: 任意

        <img src="./images/configure-federated-identity-04.png" />

- **追加** をクリックして、資格情報を生成

  <img src="./images/configure-federated-identity-05.png" />

<br />

### Task 3: ロールの割り当て

- リソース グループの **アクセス制御 (IAM)** を選択し、**追加** > **ロール割り当ての追加** をクリック

  <img src="./images/role-assignment-01.png" />

- **特権管理者ロール** > **共同作成者** を選択

  <img src="./images/role-assignment-02.png" />

- **アクセスの割当先** で **ユーザー、グループ、またはサービス プリンシパル** を選択、**＋ メンバーを選択する** をクリック

  <img src="./images/role-assignment-03.png" />

- 登録したアプリケーション名を入力

- 検出されたサービス プリンシパルを選択し **選択** をクリック

  <img src="./images/role-assignment-04.png" />

- **レビューと割り当て** をクリックし、ロールを割り当て

  <img src="./images/role-assignment-05.png" />

<br />

### Task 4: GitHub シークレットの作成

- Web ブラウザで GitHub アカウントのリポジトリを表示

- **Settings** をクリック

  <img src="./images/create-github-secrets-03.png" />

- **Secrets and variables** > **Actions** を選択、**New repository secret** をクリック

  <img src="./images/create-github-secrets-04.png" />

- **Name**, **Secret** を入力し **Add secret** をクリック

  <img src="./images/create-github-secrets-05.png" />

  - 登録するシークレット

    |Name|Secret|
    |---|---|
    |AZURE_CLIENT_ID|アプリケーション (クライアント) ID|
    |AZURE_TENANT_ID|ディレクトリ (テナント) ID|
    |AZURE_SUBSCRIPTION_ID|サブスクリプション ID|

  - アプリケーション (クライアント) ID, ディレクトリ (テナント) ID は、登録したアプリケーションの概要から取得

    <img src="./images/create-github-secrets-01.png" />

  - サブスクリプション ID は、サブスクリプションの概要から取得

    <img src="./images/create-github-secrets-02.png" />

<br />

### Task 5: OpenID Connect 認証を使用した Azure ログイン アクションの使用

- Visual Studio Code の Explorer から **.github** > **workflows** 内のワークフロー ファイルを選択

- Azure ログイン アクションの確認

  <img src="./images/create-github-secrets-07.png" />

  > GitHub Actions と Microsoft Entra ID 間でフェデレーション ID を用いた信頼関係を設定  
  > OpenID Connect の認証でシークレットを使わないセキュアな構成が可

<br />

### 参考情報

- [GitHub Actions を使用して Azure に接続する](https://learn.microsoft.com/ja-jp/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows)

<br />

## Exercise 6: GitHub Actions を用いた Azure リソースの展開

<img src="./images/Ex6.png" />

<br />

### Task 1: Bicep デプロイ用のパラメーター ファイルの作成

- Web ブラウザで GitHub アカウントのリポジトリを表示

- **Code** の **bicep** > **parameters** > **keyvault.bicepparam** を選択

- **Edit this file** をクリック

  <img src="./images/edit-bicep-parameters-01.png" />

- **keyvaultName**, **location** を指定、**Confirm changes...** をクリックし変更を確定

  - **keyvaultName**: Key Vault リソース名

  - **location**: 展開先の Azure リージョン

    <img src="./images/edit-bicep-parameters-02.png" />

<br />

### Task 2: ワークフローの実行

- **Actions** を選択, **All workflows** > **Deploy Key Vault** を選択

  <img src="./images/run-workflow-kv-01.png" />

- **Run workflow** をクリック、展開先のリソース グループ名を入力し **Run workflow** をクリック

  <img src="./images/run-workflow-kv-02.png" />

- 進行中のワークフロー (**Deploy Key Vault**) をクリックし、実行の概要を表示

  <img src="./images/deploy-key-vault-01.png" />

- ジョブ (**deploy-key-vault**) をクリック

  <img src="./images/deploy-key-vault-02.png" />

- ステップを展開し、結果を確認

  <img src="./images/deploy-key-vault-03.png" />

<br />

### Task 3: Kay Vault へのアクセス権の付与

- [Azure Portal](https://portal.azure.com/) から Key Vault の **アクセス制御 (IAM)** を選択

- **追加** > **ロールの割り当ての追加** をクリック

- **職務ロール** > **キー コンテナー管理者** を選択

  <img src="./images/role-assignment-13.png" />

- **アクセスの割当先** で **ユーザー、グループ、またはサービス プリンシパル** を選択、**＋ メンバーを選択する** をクリック

- ポータルにサインイン中のアカウントを指定し **選択** をクリック

- **レビューと割り当て** をクリックし、ロールを割り当て

<br />

### 参考情報

- [Azure 向けの GitHub Actions とは](https://learn.microsoft.com/ja-jp/azure/developer/github/github-actions)

- [GitHub Actions を使用した Bicep ファイルのデプロイ](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/deploy-github-actions?tabs=CLI%2Cuserlevel)

- [Key Vault のキー、証明書、シークレットへのアクセス権を付与する](https://learn.microsoft.com/ja-jp/azure/key-vault/general/rbac-guide?tabs=azure-cli)

- [Azure Key Vault セキュリティ](https://learn.microsoft.com/ja-jp/azure/key-vault/general/security-features)

<br />

## Exercise 7: GitHub Actions を用いた API の展開

<img src="./images/Ex7.png" />

<br />

### Task 1: マネージド ID の作成

- Azure Container Apps の **設定** > **ID** を選択

- **システム割り当て済み** タブで **状態** を **オン** に変更し **保存** をクリック

  <img src="./images/managed-identity-03.png" />

- 確認のメッセージが表示されるので **はい** をクリック

- システム割り当てマネージド ID の有効化が完了

  <img src="./images/managed-identity-04.png" />

<br />

### Task 2: ロールの割り当て

- Key Vault の **アクセス制御 (IAM)** を選択

- **追加** > **ロールの割り当ての追加** をクリック

- **職務ロール** で **キー コンテナー シークレット ユーザー** を選択し **次へ** をクリック

  <img src="./images/role-assignment-14.png" />

- **アクセスの割り当て先** で **マネージド ID** を選択し **＋ メンバーを選択する** をクリック

- **マネージドID の選択** でサブスクリプション、マネージド ID (コンテナー アプリ) を選択

- Container Apps のシステム割り当てマネージド ID をクリックし **選択** をクリック

  <img src="./images/role-assignment-15.png" />

- メンバーに選択したマネージド ID が表示されることを確認し **次へ** をクリック

- **レビューと割り当て** をクリックし、ロールを割り当て

<br />

### Task 3: コンテナー イメージのビルド、プッシュ

<br />

### Task 4: シークレットの登録

- Azure AI Search の **設定** > **キー** を選択し、クエリ キーをコピー

- Key Vault の **オブジェクト** > **シークレット** を選択し **＋ 生成/インポート** をクリック

  <img src="./images/keyvault-secret-01.png" />

- シークレットの作成

  - **アップロード オプション**: 手動

  - **名前**: 任意 (ApiKey など)

  - **シークレット値**: コピーした Azure AI Search のクエリ キー

    <img src="./images/keyvault-secret-02.png" />

- **作成** をクリックし、シークレットを登録

- Container Apps の **設定** > **シークレット** を選択し **＋ 追加** をクリック

- シークレットの追加

  - Container Apps シークレット

    |キー|種類|値|
    |---|---|---|
    |searchserviceendpoint (任意)|Container Apps シークレット|Azure AI Search の URL|
    |index (任意)|Container Apps シークレット|インデックス名|

    <img src="./images/containerapps-secret-02.png" />

  - Key Vault 参照

    - **キー**: apikey (任意)

    - **種類**: Key Vault 参照

    - **Key Vault シークレット URL**: Key Vault に登録したシークレットのシークレット識別子

    - **マネージド ID**: システム割り当て

      <img src="./images/containerapps-secret-03.png" />

- **追加** をクリックし、シークレットを登録

  <img src="./images/containerapps-secret-04.png" />

<br />

### Task 5: Azure Container Apps へのイメージの展開

<br />

### Task 6: API の動作確認

<br />