---
theme: unicorn
highlighter: shiki
lineNumbers: false
layout: center
---

<div class="text-center text-5xl pb-8">📀</div>

# Remixの凄みを紹介したい

## @aiji42_dev

---
layout: intro
introImage: 'https://storage.googleapis.com/zenn-user-upload/avatar/e738d28d01.jpeg'
---

# Who am I ?

## Uejima Aiji | Twitter: @aiji42_dev

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

パブリックされてから半年間、個人でも社内でもRemixを使い倒したので

---

## この発表をぜひ聞いて欲しい人

- Remixって最近良く聞いたり目にしたりするなー 🤔

- Next.js大好き！正直コレ一本で食っていけるよね 😎
  - この発表を聞くと、たとえRemixを使用しなくても実装の仕方の幅が広がることでしょう

- 最近Next.jsがなんか新しいLayoutに関するRF公開したよね
  - そうです、実はRemixがその先駆けです

- Cloudflareってなんか最近勢いあるよね、なんか試してみようかな 🌩
  - CloudflareはRemixを語る上で切り離せない話題です

---

## その前にお断り

この発表ではJamstackもCMSにも触れません 🙇‍

しかし、Remixが解決しようとしていることは、  
今後のReactやフロント界隈の方向性に少なからず影響を与えており、  
多くの人に触れてほしい知ってほしいという気持ちで、この発表に臨んでいます。

---

# What is Remix ?

- React SSRフレームワーク
- React Routerの開発チームが開発を主導
- 昨年11月末にv1がリリースされたタイミングでパブリックに
- Cloudflare Workersで稼働させられたり、Denoをサポートしていたり
- 📀 のアイコンがよく使われる

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

#### Next.js の getServerSideProps や API Routes のようなもの

<br>

ページコンポネントと同一ファイルに定義可能

<br>

##### loader
GETアクセス時のデータフェッチを定義  
ページ(コンポネント)からは**useLoaderData**で取得し、**useFetcher**で再フェッチ可能

##### action
POSTやDELETEなどのミューテーションを定義する
**useSubmit**や**useFormAction**、form要素からリクエストする

</div>
</div>

---

#### Remixの特徴 ②
<br>

# File system routing と Nested Routing (Layout)

レイアウトルート / 共通処理

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

ディレクトリの入れ子はそのままURLに変換される

$(ドルマーク)をつけると、  
パラメータとしてloader/actionで扱える


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

root.tsxがトップレイヤレイアウト  
ディレクトリと同一名ファイルが子レイアウト

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
(pathless layout routes)

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

root.tsxがトップレイヤレイアウト

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

ディレクトリと同一名ファイルで子レイアウトを定義

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
export const loader = async ({ params }) => {
  const data = await db.invoice.findOne({ where: { id: params.id } })
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

各ページ・レイアウトにごとに  
ErrorBoundaryを定義可能

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
export const loader = async ({ params }) => {
  const data = await db.invoice.findOne({ where: { id: params.id } })
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

エラーの伝搬を留めることができる    
フォールバックが最小限になる

</div>
</div>

---

## Nested Routes があると何が嬉しいか

- レイアウトのグルーピングと階層適応

- 並列データフェッチ
  - 各loaderは並列に処理されるため、高速化につながる
  - <img src="/fetch.png" class="h-80" />

---

- ロジックの共通化 
  - 特定ルート配下はログイン必須にするなどの、共通処理を階層的にもたせられる
  - pathless routesと組み合わせて特定のディレクトリ下は暗黙的に認証必須にするなど
 
- 差分ロード・差分再フェッチ
  - ナビゲーション時にフルページロードではなく、必要なレイアウトルート分のロードが行われる
  - 任意にページ内を更新する再フェッチ処理も実装しやすい

---

## ちょうど先日Next.jsにも同等な機能のRFCが公開された

![](/nextjs-rfc.png)
https://nextjs.org/blog/layouts-rfc

- 現プロポーザルではおおよそRemixと同等の機能をカバーする予定
  - pathlessやErrorBoundaryに関しても、ドキュメントにはないが「パート2で言及する」とのこと
    - かなりRemixのノウハウが意思決定に影響を及ぼしている印象
  - デフォルトでServerComponentになる (Remixでも同様に議論は起きている)

---
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
- もちろんセルフホスト可能

---

## Remixの一番の強みは Cloudflare Workers 上で動くという点

だと個人的には思う。

**エッジロケーションでレンダリングできるというのは今後のReact界隈において重要な意味をもつ**

---

## Reactが向かう先 - Server component / Streaming render

<style>
li {
  font-size: medium;
}
</style>

- コンポネントのレンダリングをServerサイドで行うようになる
  - 従来のSSRとはことなり、コンポネントの粒度で解決し、さらにストリームで返却する
  - <small>src: https://mxstbr.com/thoughts/streaming-ssr/</small> <img src="https://mxstbr.com/static/images/renderToString.png" class="h-60 bg-white" />
  - 各コンポネントでデータフェッチの処理を持ち、非同期的・自律的に解決する

- 同時接続数は爆発的に増加し、そしてラウンドトリップによるレイテンシが無視できなくなることが予想できる

---

## そんな未来を見据えると

- エッジファンクション(ロケーション)がオリジンとして機能する
- スケールあまり意識しなくて良い
- 1リクエストあたりのコストが安価

という点は大きなアドバンテージになる

<br>

#### Next.jsも昨年のエッジファンクション(middleware)の発表を皮切りに、エッジレンダリングを模索している  
[RFC: Switchable Next.js Runtime #34179](https://github.com/vercel/next.js/discussions/34179)

---

<div class="flex space-x-4">
<div class="flex-1">
<br>

#### Remixの特徴 ④
<br>

# Formとヘルパー

<br>

前述のactionと通信を行うための、Formコンポネントや多数のヘルパーを備えている

特に **useTransition** はフォームのsubmitの状態  
(idle / submitting / loading)を管理したり  
submit中のデータを取り扱うことが可能なので、楽観的UIの実装も容易

</div>
<div class="flex-1">
<video controls autoplay>
  <source src="https://remix.run/jokes-tutorial/img/optimistic-ui.mp4" type="video/mp4">
</video>
</div>
</div>

---

<div class="flex space-x-4">

<div class="flex-1">

```tsx
import { ValidatedForm } from "remix-validated-form";
import { withZod } from "@remix-validated-form/with-zod";

export const validator = withZod(
  // your zod role
);

export const action = async ({ request }) => {
  const result = await validator.validate(await request.formData());
  if (result.error) return validationError(result.error);

  const { firstName, lastName, email } = result.data;
  // Do something with the data
};

export default function MyPage() {
  return (
    <ValidatedForm validator={validator} method="post">
      <FormInput name="firstName" label="First Name" />
      <FormInput name="lastName" label="Last Name" />
      <FormInput name="email" label="Email" />
      <SubmitButton />
    </ValidatedForm>
  );
}
```

</div>
<div class="flex-1">

## remix-validated-form
https://www.remix-validated-form.io/

<br>

remix-validated-formとzodを使用するとactionとコンポネントを一体化できる

クライアントとサーバとでバリデーションを共通化できるだけでなく  
エラーメッセージの返却・レンダリング、例外処理の実装から開放される

別のデータソースと通信して別途バリデーションするなど、サーバサイドオンリーなバリデーションもかんたんに追加可能

</div>
</div>

---

#### Remixの特徴 ⑤

<br>

# Cookieヘルパー

CookieやSessionを取り扱うためのヘルパーが標準装備  

シリアライズ&検証の機能もデフォルトで実装されている

loader/actionと組み合わせることで、  
これまでフロントに実装していたステート管理や認証などをサーバサイドへ移譲できる

ロジックがフロントに露出しないため、  
秘匿性の高い情報の漏洩防止や、バンドルサイズの軽減につながる

---

<div class="flex space-x-4">
<div class="flex-1">

```tsx
export const loader = async ({ request }) =>
  supabaseStrategy.checkSession(request, {
    successRedirect: '/private'
  });

export const action = async ({ request }) =>
  authenticator.authenticate('sb', request, {
    successRedirect: '/private',
    failureRedirect: '/login'
  });

export default function LoginPage() {
  return (
    <Form method="post">
      <input type="email" name="email" />
      <input type="password" name="password" />
      <button>Sign In</button>
    </Form>
  );
}
```

</div>
<div class="flex-1">

## remix-auth  
https://github.com/sergiodxa/remix-auth  
(サンプルコードは[remix-auth-supabase](https://github.com/mitchelvanbever/remix-auth-supabase))

認証方法・認可ルール・フォールバックルールなどを簡単に設定・制御できる

クライアント側には一切ロジックが露出しない

</div>
</div>

---

<div class="flex space-x-4">
<div class="flex-1">

# 実用に耐えられるの？

</div>

<div class="flex-1">

<div class="text-5xl">🤔</div>


</div>
</div>

<br>


---

<div class="flex space-x-4">
<div class="flex-1">

## <Tweet id="1529131553481453569"/>

</div>

<div class="flex-1">

<img src="https://camo.qiitausercontent.com/bf17791a3291dfe19a39cb25a72449b8e70cea48/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f37303336332f35323734633438342d666563342d316137352d313164632d3361313365646637626335632e706e67" class="h-50" />


<img src="/leaderboard.png" class="h-70" />

<small>他に個人開発サービスの運用実績、社内での実サービス開発への導入実績も</small>

</div>
</div>

---

# 実際に得られた恩恵

- ステート管理ライブラリが不要
  - 前述の通り
- 認証ロジック・データフェッチロジックすべてがサーバサイド完結
  - クライアントの実装はデータの描画のみに集中できる
- 情報更新をきめ細かくリアルタイムに、かつ高速に
  - 前述の例ではSupabaseのsubscribeと組み合わせて、DBに変更が加わったらスタッツを再フェチする
  - ネストされた部分的なレイアウトのみ再フェッチ(フルページロードしない)
  - 実際のデータフェッチはloader側で行うので、ロジックは一切露出しない

---

- エッジレンダリング(Cloudflare Workers)による恩恵
  - ゼロコールドスタート
    - KV使わなくても、200-300msで応答できる(TTFB)
    - KV使えばSSRで100ms切る
    - 同一構成の Next.js on Vercel のSSRで300-500ms
    - もちろんデータソースに引っ張られるが
  - スケールを気にしなくて良い

---

# 苦しみ

- Nested Routeは想像以上に難易度が高い
  - URL設計とディレクトリ設計をセットで行わなければならない
  - 良くも悪くもロジックが分散し、脳内メモリを圧迫する
  - 実際ネストは2階層くらいにとどめておくのがよい
    - 最初のサンプルに上げたような構成は正直無理

---

Workersに限った話になるが。。。
  
- UIライブラリ入れると1MBにバンドルサイズを抑えるのは結構きつい
  - SSRなので全てバンドルしないといけない(チャンクもできない)
  - Service Bindingsを駆使して回避した
- まだまだWorker非対応なライブラリが多い
  - esbuildでpolyfillしたり、injectしたりなど職人芸が求められる
  - しかし、そもそもRemixのコンパイラ(esbuild)の設定の拡張が不可能
    - next.config.jsからwebpackの設定をいじるみたいな感じのことはできない 
    - 拡張自体がコミュニティのポリシー的にNG (マネジメントコストとリスクの問題)
      - 何度もDiscussionやPRは起票されているがことごとくリジェクト
  - 最終的に自分でRemixのesbuild を拡張可能にするプラグインを書いた
    - https://github.com/aiji42/remix-esbuild-override

---

# 相性の良いサービスやサイト

次のようなケースならNext.jsで実装するよりもRemixで実装したほうが開発コストは小さくなる(と個人的には思う)

---

- React Router で構築されたSPAサイトをSSRに移行したい、SEO対策したい
  - React Routeの思想をベースに構築されているため、大きく構造を変えずに移行できる

- 複数ページに渡るエントリーフォームを簡単に実装したい
  - 前述の通り、remix-validated-formとzodで簡単に作れる
  - ステートライブラリも不要
  - MVCなWebフレームワーク(Rails)使ってた人は比較的とっつきやすい(と個人的には思う)

- 管理画面やダッシュボードをフルスクラッチしたい
  - React Adminの導入を試みたが、データスキーム・ビジネスロジックに適したアダプターがなかったなど
  - Nested Routes と remix-validated-form, remix-auth を駆使する
  - loader/actionを各フォームやデータフィードと1対1で配置し、UIとデータロジックをそれぞれで凝縮
    - テスタビリティの向上にもつながる

---

# まとめ

<br>

なんと言っても  

- Cloudflare Workerで動く

- Nested routesの先駆け
  - レイアウトとデータフェッチロジックの分散管理
  - Next.jsに影響を及ぼす程の先見性

<br>

この2つを備えていて、ここまで完成度の高いフレームワークは他にはありません。  
メインで使うフレームワークとまはいかなくても、一度試す価値はあると思います。

Next.jsの新しいLayout戦略がGAされるまでの素振りとしても、良いサンドボックスになると思います。

---

# One more thing

## Next.jsにRemixのエッセンスを取り入れる

---

<div class="flex space-x-4">
<div class="flex-1">

```tsx
import { handle, json, Form, useFormSubmit } from 'next-runtime';

export const getServerSideProps = handle({
  async get() {
    return json({ name: 'smeijer' });
  },
  async post({ req: { body } }) {
    await db.comments.insert(body);

    return json({ message: 'thanks for your comment!' });
  },
});

export default function MyPage({ name, message }) {
  const { isSubmitting } = useFormSubmit();

  if (message) return <p>{message}</p>;
  return (
    <Form method="post">
      <input name="name" defaultValue={name} />
      <input name="message" />
      <button type="submit" disabled={pending}>
        {isSubmitting ? 'submitting' : 'submit'}
      </button>
    </Form>
  );
}
```

</div>

<div class="flex-1">

## next-runtime
https://next-runtime.meijer.ws/getting-started/1-introduction

<img src="https://github.com/smeijer/next-runtime/raw/main/docs/public/banner.png" class="h-20" />

<br>

getServerSidePropsを拡張し、リクエストメソッドごとに実装を書き分けることができる  
フォームのアクションをapi routesにせず、自パスに向ければ、擬似的なloader/actionになる

楽観的UIのためのヘルパーやCookieを取り扱うヘルパーなどが用意されており、かなりRemixに似ている

<small>というか、Remixをインスパイアされて実装したと作者がドキュメントで明言している</small>


</div>
</div>

---

こちらの記事でも紹介しています

<img src="/og-base_z4sxah.png" class="h-70 bg-white" />

https://zenn.dev/aiji42/articles/23a88a7b111694