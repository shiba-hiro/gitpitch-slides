### 2.7 フィルタ(filters)
#### 概要
「汎用的なテキストフォーマット処理を適用する仕組み」。  
DateをYYYY/mm/dd形式へ変換したり、0.5を50%として変換したり。  
「コンストラクタオプション」なので、Vueをnewするときに指定して定義する。  
注釈によれば、Vueグローバルオブジェクト自体にもfilterを定義できるAPIがあるので、アプリ全体で使いたいフィルタはそちらで定義するとよい。

template系ライブラリで言われるような「フィルタ」。  
cf. http://jinja.pocoo.org/docs/2.10/templates/#filters  
Unixのシェルで、前コマンドの出力をパイプでつないで次に渡すようなことやるけど、そういうイメージ。  
```
$ cat samplefile.txt | grep 'Sample Text' | cut -d: -f2 | xargs -I{} echo {}-sample-suffix
```

#### sample
https://jsfiddle.net/kitak/1n4s5odx/  
```
  filters: { // この節で追加したフィルタの定義
    numberWithDelimiter: function (value) {
      if (!value) {
        return '0'
      }
      return value.toString().replace(/(\d)(?=(\d{3})+$)/g, '$1,')
    }
  }

// {{1000 | numberWithDelimiter}} -> "1,000"
```

#### 私見
ここでは実践のために"1000->1,000"を自作してるけど。  
こんなCurrencyのためのfilterなんて、Open Sourceのライブラリにありがちなので、プロダクションではそれらを使うべき。  
あるいはVueの標準にプルリク出していくべき。  
https://github.com/vuejs/awesome-vue#filters
https://github.com/mazipan/vue-currency-filter


### 2.8 算出プロパティ(computed)
#### 概要
「派生するデータをプロパティとして参照する仕組み」。  
プロパティの形で、functionの実行結果を参照できる。  
template内に{{}}の形で書くと長ったらしくなったり、複数箇所のメンテが大変になったりするときに有用。  

#### sample
https://jsfiddle.net/kitak/vjf3tskz/  
```
  computed: { // 算出プロパティ
    totalPrice: function () {
      return this.items.reduce(function (sum, item) {
        return sum + (item.price * item.quantity)
      }, 0)
    }
  }
  
// items = [{price:300, quantity: 2}]のとき
// {{ totalPrice }} -> 600
```

#### 私見
たぶんテストにおいても、functionを評価する方がtemplate内でごにょごにょ書いた結果を評価よりもよほど簡潔になると思われるので、積極的に使っていくと良さそう。

応用的に、非同期処理をサポートするすばらしいライブラリもある。  
テンプレートが超絶見やすくなるはず。デザイナーとの共同作業とかもしやすそう。  
https://github.com/foxbenjaminfox/vue-async-computed  
```
new Vue({
  data: {
    postId: 1
  },
  asyncComputed: {
    blogPostContent: {
      get () {
        return Vue.http.get('/post/' + this.postId)
          .then(response => response.data.postContent)
      },
      default () {
        return 'Loading post ' + this.postId
      }
    }
  }
}
```