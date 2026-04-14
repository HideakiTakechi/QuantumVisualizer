了解しました。
そのまま実装に使える形で、**VibeCoding向け・LLM ReadableなMarkdown仕様書（Ver1 + 拡張案）**を整理しました。

---

# 📄 量子ゲート可視化ツール URL連携仕様書（Ver1）

## 1. 概要

本仕様は、量子ゲート可視化ツールに対して
URLパラメータ経由で初期状態およびゲート列を指定し、
記事（note等）からの直接リンク起動を可能にするものである。

主目的：

* 記事とツールのシームレス連携
* 初学者の即時体験
* QCoder導入支援

---

## 2. 対象バージョン

* Version: **v1 (MVP)**
* 対象: 1量子ビットのみ

---

## 3. URLパラメータ仕様

### 3.1 パラメータ一覧

| パラメータ | 型      | 必須 | 説明           |
| ----- | ------ | -- | ------------ |
| init  | string | 任意 | 初期状態         |
| gates | string | 任意 | ゲート列（カンマ区切り） |
| title | string | 任意 | 表示タイトル       |
| mode  | string | 任意 | 動作モード        |

---

### 3.2 パラメータ詳細

#### init（初期状態）

| 値      | 状態   |
| ------ | ---- |
| zero   | |0⟩  |
| one    | |1⟩  |
| plus   | |+⟩  |
| minus  | |-⟩  |
| iplus  | |i⟩  |
| iminus | |-i⟩ |

デフォルト：

```text
zero
```

---

#### gates（ゲート列）

* カンマ区切り
* 順序通り適用

例：

```text
H
H,T
H,Z,H
X
```

許可ゲート：

| 記号 | 意味      |
| -- | ------- |
| I  | 恒等      |
| X  | NOT     |
| Y  | Yゲート    |
| Z  | Zゲート    |
| H  | アダマール   |
| S  | 位相（π/2） |
| T  | 位相（π/4） |

---

#### title（表示タイトル）

* 任意のテキスト
* URLエンコード必須

例：

```text
title=QPC001_基本例
```

---

#### mode（モード）

| 値      | 説明          |
| ------ | ----------- |
| drill  | 通常学習モード     |
| qcoder | QCoder想定モード |

デフォルト：

```text
drill
```

---

### 3.3 URL例

#### 基本例

```text
index.html?init=zero&gates=H
```

#### 複合例

```text
index.html?init=plus&gates=T
```

#### 回路パズル例

```text
index.html?init=zero&gates=H,Z,H
```

#### タイトル付き

```text
index.html?init=zero&gates=H&title=QPC001&mode=qcoder
```

---

## 4. 初期化処理仕様

### 4.1 処理フロー

1. URL取得
2. パラメータ解析
3. バリデーション
4. 内部状態へ変換
5. UI反映

---

## 5. バリデーション仕様（必須）

以下を必ず実装すること：

### 5.1 init

* 未定義 → デフォルト `zero`
* 不正値 → デフォルト `zero`

---

### 5.2 gates

* 未定義 → 空配列
* 不正ゲート → 除外
* 最大長制限（推奨：5〜10）

---

### 5.3 title

* 未定義 → 空文字
* 長すぎる場合 → 切り詰め

---

### 5.4 mode

* 未定義 → `drill`
* 不正値 → `drill`

---

### 5.5 エラー処理

不正入力があった場合：

* デフォルト値にフォールバック
* UIに軽微な通知（任意）

---

## 6. 共有リンク生成機能（Ver1対象）

### 6.1 概要

現在の状態をURL化しコピー可能にする

---

### 6.2 ボタン仕様

UIボタン：

```text
「この状態を共有」
または
「リンクをコピー」
```

---

### 6.3 出力内容

現在の

* init
* gates
* title
* mode

を含むURLを生成

例：

```text
index.html?init=plus&gates=H,T&mode=drill
```

---

### 6.4 動作

* クリックでクリップボードにコピー
* コピー成功通知表示

---

## 7. UI反映仕様

* 初期状態選択をURL値で上書き
* ゲート列を段階表示
* タイトル表示（任意）
* modeに応じてUI切替（軽量）

---

# 🧪 備考（拡張検討・参考実装）

---

## A. React実装サンプル

```javascript
const params = new URLSearchParams(window.location.search);

const init = params.get("init") || "zero";
const gatesRaw = params.get("gates") || "";
const title = params.get("title") || "";
const mode = params.get("mode") || "drill";

const allowedStates = ["zero","one","plus","minus","iplus","iminus"];
const allowedGates = ["I","X","Y","Z","H","S","T"];

// validation
const safeInit = allowedStates.includes(init) ? init : "zero";

const gateList = gatesRaw
  .split(",")
  .map(g => g.trim())
  .filter(g => allowedGates.includes(g))
  .slice(0, 10);
```

---

## B. 将来拡張パラメータ案

| パラメータ    | 用途        |
| -------- | --------- |
| autoplay | 自動再生      |
| step     | 表示ステップ    |
| desc     | 説明文       |
| preset   | 問題ID      |
| target   | 目標状態      |
| speed    | アニメーション速度 |

---

## C. QCoder練習モード拡張案

```text
?mode=qcoder&init=zero&target=plus
```

* 目標状態を提示
* ユーザーがゲート列を入力
* 正誤判定

---

## D. 記事連携設計パターン

記事中に以下を埋め込む：

```markdown
[この例を試す](index.html?init=zero&gates=H)
```

---

## E. UX改善案

* 最近開いた履歴保存
* よく使う例のプリセット
* 回路パターン集

---

## F. 表示改善案

* 回転角表示
* 状態式表示
* 赤道/極の強調

---
