# Lab 5: インシデント調査の対話型エージェント

**所要時間:** 35 分  
**ゴール:** Sentinel インシデント番号を受け取り、必要な情報を取得して追加調査を案内する対話型エージェントを作る

## こんなケースはありませんか

> 「新しいインシデントごとに、まず基本情報を集めて概要をまとめる——という同じ立ち上げ作業を、担当者ごとに手作業で繰り返している」——初期トリアージは属人化しやすく、担当者によって見る項目もばらつきます。

対話型エージェントにすると、**インシデント番号を伝えるだけで、必要な情報を定型的に取得し、事実と仮説と次の確認事項を分けて提示できる**ようになります。アナリストは調査の出発点をそろえ、本質的な判断に集中できます。

> [!NOTE]
> 「対話型エージェント」とは、チャットでやりとりしながら調査を進められる「係」のことです。この Lab では、安全のために「読むだけ・1 件だけ・更新しない」という控えめな境界を先に決めます。最初は小さく作るのがコツです。

## 5-1. エージェントの境界を決める

このエージェントは次だけを行います。

1. ユーザーからインシデント番号を受け取る
2. Sentinel の `SecurityIncident` から該当行を最大 1 件取得する
3. 取得結果を事実と推測に分けて要約する
4. 次に確認すべき観点を提示する

インシデントの更新、クローズ、ユーザー無効化などの書き込みは行いません。

## 5-2. Builder へ要求する

Sentinel と Security Copilot Agent の MCP ツールを有効にし、次を送ります。

```text
Microsoft Sentinel のインシデントを調査する対話型 Security Copilot エージェントを作成してください。

条件:
- 入力は対話型エージェント必須の UserRequest 1 つだけ
- UserRequest から IncidentNumber を特定する
- SecurityIncident の実スキーマを MCP で確認する
- 1 件だけ取得する読み取り専用 KQL 子スキルを持つ
- 事実、仮説、次の確認事項を分ける
- データがない場合は捏造しない
- 自動修復やインシデント更新はしない
- SOC アナリスト向け日本語スタータープロンプトを 2 つ付ける
```

開始例は [sentinel-incident-agent.yaml](../samples/sentinel-incident-agent.yaml) です。プレースホルダー値と実際の列名を置換してください。

## 5-3. (参考) 作成されたエージェント (yaml ファイル) を確認する

- `AgentDefinitions[].PromptSkill` が `Descriptor.Name` と Agent スキル名を参照する
- `Interfaces` が `InteractiveAgent`
- 入力が必須の `UserRequest` 1 つだけ
- `RequiredSkillsets` に自身のスキルセットがある
- `ChildSkills` に KQL スキルがある
- `SuggestedPrompts` のスターターに `Title`、`Personas: [1]`、`IsStarterAgent: true` がある
- スケジュール実行が無効 (`DefaultPollPeriodSeconds: 0`)

## 5-4. アップロードしてテストする

1. YAML を Security Copilot の個人スコープへアップロードします。
2. 必要な Sentinel 設定値を入力します。
3. **Active agents** でエージェントをセットアップします。
4. **Chat with agent** を開きます。
5. 講師から渡された演習用インシデント番号でスタータープロンプトを実行します。

<img width="877" height="251" alt="image" src="https://github.com/user-attachments/assets/6bb1424f-9552-42b5-86be-603cb88dd9c7" />
<img width="2464" height="646" alt="image" src="https://github.com/user-attachments/assets/3ae55fed-16ec-43fc-8e27-93c6629c3684" />
<img width="2066" height="1143" alt="image" src="https://github.com/user-attachments/assets/993af105-dfad-4999-9a12-dd578929126c" />

## 5-5. 結果を評価する

| 観点 | 合格条件 |
|---|---|
| Grounding | KQL 結果にない事実を追加しない |
| Scope | 1 インシデントだけを扱う |
| Structure | 事実、仮説、次の確認事項が分かれる |
| Empty result | 見つからないことを明示する |
| Safety | 更新や修復を実行しない |

> [!NOTE]
> 対話型エージェントのメモリはチャットコンテキストに含まれないという既知の制限があります。重要な識別子は会話の記憶だけに依存させず、必要に応じて再提示します。

## チャレンジ

現在は Sentinel のインシデント ID を照会するようになっていますが、Defender のインシデント ID を照会するように変更してみてください。

## チェックポイント

- [ ] `UserRequest` 1 入力の対話型エージェントを作った
- [ ] KQL 子スキルを 1 件取得へ制限した
- [ ] プロンプトインジェクションを想定して境界を確認した
- [ ] 事実と仮説を分けて評価した
