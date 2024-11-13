<img src="./images/ms-cloud-workshop.png" />

Retrieval Augmented Generation (RAG) pattern for Azure AI Search  
Dec 2024

<br />

### Contents

- [Exercise 8: API Management を使用した API の発行](#exercise-8-api-management-を使用した-api-の発行)

  - [Task 1: プライベート エンドポイントの作成](#task-1-プライベート-エンドポイントの作成)

  - [Task 2: API Management の展開](#task-2-api-management-の展開)

  - [Task 3: API Management インスタンスと仮想ネットワークの統合](#task-3-api-management-インスタンスと仮想ネットワークの統合)

  - [Task 4: API の公開](#task-4-api-の公開)

<br />

## Exercise 8: API Management を使用した API の発行

<img src="./images/Ex8.png" />

<br />

### Task 1: プライベート エンドポイントの作成

- Azure AI Search の **検索管理** > **ネットワーク** を選択

- **ファイアウォールと仮想ネットワーク** タブで **パブリック ネットワーク アクセス** を **無効** に変更

  > 「このサービスのエンドポイントを接続をプライベートにしますか？」のメッセージが表示されるので **はい** をクリック

- **保存** をクリック

  <img src="./images/aisearch_private-endpoint-01.png" />

- **プライベート エンドポイント接続** > **＋ プライベート エンドポイントを作成する** をクリック

  <img src="./images/aisearch_private-endpoint-02.png" />

- プライベート エンドポイントの作成

  - **基本**

    - **プロジェクトの詳細**

      - **サブスクリプション**: ワークショップで使用中のサブスクリプション

      - **リソース グループ**: Azure AI Search が属するリソース グループ

    - **インスタンスの詳細**

      - **名前**: 任意 (pep-AI-Search など)

      - **ネットワーク インターフェイス名**: 任意 (nic-AI-Search など)

      - **リージョン**: 展開先の仮想ネットワークと同じリージョン

        <img src="./images/aisearch_private-endpoint-03.png" />

  - **リソース**

    - **サブスクリプション**: ワークショップで使用中のサブスクリプション

    - **リソースの種類**: Microsoft.Search/searchServices

    - **リソース**: Azure AI Search

    - **対象サブリソース**: searchService

      <img src="./images/aisearch_private-endpoint-04.png" />

  - **仮想ネットワーク**

    - **仮想ネットワーク**: 展開先の仮想ネットワーク

    - **サブネット**: 展開先のサブネット

    - **IP アドレスを動的に割り当てる**: オン

      <img src="./images/aisearch_private-endpoint-05.png" />
  
  - **DNS**

    - **プライベート DNS ゾーンと統合する**: はい

      <img src="./images/aisearch_private-endpoint-06.png" />
  
  - **タグ**

    <img src="./images/aisearch_private-endpoint-07.png" />

- **作成** をクリック

  <img src="./images/aisearch_private-endpoint-08.png" />

- ストレージ アカウント (Blob), Azure OpenAI Service, Key Vault に対するプライベート エンドポイントを作成

  - **ストレージ アカウント (Blob)**

    - **リソースの種類**: Microsoft.Storage/storageAccounts

    - **対象サブリソース**: blob

      <img src="./images/blob-private-endpoint-04.png" />
  
  - **Azure OpenAI Service**

    - **リソースの種類**: Microsoft.CognitiveServices/storageAccounts

    - **対象サブリソース**: account

      <img src="./images/openai-private-endpoint-04.png" />
  
  - **Key Vault**

    - **リソースの種類**: Microsoft.KeyVault/vaults

    - **対象サブリソース**: vaults

      <img src="./images/keyvault-private-endpoint-04.png" />

<br />

### Task 2: API Management の展開

- [Azure Portal](https://portal.azure.com/) のホーム画面で **＋ リソースの作成** をクリック

- **カテゴリ** > **Web** を選択、**API Management** の **作成** をクリック

- Azure API Management サービスの作成

  - **基本**

    - **プロジェクトの詳細**

      - **サブスクリプション**: ワークショップで使用中のサブスクリプション

      - **リソース グループ**: リソース グループ

    - **インスタンスの詳細**

      - **リージョン**: 展開するリージョン

      - **リソース名**: 任意 (apim-xxxx など)

      - **組織名**: 任意 (MCW など)

      - **管理者のメールアドレス**: 任意

    - **価格レベル**

      - **価格レベル**: Standard v2

      - **ユニット**: 1

        <img src="./images/api-management-01.png" />

    - **監視とセキュリティ保護**

      - **Log Analytics**: オン

        - **サブスクリプション**: ワークショップで使用中のサブスクリプション

        - **ログ分析ワークスペース**: 事前に展開済みの Log Analytics ワークスペース
      
      - **Application Insights**: オン

        - **インスタンスの選択**: 事前に展開済みの Application Insights

          <img src="./images/api-management-02.png" />
    
    - **仮想ネットワーク**

      - **接続の種類**: なし

        <img src="./images/api-management-03.png" />
    
    - **マネージド ID**

      <img src="./images/api-management-04.png" />

      > 既定のまま、有効化は不要
    
    - **タグ**

      <img src="./images/api-management-05.png" />
  
- **作成** をクリックし API Management インスタンスを展開

  <img src="./images/api-management-06.png" />

<br />

### Task 3: API Management インスタンスと仮想ネットワークの統合

- 仮想ネットワークの **設定** > **サブネット** を選択、統合に使用するサブネットをクリック

  <img src="./images/vnet-integration-01.png" />

- **サブネットをサービスに委任** に **Microsoft.Web/serverFarms** を選択し **保存** をクリック

  <img src="./images/vnet-integration-02.png" />

- API Management の **Deployment + infrastructure** > **ネットワーク** を選択

- **VNET integration** をクリック

  <img src="./images/vnet-integration-03.png" />

- **仮想ネットワーク** にチェックし、**VNET を選択してください** をクリック

  <img src="./images/vnet-integration-05.png" />

- サービスを委任したサブネットを持つ仮想ネットワーク、およびサブネットを選択し **適用** をクリック

  <img src="./images/vnet-integration-06.png" />

- **保存** をクリック

  <img src="./images/vnet-integration-07.png" />

- **VNET intagration** が **On** に変更されたことを確認

  <img src="./images/vnet-integration-08.png" />

<br />

### Task 4: API のインポートと発行

- API Management の **APIs** > **API** を選択

- **Create from Azure resource** > **Container App** をクリック

  <img src="./images/publish-api-01.png" />

- **Full** を選択し、必要な情報を入力

  - **Container App**: **Browse** をクリックし、API を展開した Contaienr Apps を選択

  - **API URL suffix**: api

    <img src="./images/publish-api-02.png" />

- **Create** をクリックし API をインポート

- **APIs** > **製品** を選択し **＋ 追加** をクリック

  - 製品の追加

    - **表示名**: MCW APIs (任意)

    - **ID**: mcw-apis (任意)

    - **説明**: Search API service (任意)

    - **発行済み**: オン

    - **サブスクリプションを要求する**: オン

      <img src="./images/publish-api-03.png" />

- **作成** をクリックし、製品を追加

- 追加した製品をクリック

  <img src="./images/publish-api-04.png" />

- **製品** > **API** を選択し **＋ 追加** をクリック、インポートした API を選択し **選択** をクリック

  <img src="./images/publish-api-05.png" />

- **製品** > **サブスクリプション** を選択し **＋ サブスクリプションの追加** をクリック

  <img src="./images/publish-api-06.png" />

- **名前**、**表示名** を入力し **作成** をクリック

  <img src="./images/publish-api-07.png" />

- 追加したサブスクリプションの **...** をクリックし **キーの表示/非表示** をクリック

- API 応答のテストを行うために **主キー** をコピー

<br />

### 参考情報

- [Azure API Management とは](https://learn.microsoft.com/ja-jp/azure/api-management/api-management-key-concepts)

- [送信要求用の VNet との統合](https://learn.microsoft.com/ja-jp/azure/api-management/integrate-vnet-outbound)

- [Container Apps Web API のインポート](https://learn.microsoft.com/ja-jp/azure/api-management/import-container-app-with-oas)

- [最初の API のインポートと発行](https://learn.microsoft.com/ja-jp/azure/api-management/import-and-publish)

- [製品を作成して発行する](https://learn.microsoft.com/ja-jp/azure/api-management/api-management-howto-add-products?tabs=azure-portal&pivots=interactive)