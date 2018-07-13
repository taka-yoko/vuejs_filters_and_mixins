# vue-cli

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

# What's this?

Udemyの`Vue JS - The Complete Guide`のSection12`Using and Creating Directives`をまとめたもの。
https://www.udemy.com/vuejs-2-the-complete-guide

# Module Introduction

filterはdataを加工するが、表示されるdata（ユーザーが見るdata）が加工されるだけで、
dataそのものを加工するわけではない。

# Creating a Local Filter

表示の際にテキストを大文字に変えるfilterを作成する。

`filters`に`toUppercase`を追加。

```javascript
export default {
    data() {
        return {
            text: 'Hello, there!'
        }
    },
    filters: {
        toUppercase(value) {
            return value.toUpperCase();
        }
    }
}
```

`filter`を使用。String Interpolation内でパイプする。
`text`の値が`toUppercase`の引数`value`として渡される。

```html
<p>{{ text | toUppercase }}</p>
```

# Global Filters and How to Chain Multiple Filters

どのコンポーネントでも使えるグローバルなfilterを作成。
main.js内で以下を記述

```javascript
Vue.filter('to-lowercase', function(value) {
  return value.toLowerCase();
});
```

そうすると、App.vueなどのすべてのコンポーネントで、filter`to-lowercase`が使えるようになる。
また、filterはパイプを繋げてチェーンすることができる。

```html
<p>{{ text | toUppercase | to-lowercase }}</p>
```

Hello, there!
↓
HELLO, THERE!
↓
hello, there!

# An (often-times better) Alternative to Filters: Computed Properties

filterを使うより、computedを使ったほうがよい例

dataにfruits arrayを追加。
filterTextに入力した文字列を元にfruitsをフィルタリングしたい。

```javascript
fruits: ['Apple', 'Banana', 'Mango', 'Melon'],
filterText: ''
```
```html
<input v-model="filterText">
<ul>
    <li v-for="fruit in fruits">{{ fruit }}</li>
</ul>
```

以下のようにfilterを使うと、パフォーマンス的によろしくない。
v-modelの変更あるなしに関わらずDOMを再レンダリングしていまう。

```html
<li v-for="fruit in fruits | someFilter">{{ fruit }}</li>
```

なのでこういう場合は、computedを使用する。
こうすると、fruits arrayや、filterTextに変化があったときのみ再計算される。

```html
 <li v-for="fruit in filteredFruits">{{ fruit }}</li>
```

```javascript
computed: {
    filteredFruits() {
        return this.fruits.filter((element) => {
            return element.match(this.filterText);
        })
    }
}
```

※↑のmatchの引数が空白（inputに何も入力なし）の場合は、nullにはならず、以下のような値が返る。なのでinput未入力の場合はfruits arrayすべてが表示される。

```javascript
["", index: 0, input: "Mango", groups: undefined]
```

# Understanding Mixins

filterは、global(main.js)に宣言すれば、component全体で使用することができる。
global filterのように、各componentで共通するロジックを重複させず、まとめる機能がmixin。

この章では、重複するロジックを持つcomponentを用意して、以下の章でmixinを使ったリファクタをしていくための準備をしている。

# Creating and Using Mixins

各componentで共通のdataやロジックを、別ファイルに書き出す
ここではdataとcomputedが重複しているので、そこを抽出して別ファイルに
まとめる。

```javascript
export const fruitMixin = {
  data() {
      return {
          fruits: ['Apple', 'Banana', 'Mango', 'Melon'],
          filterText: ''
      }
  },
  computed: {
      filteredFruits() {
          return this.fruits.filter((element) => {
              return element.match(this.filterText);
          })
      }
  }
};
```

component側で、上記のファイルをimportして、mixinとしてincludeする
こうすると、vue.jsの方で、既存のdataプロパティやcomputedプロパティ
の中にマージしてくれる。

```javascript
mixins: [fruitMixin],
data() {
    return {
        text: 'Hello, there!'
    }
},
```

# How Mixins get Merged

mixinのマージのされかた。
前章にもあるように、mixinをincludeしても、既存のdataを壊すことはない。

```javascript
mixins: [fruitMixin],
data() {
    return {
        text: 'Hello, there!'
    }
},
```

mixinと、既存のインスタンス内に同じライフサイクルフックがあっても、
上書きされることはなく、どちらも実行される。
実行の順番は、
1. mixin
2. インスタンス内の記述

```javascript
  created() {
    console.log("Created");
  }
```

# Creating a Global Mixin (Special Case!)

global mixinの設定

```javascript
// main.js
Vue.mixin({
  created() {
    console.log("global mixin - created hook");
  }
});
```

global mixinは、すべてのVueインスタンスで呼ばれる。
ここでは、以下の所で呼ばれるため、計3回
`global mixin - created hook`が表示される。

- main.jsの`new Vue`
- App.vue
- List.vue

# Mixins and Scope

mixin内に書いたdataはcomponent内で共有されるのか？

App.vue内で、以下のボタンを作成。
ボタンクリックすると、App.vueのインスタンスで宣言している
dataのfruit arrayにのみ`Mango`が追加される。
すなわち、mixinのデータは各componentで共有されず、それぞれ
複製された別々のものとなっている。

```html
<button @click="fruits.push('Mango')">Add New Item</button>
```

