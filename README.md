# dbt-pii-toolkit

> Privacy-by-design dbt macros for PII detection and masking.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![dbt](https://img.shields.io/badge/dbt-1.7+-orange)](https://www.getdbt.com/)

## なぜこのプロジェクトか

データウェアハウスに無防備に流れ込む PII（Personally Identifiable Information）は、
GDPR・改正個人情報保護法・CCPA など各種規制違反の最大要因です。にもかかわらず、
dbt エコシステムには **PII検出 + マスキングを一気通貫で扱える OSS マクロ集** が
ほぼ存在しません（2026年時点）。

`dbt-pii-toolkit` は以下を提供します:

- **検出** (`detect_pii`): カラム名・サンプル値・正規表現・LLMヒントから PII を分類
- **マスキング** (`mask_*`): 決定的ハッシュ / 部分マスキング / トークン化 / 完全削除
- **監査ログ** (`pii_audit_log`): どのモデルで誰がどうマスクしたかの自動記録
- **コンプライアンス・タグ**: GDPR Art.6 / 個情法カテゴリのモデル単位タグ付け

## 差別化ポイント

| 観点 | 既存OSS | dbt-pii-toolkit |
|------|---------|-----------------|
| PII自動検出 | 手動指定が中心 | **カラム名+正規表現+サンプルベース** の3層検出 |
| マスキング戦略 | 単一手法のみ多い | 用途別に **4種類** を切替可能 |
| 監査トレイル | 別途実装が必要 | **マクロ実行時に自動記録** |
| 規制対応 | 言及なし | GDPR / 個情法 / CCPA への **マッピング表を同梱** |
| 多DB対応 | Snowflake偏重 | **Snowflake / BigQuery / Postgres** の3DB対応を計画 |

## クイックスタート

> **注**: v0.1 開発中。現在 scaffolding 段階につき、API は流動的です。

```yaml
# packages.yml
packages:
  - git: "https://github.com/Islanders-Treasure0969/dbt-pii-toolkit.git"
    revision: main
```

```sql
-- models/marts/customers_masked.sql
{{ config(materialized='table', tags=['pii', 'gdpr_art6_consent']) }}

select
  id,
  {{ pii_toolkit.mask_email('email') }} as email_masked,
  {{ pii_toolkit.mask_phone('phone') }} as phone_masked,
  {{ pii_toolkit.hash_deterministic('national_id', salt=var('pii_salt')) }} as national_id_hash
from {{ ref('stg_customers') }}
```

## ロードマップ

詳細は [ROADMAP.md](./ROADMAP.md) を参照。

| バージョン | スコープ | 目標 |
|------------|----------|------|
| v0.1 | PII検出マクロ + 基本マスキング (Snowflake) | 2026 Q3 |
| v0.2 | BigQuery / Postgres対応 + 監査ログ | 2026 Q4 |
| v0.3 | コンプライアンスタグ + ドキュメント生成 | 2027 Q1 |

## コンプライアンス文脈

このリポジトリは、以下の規制への対応を技術的に支援することを目的とします（法的助言ではありません）:

- **GDPR** (EU): Art.5 (Data minimization), Art.25 (Privacy by design), Art.32 (Security)
- **改正個人情報保護法** (日本): 仮名加工情報・匿名加工情報の生成支援
- **CCPA / CPRA** (米国カリフォルニア): 削除権・オプトアウトの実装支援

実運用での適合性判断は必ず法務部門と確認してください。

## サンプルデータ方針

⚠️ **重要**: このリポジトリには **合成データ (synthetic data) のみ** を含めます。
実際のPIIや顧客データは絶対に commit しないでください。脆弱性の発見・報告については
[SECURITY.md](./SECURITY.md) を参照してください。

## コントリビュート

Issue / PR ともに歓迎です。設計判断の背景は [docs/adr/](./docs/adr/) にADRとして記録しています。

## ライセンス

[MIT License](./LICENSE) — Copyright (c) 2026 Kiichi Iwashita
