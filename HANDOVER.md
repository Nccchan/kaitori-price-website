# 引き継ぎ資料 — kaitori-price-website

**最終更新**: 2026-05-29
**作業者**: Claude Code (Opus 4)
**次の担当**: Claude Code（別ウェブアプリへ移行）

---

## 1. 現在の状態（一行サマリー）

公開価格表サイトを **LINE一本化リファクタ済み**。価格表示＋LINE誘導（@niko_kaitori_bot）のみ。カート/Recore/Resend/AIチャット/eKYC 全廃止済み。本番デプロイ完了。

- **本番URL**: https://kaitori-price.vercel.app/
- **規約URL**: https://kaitori-price.vercel.app/terms.html
- **最終commit**: `96d3b8f` LINE一本化リファクタ（main、push済み）
- **Vercel**: main push で自動デプロイ
- **作業ブランチ状態**: main、未コミット変更なし

---

## 2. ファイル構成（リファクタ後）

```
kaitori-price-website/
├── index.html            183行  価格表ページ（ヘッダー/タブ/検索/グリッド/LINE CTA×3）
├── app.js                549行  Sheets gviz取得・タブ・検索・カード/表切替・募集ステータス
├── terms.html            新規   利用規約14条（静的アコーディオン）
├── style.css             684行  既存 + .line-cta / .line-fab / .footer__links 追加
├── admin.html                   管理画面（ローカル専用、vercel.jsonで /admin を404化）
├── api/                  空     旧API全削除済み（chat/send-receipt/submit-offer/update-offer/terms-html）
├── images/                      商品画像
├── scripts/
│   ├── admin_server.mjs         ローカル管理サーバー（dev時のみ）
│   └── build_manifest.mjs       画像マニフェスト生成
├── vercel.json                  /admin を404化のみ
├── package.json                 @anthropic-ai/sdk / express / multer / sharp（admin用）
├── CLAUDE.md                    新アーキ反映済み
└── HANDOVER.md                  ← このファイル
```

---

## 3. 完了したこと（今回のリファクタ）

- [x] カート/チェックアウト/受領票/30分以内変更フロー UI削除
- [x] Recore API（submit-offer / update-offer）削除
- [x] Resend メール送信（send-receipt）削除
- [x] AIチャットボット（api/chat.js）削除 → LINEへ誘導
- [x] Google Docs 規約iframe（api/terms-html.js）削除 → terms.html 静的化
- [x] eKYC / セルフィー / 銀行情報 / LINEミニアプリ会員 削除
- [x] ヘッダー右上 LINE CTA 追加
- [x] フッター LINE CTA + 規約リンク 追加
- [x] 右下フローティングLINEボタン（にこにこちゃんアイコン）追加
- [x] terms.html 静的化（第1〜14条アコーディオン）
- [x] index.html: 800→183行
- [x] app.js: 2130→549行
- [x] vercel.json で /admin を404化
- [x] CLAUDE.md 更新
- [x] memory: reference_kaitori_price_website.md + MEMORY.md インデックス
- [x] commit 96d3b8f / main push / Vercel auto-deploy

---

## 4. 廃止済み機能（戻さないこと）

⚠️ **以下は明確に廃止判断済み**。「あったほうがいい」発想で復活させない：

- カート / チェックアウト / 受領票
- 30分以内の申込変更フロー
- Recore API 連携（査定情報の自社DB送信）
- Resend メール送信
- AIチャットボット（@niko_kaitori_bot に統合済み）
- Google Docs 規約iframe
- eKYC / セルフィー撮影
- 銀行口座情報フォーム
- LINEミニアプリ 会員登録

→ **査定・申込・問い合わせは LINE Bot で完結させる方針**。
→ **顧客個人情報はこのサイト/サーバーで一切扱わない**。

---

## 5. データソース

- **Spreadsheet ID**: `1PBMNNYHliomlgeNsvZgiccrfOWpIJbYPb9EMFtSAgdw`
- **シート**: `ポケモン` `ワンピース` `ドラゴンボール` `遊戯王` `設定`
- **募集停止バナー**: 「設定」シート A1=0（停止）、B1=メッセージ
- **取得方法**: GViz API、フロントから直接（環境変数不要）

---

## 6. LINE誘導URL（3箇所すべて同一）

```
https://line.me/R/ti/p/@niko_kaitori
```

配置場所：
1. ヘッダー右上 CTA
2. フッター右側 CTA
3. 右下フローティングボタン（旧AIチャット位置）

---

## 7. カテゴリ別ラベル仕様

| カテゴリ | D列(c[3]) | E列(c[4]) | boxMode |
|--------|-----------|-----------|---------|
| ポケモン | シュリンクあり | シュリンクなし | （なし） |
| ワンピース | 箱 | カートン | true |
| ドラゴンボール | 箱 | カートン | true |
| 遊戯王 | 箱 | カートン | true |

---

## 8. 関連ナレッジ（メモリ）

| memory file | 内容 |
|-------------|------|
| `reference_kaitori_price_website.md` | 本サイトの全体像 |
| `reference_line_bot_v2.md` | @niko_kaitori_bot 運用情報 |
| `reference_public_kohyo_url.md` | 公開価格表URL（app.aigive.jp/pricing/kohyo） |
| `reference_pricing_app_repo.md` | aigive-pricing repo構成 |

---

## 9. 関連リポジトリ・サービス

| サービス | URL / 場所 |
|---------|-----------|
| 本サイト repo | Nccchan/kaitori-price-website |
| 価格管理アプリ | Nccchan/aigive-pricing（別repo） |
| LINE Bot | scripts/line-bot/（org repo） |
| Discord Bot | scripts/discord-bot/（org repo） |
| Shopify | https://japantcjshop.myshopify.com/ |
| ⑥入出庫記録（在庫SSoT） | Supabase movements + zaiko |

---

## 10. 次の作業者へ（チェックリスト）

新規修正を始める前に：

- [ ] `CLAUDE.md` を読む（このサイト専用ルール）
- [ ] `git log -10` で最近の変更を確認
- [ ] `index.html` `app.js` `terms.html` が現行
- [ ] **廃止済み機能を復活させない** （§4参照）
- [ ] 顧客PIIをこのサーバーに持ち込まない
- [ ] 管理画面は /admin で404、ローカルのみ
- [ ] 価格データはシートが真実の源。サイト側でキャッシュ・保存しない

不可逆な変更（DB / 外部送信 / 大規模UI改修）は**なつきに事前確認**。
