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
  - 上記のような再フェッチを自動的に実行してくれる

---

## ちょうど先日Next.jsにも同等な機能のRFCが公開された

![](/nextjs-rfc.png)
https://nextjs.org/blog/layouts-rfc

- 現段階ではおおよそRemixと同等の機能を揃えている
  - pathlessやErrorBoundaryに関しても、「次の発表を待て」とのこと
    - かなりRemixのノウハウが意思決定に影響を及ぼしている印象
  - デフォルトでServerComponentになる (Remixでも同様に議論は起きている)

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
layout: cover-logos
logos: [
'https://cdn.worldvectorlogo.com/logos/cloudflare-1.svg',
'https://cdn.worldvectorlogo.com/logos/express-109.svg',
'https://cdn.worldvectorlogo.com/logos/deno-1.svg',
'https://cdn.worldvectorlogo.com/logos/nodejs.svg',
'https://cdn.worldvectorlogo.com/logos/cloudflare-pages-single.svg',
'https://cdn.worldvectorlogo.com/logos/netlify.svg',
'https://cdn.worldvectorlogo.com/logos/vercel.svg',
'https://seeklogo.com/images/F/fly-io-logo-BD23E4EA17-seeklogo.com.png'
]
---

#### Remixの特徴 ③
<br>

# マルチランタイム

<style>
li {
 font-size: medium;
}
</style>

- Node / web worker / Deno
- Vercel / Netlify / Cloudflare Workers・Pages, etc
  - セルフホストも標準サポート
- Next.jsとVervelみたいな関係性のものはない

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

<div class="flex space-x-4">
<div class="flex-1">
<br>


#### Remixの特徴 ④
<br>

# Formとヘルパー

前述のactionと通信を行うための、Formコンポネントや多数のヘルパーを備えている

特に **useTransition** はフォームのsubmitの状態(待機・サブミット・ロード)を管理したり、サブミット中のデータを取り扱うことが可能。

これにより楽観的UIの構築も用意になる。

</div>
<div class="flex-1">
<video controls autoplay>
  <source src="https://remix.run/jokes-tutorial/img/optimistic-ui.mp4" type="video/mp4">
</video>
</div>
</div>

---

remix-validated-form

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

#### Remixの特徴 ⑤

# Cookieヘルパー

CookieやSessionを取り扱うためのヘルパーが標準装備

```ts
import { createCookieSessionStorage } from "@remix-run/node"; // or "@remix-run/cloudflare"

const { getSession, commitSession, destroySession } =
  createCookieSessionStorage({
    // a Cookie from `createCookie` or the CookieOptions to create one
    cookie: {
      name: "__session",

      // all of these are optional
      domain: "remix.run",
      expires: new Date(Date.now() + 60_000),
      httpOnly: true,
      maxAge: 60,
      path: "/",
      sameSite: "lax",
      secrets: ["s3cret1"],
      secure: true,
    },
  });

export { getSession, commitSession, destroySession };
```

loader/actionと組み合わせることでステート管理をサーバ側へ移譲することができる。

ロジックがフロントに露出しないため、バンドルサイズも削減できる。

例えば Supabase と組み合わせれば、クライアント側に認証ロジックやデータを一切持たないということも可能。

---

remix-auth

---
logoHeader: 'https://blog.stackblitz.com/img/quotes/logo-remix.svg'
---

# 使ってみて良かったところ

- ステート管理ライブラリが不要になりクライアントコードが小さくなる
  - さらにきめ細かく再フェッチできるので、低コストで情報更新できる

---

# 使ってみて良かったところ
- エッジレンダリングはたしかに早い
  - KV使わなくても、200-300msで応答できる
  - KV使えば100ms切る
  - 同一構成の Next.js on Vercel のSSRで300-500ms
  - (もちろんデータソースに起因する)

---

# 使ってみて悪かったところ(苦しかったところ)

- Nested Routeは想像以上に難易度が高い
  - URL設計とディレクトリ設計をセットで行わなければならない
  - 良くも悪くもロジックが分散し、脳内メモリを圧迫する
  - 実際ネストは2階層くらいにとどめておくのがよい
    - 最初のサンプルに上げたような構成は正直無理

---


# 使ってみて悪かったところ(苦しかったところ)

- Github上のレスポンスやPR解決までのライフサイクルが長い
  - 私がRemixのメンバーに言われた名言(皮肉)
    - 「PRを起票するにはそれなりの根気強さと我慢が必要です」
    - 「おすすめはできないけど、実際は patch-package が一番の解決策です」
  - 一緒にリポジトリを育てるという気概が必要

---

# 使ってみて悪かったところ(苦しかったところ)

Workersに限った話になるが
  
- UIライブラリ入れると1MBにバンドルサイズを抑えるのは結構きつい
  - SSRなので全てバンドルしないといけない(チャンクできない)
  - Service Bindingsを駆使して回避した
- まだまだWorker非対応なライブラリが多い
  - esbuildでpolyfillしたり、injectしたりなど職人芸が求められる
  - しかし、そもそもesbuildの拡張が不可能
    - 何度もDiscussionに起票されているが、コミュニティのポリシー的にNG
  - 自分でRemixのesbuild を拡張可能にするライブラリを書いた

<br>

**初心者の方はランタイムNodeからスタートすることをオススメします**

---

# 向いているサービスやサイト

- React Routeで構築されたSPAサイトをSSRに移行したい、SEO対策したい
  - React Routeの思想をベースに構築されているため、大きく構造を変えずに移行できる
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

