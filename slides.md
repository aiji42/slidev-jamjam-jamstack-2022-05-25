---
theme: unicorn
highlighter: shiki
lineNumbers: false
layout: center
---

# Remixを導入してみた

## @aiji42_dev

---
layout: intro
introImage: 'https://storage.googleapis.com/zenn-user-upload/avatar/e738d28d01.jpeg'
---

# Who am I ?

## Uejima Aiji | @aiji42_dev

<br>

- 🏢 株式会社エイチームライフデザイン
- 🧘 リードエンジニア
- 🥑 最近の活動 
  - CloudflareでISRを実現したり
  - CSRなサイトをPrerenderでSSRぽくしたり
  - PrismaからSupabase APIを叩くミドルウェア作ったり
  - 社内でフロントエンド版ISSUCON開催したり

---
layout: image-center
image: 'https://remix.run/remix-v1.jpg'
---

# 今日はRemixを紹介したい

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# What is Remix ?

- React SSRフレームワーク
- React Routerの開発チームが開発を主導
- 昨年11月末にv1がリリースされたタイミングでパブリックに

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

#### Remixの特徴 ①
<br>

# loader と action

---

<div class="flex space-x-4">
<div class="flex-1">

```tsx
// app/routes/posts/$slug.tsx
export const loader = async ({ params }) => {
  const post = await db.post.findUnique({ 
    where: params.slug
  })
  
  return { post }
}

export const action = async ({ request, params }) => {
  if (request.method === 'POST') {
    await db.post.create({ ... })
  }
  if (request.method === 'DELETE') {
    await db.post.delete({ ... })
  }
  if (request.method === 'PATCH') {
    await db.post.update({ ... })
  }
}

const Page: FC = () => {
  const { post } = useLoaderData()
  return ...
}
export default Page


```

</div>

<div class="flex-1">

# loaderとaction

Next.js の getServerSideProps や API Routes と同じようなもの。

loaderでGETアクセス時のデータフェッチを定義し、actionでPOSTやDELETEなどのミューテーションを定義する。

ページからは useLoaderData でloaderデータを獲得する。

ページ(を構成するパーシャル)と同一ファイルに書くことができる。  
<small>※ Next.jsでもgetServerSidePropsでGET以外のリクエストを受けることができるが、拡張が必要であり、API Routesで受けるのが一般的</small>

</div>
</div>

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

#### Remixの特徴 ②
<br>

# File system routing と Nested Routing

レイアウトルート
共通処理

---


<div class="flex space-x-4">
<div class="flex-1">

```text
app/
├── routes/
│   ├── blog/
│   │   ├── $postId.tsx
│   │   ├── categories.tsx
│   │   ├── index.tsx
│   └── about.tsx
│   └── blog.tsx
│   └── index.tsx
└── root.tsx
```

</div>

<div class="flex-1">

ディレクトリ構成やファイル名がそのままURLになるという点は、Next.jsのpagesとよく似ている

</div>
</div>

---


<div class="flex space-x-4">
<div class="flex-1">

```text {7,9}
app/
├── routes/
│   ├── blog/
│   │   ├── $postId.tsx
│   │   ├── categories.tsx
│   │   ├── index.tsx
│   └── about.tsx
│   └── blog.tsx
│   └── index.tsx
└── root.tsx
```

</div>

<div class="flex-1">


| URL    | Matched Route        |
|--------|----------------------|
| /      | app/routes/index.tsx |
| /about | app/routes/about.tsx |


</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {4-6}
app/
├── routes/
│   ├── blog/
│   │   ├── $postId.tsx
│   │   ├── categories.tsx
│   │   ├── index.tsx
│   └── about.tsx
│   └── blog.tsx
│   └── index.tsx
└── root.tsx
```

</div>

<div class="flex-1">


| URL              | Matched Route                  |
|------------------|--------------------------------|
| /blog            | app/routes/blog/index.tsx      |
| /blog/categories | app/routes/blog/categories.tsx |
| /blog/my-post    | app/routes/blog/$postId.tsx    |


</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {3,8,10}
app/
├── routes/
│   ├── blog/
│   │   ├── $postId.tsx
│   │   ├── categories.tsx
│   │   ├── index.tsx
│   └── about.tsx
│   └── blog.tsx
│   └── index.tsx
└── root.tsx
```

</div>

<div class="flex-1">


| URL              | Matched Route                  | Layout               |
|------------------|--------------------------------|----------------------|
| /                | app/routes/index.tsx           | app/root.tsx         |
| /about           | app/routes/about.tsx           | app/root.tsx         |
| /blog            | app/routes/blog/index.tsx      | app/routes/blog.tsx  |
| /blog/categories | app/routes/blog/categories.tsx | app/routes/blog.tsx  |
| /blog/my-post    | app/routes/blog/$postId.tsx    | app/routes/blog.tsx  |


</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {3,7}
app/
├── routes/
│   ├── __authed/
│   │   ├── dashboard.tsx
│   │   └── $userId/
│   │   │   └── profile.tsx
│   ├── __authed.tsx
└── root.tsx
```

</div>

<div class="flex-1">

ダブルアンダースコアで始めると  
URL化されないレイアウトルートになる

</div>
</div>


<div class="flex space-x-4">
<div class="flex-1">

```text {4}
app/
├── routes/
│   ├── blog.tsx
│   ├── blog.$slug.tsx
└── root.tsx
```

</div>

<div class="flex-1">

ディレクトリ構造の代わりにドットでも表現可能

</div>
</div>

<div class="flex space-x-4">
<div class="flex-1">

```text {7}
app/
├── routes/
│   ├── blog/
│   │   ├── $postId.tsx
│   │   ├── categories.tsx
│   │   ├── index.tsx
│   └── $.tsx
└── root.tsx
```

</div>

<div class="flex-1">

Catch all route (*)

</div>
</div>

---

## ドキュメントで紹介されている例

<br>

<div class="flex space-x-4">
<div class="flex-1">

```text
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

</div>

<div class="flex-1">

![](/nested-route-0.png)

https://example.com/sales/invices/102000

こんな感じのダッシュボード

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {8}
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

```tsx
export default function Root() {
  return (
    <Sidebar>
      <Outlet />
    </Sidebar>
  )
}
```

</div>

<div class="flex-1">

![](/nested-route-1.png)

Outletコンポネント部分がレンダリング時に  
子レイアウト・子ページになる

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {3,7}
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

```tsx
export default function Sales() {
  return (
    <>
      <h1>Sles</h1>
      <Tabs />
      <Outlet />
    </>
  )
}
```

</div>

<div class="flex-1">

![](/nested-route-2.png)

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {4,6}
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

```tsx
export const loader = async () => {
  const overview = await fetch(...).then((res) => res.json())
  // ...
  return { overviewData, inviceListData }
}
export default function InvoiceList() {
  const { overviewData, inviceListData } = useLoaderData()
  return (
    <>
      <Overview data={overviewData}>
      <InvoiceList items={inviceListData} >
        <Outlet />
      </InvoiceList>
    </>
  )
}
```

</div>

<div class="flex-1">

![](/nested-route-3.png)

layoutにもloaderを設置可能

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {5}
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

```tsx
export const loader = async () => {
  const data = await fetch(...).then((res) => res.json())
  return { data }
}
export default function Invoice() {
  const { data } = useLoaderData()
  return (
    <InvoiceItem data={data} />
  )
}

export function ErrorBoundary({ error }) {
  return (
    <ErrorMessage>{error.message}</ErrorMessage>
  )
}
```

</div>

<div class="flex-1">

![](/nested-route-4.png)

各ページ・レイアウトにごとにErrorBoundaryを定義可能

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

```text {5}
app/
├── routes/
│   ├── sales/
│   │   ├── invoices/
│   │   │    └── $id.tsx
│   │   ├── invoices.tsx
│   └── sales.tsx
└── root.tsx
```

```tsx
export const loader = async () => {
  const data = await fetch(...).then((res) => res.json())
  return { data }
}
export default function Invoice() {
  const { data } = useLoaderData()
  return (
    <InvoiceItem data={data} />
  )
}

export function ErrorBoundary({ error }) {
  return (
    <ErrorMessage>{error.message}</ErrorMessage>
  )
}
```

</div>

<div class="flex-1">

![](/nested-route-error.png)

エラーの伝搬をその範囲に留めることができるため、  
フォールバックが最小限になる

</div>
</div>

---

## Nested Routeがあると何が嬉しいか

- 共通レイアウト
- ロジックの分散と共通化 
  - 分散により並列でデータフェッチ可能になり、高速化につながる
  - 特定ルート配下はログイン必須にするとか
  - __authed/ 配下にログイン必須ページを閉じ込めて __authed.tsx で認証チェック
    - (ダブルアンスコで始めているので、URLには現れない)
- ナビゲーション時に差分だけロードされる
  - /sales/invoices/1 から /sales/invoices/2 へ遷移する時、$id.tsxのloaderの再フェッチだけで済む

---

## ちょうど先日Next.jsにも同等な機能のRFCが公開された

![](/nextjs-rfc.png)
https://nextjs.org/blog/layouts-rfc

- 現段階ではおおよそRemixと同等の機能を揃えている
  - pathless と ErrorBoundaryに関しては言及なし
  - デフォルトでServerComponentになる (Remixでも同様に議論は起きている)

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

#### Remixの特徴 ③
<br>

# ErrorBoundary

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

#### Remixの特徴 ④
<br>

# マルチプラットフォーム

Next.jsみたいにVercelに依存しない
しかし単純比較はできない

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# 使ってみて良かったところ
- ステート管理ライブラリが基本的には不要
  - loader/actionがページではなくコンポネントに対して定義できる
  - cookieを取り扱うためのヘルパーが用意されている
  - 楽観的UIや、loader/actionとのfetcher、通信状態を管理するヘルパーが用意されている
  - クライアントで管理していたステートをサーバサイドへ移譲できる
    - RailsやPHP系のフレームワークに精通している人はとっつきやすいはず

---

# 使ってみて良かったところ
- エッジワーカー

---

# 使ってみて悪かったところ(苦しかったところ)

- Nested Routeは想像以上に難易度が高い
  - URL設計とディレクトリ設計をセットで行わなければならない
  - 良くも悪くもロジックが分散し、脳内メモリを圧迫する
  - 実際ネストは2階層くらいにとどめておくのがよい
    - 最初のサンプルに上げたような構成は正直無理

---

# 使ってみて悪かったところ(苦しかったところ)

Workersに限った話になるが
  
- UIライブラリ入れると1MBにバンドルサイズを抑えるのは結構きつい
  - SSRするのでUI系のライブラリもバンドルしないといけない
  - Cloudflare Workersにはバンドルサイズを1MB以下の制限がある
    - 申請すれば一応拡張可能ではあるのが、コールドスタートのレイテンシに影響する
  - Service Bindingを駆使して回避した
- Nodeではないので、動かないWorker非対応なライブラリは軒並み動かない
  - esbuildでpolyfillしたり、injectしたりなど職人芸が求められる
  - しかし、そもそもesbuildの拡張が不可能
    - 何度もDiscussionに起票されているが、コミュニティのポリシー的にNG
  - なので、自分でRemixのesbuild を拡張可能にするライブラリを書いた

---

# 使ってみて悪かったところ(苦しかったところ)

Workersに限った話になるが

- Edgeでレンダリングできるので早いとはいえ。。。
  - 結局データソースに起因する
    - DBのリージョンとか、同時接続数とか
  - KVとかDOとかCache APIをうまく駆使しなければならない
  - 今後D1が出てくればブレイクスルーしそう

## でもやっぱり、エッジでレンダリングできるのは夢がある

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# 向いているサービスやサイト

- React Routeで構築されたSPAサイトをSSRに移行したい、SEO対策したい
  - React Routeの思想をベースに構築されている
- 複数ページに渡るエントリーフォームを実装したい
  - 後述のremix-validate-formを導入するとサーバ - クライアント間でバリデーションを共有できる
  - 状態管理をloader/actionに任せられるので、ステートライブラリを入れなくてもよい
- 管理画面やダッシュボードをフルスクラッチしたい
  - React Adminの導入を試みたが、適したアダプターがなかったなど

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# remix-validate-form

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# remix-auth

