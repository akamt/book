# ユーティリティ型 \(Utility types\)

TypeScriptで開発するということはJavaScriptに型を付与することです。型を付与する時にあらかじめ用意されている便利な型の表現がいくつかありますのでそちらを紹介します。今回紹介するものは全てではないので、他も興味がある方は公式やソースコードを参照してください。

これから紹介するユーティリティ型は全てTypeScriptのパッケージで定義されており、ソースコードも同梱されているのでその実装方法を見ることが可能です。

本章のユーティリティ型の仕様例で、ことわりなく`Person, User`というオブジェクトを使用している部分は、下記に示すタイプエイリアスとそのオブジェクトを使うものとします。

```typescript
type Person = {
  surname: string;
  middleName?: string;
  givenName: string;
};

type User = {
  surname: string;
  middleName?: string;
  givenName: string;
  age: number;
  address?: string;
  nationality: string;
  createdAt: string;
  updatedAt: string;
};
```

## `Required<T>`

全てのプロパティから`Optional`であることを意味する`?`を取り除きます。

`Required<Person>`は以下と同じ型になります。

```typescript
type RequiredPerson = {
  surname: string;
  middleName: string;
  givenName: string;
};
```

## `Readonly<T>`

オブジェクトのプロパティに対する代入を防ぐ`readonly`を全てのプロパティに対して適用します。プロパティがオブジェクトだった場合、それが持つプロパティまでは`readonly`にしないことに注意してください。これは普通の`readonly`と同じ挙動です。

`Readonly<Person>`は以下と同じ型になります。

```typescript
type ReadonlyPerson = {
  readonly surname: string;
  readonly middleName?: string;
  readonly givenName: string;
};
```

## `Partial<T>`

全てのプロパティに`Optional`であることを意味する`?`を適用します。

`Partial<Person>`は以下と同じ型になります。

```typescript
type PartialPerson = {
  surname?: string;
  middleName?: string;
  givenName?: string;
};
```

### より便利な省略可能な引数

関数の引数を`Optional parameters, Default parameters`と`Destructuring assignment`と`Partial<T>`を組み合わせることによって省略可能でありながら見やすい関数を実装できます。これらの用語ついては関数のページにて取りあげておりますのでご参照いただければと思います。

{% page-ref page="function.md" %}

ユーザーの検索をかける関数を作り、その属性に応じて検索ができるとします。

```typescript
function findUsers(name?: string, nationality?: string, age?: number): Promise<User[]> {
  // ...
}
```

ですが、この`findUsers()`のシグネチャだと**名前、国籍は問わないが年齢だけXX才の**ユーザーが欲しい時は引数の順番を維持するために他の引数は`undefined`を入力しなければいけません。

```typescript
findUsers(undefined, undefined, 22);
```

この例題は引数が3個しかないためそこまで見辛くはないですが、多い引数の関数になると、どこに引数を入れて他を`undefined`とするかが面倒です。これを`Partial<T>`を使って見た目をよくできます。

まず引数は全てオブジェクトで受け渡しされるものとしてそのオブジェクトの型を定義します。さらにプロパティを省略可能にするために`Partial<T>`をつけます。

```typescript
type FindUsersArgs = Partial<{
  name: string;
  nationality: string;
  age: number;
}>;
```

これを関数`findUsers()`の引数にします。

```typescript
function findUsers({ name, nationality, age }: FindUsersArgs): Promise<User[]> {
  // ...
}
```

これだけではまだ呼び出し側は省略ができません。`findUsers()`を使用する時は仮に何も設定する必要がなくても引数に`{}`を与えなければいけません。

```typescript
findUsers({});
```

引数を省略できるようにするために`Default parameters`を使い省略時に`{}`が代入されるようにします。

```typescript
function findUsers({ name, nationality, age }: FindUsersArgs = {}): Promise<User[]> {
  // ...
}

findUsers();
findUsers({ age = 22 });
```

`FindUsersArgs`の右の`= {}`がそれにあたります。これにより`findUsers()`は引数がなくても呼び出せるようになります。特定の引数だけ値をすることもできます。`findUsers({ age = 22 })`がその例です。

さらに`FindUsersArgs`側にも`Default parameters`を設定することで初期値することもできます。

```typescript
function findUsers({ name = 'John Doe', nationality = 'Araska', age = 22 }: FindUsersArgs = {}): Promise<User[]> {
  // ...
}
```

## `Record<K, T>`

`Index signatures`と似たような効果を持ちます。`K`はオブジェクトのキーを意味し、`string, number, symbol`型またはそれらのユニオン型を指定できます。`T`はオブジェクトのプロパティを意味します。`Index signatures`と異なり`K`に`symbol`型も適用できることに注意してください。

`Index signatures`についてはタイプエイリアスの頁を参照ください。

{% page-ref page="type-alias.md" %}

`Person`を`Record`を使って表現すると以下になりますが`Record`はプロパティを`Optional`にする機能はないため`Person`とは完全に一致せず、上記の`Required<Person>`と同じものになります。

```typescript
type Name = 'surname' | 'middleName' | 'givenName';
type Person = Record<Name, string>;
```

## `Pick<T, K>`

ある巨大なタイプエイリアスから、必要な部分だけを抽出します。`K`はタイプエイリアス`T`のキーの部分集合である必要があります。

以下は`Pick`を使って`User`から`Person`を作る一例です。

```typescript
type Necessary = 'surname' | 'middleName' | 'givenName';
type Person = Pick<User, Necessary>;
```

### キーの部分集合である。について

部分集合と聞くと難しいかもしれませんが、言い換えると**キーに存在しないリテラルタイプを指定できない**ことを意味しています。上記例は`User`の`middleName, givenName`の`Name`は大文字から始まりますが、これを小文字にしたタイプエイリアスを定義するとこれは`Pick`では使用できません。

```typescript
type Necessary = 'surname' | 'middlename' | 'givenname';
type Person = Pick<User, Necessary>;
// -> Type '"middlename"' is not assignable to type '"surname" | "middleName" | "givenName" | "age" | "address" | "nationality" | "createdAt" | "updatedAt"'.
```

### 大元となる型の定義に追従する

書籍を扱うサービスを作ったとして、書籍を意味するオブジェクト`Book`が以下のように定義されているとします。

```typescript
type Book = {
  id: number;
  title: string;
  author: string;
  createdAt: Date;
  updatedAt: Date;
};
```

これを参考にして`Book`を作成するための入力データとして`BookInputData`を作るとします。これは外部からのリクエストで作成され、`id, createdAt, updatedAt`はこのサービスで後付けで割り当てられるとすれば`BookInputData`は以下になります。

```typescript
type BookInputData = {
  title: string;
  author: string;
};
```

ここで`author`プロパティが`string`ではなく`Person`になる必要があったとします。`Book, BookInputData`を独立して定義しているとこの変更のために都度、各々の`author`プロパティを変更する必要があります。

```typescript
type Book = {
  id: number;
  title: string;
  author: Person;
  createdAt: Date;
  updatedAt: Date;
};

type BookInputData = {
  title: string;
  author: Person;
};
```

これらの定義が近くにある状態ならまだしも、異なるファイルにあれば非常に探し辛くなります。

そこで`BookInputData`を`Pick<T, K>`を使って定義しなおします。

```typescript
type BookInputData = Pick<Book, 'title' | 'author'>;
```

このようにすれば`BookInputData`は少なくとも`Book`とコード上の繋がりができる上に、`author`プロパティの型変更を自動で追従してくれるようになります。

## `Omit<T, K>`

`Pick`と逆の動作です。つまり不必要な部分を取り除きます。`K`はタイプエイリアス`T`のキーの部分集合である**必要はありません**。タイポなどに対する検査が`Pick`と比べて貧弱なので注意してください。

以下は`Omit`を使って`User`から`Person`を作る一例です。

```typescript
type Unnecessary = 'age' | 'address' | 'nationality' | 'createdAt' | 'updatedAt';
type Person = Omit<User, Unnecessary>;
```

### キーの部分集合である必要がない。について

`Pick`と逆です。`User`の`createdAt, updatedAt`の`At`は大文字から始まりますが、これに気づかずに小文字で書いてしまってもこのことに対する指摘はなく`Omit`の結果は`createdAt, updatedAt`を含んでしまいます。

```typescript
type Unnecessary = 'age' | 'address' | 'nationality' | 'createdat' | 'updatedat';
type Person = Omit<User, Unnecessary>;
// -> 
// {
//    surname: string,
//    middleName?: string,
//    givenName: string,
//    createdAt: string,
//    updatedAt: string
// }
```

## `Exclude<T, U>`

ユニオン型`T`から`U`を取り除きます。

```typescript
type Grade = 'A' | 'B' | 'C' | 'D' | 'E';
type PassingGrade = Exclude<Grade, 'E'>;
```

この例は成績についてです。落第を示す成績が`'E'`でそれ以外は及第だとすればこのようにして及第を示すタイプエイリアス`PassingGrade`を作ることができます。

### `Exclude`の注意点

`U`は`T`の部分集合である制限がありません。つまり`Omit`と同様、タイポなどに気をつけなければいけません。

以下は`Pull Request`に関するタイプエイリアスと解釈してください。

```typescript
type PullRequestState = 'drft' | 'reviewed' | 'rejected';
type MergeableState = Exclude<PullRequestState, 'drft' | 'rejected'>;
```

`MergeableState`は`'reviewed'`を意味しますが安易に`Exclude`を使うと2点の問題があります。

#### `PullRequestState`に新しい状態が追加された時

`PullRequestState`に`'testFailed'`という`MergeableState`に含めたくない状態を追加したとします。するとこの修正に伴って`MergeableState`の`U`\(第2ジェネリクス\)も同時に修正しないといけません。これを忘れると`'testFailed'`は`MergeableState`に含まれてしまいます。

```typescript
type PullRequestState = 'drft' | 'reviewed' | 'rejected' | 'testFailed';
type MergeableState = Exclude<PullRequestState, 'drft' | 'rejected'>;
// -> 'reviewed' | 'testFailed'
```

#### タイポ

`PullRequestState`の`'drft'`は誤字だったので`'draft'`に修正しました。これも同様に`MergeableState`の第2ジェネリクスの修正を忘れると`MergeableState`に`'draft'`が含まれてしまいます。

```typescript
type PullRequestState = 'draft' | 'reviewed' | 'rejected';
type MergeableState = Exclude<PullRequestState, 'drft' | 'rejected'>;
// -> 'draft' | 'reviewed'
```

#### ユニオン型ではない型を指定する

また`Exclude`の第1ジェネリクスにユニオン型ではない型を指定しても意味がありません。

```typescript
type InputText = Exclude<string, ''>;
// -> string
```

入力した文字を表す型として`InputText`を定義したとしても、この`InputText`は`''`以外の全ての`string`とは解釈されず、ただの`string`型になります。

## `Extract<T, U>`

`Exclude`と逆です。ユニオン型`T`から`U`を抽出します。

```typescript
type Grade = 'A' | 'B' | 'C' | 'D' | 'E';
type FailingGrade = Extract<Grade, 'E'>;
```

落第を表す成績が`'E'`ならこのようにして落第を表すタイプエイリアス`FailingGrade`を作ることができます。

### `Extract`の注意点

`Eclude`と同様`U`については`T`の部分集合である制限がありません。タイポに気をつける必要があります。

#### タイポ

```typescript
type PullRequestState = 'draft' | 'reviewed' | 'rejected';
type MergeableState = Extract<PullRequestState, 'reviewd'>;
```

`MergeableState`の第2ジェネリクスはタイポです。これに気づかないと`MergeableState`は`never`という型になり、いかなる値も代入できません。

#### ユニオン型ではない型を指定する

`Extract`の第1ジェネリクスにユニオン型ではない型を指定すると`never`になります。

```typescript
type Zero = Extract<number, 0>;
// -> never
```

`0`を表す型として`Zero`を定義したとしても、これはただの`never`型になります。

