---
title: API
layout: ja/detail
description: フレームワーク API, メソッドとプロパティ
---


## コア API

### riot.mount

`riot.mount(selector: string, props?: object, componentName?: string): [RiotComponent]`

1. `selector` はページから要素を選択し、それらをカスタムコンポーネントとともにマウントします。選択したエレメントの名前は、カスタムタグの名前と一致する必要があります。is属性を持つDOMノードも自動マウントできます。

2. 消費するために `props` というオプショナルなオブジェクトがコンポーネントに渡されます。これには、単純なオブジェクトから完全なアプリケーション API まで、あらゆるものを使用できます。もしくは Flux ストアの場合もあります。実際には、クライアント側のアプリケーションをどのように構成するかによって異なります。*注意* タグに設定した属性は、`props` 引数で同じ名前を指定した属性よりも優先されます。

3. `componentName` は、マウントするノードが riot によって自動マウントされない場合のオプショナルなコンポーネントの名前です。

<strong>@returns: </strong>マウントされた [コンポーネントオブジェクト](#コンポーネントオブジェクト) の配列


例:

``` js
// 選択し、そしてページ上の全ての <pricing> タグをマウント
const components = riot.mount('pricing')

// クラス名が .customer である全てのタグをマウント
const components = riot.mount('.customer')

// <account> タグをマウントし、オプションとして API オブジェクトを渡す
const components = riot.mount('account', api)

// 既に登録された `app` コンポーネントを使用して <div id="root"> タグに API オブジェクトを渡す
const components = riot.mount('#root', api, 'app')
```

メモ: [インブラウザコンパイル](/compiler/#インブラウザコンパイル) を利用しているユーザーは、`riot.mount` メソッドをコールする前にコンポーネントのコンパイルを待つ必要があるでしょう。

```javascript
(async function main() {
  await riot.compile()

  const components = riot.mount('user')
}())
```

`props` 引数には、複数のタグインスタンス間で同じオブジェクトを共有することを避けるために関数を指定することも可能です。[riot/2613](https://github.com/riot/riot/issues/2613)

``` js
riot.mount('my-component', () => ({
  custom: 'option'
}))
```

`riot.mount` はターゲットコンポーネント下に存在する子ノードを消去しません。SSR 後に、ユーザーと対話的にコンポーネントをクライアント側でマウントする場合は、別の解決方法があります。[@riotjs/hydrate](https://github.com/riot/hydrate#usage) を確認してください。


### riot.unmount

`riot.unmount(selector: string): [HTMLElement]`

1. `selector` はページから要素を選択肢し、それらが既にマウントされていた場合はアンマウントします。
2. DOMからルートノードを削除しないようにするために使用できる `keepRootElement` というブーリアンのオプションパラメータ {% include version_badge.html version=">=4.3.0" %}

```js
// 全ての <user> タグを選択し、それらをアンマウントする
riot.unmount('user')
```

`keepRootElement` パラメータが true だった場合、ルートノードは DOM に残されます

```js
// 全ての <user> タグを選択すると、そのタグをアンマウントするが ルートノードは DOM に残される
riot.unmount('user', true)
```

<strong>@returns: </strong>マウントされた [コンポーネントオブジェクト](#コンポーネントオブジェクト) の配列

### riot.component

`riot.component(component: RiotComponentShell): function`

1. `component` - [コンポーネントシェルオブジェクト](#コンポーネントシェルインターフェース)

<strong>@returns: </strong>[コンポーネントオブジェクト](#component-object) 生成のための関数

`riot.component` メソッドを使用すると、コンポーネントをグローバルに登録せずにコンポーネントを生成、およびマウントできます:

```js
import * as riot from 'riot'
import App from './app.riot'

const createApp = riot.component(App)

const app = createApp(document.getElementById('root'), {
  name: 'This is a custom property'
})
```

{% include version_badge.html version=">=4.7.0" %}

`riot.component` によって生成されたファクトリ関数は、[有効な `slots` と `attributes` オブジェクト](https://github.com/riot/dom-bindings) を含むことができる3つ目の任意の引数も受け入れます。

```js
import { expressionTypes } from '@riotjs/dom-bindings'
import * as riot from 'riot'
import App from './app.riot'

const createApp = riot.component(App)

const app = createApp(document.getElementById('root'), {
  name: 'This is a custom property',
  class: 'custom-class'
}, {
  slots: [{
    id: 'default',
    html: 'Hello there',
    bindings: []
  }],
  attributes: [{
    type: expressionTypes.ATTRIBUTE,
    name: 'class',
    evaluate: scope => scope.props.class
  }]
})
```

### riot.install

`riot.install(plugin: function): Set`

1. `plugin` - 生成された任意のコンポーネントの [コンポーネントオブジェクト](#コンポーネントオブジェクト) を受け取る関数

<strong>@returns: </strong> インストールされた全てのプラグイン関数を含む javascript `Set`

一度インストールされたプラグイン関数 は生成された任意の Riot.js コンポーネントに対しコールされます

```js
import { install } from 'riot'

let id = 0

// このプラグインは生成された任意の riot コンポーネントに uid 属性を追加します
install(function(component) {
  component.uid = id++

  return component
})
```

### riot.uninstall

`riot.uninstall(plugin: function): Set`

1. `plugin` - 既にインストールされた関数プラグイン

<strong>@returns: </strong> インストールされた残りのプラグイン関数を含む javascript `Set`

プラグインはインストールおよびアンインストールできます:

```js
import { install, uninstall } from 'riot'
import uid from './riot-uid.js'

install(uid)

// プラグインがもう不要となった場合アンインストールする
uninstall(uid)
```

### riot.register

`riot.register(name: string, component: RiotComponentShell): Map`

1. `name` - コンポーネント名
2. `component` - [コンポーネントシェルオブジェクト](#component-shell-interface)

<strong>@returns: </strong> 全ての登録済みコンポーネントのファクトリ関数を含む javascript `Map`

```js
import { register, mount } from 'riot'
import MyComponent from './my-component.riot'

// my-component をグローバルコンポーネントとして登録
register('my-component', MyComponent)

// `<my-component>` を呼ばれるすべての DOM ノードを検索し
// 既に登録したコンポーネントと一緒にマウントする
mount('my-component')
```

### riot.unregister

`riot.unregister(name: string): Map`

1. `name` - the component name

<strong>@returns: </strong> アンマウント されていない残りのコンポーネントから生成された関数を含む javascript の `Map`

既にコンパイラか `riot.register()` を介して生成されたタグの登録を解除します。
このメソッドは、例えばアプリケーションをテストする必要があり、かつ同じ名前を使用して複数のタグを生成したい時などに有効かもしれません。

```js
import { register, unregister } from 'riot'
import TestComponent from './test-component.riot'
import TestComponent2 from './test-component2.riot'

// test コンポーネントを生成
register('test-component', TestComponent)

// マウント
const [component] = mount(document.createElement('div'), 'test-component')
expect(component.root.querySelector('p')).to.be.ok

// tag を登録解除
unregister('test-component')

// 異なるテンプレートを利用して同じコンポーネントを再生成
register('test-component', TestComponent2)
```

### riot.pure

`riot.pure(PureComponentFactoryFunction): PureComponentFactoryFunction`

1. `slots` - コンポーネント内で見つかった slot リスト
2. `attributes` - コンテキストからコンポーネントプロパティを推測するために評価可能なコンポーネントの属性式
3. `props` - `riot.component` 呼び出しによってのみ設定できる初期コンポーネントユーザープロパティ

<strong>@returns: </strong> 純粋なコンポーネントを生成するために Riot.js によって内部的に使用されるユーザーが定義したファクトリ関数

この関数は Riot.js の根本であり、デフォルトのレンダリングエンジンでは探している全ての機能(遅延読み込みやカスタムディレクティブなど…)が提供されないかもしれないような、特定の場合にのみ使用されることを意味します。

`PureComponentFactoryFunction` は、Riot.js が純粋なコンポーネントを適切にレンダーできるようにするために、常に `mount`, `update`, `unmount` メソッドを含んでいるオブジェクトを返すべきです。例:

```html
<lit-element>
  <script>
    import { pure } from 'riot'
    import { html, render } from 'lit-html'

    export default pure(({ attributes, slots, props }) => ({
      mount(el, context) {
        this.el = el
        this.render(context)
      },
      // ここでのコンテキストは親コンポーネントかundefinedのいずれかです
      render(context) {
        render(html`<p>${ context ? context.message : 'no message defined' }</p>`, this.el)
      },
      unmount() {
        this.el.parentNode.removeChild(this.el)
      }
    }))
  </script>
</lit-element>
```

### riot.version

`riot.version(): string`

<strong>@returns: </strong> 現在使用しているの riot のバージョンを文字列として返す

## コンポーネントオブジェクト

各 Riot.js コンポーネントは軽量なオブジェクトとして生成されます。`export default` を使用してエクスポートするオブジェクトには以下のプロパティがあります:

- Attributes
  - `props` - オブジェクトとして受けとる props
  - `state` - 現在のステートオブジェクト
  - `root` - ルート DOM ノード
- [生成と破壊](#生成と破壊)
  - `mount` - コンポーネントの初期化
  - `unmount` - DOM から コンポーネントを壊し、取り除く
- [ステートハンドリング](#ステートハンドリング) メソッド
  - `update` - コンポーネントのステートを更新するメソッド
  - `shouldUpdate` - コンポーネントレンダリングを一時停止するメソッド
- [ライフサイクルコールバック](#ライフサイクル)
  - `onBeforeMount` - コンポーネントがマウントされる前にコールされる
  - `onMounted` - コンポーネントのレンダリングが完了した後にコールされる
  - `onBeforeUpdate` - コンポーネントの更新前にコールされる
  - `onUpdated` - コンポーネントの更新が完了した後にコールされる
  - `onBeforeUnmount` - コンポーネントが削除される前にコールされる
  - `onUnmounted` - コンポーネントの削除が完了した時にコールされる
- [ヘルパー](#ヘルパー)
  - `$` - `document.querySelector` に似たメソッド
  - `$$` - `document.querySelectorAll` に似たメソッド


### コンポーネントインターフェース

TypeScript に詳しい人は、ここに書かれているように Riot.js コンポーネントインタフェースがどのように見えるを読むことができます:

```ts
// このインターフェースはただ公開されているのみで、どんな Riot コンポーネントも以下のプロパティを受け取る
interface RiotCoreComponent<P = object, S = object> {
  // 任意のコンポーネントインスタンスで自動的に生成
  readonly props: P
  readonly root: HTMLElement
  readonly name?: string
  // TODO: add the @riotjs/dom-bindings types
  readonly slots: any[]
  mount(
    element: HTMLElement,
    initialState?: S,
    parentScope?: object
  ): RiotComponent<P, S>
  update(
    newState?: Partial<S>,
    parentScope?: object
  ): RiotComponent<P, S>
  unmount(keepRootElement: boolean): RiotComponent<P, S>

  // ヘルパー
  $(selector: string): HTMLElement
  $$(selector: string): [HTMLElement]
}

// パブリックな RiotComponent インターフェースのプロパティはすべてオプショナル
interface RiotComponent extends RiotCoreComponent<P = object, S = object> {
  // コンポーネントオブジェクトでオプショナル
  state?: S

  // 子コンポーネントの名前をマップするオプションの別名
  components?: {
    [key: string]: RiotComponentShell<P, S>
  }

  // ステートハンドリングメソッド
  shouldUpdate?(newProps: P, currentProps: P): boolean

  // ライフサイクルメソッド
  onBeforeMount?(currentProps: P, currentState: S): void
  onMounted?(currentProps: P, currentState: S): void
  onBeforeUpdate?(currentProps: P, currentState: S): void
  onUpdated?(currentProps: P, currentState: S): void
  onBeforeUnmount?(currentProps: P, currentState: S): void
  onUnmounted?(currentProps: P, currentState: S): void
  [key: string]: any
}
```

HTML と JavaScript コードの両方で任意のコンポーネントプロパティを使用できます。例:


``` html
<my-component>
  <h3>{ props.title }</h3>

  <script>
    export default {
      onBeforeMount() {
        const {title} = this.props
      }
    }
  </script>
</my-component>
```

コンポーネントスコープには任意のプロパティを自由に設定でき、HTML に式で使用できます。例:

``` html
<my-component>
  <h3>{ title }</h3>

  <script>
    export default {
      onBeforeMount() {
        this.title = this.props.title
      }
    }
  </script>
</my-component>
```

メモ: いくつかのグローバル変数がある場合、HTML と JavaScript コードの両方でこれらの参照を使用することもできます:

```js
window.someGlobalVariable = 'Hello!'
```

```html
<my-component>
  <h3>{ window.someGlobalVariable }</h3>

  <script>
    export default {
      message: window.someGlobalVariable
    }
  </script>
</my-component>
```

<aside class="note note--warning">:warning: コンポーネントでグローバル変数を使用すると、おそらくサーバサイドレンダリングが損なわれる可能性があるため注意してください。そして、推奨しません。</aside>


### 生成と破壊

#### component.mount

`component.mount(element: HTMLElement, initialState?: object, parentScope?: object): RiotComponent;`

テンプレートをインタラクティブにレンダリングするために、任意のコンポーネントオブジェクトが DOM ノードにマウントされます。

`component.mount` メソッドを自分で呼び出すことはまずありませんが、代わりに [riot.mount](#riotmount) または [riot.component](#riotcomponent) を使用します。

#### component.unmount

`component.mount(keepRoot?: boolean): RiotComponent`

カスタムコンポーネントとその子をページから切り離します。
ルートノードを削除せずにタグをアンマウントしたい場合は、unmountメソッドに `true` を渡す必要があります。

タグをアンマウントし、DOM からテンプレートを削除します:

``` js
myComponent.unmount()
```

ルートノードを DOM に保持したままコンポーネントをアンマウントします:

``` js
myComponent.unmount(true)
```


### ステートハンドリング

#### component.state

生成された Riot.js コンポーネントには `state` というオブジェクトプロパティがあります。`state` オブジェクトは、すべての可変なコンポーネントプロパティを格納することを目的としています。例:

```html
<my-component>
  <button>{ state.message }</button>

  <script>
    export default {
      // コンポーネントステートの初期化
      state: {
        message: 'hi'
      }
    }
  </script>
</my-component>
```

この場合、初期状態のステートと一緒にコンポーネントが生成され、`component.update` を使用して内部的に修正できます。

ネストされた javascript のオブジェクトをステートプロパティに格納することは避けてください。なぜなら、ネストされた javascript の参照は複数のコンポーネントで共有され、副作用が発生する可能性があるからです。予期しない事態を避けるために、ファクトリ関数を使用してコンポーネントを生成することもできます

```html
<my-component>
  <button>{ state.message }</button>

  <script>
    export default function MyComponent() {
      // 戻り値は「オブジェクト」であることを忘れずに
      return {
        // 初期状態の `state` は常に新規に作成し、予期せぬ事態を防ぐ
        state: {
          nested: {
            properties: 'are ok now'
          },
          message: 'hi'
        }
      }
    }
  </script>
</my-component>
```

新しいコンポーネントがマウントされると、Riot.js は自動的に関数を呼び出します。

#### component.components

グローバルに Riot.js のコンポーネントを登録したくない場合、子コンポーネントをコンポーネントオブジェクトに直接マッピングすることができます。例:

```html
<my-component>
  <!-- このコンポーネントは `<my-component>` 内でのみ利用可能 -->
  <my-child/>

  <!-- このコンポーネントには別の名前が付けられ、エイリアスが設定される -->
  <aliased-name/>

  <!-- このコンポーネントは riot.register を介して既に登録されている -->
  <global-component/>

  <script>
    import MyChild from './my-child.riot'
    import User from './user.riot'

    export default {
      components: {
        MyChild,
        'aliased-name': User
      }
    }
  </script>
</my-component>
```

ファクトリ関数でコンポーネントを生成する場合、 `components` プロパティは、そのコンポーネントの静的プロパティにする必要があります（訳注: エクスポートする関数オブジェクトに直接 `components` プロパティを定義する、という意味）。もし関数としてエクスポートする場合、以下のように宣言されなければなりません:

```html
<my-component>
  <button>{ state.message }</button>

  <script>
    import MyChild from './my-child.riot'
    import User from './user.riot'

    // `components` はこのタグのファクトリ関数、すなわち `MyComponent` の静的プロパティにする
    MyComponent.components = {
      MyChild,
      'aliased-name': User
    }

    export default function MyComponent() {
      // 戻り値は「オブジェクト」であることを忘れずに
      return {
        // 初期状態の `state` は常に新規に作成し、予期せぬ事態を防ぐ
        state: {
          nested: {
            properties: 'are ok now'
          },
          message: 'hi'
        }
      }
    }
  </script>
</my-component>
```

<div class="note note--info">
  この例では、webpack, rollup, parcel, browserify を介してアプリケーションをバンドルすることを想定しています
</div>

#### component.update

`component.update(newState?:object, parentScope?: object): RiotComponent;`

コンポーネントの `state` オブジェクトを更新し、すべてのテンプレート変数を再レンダリングします。このメソッドは通常、ユーザーがアプリケーションと対話するときにイベントハンドラがディスパッチされるたびに呼び出すことができます:

``` html
<my-component>
  <button onclick={ onClick }>{ state.message }</button>

  <script>
    export default {
      state: {
        message: 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      }
    }
  </script>
</my-component>
```

このメソッドは、コンポーネントの UI を更新する必要があるときに、手動で呼び出すこともできます。これは通常 UI に関連しないイベント（`setTimeout` の後、AJAX のコール、または何らかのサーバーのイベント）の後に発生します。例:

``` html
<my-component>

  <input name="username" onblur={ validate }>
  <span class="tooltip" if={ state.error }>{ state.error }</span>

  <script>
    export default {
      async validate() {
        try {
          const {username} = this.props
          const response = await fetch(`/validate/username/${username}`)
          const json = response.json()
          // レスポンスで何かをする
        } catch (error) {
          this.update({
            error: error.message
          })
        }
      }
    }
  </script>
</my-component>
```

上記の例では、`update()` メソッドがコールされた後に UI 上にエラーメッセージが表示されます。

タグの DOM 更新をより細かくコントロールしたい場合、`shouldUpdate` 関数の戻り値を利用して設定できます。
Riot.js はその関数の戻り値が `true` の場合にのみ、コンポーネントを更新します。

``` html
<my-component>
  <button onclick={ onClick }>{ state.message }</button>

  <script>
    export default {
      state: {
        message: 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      },
      shouldUpdate(newProps, currentProps) {
        // 更新しない
        if (this.state.message === 'goodbye') return false
        // this.state.message が 'goodbye' と異なる場合、コンポーネントを更新可能
        return true
      }
    }


  </script>
</my-component>
```

`shouldUpdate` メソッドは常に2つの引数を受け取ります。最初の引数には新しいコンポーネントプロパティが含まれ、2番目の引数には現在のプロパティが含まれます。

``` html
<my-component>
  <child-tag message={ state.message }></child-tag>
  <button onclick={ onClick }>Say goodbye</button>

  <script>
    export default {
      state: {
        message = 'hi'
      },
      onClick(e) {
        this.update({
          message: 'goodbye'
        })
      }
    }
  </script>
</my-component>

<child-tag>
  <p>{ props.message }</p>

  <script>
    export default {
      shouldUpdate(newProps, currentProps) {
        // 受け取った新しいプロパティに応じて DOM を更新
        return newProps.message !== 'goodbye'
      }
    }
  </script>
</child-tag>
```



###  スロット

`<slot>` タグは、実行時にテンプレート内の任意のカスタムコンポーネントの内容を注入してコンパイルすることができる、Riot.js の特別なコア機能です。
例えば以下の riot タグ `my-post` 使ってみましょう

``` html
<my-post>
  <h1>{ props.title }</h1>
  <p><slot/></p>
</my-post>
```

いつもアプリケーションに `<my-post>` タグを入れることになるでしょう

``` html
<my-post title="What a great title">
  My beautiful post is <b>just awesome</b>
</my-post>
```

一度マウントされると、このようにレンダリングされます:

``` html
<my-post>
  <h1>What a great title</h1>
  <p>My beautiful post is <b>just awesome</b></p>
</my-post>
```

デフォルトのスロットタグ内の式は、スロット属性経由で渡さない限り、挿入されるコンポーネントのプロパティにアクセスできません。詳しくは [上位コンポーネント]({{ '/ja/api/#上位コンポーネント' | prepend:site.baseurl }}) の項を参照してください。

``` html
<!-- このタグは生成された DOM を継承するだけ -->
<child-tag>
  <slot/>

  <script>
    export default {
      internalProp: 'secret message'
    }
  </script>
</child-tag>

<my-component>
  <child-tag>
    <!-- ここでは子タグの内部プロパティは使用不可 -->
    <p>{ message }</p>

    <!-- "internalProp" はここでは許可されていないため、この式は失敗する -->
    <p>{ internalProp }</p>
  </child-tag>
  <script>
    export default {
      message: 'hi'
    }
  </script>
</my-component>
```

#### 名前付きスロット

`<slot>` タグは、コンポーネントテンプレートの特定のセクションにhtmlを挿入するメカニズムも提供します。

例えば以下の riot タグ `my-other-post` 使ってみましょう

``` html
<my-other-post>
  <article>
    <h1>{ props.title }</h1>
    <h2><slot name="summary"/></h2>
    <article>
      <slot name="content"/>
    </article>
  </article>
</my-other-post>
```

いつもアプリケーションに `<my-other-post>` タグを入れることになるでしょう

``` html
<my-other-post title="What a great title">
  <span slot="summary">
    My beautiful post is just awesome
  </span>
  <p slot="content">
    And the next paragraph describes just how awesome it is
  </p>
</my-other-post>
```

一度マウントされると、このようにレンダリングされます:

``` html
<my-other-post>
  <article>
    <h1>What a great title</h1>
    <h2><span>My beautiful post is just awesome</span></h2>
    <article>
      <p>
        And the next paragraph describes just how awesome it is
      </p>
    </article>
  </article>
</my-other-post>
```

#### 上位コンポーネント

{% include version_badge.html version=">=4.6.0" %}

`<slot>` タグを使用して上位コンポーネントを作成できます。スロットタグにセットされたすべての属性は、そのタグに挿入された html テンプレートで使用できます。
たとえば、テーマ設定可能なアプリケーションを作成し、そのアプリケーションに `<theme-provider>` コンポーネントを使用することを想像してください:

```html
<theme-provider>
  <slot theme={theme}/>

  <script>
    export default {
      theme: 'dark'
    }
  </script>
</theme-provider>
```

今、コンポーネントを `<theme-provider>` タグにラップし、常に現在のアプリケーションテーマを柔軟かつ合成可能な方法で読み取ることが可能になりました:

```html
<app>
  <theme-provider>
    <!-- ここで "theme" 変数が利用可能になることに注意 -->
    <sidebar class="sidebar sidebar__{theme}"/>
  </theme-provider>
</app>
```

上位コンポーネントを使用すると、子コンポーネントと親コンポーネントの大量のやり取りが簡略化され、アプリケーション全体のデータフローを扱うための十分な柔軟性が得られます。

### ライフサイクル

各コンポーネントオブジェクトは、次のコールバックに依存して内部状態を処理できます:

  - `onBeforeMount` - コンポーネントがマウントされる前にコールされる
  - `onMounted` - コンポーネントがレンダリングされた後にコールされる
  - `onBeforeUpdate` - コンポーネント更新される前にコールされる
  - `onUpdated` - コンポーネントが更新された後にコールされる
  - `onBeforeUnmount` - コンポーネントが削除される前にコールされる
  - `onUnmounted` - コンポーネントの削除が完了した時にコールされる

例:

```html
<my-component>
  <p>{ state.message }</p>

  <script>
    export default {
      onBeforeMount() {
        this.state = {
          message: 'Hello there'
        }
      }
    }
  </script>
</my-component>
```

すべてのライフサイクルメソッドは、`props` と `state` の2つの引数を受け取ります。これらの引数は `this.props` と `this.state` というコンポーネント属性のエイリアスです。

```html
<my-component>
  <p>{ state.message }</p>

  <script>
    export default {
      state: {
        message: 'Hello there'
      },
      onBeforeMount(props, state) {
        console.assert(this.state === state) // ok!
        console.log(state.message) // Hello there
      }
    }
  </script>
</my-component>
```

### ヘルパー

どんな Riotjs コンポーネントにも、レンダリングされたテンプレートに含まれる DOM ノードを照会するための2つのヘルパーが用意されています。

 - `component.$(selector: string): HTMLElement` - コンポーネントのマークアップにある1つのノードを返す
 - `component.$$(selector: string): [HTMLElemet]` - コンポーネントのマークアップを含むセレクタに一致するすべての DOM ノードを戻す

コンポーネントヘルパーを使用して簡単な DOM クエリを実行できます:

```html
<my-component>
  <ul>
    <li each={ item in items }>
      { item }
    </li>
  </ul>

  <script>
    export default {
      onMounted() {
        // クエリ
        const ul = this.$('ul')
        const lis = this.$$('li')

        // DOM ノードでなにかする
        const lisWidths = lis.map(li => li.offsetWidth)
        const {top, left} = ul.getBoundingClientRect()
      }
    }
  </script>
</my-component>
```

<aside class="note note--warning">:warning:
ヘルパー <code>$</code> と <code>$$</code> は 、 HTML テンプレートに含まれた子タグが Riot.js が生成するコンポーネントであっても、 DOM クエリの検索対象として動作することに気をつけてください。
</aside>

### 手動でのタグ構築

Riot.jsコンポーネントは [@riotjs/compiler](/ja/compiler) を使って javascript にコンパイルされるようになっています。ただし、あなたの好きな任意のレンダリングエンジンを使用して手動でビルドすることもできます。

#### コンポーネントシェルインターフェース

Riot.js コンパイラは、単に [コンポーネントオブジェクト](#コンポーネントインターフェース) を作成するために、riot によって内部的に変換されるシェルオブジェクトを作成する。このシェルオブジェクトを手動で作成する場合は、まずそのインターフェースを理解する必要があります:

```ts
interface RiotComponentShell<P = object, S = object> {
  readonly css?: string
  readonly exports?: () => RiotComponentExport<P, S>|object
  readonly name?: string
  template(): any
}
```

`RiotComponentShell` オブジェクトは4つのプロパティで構成されています:

- `css` - コンポーネントの css  の文字列
- `exports` - コンポーネントのパブリック API `export default`
- `name` - コンポーネント名
- `template` - コンポーネントテンプレートを管理するファクトリ機能

#### テンプレートインターフェース

テンプレート関数は、次のものと互換性のあるインタフェースを返す必要があります:

```ts
interface RiotComponentTemplate {
  update(scope: object): RiotComponentTemplate;
  mount(element: HTMLElement, scope: object): RiotComponentTemplate;
  createDOM(element: HTMLElement): RiotComponentTemplate;
  unmount(scope: object): RiotComponentTemplate;
  clone(): RiotComponentTemplate;
}
```

`RiotComponentTemplate` はオブジェクトであり、レスポンシブルにコンポーネントのレンダリングを処理します:

- `update` - メソッド: コンポーネントデータを受け取り、テンプレートの更新に使用する必要がある
- `mount` - メソッド: コンポーネントテンプレートを DOM ノードに接続するために使用する必要がある
- `createDOM` - ファクトリ関数: テンプレート DOM の構造を一度だけ作成する必要がある場合がある
- `unmount` -  メソッド: コンポーネントDOMをクリーンアップする
- `clone` - メソッド: 複数のインスタンスを操作するためにオリジナルのテンプレートオブジェクトのクローンを作成する必要がある場合がある

#### 例

この例では [@riotjs/dom-bindings (riot core template engine)](https://github.com/riot/dom-bindings) を使用しています。

```js
import { template, expressionTypes } from '@riotjs/dom-bindings'

riot.register('my-component', {
  css: ':host { color: red; }',
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  },
  template() {
    return template('<p><!----></p>', [{
      selector: 'p',
      expressions: [
        {
          type: expressionTypes.TEXT,
          childNodeIndex: 0,
          evaluate: scope => `Hello ${scope.greeting}`
        }
      ]
    }])
  }
})
```

[テンプレートエンジン API](https://github.com/riot/dom-bindings) についてご参照ください。

必要に応じて、あなたの好きな他の種類のテンプレートエンジンを使用することもできます。
この例ではテンプレートエンジンとして [lit-html](https://lit-html.polymer-project.org/) を使用しています。

```js
import {html, render} from 'lit-html'

riot.register('my-component', {
  css: ':host { color: red; }',
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  },
  template() {
    const template = ({name}) => html`<p>Hello ${name}!</p>`

    return {
      mount(element, scope) {
        this.el = element

        this.update(scope)
      },
      update(scope) {
        render(template(scope), this.el)
      },
      unmount() {
        this.el.parentNode.removeChild(this.el)
      },
      // the following methods are not necessary for lit-html
      createDOM(element) {},
      clone() {}
    }
  }
})
```

#### テンプレートなしのタグ

次のように、テンプレートなしで "ラッパータグ" を作成することもできます:

``` js
riot.register('my-component', {
  name: 'my-component',
  exports: {
    onMounted() {
      console.log('I am mounted')
    }
  }
})

```

この場合、`my-component` という名前のタグをマウントすると、riot はコンポーネントのマークアップを解析せずにそのままにするでしょう:

```html
<html>
<body>
  <my-component>
    <p>I want to say Hello</p>
  </my-component>
</body>
</html>
```

このテクニックはおそらくサーバーサイドでレンダリングされたテンプレートを強化するために使用されます。
