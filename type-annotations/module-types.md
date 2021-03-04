# 模組化 \( Module Types \)

## 引入、匯出型別 <a id="import-export-type"></a>

我們常常會需要以模組為單位的定義、引用我們的型別 \( 以檔案為單位 \)。  
在 Flow 中可以匯出 type、interfaces、或是 class 等類型。

#### exports.js

```javascript
// @flow
export type Person = {
  name: string,
  age: number,
};

export class Book {
  // ...
}

export interface Swim {
  doSwim(): void;
}

```

#### import.js

若要引入型別的話，要使用 `import type` 關鍵字來引入

```javascript
// @flow
import type BookType, { Person, Swim } from './exports';
import Book from './my-modules'; // 因為要使用 new Book()，所以不能只使用 type

const book: BookType = new Book();

const person: Person = {
  name: 'Wayne',
  age: 26,
}

class Fish implements Swim {
  doSwim() {
    console.log('Fish Swim ~~~')
  }
}
```

## 引入、匯出值 <a id="import-export-value"></a>

Flow 也支援使用 `import typeof` 關鍵字，直接引入某個值的型別

#### exports.js

```javascript
// @flow
export default {
  name: 'Wayne',
  age: 26,
};

export const book = {
  name: '原子習慣',
  price: 180,
}

export const user = {
  name: 'Wayne',
  password: '123456',
}
```

#### import.js

```javascript
// @flow
import typeof Person, { price as Price, isSoldOut } from './exports';

({ name: 'Wayne', age: 26 }: Person);
(180: Price);
(false: isSoldOut);
('false': isSoldOut); // 錯誤!

```

