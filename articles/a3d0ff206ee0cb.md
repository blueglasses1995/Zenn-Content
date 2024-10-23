---
title: "Remix Route Convention"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
app/route/以下のディレクトリを切ることでどのようなルートが設定されるか確認。

前提
- root.tsxファイルは必須
  - 基本default exportがReactツリーのエントリーポイント
  - 任意でLayoutコンポーネントをnamed exportできる
    - この場合LayoutがReactツリーのエントリーポイント
    - childrenをpropsに持つ
    - default exportをchildrenに表示する
    - https://remix.run/docs/en/main/file-conventions/root#layout-export
- 各ファイルのdefault exportされたコンポーネントを見ている。

```
app/
├── routes/
│   ├── _index.tsx
│   └── about.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/about) return about.tsx
```

```
app/
├── routes/
│   ├── _index.tsx
│   ├── index.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/index) return index.tsx <- aboutと同じで「index」は固定値として扱われる
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies.new.tsx
│   └── movies.classic.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies/new) return movies.new.tsx <-ドットがスラッシュに置換される
if (/movies/classic) return movies.classic.tsx <-ドットがスラッシュに置換される
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies.new.tsx
│   └── movies.$movieId.tsx <- $プレフィックスで任意の値を示す
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies) {
    if (/) return movies._index.tsx
    if (/new) return movies.new.tsx <- 固定文字列のルートがある場合、そちらを優先して表示
    return movies.$movieId.tsx <-固定文字列のルートがない場合、$プレフィックスのルートを表示
}
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies._index.tsx
│   └── movies.new.tsx
│   └── movies.$movieId.tsx
│   └── movies.$movieId.detail.tsx 
│   └── movies.$movieId.$reviewId.tsx <- $プレフィックスを複数指定可能
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies) {
    if (/) return movies._index.tsx
    if (/new) return movies.new.tsx
    return movies.$movieId.tsx <- detail, 1などを後ろにつけても常にここに到達
}
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies._index.tsx
│   └── movies.new.tsx
│   └── movies.$movieId._index.tsx <- movies.$movieIdのルートのファイルとして_indexに改名する
│   └── movies.$movieId.detail.tsx 
│   └── movies.$movieId.$reviewId.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies) {
    if (/) return movies._index.tsx
    if (/new) return movies.new.tsx

    if (/$movieId/) return movies.$movieId.tsx
    if (/$movieId/detail) return movies.$movieId.detail.tsx
    return movies.$movieId.reviewId.tsx
}
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies._index.tsx
│   └── movies.new.tsx
│   └── movies.$movieId.tsx <- このコンポーネントの中にOutletを指定してあげるとレイアウトとして機能する
│   └── movies.$movieId._index.tsx
│   └── movies.$movieId.detail.tsx 
│   └── movies.$movieId.$reviewId.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies) {
    if (/) return movies._index.tsx
    if (/new) return movies.new.tsx

    <!-- 以下は全てmovies.$movieId.tsxのdefault exportのコンポーネントにラップされて表示される -->
    if (/$movieId/) return movies.$movieId.tsx
    if (/$movieId/detail) return movies.$movieId.detail.tsx
    return movies.$movieId.reviewId.tsx
}
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── movies._index.tsx
│   └── movies.new.tsx
│   └── movies.$movieId.tsx
│   └── movies.$movieId._index.tsx
│   └── movies_.$movieId.detail.tsx <- movies_とする
│   └── movies.$movieId.$reviewId.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/movies) {
    if (/) return movies._index.tsx
    if (/new) return movies.new.tsx

    <!-- movies.$movieId.tsxのdefault exportのコンポーネントにラップされない -->
    if (/$movieId/detail) return movies.$movieId.detail.tsx
    <!-- 以下は全てmovies.$movieId.tsxのdefault exportのコンポーネントにラップされて表示される -->
    if (/$movieId/) return movies.$movieId.tsx
    return movies.$movieId.reviewId.tsx
}
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── _auth.login.tsx
│   └── _auth.register.tsx
│   └── _auth.tsx <- レイアウト用
└── root.tsx
```
↓
```
if (/) return _index.tsx
<!-- _auth.tsxに包まれる -->
if (/login) return _auth.login.tsx
<!-- _auth.tsxに包まれる -->
if (/register) return _auth.register.tsx
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── ($lang)._auth.login.tsx
│   └── ($lang)._auth.register.tsx
│   └── ($lang)._auth.tsx <- 同様に($lang).をつけないとレイアウトとして扱えない
└── root.tsx
```
↓
```
if (/) return _index.tsx
<!-- _auth.tsxに包まれる -->
if (/login) return ($lang),_auth.login.tsx
<!-- _auth.tsxに包まれる -->
if (/register) return ($lang)._auth.register.tsx
<!-- _auth.tsxに包まれる -->
return Outletなし <-他条件でマッチしない任意のパスでOutletがない画面が表示されてしまうことに注意
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── ($lang)._auth.login.tsx
│   └── ($lang)._auth.register.tsx
│   └── ($lang)._auth._index.tsx <- 他条件でマッチしない任意のパスで表示するOutlet
│   └── ($lang)._auth.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
<!-- _auth.tsxに包まれる -->
if (/login) return ($lang),_auth.login.tsx
<!-- _auth.tsxに包まれる -->
if (/register) return ($lang)._auth.register.tsx
<!-- _auth.tsxに包まれる -->
return ($lang)._auth._index.tsx <- 他条件でマッチしない任意のパスでOutletがない画面が表示されてしまうことに注意
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── about.tsx
│   └── about.$.tsx <- $はスラッシュで区切られた残り全て
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/about/*) return about.tsx <- about以降のどのパスにもマッチするが、画面は出し分けられない、/aboutではメインコンテンツとして、それ以外はレイアウトとしてabout.tsxが表示される
```

```
app/
├── routes/
│   ├── _index.tsx
│   └── about._index.tsx <- _indexを加える
│   └── about.$.tsx
└── root.tsx
```
↓
```
if (/) return _index.tsx
if (/about) {
    if (/) return about._index.tsx
    return ($lang)._auth.register.tsx <- 画面を出し分けられる
}
```
