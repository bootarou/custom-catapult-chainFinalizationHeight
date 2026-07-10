# BNL Custom Catapult — Chain Finalization Height

> **Disclaimer:** This is an **unofficial, independent fork** of the Symbol monorepo. It is
> **not affiliated with, endorsed by, or connected to** the official Symbol / NEM projects, the
> NEM Group, or any Symbol (XYM) cryptocurrency, network, or organization. "Symbol", "NEM", and
> related names are used only to describe the upstream software this fork derives from. Do **not**
> treat this project as official Symbol software. Use at your own risk.

> **免責事項:** 本プロジェクトは Symbol モノレポの**非公式かつ独立したフォーク**です。
> 公式 Symbol / NEM プロジェクト、NEM Group、および Symbol（XYM）暗号資産・ネットワーク・関連団体とは
> **一切関係がなく、提携・承認・後援も受けていません**。「Symbol」「NEM」等の名称は、派生元のソフトウェアを
> 説明する目的でのみ使用しています。本プロジェクトを公式 Symbol ソフトウェアとして扱わないでください。
> 利用は自己責任でお願いします。

Symbol/catapult のカスタムフォークです。**暗号方式は上流のまま**（ed25519。ポスト量子化はしていません）で、
安定化修正と **Chain Finalization（チェーン確定高さ）** 機能を追加しています。
ポスト量子（PQC）実験は本リポジトリを土台にした別リポジトリ
[bnl-catapult-pqc](https://github.com/bootarou/bnl-catapult-pqc) で行っています。

## Chain Finalization Height（本フォークの主機能）

`config-network.properties` の `[chain]` に追加された **opt-in** 設定です。
チェーンが指定高さに到達すると**ブロック生成を停止し、それ以降のブロックを拒否**して
チェーンを読み取り専用の確定状態にします。有限で「完結する」チェーンを運用するための機能です。

```ini
[chain]
# Height(0) = 無効（従来どおり無限に伸びる。既定値・後方互換）
chainFinalizationHeight = 0
```

実装:

- **model**: `BlockchainConfiguration` にオプショナルな `ChainFinalizationHeight` を追加
  （`Height(0)` で無効化 = 後方互換）
- **harvesting**: `Harvester::harvest` は、候補ブロックの高さが確定高さを超える場合
  `nullptr` を返してブロック生成を停止
- **coresystem**: 新しい stateful validator `ChainFinalizationValidator` が確定高さ超過の
  ブロックを `Failure_Core_Chain_Finalization_Height_Exceeded` で拒否
- **tests**: 設定ロード・harvester・validator のテストを同梱

## 安定化修正（上流に対する主な fix）

- TLS ソケットの graceful close（Windows での RST ストーム抑止）
- Windows のハンドル解放遅延に対する RocksDB / state ディレクトリ操作のリトライ
- `LocalNode` boot/shutdown の再入ガード
- `BlockchainSyncConsumer` の null 参照修正
- `CatRealloc` の copyTo 引数順修正・解放済みプールスロットのワイプ
- mongo プラグインのテストソースのビルド修正
- ccache のオプトアウト（`NO_CCACHE`）ほか

## ブランチ構成

| ブランチ | 内容 |
|---|---|
| `main` | 上流 + 安定化修正 + Chain Finalization Height（本フォークの完成形） |
| `dev` | 上流 + 安定化修正のみ（Chain Finalization 追加前） |

## ビルド

上流 catapult と同一のツールチェーンです（[`client/catapult`](client/catapult) を参照）。
`symbolplatform/symbol-server-build-base` イメージ内でのビルドを推奨します。

## 関連リポジトリ

| | |
|---|---|
| [blockchain-network-launcher](https://github.com/bootarou/blockchain-network-launcher) | **BNL 本体** — カスタムブロックチェーンネットワークの起動・管理ツール（本フォークの利用元） |
| [bnl-catapult-pqc](https://github.com/bootarou/bnl-catapult-pqc) | 本リポジトリを土台にしたポスト量子（ML-DSA-44 / ML-KEM-768 / iVRF）実験フォーク |
| [symbol/symbol](https://github.com/symbol/symbol) | 派生元（上流）の Symbol モノレポ |
