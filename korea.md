

サイトの構造、フォーム送信の仕組み、config.jsの依存関係、画像参照パスなどを踏まえて、韓国語版の仕様をまとめます。

---

# 韓国語版ページ 構造仕様書

## 1. ディレクトリ構造

```
JP-HomePage/
├── index.html              ← 日本語版（既存・変更あり）
├── terms.html              ← 日本語 利用規約（変更あり）
├── privacy.html            ← 日本語 プライバシーポリシー（変更あり）
├── config.js               ← 共有（変更なし）
├── abc.md
├── CNAME
├── images/                 ← 共有画像（変更なし）
│   ├── jboundex_logo.png
│   ├── goal.jpg
│   ├── zoom.jpg
│   ├── AI.jpg
│   ├── エクセル.jpg
│   ├── チャット.jpg
│   └── 握手.jpg
│
└── ko/                     ← 韓国語版（NEW）
    ├── index.html           ← 韓国語版トップ
    ├── terms.html           ← 韓国語版 利用規約
    └── privacy.html         ← 韓国語版 プライバシーポリシー
```

**`ko/images/` は作らない。** 現在の画像（ロゴ、写真）はすべて言語非依存なので、`../images/` で参照して重複を避ける。韓国語テキストが埋め込まれた画像が将来必要になった場合のみ `ko/images/` を新設する。

---

## 2. HTML変更仕様

### 2-1. `<html>` タグ

| 版 | 指定 |
|---|---|
| 日本語版 | `<html lang="ja">` （既存のまま） |
| 韓国語版 | `<html lang="ko">` |

### 2-2. hreflang タグ（全ページ共通で `<head>` に追加）

日本語版・韓国語版の**両方すべてのページ**の `<head>` 内に、以下3行を追加する。

```html
<!-- index.html の場合 -->
<link rel="alternate" hreflang="ja" href="https://（ドメイン）/" />
<link rel="alternate" hreflang="ko" href="https://（ドメイン）/ko/" />
<link rel="alternate" hreflang="x-default" href="https://（ドメイン）/" />
```

```html
<!-- terms.html の場合 -->
<link rel="alternate" hreflang="ja" href="https://（ドメイン）/terms.html" />
<link rel="alternate" hreflang="ko" href="https://（ドメイン）/ko/terms.html" />
<link rel="alternate" hreflang="x-default" href="https://（ドメイン）/terms.html" />
```

```html
<!-- privacy.html の場合 -->
<link rel="alternate" hreflang="ja" href="https://（ドメイン）/privacy.html" />
<link rel="alternate" hreflang="ko" href="https://（ドメイン）/ko/privacy.html" />
<link rel="alternate" hreflang="x-default" href="https://（ドメイン）/privacy.html" />
```

**`x-default` は日本語版を指定**（メイン市場が日本のため）。

### 2-3. 言語切り替えリンク（ヘッダーに追加）

Google公式が「自動リダイレクトはするな、ユーザーが自分で切り替えられるようにせよ」と明記しているため、両言語版のヘッダーに言語切り替えを設置する。

**日本語版ヘッダー：**
```html
<nav class="header-nav">
    <a href="#services">対応業務範囲</a>
    <a href="#pricing">報酬体系</a>
    <a href="#faq">FAQ</a>
    <a href="ko/" class="lang-switch">한국어</a>       <!-- NEW -->
    <a href="#contact" class="header-cta">お問い合わせ</a>
</nav>
```

**韓国語版ヘッダー：**
```html
<nav class="header-nav">
    <a href="#services">업무 범위</a>
    <a href="#pricing">요금 체계</a>
    <a href="#faq">FAQ</a>
    <a href="../" class="lang-switch">日本語</a>        <!-- NEW -->
    <a href="#contact" class="header-cta">문의하기</a>
</nav>
```

**追加CSS（両版共通）：**
```css
.lang-switch {
    padding: 4px 14px;
    border: 1px solid rgba(188, 0, 45, 0.4);
    border-radius: 999px;
    font-size: 13px;
    color: var(--primary) !important;
    transition: background-color 0.3s, color 0.3s;
}
.lang-switch:hover {
    background-color: var(--primary);
    color: #fff !important;
}
```

---

## 3. パス参照のルール

韓国語版（`ko/` 配下）からの相対パス変換表：

| 対象 | 日本語版（既存） | 韓国語版（`ko/index.html` から） |
|---|---|---|
| 画像 | `images/AI.jpg` | `../images/AI.jpg` |
| ファビコン | `images/jboundex_logo.png` | `../images/jboundex_logo.png` |
| config.js | `config.js` | `../config.js` |
| ロゴリンク | `index.html` | `index.html`（ko内のトップ） |
| プライバシーポリシー | `privacy.html` | `privacy.html`（ko内のものへ） |
| 利用規約 | `terms.html` | `terms.html`（ko内のものへ） |
| 日本語版トップへ | ー | `../index.html` または `../` |

---

## 4. フォーム送信の変更

### 4-1. フォームID・hidden値

```html
<!-- 韓国語版 -->
<form id="contact-form-ko" method="post">
    <input type="hidden" name="lang" value="ko">
    ...
```

既存の `config.js` を `../config.js` で読み込み、同じ `formAction` エンドポイントに送信する。`name="lang"` の値が `ko` になるので、Google Apps Script側（受信側）で日韓を判別できる。

### 4-2. JavaScript のセレクタ変更

韓国語版のスクリプトブロックで、フォームセレクタを変更する。

```javascript
const form = document.querySelector('#contact-form-ko');
```

送信完了・エラーのアラートメッセージも韓国語にする。

```javascript
alert('전송이 완료되었습니다. 감사합니다.');  // 送信完了
alert('전송에 실패했습니다. 잠시 후 다시 시도해주세요.');  // 送信失敗
```

ボタンテキストも連動させる。

```javascript
submitButton.textContent = '전송 중...';  // 送信中
submitButton.textContent = '전송하기';     // 送信する（初期・リセット後）
```

---

## 5. `<title>` と `<meta>` タグ

### 韓国語版 index.html

```html
<title>라쿠텐・Amazon 운용 대행 서비스 | JBoundex | 이익 중심 운용 지원</title>
<meta name="description" content="매출이 아닌 이익을 기준으로 라쿠텐・Amazon 사업을 성장시킵니다. 광고, 페이지, 재고까지 일괄 운용 대행.">
```

### 日本語版 index.html に追加

```html
<meta name="description" content="売上ではなく利益を軸に、楽天・Amazon事業を成長させます。広告・ページ・在庫まで丸投げで一括運用代行。">
```

（現在 description が未設定なので、この機会に日本語版にも追加を推奨）

---

## 6. フッター

### 韓国語版のフッターリンク

```html
<ul class="footer-list">
    <li><a href="privacy.html">개인정보 처리방침</a></li>
    <li><a href="terms.html">이용약관</a></li>
    <li><a href="index.html">홈페이지</a></li>
    <li><a href="../">日本語サイト</a></li>   <!-- 日本語版へのリンク -->
</ul>
```

### 日本語版のフッターにも追加

```html
<li><a href="ko/">한국어 사이트</a></li>
```

---

## 7. 翻訳対象の洗い出し

翻訳が必要なテキスト要素の一覧：

| 区分 | 内容 |
|---|---|
| **ヘッダー** | ナビゲーションテキスト4項目、CTAボタン |
| **ヒーロー** | 見出し、説明文、バッジ4つ、CTAボタン |
| **課題セクション** | 見出し、サブタイトル、カード4枚 |
| **運用方針セクション** | 見出し、サブタイトル、カード3枚（各h3・p・li） |
| **対応業務範囲** | 見出し、サブタイトル、サービスカード9枚（全リスト項目） |
| **運用開始までの流れ** | 見出し、サブタイトル、ステップ5つ |
| **実績・導入事例** | 見出し、サブタイトル、事例カード2枚（ラベル・タイトル・本文・指標・注記） |
| **報酬体系** | 見出し、サブタイトル、プランカード2枚、粗利定義の注記 |
| **価格理由セクション** | 見出し、サブタイトル、カード2枚 |
| **FAQ** | 見出し、サブタイトル、Q&A 4組 |
| **最終CTA** | 見出し、説明文 |
| **お問い合わせ** | 見出し、説明文、会社情報、フォームラベル全項目、select option全項目、同意文、送信ボタン |
| **フッター** | リンクテキスト、コピーライト |
| **terms.html** | 全文 |
| **privacy.html** | 全文 |

**翻訳しないもの：** ブランド名「JBoundex」、住所（日本語のまま）、メールアドレス、CSS、JavaScript のロジック部分、画像ファイル

---

## 8. 韓国語版で調整が必要な固有事項

### 8-1. 通貨表記

日本語版の「¥」「万円」表記はそのまま円建てで残す（日本市場向けサービスのため）。ただし韓国の方にわかりやすいよう、初出時に補足を入れることを検討する。

```
월액: 15만 엔(약 150만 원)~
```

### 8-2. フォームの「商品カテゴリ」select options

韓国語に翻訳する。

```html
<option value="">카테고리 선택</option>
<option value="beauty">뷰티・퍼스널 케어</option>
<option value="health">헬스・웰니스</option>
<option value="electronics">전자기기</option>
<option value="home">홈・키친</option>
<option value="food">식품・음료</option>
<option value="fashion">패션・의류</option>
<option value="other">기타</option>
```

### 8-3. 「海外からの輸入対応」セクション

韓国企業からの問い合わせを想定すると、このセクションは特に重要。韓国語版では「한국에서 일본으로의 수출을 지원합니다」のように、韓国→日本の文脈であることを明確にする。

### 8-4. フォント

`font-family` に韓国語対応フォントを追加する。

```css
/* 韓国語版のbody */
body {
    font-family: 'Segoe UI', 'Malgun Gothic', '맑은 고딕', 'Apple SD Gothic Neo', sans-serif;
}
```

---

## 9. 作業手順（推奨順序）

| 順番 | 作業 | 対象ファイル |
|---|---|---|
| 1 | `ko/` ディレクトリ作成 | リポジトリ |
| 2 | `index.html` を `ko/index.html` にコピー | ko/index.html |
| 3 | `<html lang="ko">` に変更、パス書き換え（`../images/` 等） | ko/index.html |
| 4 | テキスト翻訳（全セクション） | ko/index.html |
| 5 | フォームID・hidden値・JSアラート文を韓国語版に変更 | ko/index.html |
| 6 | font-family を韓国語フォントに変更 | ko/index.html |
| 7 | `terms.html` → `ko/terms.html` を翻訳して作成 | ko/terms.html |
| 8 | `privacy.html` → `ko/privacy.html` を翻訳して作成 | ko/privacy.html |
| 9 | 日本語版の全ページに hreflang タグ追加 | index.html, terms.html, privacy.html |
| 10 | 韓国語版の全ページに hreflang タグ追加 | ko/index.html, ko/terms.html, ko/privacy.html |
| 11 | 日本語版ヘッダー・フッターに韓国語版リンク追加 | index.html |
| 12 | 日本語版に `<meta name="description">` 追加 | index.html |
| 13 | 動作確認（リンク、フォーム送信、画像表示） | 全ファイル |
| 14 | Git commit & push | リポジトリ |

---

---



# Codex 指示書：韓国語版ページ作成

## 概要

日本語版サイト（`index.html`, `terms.html`, `privacy.html`）をベースに、韓国語版を `ko/` サブディレクトリに作成する。同時に、日本語版にも hreflang タグ・言語切り替えリンク等の改修を行う。

---

## 前提

- リポジトリ：`JP-HomePage`
- ホスティング：GitHub Pages（カスタムドメインあり、CNAME参照）
- 既存ファイル構成は仕様書の通り共有済み
- `config.js` は日韓共通で使用する（変更不要）
- 画像は `images/` を共有参照する（`ko/images/` は作らない）
- ブランド名「JBoundex」、住所、メールアドレス、数値（売上実績等）は翻訳しない
- CSSは各HTMLの `<style>` 内にインラインで記述されている（外部CSSファイルは存在しない）

---

## タスク一覧

### タスク1：`ko/index.html` の作成

`index.html` をコピーし、以下をすべて適用する。

**1-1. `<html>` タグ**
```html
<html lang="ko">
```

**1-2. `<head>` 内**

```html
<title>라쿠텐・Amazon 운용 대행 서비스 | JBoundex | 이익 중심 운용 지원</title>
<meta name="description" content="매출이 아닌 이익을 기준으로 라쿠텐・Amazon 사업을 성장시킵니다. 광고, 페이지, 재고까지 일괄 운용 대행.">
```

hreflang タグを追加（`<meta charset>` の直後あたりに配置）：
```html
<link rel="alternate" hreflang="ja" href="https://jboundex.com/" />
<link rel="alternate" hreflang="ko" href="https://jboundex.com/ko/" />
<link rel="alternate" hreflang="x-default" href="https://jboundex.com/" />
```

ファビコンのパスを修正：
```html
<link rel="icon" type="image/png" href="../images/jboundex_logo.png">
```

**1-3. CSS の font-family 変更**

`body` の `font-family` を以下に変更：
```css
font-family: 'Segoe UI', 'Malgun Gothic', '맑은 고딕', 'Apple SD Gothic Neo', sans-serif;
```

言語切り替えリンク用CSSを `<style>` 内に追加：
```css
.lang-switch {
    padding: 4px 14px;
    border: 1px solid rgba(188, 0, 45, 0.4);
    border-radius: 999px;
    font-size: 13px;
    color: var(--primary) !important;
    transition: background-color 0.3s, color 0.3s;
}
.lang-switch:hover {
    background-color: var(--primary);
    color: #fff !important;
}
```

**1-4. 画像パス**

すべての画像参照を `../images/` に書き換える。対象：
- `images/jboundex_logo.png` → `../images/jboundex_logo.png`
- `images/エクセル.jpg` → `../images/エクセル.jpg`
- `images/握手.jpg` → `../images/握手.jpg`
- `images/チャット.jpg` → `../images/チャット.jpg`
- `images/AI.jpg` → `../images/AI.jpg`
- `images/goal.jpg` → `../images/goal.jpg`
- `images/zoom.jpg` → `../images/zoom.jpg`

**1-5. `config.js` のパス**

```html
<script src="../config.js"></script>
```

**1-6. ヘッダー**

```html
<div class="header-content">
    <a href="index.html" class="logo">JBoundex</a>
    <nav class="header-nav">
        <a href="#services">업무 범위</a>
        <a href="#pricing">요금 체계</a>
        <a href="#faq">FAQ</a>
        <a href="../" class="lang-switch">日本語</a>
        <a href="#contact" class="header-cta">문의하기</a>
    </nav>
</div>
```

**1-7. 全テキストの韓国語翻訳**

以下のルールで全セクションを翻訳する。

翻訳対象：
- ヒーローセクション（h1、p、バッジ4つ、CTAボタン）
- 課題セクション（h2、サブタイトル、カード4枚）
- 運用方針セクション（h2、サブタイトル、カード3枚の見出し・説明文・リスト項目）
- 対応業務範囲セクション（h2、サブタイトル、サービスカード9枚の見出し・全リスト項目）
- 運用開始までの流れ（h2、サブタイトル、ステップ5つ）
- 実績・導入事例（h2、サブタイトル、事例カード2枚のラベル・タイトル・本文・指標ラベル・注記・キャプション）
- 報酬体系（h2、サブタイトル、プランカード2枚、粗利定義の注記）
- 価格理由セクション（h2、サブタイトル、カード2枚）
- FAQ（h2、サブタイトル、Q&A 4組）
- 最終CTA（h2、p）
- お問い合わせセクション（h2、説明文、会社情報ラベル、フォームラベル全項目、select option全項目、同意文、送信ボタン）
- フッター（リンクテキスト、コピーライト）
- 画像の `alt` 属性テキスト

翻訳しないもの：
- 「JBoundex」
- 住所「〒160-0023 東京都新宿区西新宿三丁目3番13号 西新宿水間ビル6階」
- メールアドレス「contact@jboundex.com」
- 数値（120万、650万、80万、650%、363%、500%、8ヶ月、3ヶ月、6点、1点 等）
- CSSクラス名、id名
- HTMLの構造・属性名

通貨表記は円建てのまま残し、初出箇所に韓国ウォン概算を併記する：
```
월액: 15만 엔(약 150만 원)~
```

「海外からの輸入対応」セクションは、韓国→日本の輸出支援の文脈であることが伝わる訳にする。

**1-8. フォーム**

フォームID と hidden 値を変更：
```html
<form id="contact-form-ko" method="post">
    <input type="hidden" name="lang" value="ko">
```

フォーム内のリンクを韓国語版に向ける：
```html
<a href="privacy.html" target="_blank" rel="noopener noreferrer">개인정보 처리방침</a>
```

**1-9. JavaScript**

フォームのセレクタを変更：
```javascript
const form = document.querySelector('#contact-form-ko');
```

アラートメッセージとボタンテキストを韓国語にする：
```javascript
// 送信完了
alert('전송이 완료되었습니다. 감사합니다.');
// 送信失敗
alert('전송에 실패했습니다. 잠시 후 다시 시도해주세요.');
// 送信先エラー
alert('전송처 설정에 문제가 있습니다. 잠시 후 다시 시도해주세요.');
// ボタンテキスト（送信中）
submitButton.textContent = '전송 중...';
// ボタンテキスト（初期・リセット後）
submitButton.textContent = '전송하기';
```

スムーススクロールとFAQアコーディオンのJSロジックはそのまま維持。

**1-10. フッター**

```html
<ul class="footer-list">
    <li><a href="privacy.html">개인정보 처리방침</a></li>
    <li><a href="terms.html">이용약관</a></li>
    <li><a href="index.html">홈페이지</a></li>
    <li><a href="../">日本語サイト</a></li>
</ul>
```
```html
<p class="footer-copy">&copy; 2025 JBoundex. All rights reserved.</p>
```

---

### タスク2：`ko/terms.html` の作成

既存の `terms.html` をコピーし以下を適用：
- `<html lang="ko">`
- `<head>` に hreflang タグ（terms.html 用のもの）追加
- 全テキストを韓国語に翻訳
- パス（画像、config.js）を `../` 付きに修正
- ヘッダー・フッターがある場合はタスク1と同じ言語切り替え構造にする
- font-family を韓国語版に変更

---

### タスク3：`ko/privacy.html` の作成

既存の `privacy.html` をコピーし以下を適用：
- `<html lang="ko">`
- `<head>` に hreflang タグ（privacy.html 用のもの）追加
- 全テキストを韓国語に翻訳
- パス（画像、config.js）を `../` 付きに修正
- ヘッダー・フッターがある場合はタスク1と同じ言語切り替え構造にする
- font-family を韓国語版に変更

---

### タスク4：日本語版 `index.html` の改修

既存の `index.html` に以下を追加。既存コンテンツの翻訳や削除はしない。

**4-1. `<head>` に追加**
```html
<link rel="alternate" hreflang="ja" href="https://jboundex.com/" />
<link rel="alternate" hreflang="ko" href="https://jboundex.com/ko/" />
<link rel="alternate" hreflang="x-default" href="https://jboundex.com/" />
<meta name="description" content="売上ではなく利益を軸に、楽天・Amazon事業を成長させます。広告・ページ・在庫まで丸投げで一括運用代行。">
```

**4-2. `<style>` 内に追加**
```css
.lang-switch {
    padding: 4px 14px;
    border: 1px solid rgba(188, 0, 45, 0.4);
    border-radius: 999px;
    font-size: 13px;
    color: var(--primary) !important;
    transition: background-color 0.3s, color 0.3s;
}
.lang-switch:hover {
    background-color: var(--primary);
    color: #fff !important;
}
```

**4-3. ヘッダーナビに言語切り替えリンクを追加**

`<a href="#faq">FAQ</a>` の直後に挿入：
```html
<a href="ko/" class="lang-switch">한국어</a>
```

**4-4. フッターに追加**

`footer-list` の `</ul>` の直前に挿入：
```html
<li><a href="ko/">한국어 사이트</a></li>
```

---

### タスク5：日本語版 `terms.html` の改修

`<head>` に hreflang タグ（terms.html 用）を追加。それ以外の変更はしない。

---

### タスク6：日本語版 `privacy.html` の改修

`<head>` に hreflang タグ（privacy.html 用）を追加。それ以外の変更はしない。

---

## hreflang ドメインについて

上記では `https://jboundex.com` をプレースホルダとして使っている。実際のドメインは `CNAME` ファイルの内容を確認して、正しいドメインに置き換えること。

---

## 注意事項

- `ko/` 配下に `config.js` や `images/` をコピーしないこと
- 日本語版のCSS・JavaScript ロジック・HTML構造は変更しないこと（追加のみ）
- 韓国語版のデザイン・レイアウト・配色・アニメーションは日本語版と完全に同一にすること
- フォームの `name` 属性値（`company`, `person`, `email`, `website`, `category`, `sales`, `goals`, `contact-method`, `privacy-consent`）は変更しないこと（受信側スクリプトとの整合性のため）
- select の `value` 属性（`beauty`, `health`, `electronics` 等）も変更しないこと。表示テキストのみ翻訳する
