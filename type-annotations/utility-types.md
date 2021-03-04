---
description: 提供一些實用工具涵式，處理複雜的型別操作
---

# Utility Types

## $Keys&lt;T&gt; <a id="keys"></a>

Flow 可以定義 [union types](https://flow.org/en/docs/types/unions/) 來達到類似 enum 的效果。

```javascript
// @flow
type Country = 'US' | 'IT' | 'FR';

const us: Country = 'US';
const jp: Country = 'JP'; // 錯誤! 因為 Countries 沒有定義 JP
```

這是一個很簡單的國別碼範例，但是當面對到需求擴增的時候可能會顯得功能有點不足。  
假設我們的需求從簡單的國別碼擴增到使用國別碼取得國家全名。

那我們的做法會變成這樣:

```javascript
// @flow
type Country = 'US' | 'IT' | 'FR';

const countries = {
    US: '美國',
    IT: '義大利',
    FR: '法國',
};

function printCountryName(code: Country): void {
    console.log(countries[code]);
}

printCountryName('IT'); // 義大利
printCountryName('JP'); // 錯誤! 因為 Countries 沒有定義 JP

```

這樣會動是沒錯，但是不夠好 _\( doesn’t feel very DRY \)_，因為我們定義了兩次國別碼。

在這種情況下，`$Keys<T>` 會是我們的好朋友。  
我們用原本的範例，使用 `$Keys<T>` 來改寫。

```javascript
// @flow
const countries = {
    US: '美國',
    IT: '義大利',
    FR: '法國',
};

type Country = $Keys<typeof countries>;

function printCountryName(code: Country): void {
    console.log(countries[code]);
}

printCountryName('IT'); // 義大利
printCountryName('JP'); // 錯誤! 因為 Countries 沒有定義 JP
```

範例中的 `type Country = $Keys<typeof countries>` 就相等於前面用的 `type Country = 'US' | 'IT' | 'FR'` 差別在於後者是使用 `$Keys<T>` 解析 `countries` 物件所產生的。

{% hint style="info" %}
有了 `$Keys<T>，`我們不用再自己定義 Unions 類別，取而代之的是解析物件本身的 keys 來取得類別。
{% endhint %}

## $Values&lt;T&gt; <a id="values"></a>

$Values&lt;T&gt; 會從物件中提取值的屬性別，最終以 [union types](https://flow.org/en/docs/types/unions/) 回傳。  
\( 注意! 不是從 `key` 哦! 他是去解析 `value` 的 \)

```javascript
// flow
type Person = {
    name: string,
    age: number,
    height: number,
    alive: boolean,
};

// 這兩種型態是一樣的
type PersonValue = string | number | boolean;
type Person$Value = $Values<Person>;

const name: Person$Value = 'Wayne';
const age: Person$Value = 26;
// 錯誤! 因為從 Person 型別中解析出來的 union types 中不含有 function
const swim: Person$Value = () => {};
```

{% hint style="info" %}
`$Values<T>` 可以幫我們快速從已存在的型態中，取出其 union types。

\( 但是老實說，我也還沒有找到具體使用的情境 \)
{% endhint %}

### 小陷阱 <a id="trap"></a>

假設我們定義了一個 `children` 型態是 `Array<String>`。

```javascript
// @flow
type Person = {
    name: string,
    age: number,
    children: Array<string>, // 新定義 Array
}

type Person$Value = $Values<Person>;

const children: Person$Value = ['Willy', 'Lee']; // ok
```

但是當我們現在有多一個 `account` 型態是 `Array<number>`。\( 先不要管為什麼要這樣，我就演示演示而已~ \)

```javascript
// @flow
type Person = {
    name: string,
    age: number,
    children: Array<string>, // 新定義 Array<string>
    account: Array<number>, // 多一個 Array<number>
};

type Person$Value = $Values<Person>;

const children: Person$Value = ['Willy', 'Lee']; // 錯誤!
const account: Person$Value = [1, 12, 123]; // 錯誤!
```

我們預期兩種都接受的型態應該是 `Array<string | number>`  
但是它幫我們解析成 `Array<string> | Array<number>`   
所以就不行囉~~ 要小心。

## $ReadOnly&lt;T&gt; <a id="readonly"></a>

會將已經定義好的型別轉換成 `ReadOnly`修飾 \( 只可讀 \)，詳細請參考 [ReadOnly ](https://flow.org/en/docs/types/interfaces/#toc-interface-property-variance-read-only-and-write-only)章節。

這意味這這兩個是相等的:

```javascript
// @flow
type ReadOnlyObj = {
    +key: any,
};
```

```javascript
// @flow
type ReadOnlyObj = $ReadOnly<{
    key: any
}>;
```

這兩個同樣具有不可被寫入只可讀取值的特性。

`$ReadOnly<T>` 在需要將一個既存物件轉為一個 `ReadOnly`版本的資料型態是很方便的，不需要重新定義一個同樣結構、卻只是在每個 `key`上加個 `ReadOnly`修飾的物件。

```javascript
// @flow
type Person = {
    name: string,
    age: number,
};

type ReadOnlyPerson = $ReadOnly<Person>;

function fn(person: ReadOnlyPerson) {
    // 皆會有錯誤 
    // ... 略過 ... property `name` is not writable.Flow(cannot-write)
    person.name = 'Wayne';
    // ... 略過 ... property `age` is not writable.Flow(cannot-write)
    person.age = 26;
}
```

## $Exact&lt;T&gt; <a id="exact"></a>

可以從原先定義的型別轉換為精確匹配型別，參考 [Exact object types](https://flow.org/en/docs/types/objects/#toc-exact-object-types)。

```javascript
// @flow
type Person = {
    name: string,
    age: number,
};

function fn(person: Person) {
    // ... 跑跑跑 ...
}

// 成功，因為預設的型別判斷是只要定義的都有就好
fn({
    name: 'Wayne',
    age: 26,
    children: [],
});

```

現在改用 `$Exact<T>` 來做精確限制。

```javascript
// @flow
type Person = {
    name: string,
    age: number,
};

type ExactPerson = $Exact<Person>;

function exactFn(person: ExactPerson) {
    // ... Exact 跑跑跑 ...
}

// 錯誤! 因為改成了精確匹配
exactFn({
    name: 'Wayne',
    age: 26,
    children: [], // 多定義了 :(
});

```

{% hint style="info" %}
補充說明:

```javascript
type ExactUser = $Exact<{name: string}>;
type ExactUserShorthand = { | name: string | };
```

這兩個是一樣的哦~~~
{% endhint %}

## $Diff&lt;A, B&gt; <a id="diff"></a>

顧名思義，他會比較 `A` 與 `B` 的差別，我的了解是:

> 以 A 為主，去除 B 的型別，取出剩餘的型別。

{% code title="借用官方範例，實在精妙呀!" %}
```javascript
// @flow
type Props = {
    name: string,
    age: number,
};
type DefaultProps = { age: number };
type RequiredProps = $Diff<Props, DefaultProps>;

function setProps(props: RequiredProps) {
    // ...
}

setProps({ name: 'foo'}); // ok
setProps({ name: 'foo', age: 42, baz: false }); // 可以傳入額外的屬性，沒問題的
setProps({ age: 42 }); // 錯誤! 一定要有 name
```
{% endcode %}

眼尖的朋友或許注意到了，這個跟我們在 React 定義 Props 型別與預設 Props 是不是很像呢?  
哈哈沒錯，官方也這麼說道了:

> `$Diff` is exactly what the React definition file uses to define the type of the props accepted by a React Component.

就是用在 React Component 中的囉~ 給有發現的自己拍拍手 ^^   
\( 我自己要不是整理這份筆記，我也沒發現 \)

拉回來，繼續講 `$Diff<A, B>`，其實個人認為在理解上，可以把 `$Diff<A, B>` 方法以 `A - B` 去理解，會比較好理解。

我們的 `A` 有 `name`, `age` 兩個屬性，`B` 有一個 `age` 屬性，這樣 `Diff<A, B>`比較下去，就會只留下 `A` 的 `name` 屬性。

```javascript
// @flow
type Props = {
    name: string,
    age: number,
};
type DefaultProps = {
    age: number
};
type RequiredProps = $Diff<Props, DefaultProps>; // 只剩下 name 一個屬性
```

為什麼我前面要不厭其煩的不斷寫道 `$Diff<A, B>` ?  
因為我要讓各位記住， `A` 與 `B`放置的位置是有不同意義的! 就像減法中 `10 - 5` 與 `5 - 10` 得出來的結果不會相等一樣。而 `$Diff<A, B>` 位置放錯是會有錯誤的哦~

`$Diff<A, B>` 的要求是，`B` 之中的屬性一定要是 `A` 所存在的。

```javascript
// @flow
type Props = {
    name: string,
    age: number,
};
type DefaultProps = {
    age: number,
    other: string,
};

// 錯誤!
// Cannot instantiate `$Diff` because undefined property `other` [1] ...
type RequiredProps = $Diff<Props, DefaultProps>;
```

`B` 的 `other`屬性 `A` 沒有，所以就會報出「 `B` 含有 `A` 所沒有的屬性」，這種錯誤了。

## $Rest&lt;A, B&gt; <a id="rest"></a>

要瞭解 `$Rest<A, B>` 的話一定要先瞭解 JavaScript 的 [解構賦值 \( Destructuring assignment \)](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

```javascript
const person = {
    name: 'Wayne',
    age: 26,
    children: ['Andy', 'Tom', 'Jayson'],
    foo: 'Test',
    bar: 123,
    baz: false,
}

const { name, children, ...other } = person;

console.log(name); // Wayne
console.log(children); // [ 'Andy', 'Tom', 'Jayson' ]
console.log(other); // { age: 26, foo: 'test', bar: 123, baz: false }
```

大致演示一下它的概念。  
我們有一個 `person` 物件，含有多個屬性，當我們使用解構方法提取出 `name` 與 `children` 後，再將剩餘的屬性放入 `other` 物件中，所以 `other` 物件就會有除了剛才被提去出去的 `name` 與 `children` 以外的全部屬性。

那 $Rest&lt;A, B&gt; 的概念也就類似這樣，它會保留 A 擁有的且 B 沒有的屬性，flow 判定擁有屬性是以[精確指定 \( Exact Object Types \)](https://flow.org/en/docs/types/objects/#toc-exact-object-types)為擁有依據哦!。因此是這種定義方法 `{| name: string, age: number |}` 而不是 `{ name: string, age: number }` 哦!

```javascript
// @flow
type Person = {
    name: string,
    age: number,
    children: Array<string>,
    foo: string,
}

type RestPerson = $Rest<Person, {| age: number, foo: string |}>

function fn(person: RestPerson) {
    person.name // ok!
    person.age // 錯誤!
    person.children // ok!
    person.foo // 錯誤!
}

```

可以看到因為我們送給 `$Rest<A, B>` 的 `B` 當中含有 `age` 與 `foo` 兩個屬性，因此在下面呼叫 `person.age` 與 `person.foo` 的時候都會報錯。

### 與 $Diff&lt;A, B&gt; 的差別 <a id="between-diff"></a>

這樣乍看下來... 跟 `$Diff<A, B>` 是不是很像? 確實是滿像的，但還是有差別的! 他們最主要的差別就展現在 [必須有 與 optional](https://flow.org/en/docs/types/primitives/#toc-optional-object-properties) 上面。  
當我們使用 `$Rest<{| n: number |}, {}>`  與 `$Diff<{| n: number |}, {}>` 時，答案就呼之欲出了。

```javascript
// @flow
type RestType = $Rest<{| n: number |}, {}> // {| n?: number |}
type DiffType = $Diff<{| n: number |}, {}> // {| n: number |}

```

差別就在 optional 囉!!

## $PropertyType&lt;T, k&gt; <a id="propertytype"></a>

傳入 T \( Type \) 與 k \( Key \)，回傳指定 `k` 的型態，  
`T` 必須是類型，  
而 `k` 呢，必須是一個字串。

```javascript
// @flow
type Person = {
  name: string,
  age: number,
  parent: Person
};

function nameFn(name: $PropertyType<Person, 'name'>) {
  // ...
}

nameFn('Wayne'); // ok!
nameFn(123); // 錯誤，必須是 string


```

可以看到若是指定 `name` 這個 `key` 進去，它會提取出 `Person` 中 `name` 的型別，也就是 `string`。

剛才上面的是我個人的理解方式，接下來用官方的範例吧！

```javascript
// @flow
type Person = {
  name: string,
  age: number,
  parent: Person
};

// ok
const newName: $PropertyType<Person, 'name'> = 'Toni Braxton';
// ok
const newAge: $PropertyType<Person, 'age'> = 51;
// 錯誤，因為要是 Person 形態
const newParent: $PropertyType<Person, 'parent'> = 'Evelyn Braxton';
// 如果要正確的話要這樣哦！
const newParent2: $PropertyType<Person, 'parent'> = {
  name: 'Wayne',
  age: 123,
  parent: {}
};

// 錯誤! children 不存在
const newChildren: $PropertyType<Person, 'children'> = {};
```

`$PropertyType<T, K>` 在 React 的用例場景上尤其有用。

```javascript
// @flow
import React from 'react';

// 定義 Props
type Props = {
  text: string,
  onMouseOver: ({x: number, y: number}) => void
}

// 指定這個 Component 會有 text 跟 onMouseOver 兩個 Prop 傳入
class Tooltip extends React.Component<Props> {
  props: Props;
}

// 存 Tooltip 中取得 props 屬性的型別，剛好就是剛才定義的 Props
const someProps: $PropertyType<Tooltip, 'props'> = {
  text: 'foo',
  onMouseOver: (data: {x: number, y: number}) => undefined
};

// 錯誤
const otherProps: $PropertyType<Tooltip, 'props'> = {
  text: 'foo'
  // 缺少 `onMouseOver` 定義
};

```

延續剛才的範例，我們也可以使用巢狀的定義 `$PropertyType` 像是這樣

```javascript
// @flow
// 指定這個 Component 會有 text 跟 onMouseOver 兩個 prop 傳入
type Props = {
  text: string,
  onMouseOver: ({x: number, y: number}) => void
}

class Tooltip extends React.Component<Props> {
  props: Props;
}

type PositionHandler = $PropertyType<$PropertyType<Tooltip, 'props'>, 'onMouseOver'>;

const handler: PositionHandler = (data: {x: number, y: number}) => undefined; // ok
const handler2: PositionHandler = (data: string) => undefined; // 錯誤!傳入參數錯誤
```

稍微來解析一下，先從 `$PropertyType<Tooltip, 'props'>` 中由 **props** 字串轉出 `Props` 類別，轉出來後，再由 `$PropertyType<$PropertyType<Tooltip, 'props'>, 'onMouseOver'>` 中解析出 **onMouseOver** 這個 `function`  類型。

## $ElementType&lt;T, K&gt; <a id="elementtype"></a>

> 之後會用比較多 [Type Casting Expressions](https://flowjs.judysocute.com/xing-bie-ban-yan-biao-da-shi-type-casting-expressions)，不熟的話再去複習一下吧！

`$ElementType<T, K>` 跟 `$PropertyType<T, k>` 一樣  
都是傳入 `T` 與 `k`，回傳指定 `k` 的型態。

在特定場景下，它的用法會跟 `$PropertyType<T, k>` 一樣

```javascript
// @flow
type Person = {
    name: string,
    age: number,
}

(123: $ElementType<Person, 'age'>); // ok
(123: $PropertyType<Person, 'age'>); // ok

// children 不存在
(123: $ElementType<Person, 'children'>); // 錯誤!
(123: $PropertyType<Person, 'children'>); // 錯誤!
```

在這種情況下確實是會有一樣的回傳結果，只是 ElementType&lt;T, K&gt; 的 T 不一定要接收物件型別，而是可以接收一個物件、陣列、甚至是 Tuple 的型別。接收到後再根據 K 傳入的值去解析出相對應的欄位型別。

```javascript
// @flow
type Tuple = [boolean, string];

(true: $ElementType<Tuple, 0>); // ok!
('123': $ElementType<Tuple, 1>); // ok!
('123': $ElementType<Tuple, 2>); // 錯誤! 長度並沒有到 2

(true: $PropertyType<Tuple, 0>); // 錯誤! $PropertyType<T, k> 中，k 必須是字串
```

可以看到我們這次用了 Tuple 類別，`$ElementType<T, K>` 可以使用位置解析出型別，但是 `$PropertyType<T, k>` 不行，因為 `$PropertyType<T, k>` 的 `k` 是只可以傳入字串的，不接受其他型別的傳入，  
而 `$ElementType<T, K>` 並沒有這個限制，只要傳入的 `K` 是 `T` 存在的屬性就好，這也是 `$ElementType<T, K>` 跟 `$PropertyType<T, k>` 最大的差別囉！

```javascript
// @flow

// 物件解析
type Obj = {
    [key: string]: number,
};

(42: $ElementType<Obj, string>); // ok!
(42: $ElementType<Obj, boolean>); // 錯誤! Obj 內並沒有 boolean 型別
(true: $ElementType<Obj, string>); // 錯誤! 回傳值是 number 型別

// 解析陣列，由於一個陣列我們通常不知道它確切的長度，因此我們直接使用 number 作為 key 就好了
type Arr = Array<boolean>;

(true: $ElementType<Arr, number>);
(true: $ElementType<Arr, boolean>); // 錯誤! 陣列中並沒有 boolean 型別作為 key
('foo': $ElementType<Arr, number>); // 錯誤! 回傳值是 boolean 型別
```

也可以使用巢狀的使用 `$ElementType` ，在需要取得型別中的型別時會特別有用。

像這樣

```javascript
// @flow
type NumberObj = {
  nums: Array<number>,
};

(42: $ElementType<$ElementType<NumberObj, 'nums'>, number>);
```

稍微來解析一下，先從 `$ElementType<NumberObj, 'nums'>` 中由 **nums** 字串轉出 `NumberObj` 陣列類別，轉出來後，再由 `$ElementType<$ElementType<NumberObj, 'nums'>, number>` 透過 array key \( number \) 解析出 `number`  類型。

巢狀有點複雜，可以多看幾次！

## $NonMaybeType&lt;T&gt; <a id="nonmaybetype"></a>

顧名思義，`$NonMaybeType<T>` 會將收到的 `T` 轉換為 **non-mayby type**，簡單一點瞭解就是將允許 `null` 或 `undefined` 的定義轉為不允許啦~

```javascript
// @fow
type MaybeName = ?string;
type Name = $NonMaybeType<MaybeName>;

('Wayne': MaybeName); // ok
(null: MaybeName); // ok

('Wayne': Name); // ok
(null: Name); // 錯誤
```

## $ObjMap&lt;T, F&gt; <a id="objmap"></a>

> 這段比較不好懂，會用到比較多的泛型  \( Generic \)，與  js 方法，如果一時半刻看不懂，不要怪自己，是很正常的。只要花點時間細看一下，我相信一定能看懂！功力也會有些許進步的。  
> 也不要放棄唷！因為接下來的很多都要在這個的基礎上做延伸嘿！

這個不太好懂，如果一時半刻看不懂，不要怪自己，是很正常的！

首先我們來看一個純 js `function`

```javascript
function run(o) {
  return Object.keys(o).reduce(
    (acc, k) => Object.assign(acc, { [k]: o[k]() }),
    {}
  );
}

const obj = run({
  foo: () => 2,
	bar: () => true,
	baz: () => 'test'
})
```

這很簡單，就是一個 `function` 接收一個物件，而這個 `function` 會把物件中每個屬性都當成 `function` 去跑，再將回傳值放入一個空物件中，用 `reduce()` 方法做出新的物件回傳。 \( 不熟的話麻煩自己再去補一下 `reduce()` 跟 `Object.assign()` 囉！ \)

但是這樣的回傳我們要怎麼知道 `obj` 他的回傳型別呢？  
我怎麽知道 `foo` 是 `number`、`bar` 是 `boolean` 呢？

這就是 `$ObjMap<T, F>` 的使用時機了！

`$ObjMap<T, F>` 收取  
物件形態 `T`  
`function` 形態 `F`

會從 F 中提取回傳形態作為該屬性最終的形態依據，換句話說，`$ObjMap<T, F>` 會解析 `T` 的所有屬性，並且一一執行 `F` 取得回傳值。

再多說下去就要茫了~直接來看範例吧！

```javascript
// @flow
type ExtractReturnType = <V>(() => V) => V;

function run<O: {[key: string]: Function}>(o: O): $ObjMap<O, ExtractReturnType> {
  return Object.keys(o).reduce(
    (acc, k) => Object.assign(acc, { [k]: o[k]() }),
    {}
  );
}

const obj = run({
  foo: () => 2, // ok!
	bar: () => true, // ok!
	baz: () => 'test' // ok!
})

(obj.foo: number);
(obj.bar: bool);
(obj.baz: string);
(obj.baz: bool); // 錯誤!

```

我們在前面先定義了一個 `function` 形態 **`ExtractReturnType`**，他的內容就是簡單地將收去到的形態照原本回傳回去而已。

而 `run()` 做的事情跟原先一樣，只是多加了些型別的判斷、使用，  
首先 `run<O: {[key: string]: Function}>` 是限制傳入的參數必須為物件、且物件的 `key` 必須是字串型式、`value` 必須是個 `Function`。

再來`run<O: {[key: string]: Function}>(o: O): $ObjMap<O, ExtractReturnType>` 的後半段，`$ObjMap<O, ExtractReturnType>` 就是將收到的物件 `O` 與剛才定義 **`ExtractReturnType`** `F` 傳入，`$ObjMap<T, F>` 就會執行它的任務了。

## $ObjMapi&lt;T, F&gt; <a id="objmapi"></a>

`$ObjMapi<T, F>` 與 `$ObjMap<T, F>` 看起來很類似，差別在於 `$ObjMapi<T, F>` 的 `Function` `F` 會呼叫 `T` 的 `key` 與 `value`，而不是只針對 `value` 做處理。

```javascript
// @flow
const person = {
  name: () => 'Wayne',
  age: () => 26
};

type ExtractReturnObjectType = <K, V>(K, () => V) => { k: K, v: V };

declare function run<O: Object>(o: O): $ObjMapi<O, ExtractReturnObjectType>;

(run(person).name: { k: 'name', v: string}); // ok!
(run(person).age: { k: 'age', v: number}); // ok!

(run(person).name: { k: 'age', v: string}); // 錯誤! name 的 k 應該是 name
(run(person).age: { k: 'age', v: string}); // 錯誤! age 的 v 應該是 number
```

## $TupleMap&lt;T, F&gt; <a id="tuplemap"></a>

`$TupleMap<T, F>` 接收一個可迭代的型別 `T` \( 像是 Tuple、陣列... \)，另外再接收一個 `function` 形態的 `F`。  
最終回傳另一個可迭代的物件，物件中每個元素都經過 `F` 的回傳，有點像是 JavaScript 的 `map()` 方法。

此功能的用例上也是比較特殊一點的，假設我們有一個陣列裡面每個元素都是 function，我們需要取得每個元素的回傳值作為最終的形態。

```javascript
// @flow
type ExtractReturnType = <V>(() => V) => V;

function run<A, I: Array<() => A>>(iter: I): $TupleMap<I, ExtractReturnType> {
  return iter.map(fn => fn());
}

const arr = [
  () => 'foo',
  () => 'bar',
  () => 'baz',
];

(run(arr)[0]: string); // ok
(run(arr)[1]: string); // ok
(run(arr)[2]: string); // ok
(run(arr)[2]: number); // 錯誤! 應該是 string 才對

```

## $Call&lt;F, T...&gt; <a id="call"></a>

`$Call<F, T...>` 接收一個 `F` 參數與 **0 個或多個** `T` 參數，並且已 `F` 的回傳做作為最終的形態。

注意到了！不一定要有 `T` 參數也是可以的哦！

```javascript
// @flow

// 一個固定會回傳 number 型態的 function
type ExtractNumberType = () => number;

type NumberType = $Call<ExtractNumberType>;


(26: NumberType); // ok
('Wayne': NumberType); // 錯誤! 必須要是 number 形態

```

上例雖然沒有什麼意義，就是演示沒有接收 `T` 參數，`$Call<F, T...>` 也是可以正常運作的~

再來我們看一下接收一個參數的幾個範例

```javascript
// @flow
// 解析接收到的物件中 prop 屬性的的型態
type ExtractPropType = <T>({prop: T}) => T;

type PropObj = { prop: number };
type NopeObj = { nope: number };

type PropType = $Call<ExtractPropType, PropObj>;
type NopeType = $Call<ExtractPropType, NopeObj>; // 錯誤


(26: PropType); // ok!
('Wayne': PropType); // ok!
(26: NopeType); // 錯誤

```

```javascript
// @flow
type ExtractReturnType = <R>(() => R) => R;
type Fn = () => number;
type Return = $Call<ExtractReturnType, Fn>;

(26: Return); // ok!
('Wayne': Return); // 錯誤! 必須是 number 型態

```

再來些進階的用法

```javascript
// @flow
type NestedObj = {
  status: number,
  data: $ReadOnlyArray<{
    foo: {
       bar: number,
    },
  }>,
};

type Fn = <T>({
  data: $ReadOnlyArray<{
    foo: {
      bar: T,
    },
  }>
}) => T

// $Call<F, T...> 也是可以往下解析型態的哦！
type BarType = $Call<Fn, NestedObj>;

(26: BarType); // ok!
('Wayne': BarType); // 錯誤! 必須是 number 型態

```

`$Call<F, T...>` 還可以解析 Map 等等型態，就等你們再去挖掘囉~

## Class&lt;T&gt; <a id="class"></a>

接收一個 `T` 參數回傳一個實體 Class 型態 `C`，`C` 型態是 Class 本身而不是被 new 出來的 Class 實體。

```javascript
// @flow
class Store {}

// 接收 Store 型態
function getStore(newStore: Store) {
  return newStore;
}

// 接收 Class<Store> 型態
function makeStore(storeClass: Class<Store>) {
  return new storeClass();
}

(getStore(Store): Store); // 錯誤! 必須接收 Class 實體
(getStore(new Store()): Store); // ok

(makeStore(Store): Store); // ok
(makeStore(new Store()): Store); // 錯誤! 必須接收 Class 本身
```

Class 的建構子需要接收參數的話，我們也需要提供參數的型別

```javascript
// @flow
class ParamStore<T> {
  constructor(data: T) {
  }
}

function makeStore<T>(storeClass: Class<ParamStore<T>>, data: T): ParamStore<T> {
  return new storeClass(data);
}

(makeStore(ParamStore, 26): ParamStore<number>); // ok
(makeStore(ParamStore, 'Wayne'): ParamStore<string>); // ok
(makeStore(ParamStore, 'Wayne'): ParamStore<bool>); // 錯誤!
```

## $Shape&lt;T&gt; <a id="shape"></a>

接收一個物件型態的 `T`，他會去解析 `T` 的所有屬性，並回傳一個只要符合一個屬性就通過的型別。

```javascript
// @flow
type Person = {
  age: number,
  name: string,
}
type PersonDetails = $Shape<Person>;

const person001: Person = { age: 26, name: 'Wayne' }; // ok
const person002: PersonDetails = { age: 26, name: 'Wayne' }; // ok

const person003: Person = { age: 26 }; // 錯誤!
const person004: PersonDetails = { age: 26 }; // ok
const person005: PersonDetails = { name: 'Wayne' }; // ok
```

{% hint style="info" %}
注意了！`$Shape<T>` 跟 T 使用 Optional 型別定義是不一樣的哦！
{% endhint %}

## $Exports&lt;T&gt; <a id="exports"></a>

官網給出來的解釋是這樣的

```javascript
// @flow
import typeof * as T from 'my-module';
```

```javascript
// @flow
type T = $Exports<'my-module'>;
```

恩... 當然是沒錯啦！但是我個人在看的時候為了搞懂這兩句可是花了好大的心思呀！

為什麼呢？因為這短短兩行，我們就必須要去補 flow module 的部分，看看看還會被帶到 [flow-typed](https://github.com/flow-typed/flow-typed) 、 [Library Definitions](https://flow.org/en/docs/libdefs/creation/) 等等的地方，是很容易迷路的！

這邊我給大家個範例！加速理解，有空大家再自己去補技術細節就好。

先在 _/flow-typed_ 資料夾內建立一個叫做 **'test-module.js'** 的檔案 \( _/flow-typed_ 資料夾是預設的全域 `lib`，詳細在該章節細細討論 \)

```javascript
// @flow
declare module "test-module" {
  declare export function concatPath(dirA: number, dirB: string): string;
  declare export var Name: string;
  declare export var PI: number;
}
```

接著在專案內引入

`typeof * as T` 方法引入

```javascript
// @flow
import typeof * as T from 'test-module';

// ok!
const a: T = {
  concatPath: (a, b) => b,
  PI: 3.14,
  Name: 'Wayne',
}


// 錯誤
const b: T = {
  concatPath: (a, b) => b,
  PI: 3.14,
  Name: 'Wayne',
  test: 'Hello',
}
```

`$Exports<T>` 方法引入

```javascript
// @flow
type T = $Exports<'test-module'>;

// ok!
const a: T = {
  concatPath: (a, b) => b,
  PI: 3.14,
  Name: 'Wayne',
}

// 錯誤
const b: T = {
  concatPath: (a, b) => b,
  PI: 3.14,
  Name: 'Wayne',
  test: 'Hello',
}
```

