# フロントエンド開発のためのテスト入門

## 第１章　テストの目的と障壁

リグレッションテスト ... デグレのテストのこと
E2E テスト ... クライアントから DB サーバまで含んだテスト、実際の Web アプリケーションに近い状況を再現する

### テストを書く目的

1. バグを減らすため
2. リファクタリングを容易に行い、コードの品質を高く維持するため
   1. リファクタリングによって悪影響がないかすぐに調べられる
3. アクセシビリティの向上
   1. テストコードはアクセシビリティ由来の要素取得 AI を利用してテストを書くことが増えたので、自ずとアクセシビリティを担保したコードになる
4. コミュニケーションの円滑化
   1. プルリクを出した時にレビュワーがどのような仕様で作った機能なのかが、テストに書いてあるのでわかりやすい
5. デグレを防ぐため

### テストを書く障壁

1. テストを書かない風土
   1. 自分がテストを書けば、それを真似する人が出てくるかも
2. 時間がない
   1. テストを書かない方がバグを生むし、それによる手戻りを考えると、テストを書く方が時間効率が良い

## 第２章　テスト手法とテスト戦略

### 範囲と目的で考えるテスト

#### テストの範囲

フロントエンドの自動テストでは、以下のどこからどこまでを対象とするかを考える。

1. ライブラリが提供する関数
2. ロジックを担う関数
3. UI を表現する関数
4. Web API クライアント
5. API サーバ
6. DB サーバ

#### 静的解析

TypeScript や ESLint による静的解析
1 つ 1 つのモジュール検証だけでなく、2 と 3 といった**「隣接するモジュール間の不整合」**も検出できる

#### 単体テスト

2 のみ、3 のみというように、**「モジュール単体が提供する機能」**に着目したテスト
独立した検証が行えるため、アプリケーション稼働時には滅多に発生しないケース（コーナーケース）の検証に向いている
Jest や Testing Library を使う

#### 結合テスト

1~4 まで、2~3 までというように**「モジュールをつなげることで提供できる機能」**に着目したテスト
広範囲の対象を効率よくテストできるが、ざっくりした検証になる傾向がある
モックやスタブを多用して、外部依存を除外しながら「連携部分の挙動」を検証する
E2E テストとの境界が曖昧になりがちなので、モジュール単位〜ページ単位くらいが適切

#### E2E テスト

1~6 を通し、ヘッドレスブラウザ（見た目を提供しないブラウザ）と UI オートメーション（クリックなどの人の動作を自動化する）で実施するテスト
最も広範囲なテストであり、アプリ稼働状況に忠実なテスト

| テスト種別 | 対象範囲           | 特徴                       |
| ---------- | ------------------ | -------------------------- |
| 静的解析   | 型・文法・隣接関係 | 実行なしで問題検出         |
| 単体テスト | モジュール単体     | 小規模・細かな動作確認     |
| 結合テスト | モジュール間連携   | 中規模・連携の確認         |
| E2E テスト | 実アプリの全体     | 実際の挙動に近い・コスト高 |

### テストの目的

ソフトウェアテストは主に、機能テスト、非機能テスト、リグレッションテスト、ホワイトボックステストに分類される。
Web フロントエンドテストでは以下の 3 つが代表的。

#### 機能テスト（インタラクションテスト）

開発対象に不具合がないかを見るテスト。
フロントエンドの場合、インタラクション（クリックなどユーザの操作）のテストが主になる。
ブラウザの API を利用する場合、ヘッドレスブラウザや UI オートメーションを組み合わせる。

#### 非機能テスト（アクセシビリティテスト）

心身特性に隔てのない製品を提供できているかを測るテスト。

#### リグレッションテスト

デグレを見るテスト

### テスト戦略

単体テスト > 結合テスト > E2E テスト で項目数が多くなるように心がける
理由はテストにかかるコストは E2E テストが１番重いから

## 第３章 はじめての単体テスト

### テストの作成

- test 関数
  - 第一引数
    - テストタイトル ... テストの内容を表す文字列
  - 第二引数
    - テスト関数 ... アサーション（検証ちが期待通りか確認する関数）
      - アサーション ... expect(検証値).toBe(期待値)
      - マッチャー ... toBe(期待値)

```ts
test("1 + 2 = 3", () => {
  expect(add(1, 2)).toBe(3);
});
```

### テストグループの作成

関連するいくつかのテストを describe によってグルーピングする。
describe(グループタイトル, グループ関数)

```ts
describe("add", () => {
  test("1 + 1 = 2", () => {
    expect(add(1, 1)).toBe(2);
  });
  test("1 + 2 = 3", () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

test 関数はネストできないが、describe 関数はネストできる

```ts
describe("四則演算", () => {
  describe("add", () => {
    test("1 + 1 = 2", () => {
      expect(add(1, 1)).toBe(2);
    });
    test("1 + 2 = 3", () => {
      expect(add(1, 2)).toBe(3);
    });
  });
  describe("sub", () => {
    test("1 - 1 = 0", () => {
      expect(sub(1, 1)).toBe(0);
    });
    test("2 - 1 = 1", () => {
      expect(sub(2, 1)).toBe(1);
    });
  });
});
```

### テストの実行

```ts
// 全てのテストを実行
npm test

// 特定のテストを実行
npm test ディレクトリ名

// VSCodeの拡張機能「Jest Runner」を使う
```

### 例外のスローを検証するテスト

```ts
// 引数には値ではなく、関数を入れる
expect(() => add(-10, 110)).toThrow();

// エラーメッセージが期待通りか検証
test("引数が0~100の場合、例外をスローする", () => {
  expect(() => add(110, -10)).toThrow("0~100の間で入力してください。");
});
```

### instanceof 演算子による詳細な検証

Error クラスを拡張したクラスを作る

```ts
export class HttpError extends Error {}
export class RangeError extends Error {}

if (err instanceof HttpError) {
  // throw new HttpError("hoge") で入ってくる
}
if (err instanceof RangeError) {
  // throw new RangeError("hoge") で入ってくる
}
```

テストコード

```ts
expect(() => add(110, -10)).toThrow(RangeError);
// スーパークラスなのでテストが通る
expect(() => add(110, -10)).toThrow(Error);
```

### マッチャーの種類

#### 真偽値の検証

- toBeTruthy ... 真かどうか。
- toBeFalthy ... 偽かどうか、null や undefined も通る。
- toBeNull ... null かどうか
- toBeUndefined ... undefined かどうか

#### 数値の検証

- toBe, toEqual ... 等しい
- toBeGraterThan ... 期待値よりも大きい
- toBeGraterThanOrEqual ... 期待値と等しいか大きい
- toBeLessThan ... 期待値よりも小さい
- toBeLessThanOrEqual ... 期待値と等しいか小さい

#### 文字列の検証

- toBe, toEqual ... 等しい
- toContain ... 部分一致
- toMatch ... 正規表現
- toHaveLength ... 文字数の一致
- stringContaining ... オブジェクト内の文字列の部分一致
- stringMatching ... オブジェクト内の正規表現

```ts
expect(obj).toEqual({
  message1: expect.stringContain("hoge");
  message2: expect.stringMatching(/hoge/);
});
```

#### 配列の検証

- toContain ... 値が配列に含まれるか
- toHaveLength ... 配列の長さ
- toContainEqual ... 配列に引数で与えたオブジェクトが入っているかどうか
- arrayContaining ... 引数に配列を与えて、それらが全て含まれているか

#### オブジェクトの検証

- toMatchObject ... 引数で指定した部分的にプロパティが一致するかどうか
- toHaveProperty ... 引数で指定したプロパティが存在するかどうか

完全一致は toEqual でやれば良い

### 非同期処理のテスト

非同期処理のテストでは、**`async / await` を使わないと `return` が必要**になる。

Jest の内部処理

1. `test()` や `it()` の中の関数が呼ばれる
2. その関数の**戻り値**が何かを確認
3. **戻り値が Promise であれば、「非同期処理がある」と判断して完了まで待つ**
4. それ以外（非 Promise）なら、すぐにテスト完了とみなす

```ts
// thenを使った基本的なPromiseのテスト
test("非同期処理のテスト", () => {
  // テスト関数を同期的に書く場合はreturnする
  // でないとundefined(void,つまり非Promise)と判断され、評価されず通ってしまう
  return wait(50).then((duration) => {
    expect(duration).toBe(50);
  });
});

// resolvesマッチャを使ったテスト
test("非同期処理のテスト", () => {
  // テスト関数を同期的に書く場合はreturnする
  return exxpect(wait(50)).resolves.toBe(50);
});

// async/await + resolves
test("非同期処理のテスト", async () => {
  await expect(wait(50)).resolves.toBe(50);
});

// async/await
test("非同期処理のテスト", async () => {
  expect(await wait(50)).toBe(50);
});
```

#### reject の検証

```ts
// catchメソッドにアサーションを書く
test("rejectの検証", () => {
  return timeout(50).catch((duration) => {
    // テスト関数を同期的に書く場合はreturnする
    expect(duration).toBe(50);
  });
});

// rejectマッチャを使う
test("rejectの検証", () => {
  return expect(timeout(50)).rejects.toBe(50); // テスト関数を同期的に書く場合はreturnする
});

test("rejectの検証", async () => {
  await expect(timeout(50)).rejects.toBe(50);
});

// try-catchを使う
test("rejectの検証", async () => {
  expect.assertions(1); // アサーションを1回することを期待する
  try {
    timeout(50);
  } catch (err) {
    expect(err).toBe(50);
  }
});
```

## 第４章 モック

モック ... テストしたいが、環境構築に時間がかかるものや、用意できないものの代替品として用意するもの

モックを使用しないと、多くの場合、API からのデータ取得が失敗したケースの判定が難しい。

| 用語               | 一言で言うと                             | 主な目的               |
| ------------------ | ---------------------------------------- | ---------------------- |
| **スタブ（Stub）** | **戻り値を差し替えるもの**               | **振る舞いを制御する** |
| **スパイ（Spy）**  | **呼び出し（入出力）状況を記録するもの** | **振る舞いを観察する** |

### モジュールをスタブに置き換える

```ts
import { greet } from "./greet";
test("jest.mockを使わない", () => {
  expect(greet("Taro")).toBe("Hello, Taro");
});

// jest.mockを使うと、指定した関数の置き換えが行われる
import { greet } from "./greet";
jest.mock("./greet");
test("jest.mockを使う", () => {
  expect(greet("Taro")).toBe(undefined); // 関数が置き換えられているため、greet関数はundefinedを返す
});

// 内部実装をする
import { greet } from "./greet";
jest.mock("./greet", () => ({
  sayGoodBye: (name: string) => `GoodBye, ${name}`,
}));
test("jest.mockを使い、仮実装をする", () => {
  expect(greet("Taro")).toBe(GoodBye, Taro);
});

// 1つは本実装、1つは仮実装
import { greet, sayGoodBye } from "./greet";
jest.mock("./greet", () => ({
  ...jest.requireActual("./greet"),
  sayGoodBye: (name: string) => `GoodBye, ${name}`,
}));
```

### ライブラリを置き換える

```ts
jest.mock("next/router", () => require("next-router-mock"));
```

### Web API のスタブ実装

リクエスト成功

```ts
// jest.spyOn(対象のオブジェクト, 対象の関数);
// spyOnで対象オブジェクトの対象の関数が何を返り値として返すかを設定する。
// 以下の場合はFeaturesに存在するgetMyProfile関数が{id, email}のオブジェクトを返すという定義
jest.spyOn(Features, getMyProfile).mockResolvedValueOnce({
  id: hoge,
  email: hoge,
});
await expect(getGreet()).resolves.toBe("Hello");
```

リクエスト失敗

```ts
jest.spyOn(Features, getMyProfile).mockRejectedValueOnce(httpError);
await expect(getGreet()).rejects.toMatchObject({
  err: { message: "internal server error" },
});
```

モック生成関数を使った場合

```ts
// fixtures ディレクトリにあらかじめレスポンス用のデータを用意
exporrt function mockGetMyArticles(status = 200) {
  if(status > 299) {
    return jest.spyOn(Features, "getMyArticles").mockRejectedValueOnce(httpError)
  }
  return jest.spyOn(Features, "getMyArticles").mockResoledValueOnce(getMyArticlesData)
}

test("テストが成功した時", () => {
  mockGetMyArticles();
})
test("テストが失敗した時", () => {
  mockGetMyArticles(500);
})
```

### スパイ

```ts
test("スパイの使い方", () => {
  const mockFn = jest.fn();
  mockFn();
  expect(mockFn).toBeCalled(); // mockFn関数が呼ばれたか
  expect(mockFn).not.toBeCalled(); // mockFn関数が呼ばれていないか
  expect(mockFn).toHaveBeenCalledTimes(1); // mock関数が1回呼ばれたか

  function greet(message: string) {
    mockFn(message);
  }
  greet("Hello")
  expect(mockFn).toHaveBeenCalledWith("Hello"); // 引数がHelloで呼び出されたか
}
```

### 時刻を設定する

jest.useFakeTimers : Jest に偽のタイマーを使用するように指示
jest.setSystemTime : 偽のタイマーで使用される現在システム時刻を設定
jest.useRealTimers : Jest に真のタイマーを使用するように指示

```ts
describe("greetByTime", () => {
  beforEach(() => {
    jest.useFakeTimes();
  });
  afterEach(() => {
    jest.useRealTimers();
  });
  test("8時", () => {
    jest.setSystemTime(new Date(2022, 7, 20, 8, 0, 0));
    expect(greetByTime()).toBe("おはよう");
  });
});
```
