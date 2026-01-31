# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

PageIndex は、ベクトルDBやチャンキングを使わず、LLM推論ベースの階層ツリーインデックスを構築するRAGシステム。PDF/Markdownから目次構造を抽出し、ツリー検索で関連ページを特定する。

## コマンド

```bash
# 依存関係インストール
uv sync

# PDF処理
uv run python run_pageindex.py --pdf_path /path/to/document.pdf

# Markdown処理
uv run python run_pageindex.py --md_path /path/to/document.md

# Markdown処理（サマリ生成あり）
uv run python run_pageindex.py --md_path /path/to/document.md --if-add-node-summary yes
```

テスト・lint用の専用コマンドは未整備。

## 環境設定

- Python 3.11以上（`.python-version` で管理）
- パッケージ管理は `uv` を使用（`pyproject.toml` + `uv.lock`）
- `.env` ファイルに `CHATGPT_API_KEY` を設定（OpenAI APIキー）

## アーキテクチャ

### パッケージ構成 (`pageindex/`)

- **`page_index.py`** — PDF処理のコアロジック（TOC検出→ツリー構築→検証）。主要エントリポイントは `page_index_main()`
- **`page_index_md.py`** — Markdown処理。ヘッダ階層からツリー構築。エントリポイントは `md_to_tree()`
- **`utils.py`** — OpenAI API呼び出し（リトライ付き）、トークン計算、PDF文字列抽出、ツリー操作ユーティリティ
- **`config.yaml`** — デフォルト設定（モデル名、ノードあたりの最大ページ数/トークン数、サマリ生成有無など）

### 処理フロー

1. PDF/MDからテキスト抽出
2. TOCページ検出・構造抽出（LLM使用）
3. 階層ツリーインデックス生成（ページ番号マッピング付き）
4. LLMによる検証・修正（`fix_incorrect_toc()`）
5. ノードサマリ生成（非同期並行処理）
6. JSON出力（`./results/`）

### ツリーノード構造

```json
{
  "title": "Section Name",
  "node_id": "0006",
  "start_index": 21,
  "end_index": 22,
  "summary": "...",
  "nodes": []
}
```

### 設計上の特徴

- **非同期処理**: asyncioによるAPI並行呼び出し
- **リトライ**: OpenAI API呼び出しは最大10回リトライ
- **トークン制限**: ノードごとに最大トークン数を管理し、超過時は再帰的に分割
- **デフォルトモデル**: `gpt-4o-2024-11-20`

## 言語

作成および返信は日本語で行うこと。
