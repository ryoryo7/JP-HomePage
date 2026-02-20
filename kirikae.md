

# Codex 指示書：Language ドロップダウン切り替え実装

## 概要

日本語版・韓国語版の両ヘッダーにある言語切り替えリンク（`.lang-switch`）を、「Language ▾」ボタンのホバー式ドロップダウンメニューに置き換える。選択肢の言語名は英語表記（Japanese / Korean）とする。

---

## 対象ファイル

- `index.html`（日本語版）
- `ko/index.html`（韓国語版）

---

## タスク1：CSS の変更（両ファイル共通）

両ファイルの `<style>` 内にある以下の既存CSSを**削除**する：

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

削除した場所に以下のCSSを**追加**する：

```css
/* Language switcher */
.lang-menu {
    position: relative;
}
.lang-toggle {
    padding: 6px 16px;
    border: 1px solid rgba(188, 0, 45, 0.4);
    border-radius: 999px;
    font-size: 13px;
    font-weight: 600;
    color: var(--primary);
    background: transparent;
    cursor: pointer;
    transition: background-color 0.3s, color 0.3s;
    text-decoration: none;
    display: flex;
    align-items: center;
    gap: 6px;
    font-family: inherit;
}
.lang-toggle:hover {
    background-color: var(--primary);
    color: #fff;
}
.lang-toggle::after {
    content: "▾";
    font-size: 11px;
}
.lang-dropdown {
    display: none;
    position: absolute;
    top: calc(100% + 8px);
    left: 50%;
    transform: translateX(-50%);
    background: #fff;
    border: 1px solid rgba(188, 0, 45, 0.12);
    border-radius: 10px;
    box-shadow: 0 12px 32px rgba(0, 0, 0, 0.12);
    min-width: 150px;
    z-index: 200;
    overflow: hidden;
}
.lang-menu:hover .lang-dropdown,
.lang-menu:focus-within .lang-dropdown {
    display: block;
}
.lang-dropdown a {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 12px 18px;
    font-size: 13px;
    font-weight: 600;
    color: var(--ink);
    text-decoration: none;
    transition: background-color 0.2s;
    white-space: nowrap;
}
.lang-dropdown a:hover {
    background-color: #fff0f3;
    color: var(--primary);
}
.lang-dropdown a.active {
    color: var(--primary);
    background-color: #fff7f9;
    pointer-events: none;
}
```

---

## タスク2：日本語版 `index.html` のヘッダーHTML変更

ヘッダーナビ内の以下を：

```html
<a href="ko/" class="lang-switch">한국어</a>
```

以下に**置き換え**る：

```html
<div class="lang-menu">
    <button class="lang-toggle" aria-haspopup="true">Language</button>
    <div class="lang-dropdown">
        <a href="index.html" class="active">Japanese</a>
        <a href="ko/">Korean</a>
    </div>
</div>
```

変更後のヘッダーナビ全体は以下のようになる：

```html
<nav class="header-nav">
    <a href="#services">対応業務範囲</a>
    <a href="#pricing">報酬体系</a>
    <a href="#faq">FAQ</a>
    <div class="lang-menu">
        <button class="lang-toggle" aria-haspopup="true">Language</button>
        <div class="lang-dropdown">
            <a href="index.html" class="active">Japanese</a>
            <a href="ko/">Korean</a>
        </div>
    </div>
    <a href="#contact" class="header-cta">お問い合わせ</a>
</nav>
```

---

## タスク3：韓国語版 `ko/index.html` のヘッダーHTML変更

ヘッダーナビ内の以下を：

```html
<a href="../" class="lang-switch">日本語</a>
```

以下に**置き換え**る：

```html
<div class="lang-menu">
    <button class="lang-toggle" aria-haspopup="true">Language</button>
    <div class="lang-dropdown">
        <a href="../">Japanese</a>
        <a href="index.html" class="active">Korean</a>
    </div>
</div>
```

変更後のヘッダーナビ全体は以下のようになる：

```html
<nav class="header-nav">
    <a href="#services">업무 범위</a>
    <a href="#pricing">요금 체계</a>
    <a href="#faq">FAQ</a>
    <div class="lang-menu">
        <button class="lang-toggle" aria-haspopup="true">Language</button>
        <div class="lang-dropdown">
            <a href="../">Japanese</a>
            <a href="index.html" class="active">Korean</a>
        </div>
    </div>
    <a href="#contact" class="header-cta">문의하기</a>
</nav>
```

---

## 変更しないもの

- フッターの言語リンク（`한국어 사이트` / `日本語サイト`）はそのまま維持
- JavaScript は変更なし（ドロップダウンはCSS `:hover` / `:focus-within` で動作するためJSは不要）
- `ko/terms.html`、`ko/privacy.html`、`terms.html`、`privacy.html` は変更なし

---

## 注意事項

- `.lang-switch` クラスは両ファイルのCSS・HTMLの両方から完全に削除すること（残骸を残さない）
- `<button>` の `font-family: inherit;` を忘れないこと（ブラウザのデフォルトフォントが適用されるのを防ぐため）
- ドロップダウンの `z-index: 200` はヘッダーの `z-index: 100` より高いことを確認済み