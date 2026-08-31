# Lab 2: Microsoft Sentinel MCP サーバーと Azure MCP Server へ接続する

**所要時間:** 30 分<br>
**ゴール:** Sentinel のデータ探索、Security Copilot エージェント作成、Azure リソース確認の MCP ツールを VS Code から利用する

## こんなケースはありませんか

> 「調査のたびにどのテーブルに目的のデータがあるのか思い出せず、ドキュメントや過去の KQL を探し回っている」——サイロのテーブルが多く、列名も環境ごとに違うため、ベテランでもスキーマ確認に時間を取られがちです。

本 Lab で MCP サーバーを接続すると、**自然言語で「サインイン失敗を調べたい」と伝えるだけで、AI が実在するテーブルと列を探してくれる**ようになります。テーブル名を暗記する必要はなく、同じ MCP ツールが後の Lab で作るカスタムエージェントの土台にもなります。

> [!NOTE]
> 「MCP」は、VS Code から外部のデータやサービスにつなぐ「差し込み口」だと思ってください。この Lab では 3 つの MCP 接続を利用します。Sentinel のデータ探索用、Security Copilot のエージェント作成用、Azure リソース確認用です。

> [!IMPORTANT]
> Sentinel の 2 つはワークスペースへ登録するリモート MCP コレクションです。Azure MCP Server は VS Code 拡張機能の Provider から提供され、サインインしたユーザーの Azure RBAC 権限の範囲で Azure リソースへアクセスします。最初は Reader ロールなどの読み取り権限で試してください。

| 表示名または Server ID | 用途 | 接続方式 |
|---|---|---|
| `sentinel-data-exploration` | テーブル検索、データ取得、エンティティ分析 | HTTP: `https://sentinel.microsoft.com/mcp/data-exploration` |
| `security-copilot-agent-creation` | エージェントの作成、ツール検索、デプロイ | HTTP: `https://sentinel.microsoft.com/mcp/security-copilot-agent-creation` |
| `Azure MCP Server` | Azure のサブスクリプション、リソースグループ、各種リソースの確認と操作 | VS Code 拡張機能 Provider |

## 2-1. データ探索コレクションを追加する

1. `Ctrl+Shift+P` を押します。
2. **MCP: Add Server** を選択します。<img width="908" height="421" alt="image" src="https://github.com/user-attachments/assets/1c39fb04-6a92-41da-a4df-3c8b490ce534" />

3. **HTTP (HTTP or Server-Sent Events)** を選択します。<img width="904" height="319" alt="image" src="https://github.com/user-attachments/assets/546ffdd8-9f23-49fa-b805-a2598c875c48" />

4. 次の URL を入力します。大文字と小文字を変えないでください。

```text
https://sentinel.microsoft.com/mcp/data-exploration
```

<img width="912" height="141" alt="image" src="https://github.com/user-attachments/assets/130ada2f-c431-43e1-9a32-402da6f64d36" />

5. Server ID に `sentinel-data-exploration` と入力します。<img width="908" height="145" alt="image" src="https://github.com/user-attachments/assets/195f1085-1202-4e84-bf73-d8e7539b5f41" />

6. **Workspace** を選びます。<img width="892" height="166" alt="image" src="https://github.com/user-attachments/assets/25712215-0660-4d41-988c-ba562d966f59" />

7. 信頼を求められたら、URL が `https://sentinel.microsoft.com/` であることを確認して許可します。<img width="810" height="190" alt="image" src="https://github.com/user-attachments/assets/00e115b9-cda7-47b0-8416-6ec533a6241f" />

8. 演習対象テナントのアカウントで認証します。

<img width="903" height="269" alt="image" src="https://github.com/user-attachments/assets/bb216b7d-a00b-4bd8-92e7-b4c66b097882" />


## 2-2. エージェント作成コレクションを追加する

同じ手順でもう 1 台追加し、次を指定します。

```text
URL: https://sentinel.microsoft.com/mcp/security-copilot-agent-creation
Server ID: security-copilot-agent-creation
Scope: Workspace
```

**期待結果:** ワークスペースの `.vscode/mcp.json` に 2 つのサーバーが表示され、どちらも起動できます。

> [!CAUTION]
> 生成された `mcp.json` にアクセストークンを追記しないでください。認証は VS Code の認証フローに任せます。

<img width="1141" height="682" alt="image" src="https://github.com/user-attachments/assets/ff6cb86c-67c7-4d42-be58-fb9f9ba6fcb9" />

## 2-3. Azure MCP Server を追加する

[Azure MCP Server](https://learn.microsoft.com/azure/developer/azure-mcp-server/get-started/tools/visual-studio-code) は、AI エージェントが Azure のリソースを確認・操作するための Microsoft 製 MCP サーバーです。この演習では、リソースを変更せず一覧だけを取得します。

### 前提条件を確認する

- Azure サブスクリプションへアクセスできるアカウント
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)（演習対象のアカウント確認に使用）
- 対象スコープの Reader ロール、または同等の読み取り権限

ターミナルで次を実行し、演習対象のアカウントとサブスクリプションを確認します。

```powershell
az login
az account show --output table
```

複数のサブスクリプションを利用できる場合は、演習対象を明示します。

```powershell
az account set --subscription "<サブスクリプション名または ID>"
```

> [!CAUTION]
> `az account show` の結果にあるテナントとサブスクリプションが演習対象であることを確認してください。Azure MCP Server は、VS Code などのローカル開発ツールでサインインした認証情報と Azure RBAC 権限を使います。

### 拡張機能 Provider を有効にする

1. VS Code の左側にある **Extensions** を選択します。
2. `Azure MCP Server` を検索します。
3. 発行元が **Microsoft** の [Azure MCP Server](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azure-mcp-server) 拡張機能を選択し、**Install** を選択します。
4. 要求された場合は VS Code を再読み込みします。
5. **View > Chat** を開き、モードを **Agent** にします。
6. プロンプト欄のツールアイコンを選択し、ツール一覧を更新します。
7. `Azure MCP` で検索し、Azure MCP Server のツールが表示されることを確認します。
8. Azure へのサインインを求められた場合は、先ほど確認した演習対象アカウントで認証します。

> [!NOTE]
> Azure MCP Server 拡張機能は MCP Server Definition Provider として Azure MCP を VS Code へ登録します。そのため、Azure MCP 用の設定を `.vscode/mcp.json` へ追記する必要はありません。拡張機能を使うとプレビュー版も自動更新されます。

## 2-4. ツールを確認する

1. **View > Chat** を開きます。
2. モードを **Agent** にします。
3. プロンプト欄のツールアイコンを選択します。
4. `sentinel-data-exploration` のデータ探索ツールが見えることを確認します。
5. `security-copilot-agent-creation` にエージェント作成用ツールが見えることを確認します。
6. `Azure MCP Server` に Azure リソース用ツールが見えることを確認します。

<img width="897" height="110" alt="image" src="https://github.com/user-attachments/assets/3695501f-b1b8-4015-a907-9a27132dfa44" />

## 2-5. Azure リソースと Sentinel テーブルを検索する

最初に Azure MCP Server のツールだけを有効にし、リソースを変更しないよう明示して次を送ります。

```text
Azure MCP Server を使い、現在選択されているサブスクリプション名とリソースグループの一覧を取得してください。
読み取り操作だけを行い、リソースの作成、変更、削除はしないでください。
```

**期待結果:** Azure MCP のツール実行確認が表示され、許可するとサブスクリプション名とリソースグループが返ります。対象サブスクリプションが違う場合は、処理を続けず `az account set` で切り替えます。

> [!IMPORTANT]
> ツールの実行確認では、操作名と引数を毎回確認してください。この演習では読み取り操作だけを許可し、ワークスペース全体や今後のセッションに対する自動許可は設定しません。

次に、Azure MCP Server のツールを無効にしてデータ探索コレクションだけを有効にします。

```text
Microsoft Sentinel で過去 24 時間のサインイン失敗を調べるために、利用可能な関連テーブルを検索してください。
まだログ本体は取得せず、候補テーブル名、用途、主要な時刻列と結果列を表にしてください。
```

**期待結果:** MCP ツールが呼び出され、テナントで利用可能なテーブル候補が返ります。候補は環境によって異なります。

<img width="1403" height="759" alt="image" src="https://github.com/user-attachments/assets/7bccabe4-fa34-4004-bc3e-0e28f834a64c" />


次に、候補の 1 つを指定して少量だけ確認します。

```text
先ほど見つけた最適なテーブルを使い、過去 24 時間のサインイン失敗を最大 10 件だけ取得してください。
返す列は時刻、ユーザー、結果コード、IP アドレスに限定してください。使用した KQL も示してください。
```

**確認ポイント:** 時間条件、`project`、`take` または `top` が入り、不要な列や大量データを返していないこと。

## 2-6. 接続トラブルを切り分ける

| 症状 | 確認すること |
|---|---|
| 401/403 | 対象テナント、Security Reader、Security Copilot/データソース権限 |
| サーバーが起動しない | URL の綴り、VS Code の更新、組織のプロキシ設定 |
| ツールが表示されない | Agent モード、対象 MCP サーバーの起動、ツールの有効化 |
| Azure MCP のツールが表示されない | Azure MCP Server 拡張機能がインストール済みで有効か、VS Code を再読み込みしたか、Tools 一覧を更新したか |
| Azure の認証に失敗する | `az login` と `az account show`、対象テナント、条件付きアクセス |
| Azure リソースが見えない | 選択中のサブスクリプション、対象スコープの Reader ロール、リソースの存在 |
| テーブルが見つからない | Sentinel data lake のオンボード、データコネクタ、対象期間 |
| Builder がスキーマを取得しない | 必要な MCP コレクションが有効か、Builder の `allowed-tools` と実際のツール名が一致するか |

## チェックポイント

- [ ] Sentinel の 2 つの MCP コレクションと Azure MCP Server の用途を説明できる
- [ ] Sentinel の 2 サーバーが起動し、Azure MCP Server Provider のツールが表示される
- [ ] Azure MCP Server で対象サブスクリプションとリソースグループを読み取り専用で確認できる
- [ ] テナントで利用可能なテーブルを MCP 経由で検索できる
- [ ] 最大 10 件、必要列のみのログ検索を実行できる
