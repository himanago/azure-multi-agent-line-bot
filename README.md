# Azure Multi-Agent LINE Bot

Azure AI Foundry Agent Service を使用してマルチエージェント機能を実装した LINE Bot です。Durable Functions を使用してオーケストレーションを行い、複数のメッセージを並行処理しながら、AI エージェントとの対話を実現します。

## 使用技術

- C# (.NET 8 isolated)
- Azure Functions (Durable Functions)
- Azure AI Foundry Agent Service
- Azure Key Vault
- Azure Application Insights
- Azure Developer CLI (azd)
- LINE Messaging API
- OpenAPI Generator

## アーキテクチャ

このプロジェクトは以下のアーキテクチャで構成されています：

1. **LINE Webhook** - LINE からのメッセージを受信
2. **Durable Functions Orchestrator** - メッセージ処理のオーケストレーション
3. **AI Agent Activity** - Azure AI Foundry Agent Service との連携
4. **LINE Reply Activity** - LINE への応答送信
5. **Entity Storage** - ユーザーセッションとリプライトークンの管理

### 主な機能

- **マルチエージェント対応**: Azure AI Foundry Agent Service を使用
- **マルチモーダル対応**: テキストと画像の両方を処理
- **並行処理**: Durable Functions による複数メッセージの同時処理
- **セッション管理**: ユーザーごとのスレッド管理
- **エラーハンドリング**: 堅牢なエラー処理とリトライ機構

詳細はスライド資料： [Azure AI＆サーバーレスで構築するマルチエージェント LINE Bot](https://www.docswell.com/s/himanago/KDJ4X4-azure-paas-agents-linebot) を参照してください。

## 使い方

### 1. 前提条件

- Azure サブスクリプション
- Azure AI Foundry プロジェクト
- LINE Developer アカウント
- Azure CLI または GitHub Codespaces

### 2. GitHub Codespaces で開く

本リポジトリを GitHub Codespaces で開くか、ローカルにクローンします。

### 3. 準備完了するまで待つ

Codespace の場合、openapi-generator-cli により LINE OpenAPI の定義から SDK を `sdk` フォルダに生成します。

```bash
./generate-sdk.sh
```

### 4. LINE Messaging API チャネルの作成

LINE Developers のドキュメント、[Messaging APIを始めよう](https://developers.line.biz/ja/docs/messaging-api/getting-started/) を参考に、Messaging API チャネルを作成します。

### 4. Azure Developer CLI のログイン

ターミナルで以下のコマンドを実行し、`azd` で Azure にログインします。

```bash
azd auth login --use-device-code
```

表示されるコードをコピーし、Enter 押下後に開くタブに貼り付け、使用する Azure アカウントでサインインしてください。

### 5. Azure AI Foundry プロジェクトとエージェントの作成

Azure AI Foundry リソースおよびプロジェクト、エージェントは Bicep により作成されません。

あらかじめ Azure Portal / Azure AI Foundry portal で作成しておいてください。

#### マルチエージェントの作成

このプロジェクトではマルチエージェント構成を推奨します。

[docs/agent-prompts.md](./docs/agent-prompts.md) を参考に、各エージェントを作成してください。

#### プロジェクト情報の記録

1. **プロジェクトエンドポイント**:
   - プロジェクトの **「Settings」** → **「Overview」** からエンドポイント URL をコピー
   - 形式: `https://your-project.cognitiveservices.azure.com/`

2. **メインエージェント ID**:
   - メインエージェントの詳細画面でエージェント ID をコピー
   - 形式: `asst_xxxxxxxxx`

### 6. Azure Developer CLI によるリソースの作成とデプロイ

リソースの作成とデプロイを実行：

```bash
azd up
```

実行すると、以下の値を対話的に入力するよう求められます：

- **LINE Channel Access Token**: LINE Messaging API チャネルのアクセストークン
- **Azure AI Project Endpoint**: Azure AI Foundry プロジェクトのエンドポイント URL
- **Azure AI Agent ID**: 作成したメインエージェント（オーケストレーター）の ID

### 7. LINE Bot の Webhook URL の設定

デプロイ完了後、Azure Functions の `WebhookEndpoint` 関数の URL を LINE Messaging API チャネルの Webhook URL に設定します。

### 8. 動作確認

LINE で Bot を友だち追加し、メッセージを送信してテストします。AI Agent が応答を返すことを確認してください。

## 開発者向け情報

### プロジェクト構造

```
src/LineBotFunctions/
├── WebhookEndpoint.cs          # LINE Webhook エンドポイント
├── WebhookHandler.cs           # Webhook イベント処理
├── LineMessageOrchestrator.cs  # メッセージ処理オーケストレーター
├── CallAIAgentActivity.cs      # AI Agent 呼び出し
├── SendLineReplyActivity.cs    # LINE 応答送信
├── LineTalkEntity.cs           # エンティティ状態管理
├── Services/
│   └── LineImageService.cs     # LINE 画像取得サービス
├── Models/
│   └── AIAgentModels.cs        # データモデル定義
└── Config/
    ├── AIAgentSettings.cs      # AI Agent 設定
    └── LineBotSettings.cs      # LINE Bot 設定
```

### 環境変数

| 変数名 | 説明 | 必須 |
|-------|------|------|
| `LINE_CHANNEL_ACCESS_TOKEN` | LINE チャネルアクセストークン | ✓ |
| `AZURE_AI_PROJECT_ENDPOINT` | Azure AI Foundry プロジェクトエンドポイント | ✓ |
| `AZURE_AI_AGENT_ID` | メインエージェント（オーケストレーター）の ID | ✓ |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault エンドポイント | ✓ |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | Application Insights 接続文字列 | ✓ |

### カスタマイズ

- `WebhookHandler.cs` でメッセージタイプごとの処理をカスタマイズ
- `CallAIAgentActivity.cs` で AI Agent との連携ロジックを調整
- `LineTalkEntity.cs` でセッション管理の動作を変更

## トラブルシューティング

1. **AI Agent が応答しない**
   - Azure AI Foundry プロジェクトのエンドポイントと Agent ID を確認
   - Azure Functions の環境変数設定を確認

2. **LINE メッセージが届かない**
   - Webhook URL が正しく設定されているか確認
   - LINE チャネルアクセストークンが正しく設定されているか確認

3. **デプロイエラー**
   - Azure サブスクリプションの権限を確認
   - リージョンでのリソース利用可能性を確認

## その他

### SDK の更新

LINE OpenAPI の最新版に基づいて SDK を更新する場合：

```bash
./generate-sdk.sh
```

## セキュリティ設定

### 自動設定される権限

Bicep テンプレートにより、以下の権限が Azure Functions のマネージド ID に自動的に設定されます：

- **Azure AI User**: Azure AI Foundry Agent Service への呼び出し権限
- **Key Vault Secrets User**: Key Vault からのシークレット取得権限
- **Storage Blob Data Owner**: Durable Functions 用ストレージアクセス権限


## 参考

- [Azure AI Foundry](https://ai.azure.com/)
- [LINE Messaging API](https://developers.line.biz/ja/docs/messaging-api/)
- [Azure Durable Functions](https://docs.microsoft.com/azure/azure-functions/durable/)
- [Azure AI Foundry Agent Service](https://docs.microsoft.com/azure/ai-services/agents/)
- [Azure Functions HTTP trigger](https://docs.microsoft.com/azure/azure-functions/functions-bindings-http-webhook-trigger)
- [Azure Developer CLI (azd)](https://docs.microsoft.com/azure/developer/azure-developer-cli/)
