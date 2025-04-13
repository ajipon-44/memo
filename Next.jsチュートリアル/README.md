# Next.js チュートリアル

## Chapter 3 - Optimizing Fonts and Images -

### フォントの最適化

#### フォントの種類

フォールバックフォント ... HTML や OS に標準で備わっているフォント（h1 や bold など）。別名システムフォント。
カスタムフォント ... CSS などで装飾されたフォント。サーバや外部リソースから取得する。

#### ブラウザの挙動

1. ページを読み込み、システムフォントで出力する。
2. CSS やフォントファイルをロードする（ここでファイル取得や読み込みのための時間がかかってしまう）。
3. 装飾されたページが表示される。

#### レイアウトシフト

システムフォントからカスタムフォントに切り替わった時に、文字の大きさや文字同士の間隔の差異によって起こるページのずれ

#### CLS（累積レイアウトシフト）

Google のサイト評価の指標の 1 つ。これが大きいと、ページの表示に時間がかかると判定される。

#### next/font

ビルド時にフォントに関するロードを行う
よって、ユーザが使う時に、オーバーヘッドが生じない

### 画像の最適化

#### 画像の最適化をすべき理由

画像の読み込みには一定の時間がかかるので、最適化の有無によって SEO や UX に大きく影響する

#### 従来の画像の最適化

以下の項目は従来は手動で最適化を行なっていた。

1. **レスポンシブ画像の確保**
   画像が異なる画面サイズに対応できるようにする。
2. **異なるデバイス用の画像サイズ指定**
   モバイルデバイスとデスクトップで異なるサイズの画像を使い分ける。
3. **レイアウトシフトの防止**
   画像が読み込まれる際に、ページが動かないようにする。
4. **遅延読み込み（Lazy loading）の実装**
   ユーザーが実際に画像を見るまで、画像を読み込まないようにする。

#### Image コンポーネント

HTML の img タグの拡張版で手動で行っていた項目を自動で行う

1. **最新フォーマットでの提供**

   WebP や AVIF といった、ブラウザがサポートしている最新の画像フォーマットを使用して、画像サイズを圧縮する。
   これにより、読み込み速度がさらに向上する。
   ブラウザがこれらのフォーマットをサポートしていない場合は、他の形式の画像を使う。

2. **異なるデバイス用の画像サイズ指定**
   デバイスを検知して、自動で画像をリサイズする。

3. **レイアウトシフトの防止**
   （元の画像サイズや CSS などから算出したサイズをもとに）画像のサイズ表示領域をあらかじめ確保しておくことによって、画像が表示されたときに、ページのレイアウトが崩れることを防ぐ。
   よって、`width`と`height`の属性を指定することが必須。

4. **遅延読み込み（Lazy loading）の実装**
   デフォルトで遅延読み込みを行うので、パフォーマンスが向上する。

## Chapter 4 - Creating Layouts and Pages -

### ルーティングの方法

app ディレクトリ配下にディレクトリを作り、page.tsx ファイルを設置する。
例えば、app/dashboard/page.tsx にアクセスする URL は https://localhost:8080/dashboard になる

### 特別なファイル

#### layout.tsx

複数のファイルで共有される UI を表すファイル
layout.tsx が存在する同階層とその下の階層のファイルを Children として受け取る

### page.tsx

## Chapter 5 - Navigating Between Pages -

### Link コンポーネント

HTML の a タグの代わりに Link コンポーネントを使用する。
クライアントサイドナビゲーションによってページ遷移する。

#### クライアントサイドナビゲーション

- Link コンポーネントを使用する
- 必要な JS ファイルやデータだけを取得し、ページの変更箇所だけを更新するので、高速
  - ネットワーク通信量が少ない
  - ページ全体のロードをしないので動作がスムーズ
- JavaScript を利用してページ遷移を行う

#### サーバサイドナビゲーション

- a タグを使用する
- ページ遷移時、サーバ側にリクエストを送信する
- レンダリング済みのページが返ってくるので、それを表示する
  - ページ全体が再読み込みされるため、少し遅く感じる

#### コード分割

Next.js ではビルド時にページ単位で固有の JS ファイルにバンドルを行う。
このようにすることで、大規模なアプリだとしても初回ロードに時間がかかりすぎない

#### プリフェッチング

ユーザのスクロールなどにより、リンクが画面内に見える位置になった、または近づいた場合にリンク先のページをバックグラウンドで取得する仕組み
Intersection Observer API を活用して画面の動きを監視している
取得されたファイルはブラウザや Next.js のキャッシュに保存され、リンクがクリックされると取得される。

#### まとめ

1. ビルドするとページ単位でコード分割されて JS ファイルにバンドルされる
2. 初回ロード時には表示されるページしかフェッチされない
3. ユーザがスクロールし、リンクが近くなると、バックグラウンドでリンク先のページをプリフェッチし、キャッシュに保存する
4. ユーザがリンクを押下すると、キャッシュからページを取得し、表示する

### アクティブリンク

アクティブリンク ... タブに複数の項目がある場合などに、現在アクセスしている項目を分かりやすく色付けするもの

usePathname() ... 現在の URL を返す hooks。https://example.com/about にアクセスすると /about を返す

#### Client Component

- 動作
  - ブラウザ上で実行されるコンポーネント
- 特徴
  - JS を利用してインタラクティブな要素（ボタンのクリックやフォーム入力など）を実現する
  - 状態管理やイベントハンドラに対応（useState などの hooks、window オブジェクトなど）
  - 必要に応じてサーバからデータを取得し、クライアントサイドでデータを処理

```jsx
"use client";
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

#### Server Component

- 動作
  - サーバ上で実行されるコンポーネント
- 特徴
  - サーバ上で JS を実行し、静的な HTML を生成
  - 高速な初期レンダリングと SEO に適している
  - 状態管理やブラウザ固有の API（例: `window` や `useEffect`）は使用できない。

```jsx
export default async function UserList() {
  const res = await fetch("https://api.example.com/users");
  const users = await res.json();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

Server コンポーネントによって軽量で効率的な初期レンダリングを提供し、必要な部分だけを Client コンポーネントでインタラクティブにするのがベストプラクティス

## Chapter 6 - Setting Up Your Database -

省略

## Chapter 7 - Fetching Data -

Server Component によって、サーバ上から DB に問い合わせをできるため、ロジックを持つことができ、結果だけをクライアント側に送ることができる

### 並列データフェッチ

Promise.all や Promise.allSettled を使うことで、全てのプロミスを同時に開始することができる

```javascript
const data = await Promise.all([
  invoiceCountPromise,
  customerCountPromise,
  invoiceStatusPromise,
]);
```

メリット

- 各リクエストの完了を待たずにフェッチできるので、高速化が図れる
- ネイティブな JS なので、どんなライブラリにも対応できる

デメリットは次章で解説

## Chapter 8 - Static and Dynamic Rendering -

### Static Rendering

データのフェッチとレンダリングはビルド時とデータの再検証時に行われる
ユーザがアクセスするたびにキャッシュされた結果が表示される

#### メリット

- Web サイトの高速化
  - プリレンダリングされたコンテンツをキャッシュして配信する
- 負荷軽減
  - キャッシュされたデータを提供するので、ユーザのリクエストごとにコンテンツを生成する必要がない
- SEO 対策
  - プリレンダリングされたコンテンツは利用可能状態になっているため、検索エンジンのクローラーがインデックスしやすい

#### 使い所

静的なブログ記事や製品ページなど、データのない UI やユーザ間で変わらないページ

### Dynamic Rendering

リクエスト時に各ユーザに対して、サーバ上でコンテンツがレンダリングされる

#### メリット

- リアルタイム性
  - リアルタイム、または頻繁に更新されるデータを表示させることができる
- ユーザ固有のコンテンツ
  - パーソナライズされたコンテンツを提供できる
- リクエスト時の情報
  - Cookie や検索パラメータなどリクエスト時にしか分からない情報にアクセスできる

#### デメリット

- データの取得が遅い場合、全体のパフォーマンスがそれに引きづられる

## Chapter 9 - Streaming -

### Streaming

大きなデータをチャンクに分けて、準備ができたものから送信する送信形式
重いデータの送信をノンブロッキングで処理できる

#### 実際の動作

最初にレンダリング可能な部分がクライアントに送信され、非同期のデータフェッチが完了するたびに次にコンポーネントがレンダリングされる。

- クライアントコンポーネントの場合
  - 最初はローディング画面やローディングスケルトンなどのプレースホルダーが表示され、処理が完了したものから順に表示される
    - ローディングスケルトン ... 文字や画像が空白で枠やデザインだけ見えているようなコンテンツ
- サーバコンポーネントの場合
  - レンダリングが完了した部分からクライアントに送信されて表示される

### ページレベルでのストリーミング

1. loading.tsx を使用する
   1. loading.tsx を使用すると自動的に <Suspense> でラップされる
   2. 最初に loading.tsx がレンダリングされ、その間にページ全体を非同期でレンダリングする
      1. 設置されたディレクトリよりも下位の階層でも適用される（Route Group で変更可能）
   3. ページ単位で大まかにストーミングをしたい時、シンプルにしたい時に使う

#### Route Group

Next.js はディレクトリ構造が URL になるが、URL に影響を与えずにディレクトリ構造を作る仕組みを Route Group という。

`(folderName)` で作成できる

- メリット
  - layout.tsx などの UI を分けることができる
  - 特定のページにだけローディングスケルトンを設置できる

### コンポーネントレベルでのストリーミング

1. <Suspense> コンポーネントを使用する
   1. ページレベルよりも細かい単位でストリーミングの制御ができる
   2. 非同期で読み込むコンポーネントを <Suspense> でラップする
   3. ページの一部だけをストリーミングしたい時に使う

```jsx
// fallbackでロード中に表示したいコンポーネントを指定
<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
```

以下のように同じコンポーネントが複数あるときは、ラップしたものを <Suspense> でラップするべき
個別に <Suspense> をすると次々に表示されて UX がよくない

```jsx
<Card />
<Card />
<Card />
// ↓
<Suspense fallback={<CardsSkeleton />}>
    <CardWrapper />
</Suspense>
```

### どのレベルでストリーミングをするか

正解はない。
ユーザにどのような体験をして欲しいか、どのコンテンツを優先したいかなどで決める。

## Chapter 10 - Partial Prerendering -

### Partial Prerendering（PPR、部分プリレンダリング）とは

同じ URL で静的レンダリングと動的レンダリングを組み合わせることができる仕組み

**ページの一部だけを事前に作っておいて、残りは後から動的に表示する仕組み**
これにより、**ページの表示を速くしつつ、最新のデータも取得できる**ようになります。

### PPR の仕組み

静的な部分はビルド時に HTML にしておく。

Suspense を利用して、条件が満たされるまで（例えばデータの取得など）画面の一部をフォールバックなどを表示して、レンダリングを延期させる。（Suspense を使用することで、Next.js がどこが動的なページか分かるようになる）

データの取得が完了したら、ページの動的な部分だけをレンダリングし、クライアントサイドで更新処理をする。

### 実装

next.config.ts に PPR に関する記載をする

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {
    // incremental は特定のルートにのみ PPR を適用させる
    ppr: "incremental",
  },
};

export default nextConfig;
```

### 注意点

サーバコンポーネントでフェッチするときに PPR を使うべきではない
サーバコンポーネントはデータの取得が完了してから、クライアントサイドにページを送信するので、データの取得が終わるまで、静的な部分もクライアント側に送信できない。

## Chapter 11 - Adding Search and Pagination -

### 検索

検索に URL パラメータを使う理由

1. ブックマークでき、共有できる
2. どの URL が多く叩かれているかが分かるため、ユーザがどの検索を多く使うのかなどのトラッキングができる

#### 実装

使う Hooks

- useSearchParams ... 現在のクエリパラメータを読み取れる。/dashboard/invoices?page=1&query=pending だったら ? 以降
- usePathName ... パスを返す。/dashboard/invoices?page=1&query=pending だったら /dashboard/invoices
- useRouter ... ページ遷移などに使われる。

#### 実装ステップ

1. ユーザーの入力をキャプチャする、クライアントコンポーネントでユーザの入力をキャッチする

```jsx
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
/>
```

2. URL を検索パラメータで更新する。フォームを入力すると URL のクエリパラメータが書き換わる。

```jsx
import { useSearchParams, usePathname, useRouter } from "next/navigation";

export default function Search({ placeholder }: { placeholder: string }) {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
    replace(`${pathname}?${params.toString()}`);
  }
  // 省略
}
```

3. URL のクエリパラメータと入力フォーム内の文字列を同期させる。

```jsx
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get("query")?.toString()} // ここを追加
/>
```

4. 入力フォームに文字列を入れると、検索クエリが変わり、サーバ上にデータをフェッチしに行き、テーブルを更新する。

### デバウンス

関数が起動する時間を制限する手法。これによりユーザが動きを止めた時だけ、DB に問い合わせることができる。
上の実装方法では、ユーザがキーを打つたびに DB にクエリを投げているのでパフォーマンスが悪い

1. デバウンスの対象となるイベント（キーの入力など）が発生すると、タイマーが起動する
2. タイマーが起動している間に、再度イベントが起こるとタイマーがリセットされる
3. タイマーが 0 になると関数が実行される

#### 実装

1. ライブラリをインストール

```shell
pnpm i use-debounce
```

2. useDebouncedCallback を使う、下の例だとタイマーが 300ms

```jsx
import { useDebouncedCallback } from "use-debounce";

export default function Search({ placeholder }: { placeholder: string }) {
  const handleSearch = useDebouncedCallback((term) => {
    console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);
```

### ページネーション

#### 実装

1. URL からパス名を取得、useSearchParams を使用して、現在のページ番号を取得

```jsx
const pathname = usePathname();
const searchParams = useSearchParams();
const currentPage = Number(searchParams.get("page")) || 1;
```

2. URL を作成する

```jsx
const createPageURL = (pageNumber: number | string) => {
  const params = new URLSearchParams(searchParams);
  params.set("page", pageNumber.toString());
  return `${pathname}?${params.toString()}`;
};
```

※useSearchParams に関する補足

| 変数名       | 役割                                                                                                          | 取得元                | 変更可否      | メソッド                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------- | --------------------- | ------------- | ------------------------------------------------------------------------------------------------ |
| searchParams | 現在のクエリパラメータを取得<br />useState の getter のようなもの                                             | useSearchParams()     | ❌ (変更不可) | .getter                                                                                          |
| params       | searchParams をコピーし、編集可能にしたもの<br />useState の setter のようなものだが、getter 関数も持っている | new URLSearchParams() | ✅ (変更可)   | .getter : 取得<br />.setter : 置き換え<br />.append : 新しいパラメータの追加<br />.delete : 削除 |

以下のようにも宣言できたが、意図的にしなかったと思われる。

```jsx
const [searchParams, setSearchParams] = useSearchParams();
```

- 意図
  - 関数内だけでしか set を使わないので get しか宣言しなかった。
  - 関数内でクエリパラメータを変更するということを明確化したかった。

3. ページネーションのリンクに作成した URL を入れ込む

```jsx
<PaginationArrow
  direction="left"
  href={createPageURL(currentPage - 1)}
  isDisabled={currentPage <= 1}
/>
```

## Chapter 12 - Mutating Data -

### Server Actions とは

非同期コードなどデータ処理やロジックをサーバ上で実行する仕組み。

メリット

✅ **サーバー上で実行** されるため、ブラウザの JavaScript が無効でも動作する
✅ **クライアントに処理負担をかけず** にデータの取得・変更ができる
✅ **ネットワークが不安定でも** リクエストを送れれば処理が完了する
✅ **機密情報を安全に処理**（例: データベースの操作や認証）

#### 作り方

ファイル内で "use server" を宣言し、中で関数を定義する

```ts
"use server";

export async function createInvoice(formData: FormData) {
  const rawFormData = {
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  };

  console.log(rawFormData); // サーバーで実行される
}
```

ファイル内の関数をクライアントコンポーネントやサーバーコンポーネントにインポートして使用する。
使われない関数はバンドル時に削除される。

クライアントコンポーネントでの使用例：

```tsx
import { createInvoice } from "@/app/lib/actions";

export default function Form() {
  // action={createInvoice} でフォームの送信時に createInvoice が 自動で FormData を受け取る
  // ここに設定する関数は FormData を引数に取る関数である必要がある
  return <form action={createInvoice}>...</form>;
}
```

#### form の action 属性 (※Next.js の場合は拡張されて、関数も指定できるようになっている)

**通常の HTML の `form` の `action` の役割：**

- `action` に指定した URL へフォームデータを送信する。
- `method="POST"` ならば、フォームのデータを HTTP リクエストのボディとして送信する。
- `method="GET"` ならば、フォームのデータを URL クエリパラメータとして送信する。

```html
<form action="/submit" method="POST">
  <input type="text" name="username" />
  <button type="submit">送信</button>
</form>
```

#### 引数を入れたいとき

bind 関数を使用する

```tsx
  // updateInvoice(id: string, formData: FormData)
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
  return (
    <form action={updateInvoiceWithId}>
```

### Revalidate and redirect

Next.js では、ルートセグメントをブラウザにキャッシュするクライアントサイドルーターキャッシュがある。
クライアントサイドルーターキャッシュは UI とデータをキャッシュしている。

✅ **プリフェッチ**（事前取得）と組み合わせて、ページ遷移を高速化
✅ **ブラウザにキャッシュされたデータを使い、リクエスト数を減らす**

```
/app
  ├── page.tsx        ← `/` のルートセグメント
  ├── dashboard
  │   ├── page.tsx    ← `/dashboard` のルートセグメント
  │   ├── invoices
  │   │   ├── page.tsx  ← `/dashboard/invoices` のルートセグメント
```

### キャッシュのクリア

クライアントサイドルーターキャッシュのデータをクリアするときに使うのが **revalidatePath("/pathname")**

```ts
import { revalidatePath } from "next/cache";

export async function updateInvoice(formData: FormData) {
  // データを更新する処理
  await db.updateInvoice(formData);

  // `/dashboard/invoices` のキャッシュを削除し、最新のデータを取得
  revalidatePath("/dashboard/invoices");
}
```

**UI も更新** したいなら、ページごとキャッシュを無効化する `revalidateTag()` や `revalidatePath()`なども使う。

### 編集ページ

```tsx
// /app/dashboard/invoices/[id]/edit/page.tsx
import Form from "@/app/ui/invoices/edit-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchInvoiceById, fetchCustomers } from "@/app/lib/data";

// page.tsxのpropsは特別で、URLのパスパラメータやクエリパラメータなどが入っている
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  // SSRなので、awaitが直接使え、レンダリングされると実行される
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: "Invoices", href: "/dashboard/invoices" },
          {
            label: "Edit Invoice",
            href: `/dashboard/invoices/${id}/edit`,
            active: true,
          },
        ]}
      />
      <Form invoice={invoice} customers={customers} />
    </main>
  );
}
```

### キャッシュ戦略

Next.js には DB に近い順から以下の 4 つのキャッシュが存在する

| メカニズム                                                                                               | 内容                 | 保存場所 | 目的                                            | 期間                            |
| -------------------------------------------------------------------------------------------------------- | -------------------- | -------- | ----------------------------------------------- | ------------------------------- |
| [Data Cache](https://nextjs.org/docs/app/building-your-application/caching#data-cache)                   | Data                 | サーバ   | Store data across user requests and deployments | Persistent (can be revalidated) |
| [Request Memoization](https://nextjs.org/docs/app/building-your-application/caching#request-memoization) | 関数の戻り値         | サーバ   | Re-use data in a React Component tree           | Per-request lifecycle           |
| [Full Route Cache](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)       | HTML and RSC payload | サーバ   | Reduce rendering cost and improve performance   | Persistent (can be revalidated) |
| [Router Cache](https://nextjs.org/docs/app/building-your-application/caching#client-side-router-cache)   | RSC Payload          | ブラウザ | Reduce server requests on navigation            | User session or time-based      |

<img src="/Users/yanaikeigo/Library/Application Support/typora-user-images/スクリーンショット 2025-03-16 19.20.03.png" alt="スクリーンショット 2025-03-16 19.20.03" style="zoom:40%;" />

#### Data Cache

#### Request Memoization

リクエストをメモ化する機能。同じ URL とオプション

#### Full Route Cache

#### Router Cache

ゆくゆく追記する

## Chapter 13 - Handling Errors -

### try-catch

Server Actions で try catch を使ってエラーハンドリングをする。

```ts
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  try {
    await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    console.error(error);
  }

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

### error.tsx

1. コンポーネント内でエラーが発生

2. エラーが発生したコンポーネントを含む DOM ツリーが破棄される

3. 最も近い error.tsx に置き換えられる。

   - `app/error.tsx` があれば全体適用

   - `app/dashboard/error.tsx` なら `dashboard/` 配下のエラーだけ適用

4. **`reset()` を呼ぶとエラーリカバリを試みる**

   - コンポーネントが再レンダリングされ、エラーが解消されれば通常の UI に戻る

### 404 Page

not-found.tsx というファイルを作る。

next/navigation の notFound 関数を叩くと 404 ページに遷移する

## Chapter 14 - Improving Accessibility -

### アクセシビリティとは

個人の特性によらず、キーボードナビゲーション、セマンテック HTML(input タグなど)、画像、色、動画などが利用できること。

### next lint

Next.js の Linter

Next.js の文法エラーや型の安全性だけでなく、画像の alt テキストや aria 属性など a11y (accesibility) に関するチェックを行う

### useActionState

クライアントコンポーネントで利用できる Hooks

第一引数 ... アクション、関数
第二引数 ... 初期状態

返り値 1 ... フォームの状態
返り値 2 ... フォームが送信されるときに実行する関数

```tsx
"use client";
import { useActionState } from "react";

export default function Form({ customers }: { customers: CustomerField[] }) {
  const [state, formAction] = useActionState(createInvoice, initialState);

  return <form action={formAction}>...</form>;
}
```

## Chapter 15 - Adding Authentication -

## Chapter 16 - Adding Metadata -
