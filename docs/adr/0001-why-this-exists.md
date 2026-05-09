# ADR-0001: なぜ dbt-pii-toolkit を作るのか

- **Status**: Accepted
- **Date**: 2026-05-10
- **Decider**: Kiichi Iwashita

## Context

データウェアハウス上での PII (Personally Identifiable Information) の取り扱いは、
GDPR・改正個人情報保護法・CCPA 等の規制対応において中心的な課題である。
dbt は分析エンジニアリングのデファクトスタンダードとなったが、
PII を「検出 → マスキング → 監査」する一気通貫の OSS マクロ集は **2026年5月時点で存在しない**。

既存ソリューションの整理:

| カテゴリ | 例 | 課題 |
|----------|----|----|
| 商用 DLP | BigID, Privacera, Immuta | 高価格・dbt統合は周辺機能 |
| dbt 単機能マクロ | `dbt_utils.surrogate_key` | ハッシュは可能だが PII 検出は範囲外 |
| DBエンジン機能 | Snowflake Dynamic Data Masking | DB依存・ポータビリティなし |
| クラウドネイティブ DLP | GCP Sensitive Data Protection | dbt との統合は手動 |
| 自作スクリプト | 各社内製 | 共有されず再発明が続く |

## Problem

dbt プロジェクトで PII を扱う開発者が、毎回ゼロから以下を実装している:

1. PII カラムの特定 (カラム名 + サンプル値ベースの判定)
2. マスキング戦略の選択 (ハッシュ / 部分マスク / トークン化)
3. 監査トレイルの記録 (誰がいつ何をマスクしたか)
4. 規制要件とのマッピング (GDPR Art.X / 個情法第Y条)

この再発明は、組織ごとに **品質のばらつき** と **コンプライアンス事故のリスク** を生んでいる。

## Decision

**dbt-pii-toolkit を OSS として公開する**。

具体的には以下を提供する:

1. **検出マクロ群** (`detect_pii`, `find_pii_columns`)
   - カラム名パターン + 正規表現 + サンプル値の3層判定
2. **マスキングマクロ群** (`mask_email`, `mask_phone`, `hash_deterministic`, `tokenize`)
   - 用途別の4戦略を切替可能
3. **監査ログ機能** (`pii_audit_log`)
   - マクロ実行時に自動でメタデータをテーブル記録
4. **コンプライアンスタグ**
   - モデル単位で GDPR / 個情法カテゴリを宣言
5. **多DB対応** (Snowflake → BigQuery → Postgres の順)

## Alternatives Considered

### 代替案A: 既存の `dbt-utils` への PR で機能追加

却下理由:
- `dbt-utils` のスコープは汎用ユーティリティであり、PII特化機能はメンテナの方針に合わない
- 監査ログのような副作用を持つ機能は別パッケージとして独立させるべき

### 代替案B: 商用 SaaS として開発

却下理由:
- OSSとして公開する方が採用障壁が低く、コミュニティフィードバックを得やすい
- 著者のキャリア戦略 (OSS実績の蓄積) と整合する
- 商用機能 (audit log SaaS等) は将来的に別プロジェクトとして派生可能

### 代替案C: Snowflake / BigQuery のネイティブマスキング機能の使用を推奨するだけ

却下理由:
- DB エンジン固有機能はポータビリティがなく、マルチクラウド環境で再実装が必要
- マスキングポリシーは「アクセス制御」であり、ストレージ上で物理的にマスクする本Toolkitとは目的が異なる
- 補完関係であり、本Toolkitと併用が推奨される (README で明記)

## Consequences

### Positive

- ✅ 日本語圏で初の dbt PII マクロ集として認知獲得が期待できる
- ✅ コンプライアンス要件 (改正個情法 2024) に苦慮する日系企業へのアピール
- ✅ 著者の専門性 (データ基盤 + プライバシー) のショーケース
- ✅ コミュニティ寄稿により多DB対応が加速できる可能性

### Negative

- ⚠️ メンテナンス負荷 (3DB × dbt バージョン × 規制更新)
- ⚠️ 法的助言と誤解されるリスク → README / SECURITY に明記して回避
- ⚠️ 商用 DLP との競合に見られる可能性 → 「補完」のポジショニングを明確化

### Neutral

- 著者が当面メインメンテナとなる前提。コア寄稿者が3名以上集まった時点でガバナンス見直し。

## Related Decisions

- (今後) ADR-0002: マクロのディスパッチ戦略 (per-adapter dispatch vs unified)
- (今後) ADR-0003: 監査ログのスキーマ設計

## References

- [GDPR Art.25 — Privacy by design](https://gdpr-info.eu/art-25-gdpr/)
- [改正個人情報保護法 (2024)](https://www.ppc.go.jp/)
- [dbt-utils](https://github.com/dbt-labs/dbt-utils)
- [Snowflake Dynamic Data Masking](https://docs.snowflake.com/en/user-guide/security-column-ddm)
