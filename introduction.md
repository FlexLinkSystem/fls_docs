# FLSのご紹介
FLSとはWindowsとUbuntu22.04を対象としたプロセス間通信ライブラリです。

環境構築とコーディングの容易さを両立することを目的として開発しました。
使用言語はRustで「Rustを先日初めて知ったよ！」みたいなレベルでも開発可能であると自負しております。

この資料を読んで***FlexLinkSystem***、通称***FLS***への理解を深め開発に参加してくれることを願っています。

## このライブラリの意味
ずばり、<u>このライブラリがどのような機能を持つのか</u>ということについての解析です。

プログラミングをするときには色々な開発方法があると思いますが、このライブラリでは共同開発に適した開発方法であるパッケージ開発をサポートしております。

### パッケージ開発ってなんぞや
```mermaid
flowchart TD

パッケージの作成　--> a2[パッケージで使う外部のライブラリを指定する]
a2 --> a3[srcフォルダ内のコードを編集してソフトウェアを作成する]
a3 --> a4[パッケージをビルド（コンパイル）する]
a4 --> a5[焦土作戦実行！！]
```

上記にパッケージ開発の核となるような部分について示しました。

ここでいう<u>パッケージ</u>とは<u>パッケージ</u>で使いたいライブラリを指定しているファイルとソフトウェアを構造するソースコードを含んだフォルダのことを示します。

また<u>ライブラリ</u>とは外部のソースコードを示します。ソフトウェア開発の際には０から１００まですべて自分で書いているというわけではありません。開発を高速に行うためにも<u>誰かが書いたことのあるコードは引用したい</u>ものです。そんなときにソフトウェアにおける<u>ライブラリ</u>機能が役立ちます。<u>**人が書いたコードを引用するときにライブラリを用いるということです。**</u>

最後に<u>**ビルド**</u>についてです。通常、授業などでプログラムを機械語に翻訳することを<u>**コンパイル**</u>と呼ぶと思います。ですがパッケージで開発しているときには先述したライブラリをインストールしたりする操作も含みます。よってそれらの操作をまとめて<u>**ビルド**</u>と呼んでいるのだなと私は理解しています。

### プロセスとスレッド
**プロセス**と**スレッド**についても軽く解説しておきます。

プロセスとはソフトウェアの単位のようなもので以下のような感じで１プロセスが実行されます。

#### １プロセス１スレッド
```mermaid
flowchart TD

subgraph　プロセス1

コントローラーの情報受け取る　--> a2[使いたい値のみ抜き出す]
a2 --> a3[足回りや機構に応じて計算する]
a3 --> a4[無線or有線でロボットに送信！]
a4 --> a5[焦土作戦実行！！]

end
```

これは一つのプロセスでロボットの操縦に必要なコントローラー情報の受け取りや外部との通信を１プロセスで行っているケースです。これは<u>**分かりやすくコーディングできる**</u>反面、<u>**１プロセスが行う作業量が多いため動作に遅延が出る**</u>恐れがあります。また、一度作ったプログラムを使い回せないという欠点があります。

スレッドやプロセスなどそれぞれの言葉に馴染みがなくてもどうか我慢してください。

完全に理解するというより「へ〜そうなんだ」ぐらいで読んでくれてもかまいませんし、私はどっちかと言うとそのようにソフトウェアを学んだタイプだと思っています。

次にスレッドが複数あるときのプログラムです。

実装している機能については先程のものと同じです。

#### １プロセス３スレッド
```mermaid
flowchart TD

subgraph　スレッド1
コントローラーの情報受け取る
end

subgraph スレッド2
ロボットと通信する --> 焦土作戦実行
end

subgraph スレッド3
コントローラーの情報受け取る　--> a2[使いたい値のみ抜き出す]
a2 --> a3[足回りや機構に応じて計算する]
a3 --> ロボットと通信する
end
```

これは最初のプログラムと違ってスレッドが分かれています。物を工場で作成するような工程で想像してみると最初のプログラムは一つの生産ラインに作業する人が１人しかいない状態で２つ目のプログラムは工場に作業する人が何人もいて分担してるようなものだと私は認識しています。

よって複数のスレッドを立てることはとても「強い」ソフトウェアを開発することに長けているように聞こえますが、これを<u>**実現するには高いプログラム能力が求められてしまう**</u>他、<u>**一人が作り始めたものに他人が参加しづらい**</u>というデメリットもあります。

### FLSの良さ
これまで色々なソフトウェアの開発の仕方を見てきましたが「じゃあ、どうすればいいんだよ」って話なんですよ。ここで私が開発した***FLS***を使うのです。１番最初に***FLS***はプロセス間通信ライブラリであるとお話しました。では<u>**プロセス間通信**</u>とは何者なのでしょうか。

```mermaid
flowchart TD

subgraph　プロセス1
コントローラーの情報受け取る
end

subgraph プロセス3
ロボットと通信する --> 焦土作戦実行
end

subgraph プロセス2
コントローラーの情報受け取る　--> a2[使いたい値のみ抜き出す]
a2 --> a3[足回りや機構に応じて計算する]
a3 --> ロボットと通信する
end
```

またロボットを動かすための機能について簡単に図示してみました。これをみると、実は「スレッドがプロセス」になっただけです。しかし意味は全く異なってきます。１プロセスで複数のスレッドを取り扱う開発は共同開発がとても難しいという話をしました。それに対してプロセス間通信はプロセスを分けることが許されているため<u>**各機能を１パッケージずつ開発することが可能**</u>です。

例えば「Aくんはコントローラーの情報を読み取ってそれをプロセス間通信に発信するパッケージを作ってよ」とか「Bくんはプロセス間通信で受け取った情報でロボットと通信するパッケージをつくって」とか機能ごとに人を割り当てるような開発が可能です。

さらにパッケージの使い回しも効くようになります。先述したコントローラー情報を扱う機能は毎年必要なはずです。そこで「コントローラー情報を発信するパッケージ」として完結して開発していると来年もその情報を受信する部分のみ書き換えることによって使い回しが可能となるのです。