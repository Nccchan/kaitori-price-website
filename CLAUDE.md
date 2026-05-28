# CLAUDE.md — にこにこ買取 価格表サイト

Claude Code 起動時に自動で読み込まれるファイルです。

---

## プロジェクト概要

- **サービス名**: にこにこ買取
- **内容**: トレーディングカード（ポケモン・ONE PIECE・ドラゴンボール・遊戯王）の買取価格表
- **本番URL**: https://kaitori-price.vercel.app/
- **査定・申込導線**: すべて LINE Bot (@niko_kaitori) に一本化

### 主要ファイル
- `index.html` — 価格表ページ
- `app.js` — 価格データ読み込み・タブ・検索・カード/表切替・「詳細」モーダル・募集ステータス
- `terms.html` — 利用規約（静的・アコーディオン）
- `style.css` — スタイル全般（LINE CTA / LINE FAB 含む）
- `admin.html` / `scripts/admin_server.mjs` — 管理画面（ローカルのみ・非公開）
- `scripts/build_manifest.mjs` — 画像マニフェスト生成

### 廃止済み（2026-05-29）
- カート・チェックアウト・受領票・申込変更フロー
- Recore API連携 (`api/submit-offer.js` / `api/update-offer.js`)
- Resend メール送信 (`api/send-receipt.js`)
- AIチャットボット (`api/chat.js`)
- Google Docs 規約iframe (`api/terms-html.js`) → `terms.html` 静的化
- eKYC / セルフィー / 銀行情報フォーム / LINEミニアプリ会員機能

すべての査定・申込・問い合わせは LINE Bot で処理する設計に切替。
顧客個人情報はこのサイト/サーバーで一切扱わない。

---

## アーキテクチャ

```
[ブラウザ]
  ├─ 価格表示   ← Google Sheets (GViz API) リアルタイム取得
  ├─ 募集ステータス ← Sheets「設定」シート A1=0/1
  └─ LINE誘導   → https://line.me/R/ti/p/@niko_kaitori
        ヘッダーCTA・フッターCTA・右下フローティングボタン (3箇所)
```

### 環境変数
- 不要（フロントエンドのみ。Recore/Resend/Anthropic キーはすべて廃止）

### 価格データソース
- Spreadsheet ID: `1PBMNNYHliomlgeNsvZgiccrfOWpIJbYPb9EMFtSAgdw`
- シート名: `ポケモン` / `ワンピース` / `ドラゴンボール` / `遊戯王` / `設定`
- 「設定」A1=0 で募集停止バナー表示（B1=任意メッセージ）

---

## カテゴリ別ラベル仕様

| カテゴリ | D列(c[3]) | E列(c[4]) | boxMode |
|--------|-----------|-----------|---------|
| ポケモン | シュリンクあり | シュリンクなし | (なし) |
| ワンピース | 箱 | カートン | true |
| ドラゴンボール | 箱 | カートン | true |
| 遊戯王 | 箱 | カートン | true |

---

## 商品・カテゴリ追加手順

### A. 既存カテゴリに商品を追加

1. **Google スプレッドシートに行を追加**
   - 該当シートに 商品コード・商品名・価格を入れる
   - 保存だけで本番サイトに即時反映（キャッシュなし）

2. **商品画像をGitHubにアップロード**
   - ファイル名 = 商品コード（例: `OP-15.png`）
   - アップロード先: `images/{pokemon|onepiece|dragonball|yugioh}/`
   - 推奨: 縦横比3:4・500KB前後

3. **manifest.json を更新**
   - Claude Code に「manifest を更新して」と依頼
   - または `node scripts/update-manifest.mjs && git commit && git push`

> GitHub Web UI でアップロードした画像は `main` に入る。
> 別ブランチで作業中の場合は `git cherry-pick` で取り込む。

画像なしでも動作する（`images/placeholder.svg`）。

### B. 新カテゴリを追加

1. シートを追加（例: `遊戯王`）
2. `app.js` の `CATEGORIES` にエントリ追加
3. `index.html` のタブに `<button class="tab" data-category="..." ...>` を追加
4. `images/{key}/` フォルダ作成・画像配置
5. `manifest.json` 更新
6. このCLAUDE.md「カテゴリ別ラベル仕様」表を更新

---

## 利用規約の更新方法

`terms.html` を直接編集する。バージョン番号と最終更新日は冒頭の `terms-page__meta` を更新。
（旧 Google Docs プロキシは廃止）

---

## 開発ルール

- **ブランチ**: `claude/` プレフィックス
- **デザイン**: シンプル・モバイルファースト
- 顧客の個人情報を扱わない（=この repo に PII を持ち込まない）
- 管理画面は**ローカルのみ**（`vercel.json` で `/admin` を 404）
- LINE誘導以外の申込導線は新設しない（混乱の元）

---

## 変更ログ（要点）

### 2026-05-29 — LINE一本化リファクタ
- カート・チェックアウト・受領票・Recore・Resend・AIチャット・規約iframeを全廃止
- ヘッダー右上・フッター・右下フローティング に LINE CTA を3箇所配置
- `terms.html` を静的化、アコーディオン形式
- `index.html`: 800→183行、`app.js`: 2130→549行に縮約

### 〜2026-02-28 (旧アーキテクチャ・参考用)
詳細は git history (`git log -- index.html app.js`) を参照。
カート/チェックアウト/Recore/Resend/AIチャット/eKYC/LINEミニアプリ会員 すべて実装→廃止の経緯あり。
