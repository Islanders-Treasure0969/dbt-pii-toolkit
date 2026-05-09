# Roadmap

> 6ヶ月マイルストーン。バージョン番号は SemVer (0.x はAPI非互換変更ありえる)。

## v0.1 — Foundation (2026 Q3 / 〜7月末)

**ゴール**: Snowflake環境で PII を検出してマスキングできる最小構成を提供する。

### Must-have
- [ ] `pii_toolkit.detect_pii(model, columns)` — カラム名 + 正規表現での検出
- [ ] `pii_toolkit.mask_email(column)` — `j***@example.com` 形式
- [ ] `pii_toolkit.mask_phone(column)` — 末尾4桁のみ残す
- [ ] `pii_toolkit.hash_deterministic(column, salt)` — SHA-256ベース決定的ハッシュ
- [ ] `pii_toolkit.tokenize(column)` — UUID置換 + マッピング表
- [ ] dbt 1.7+ 対応
- [ ] Snowflake 対応
- [ ] 合成データを使った integration test
- [ ] README / SECURITY / CONTRIBUTING の整備

### Nice-to-have
- [ ] 検出ルールのカスタマイズ機能 (YAML設定)
- [ ] dbt docs 生成時に PII カラムをハイライト

## v0.2 — Multi-DB + Audit (2026 Q4 / 〜10月末)

**ゴール**: BigQuery / Postgres に対応し、監査ログ機能を追加する。

### Must-have
- [ ] BigQuery dispatch 実装
- [ ] PostgreSQL dispatch 実装
- [ ] `pii_toolkit.pii_audit_log` テーブルへの自動記録
- [ ] テストマトリックス (Snowflake × BigQuery × Postgres)
- [ ] CI で 3DB 全てに対する integration test

### Nice-to-have
- [ ] Redshift 対応
- [ ] Databricks 対応 (コミュニティ要望次第)

## v0.3 — Compliance & DX (2027 Q1 / 〜1月末)

**ゴール**: コンプライアンス文脈との接続を強化し、開発者体験を向上させる。

### Must-have
- [ ] モデル単位の compliance タグ (`gdpr_art6_*`, `kojin_kameikako` 等)
- [ ] `pii_toolkit.compliance_report` — タグから自動でレポート生成
- [ ] dbt-pii-toolkit 専用 docs サイト (Mintlify or Docusaurus)
- [ ] チュートリアル (3本以上): 入門 / 監査ログ活用 / 多DB移行

### Nice-to-have
- [ ] `dbt-osmosis` 連携での自動タグ伝播
- [ ] Great Expectations 連携での PII 漏洩検知

## v1.0 — Production Ready (2027 Q2)

**ゴール**: 本番運用に耐える品質と、長期メンテナンスのコミットメント。

- [ ] API 安定化 (破壊的変更は v2 まで凍結)
- [ ] 全マクロのカバレッジ 90%+
- [ ] 実運用事例 3社以上の収集 (匿名でも可)
- [ ] セキュリティ監査の実施

## 仕様外 (Non-goals)

このリポジトリは以下を **扱いません**:

- **DLP製品の代替** — 暗号化キー管理や鍵ローテーションは Vault / KMS を推奨
- **クライアントサイド暗号化** — クエリ可能性が損なわれるため
- **動的データマスキング (DDM)** — DBエンジン固有機能 (Snowflake masking policy等) を併用してください
- **個人情報の自動削除権 (RTBF) 実装** — 専用のフレームワーク (e.g. `gdpr-tooling`) を別途検討してください

## フィードバック

Roadmap 上の優先度は GitHub Issues / Discussions で議論しましょう。
ユースケースが分かるほど良いマクロが書けるので、ぜひあなたの環境を教えてください。
