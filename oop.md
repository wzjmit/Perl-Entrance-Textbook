# はじめに

この資料では, Perlにおけるオブジェクト指向について説明します.
｢Perlでオブジェクト指向をどのように実現するか｣を中心に扱い, オブジェクト指向の基本的な知識(理論)については**扱いません.**

# Perlのオブジェクト指向

Perlは良くも悪くも, 後方互換性を非常に重視した言語です.
そのため, 当初オブジェクト指向に関する機能を持っていなかったPerlがオブジェクト指向に対応する際も, 後方互換性を維持しながらオブジェクト指向に対応しました.
以下解説していきますが, そのような理由により, Perlのオブジェクト指向はJavaやRubyなど, 他のオブジェクト指向な言語に比べて非常に｢歪｣です.
他言語使いからすれば非常に理解に苦しむ部分も多いとは思いますが, ｢新しい言語に慣れるトレーニング｣と思って頑張っていきましょう.

これ以降は, 簡単なPerlのモジュールを作りながら, Perlのオブジェクト指向プログラミングや, それを支える各種モジュールの使い方を説明していきます.

# Minilla

Perlのモジュールを作るにあたって, まずそのひな形を用意します.
かつてはModule::Starterなど様々なツールが提案されていましたが, ここ数年は[@tokuhirom](https://twitter.com/tokuhirom)さんが開発した[Minilla](https://metacpan.org/pod/Minilla)がデファクトスタンダードになっています.
Minillaは, モジュールのひな形を作るだけでなく, モジュール開発の様々な支援(例えばテストの実施, モジュールのインストール, モジュールのファイルをまとめたtarballの作成, そしてモジュールのCPANへのアップロードなど)を行ってくれるので, モジュールを開発するのであれば積極的に利用すると良いでしょう.

plenvを利用したPerlの環境は構築済みだと思いますので, `cpanm`コマンドでMinillaをインストールします.

```
$ cpanm Minilla
--> Working on Minilla
Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Minilla-v2.4.1.tar.gz ... OK
Configuring Minilla-v2.4.1 ... OK
    ... 中略 ...
Successfully installed Minilla-v2.4.1 (upgraded from v2.1.1)
4 distributions installed
```

Minillaのインストールが完了しました(元々インストール済みだったので, 表示上は`upgraded`になっていますが).
これで, Minillaが提供する`minil`コマンドが利用できるようになっているはずです.

```
$ which minil
/Users/username/.anyenv/envs/plenv/shims/minil
```

それでは早速, `minil`コマンドからモジュールのひな形を作りましょう.
今回はチュートリアル用なので, `PerlEntrance::OOPTutorial`というモジュールを作ることとします.

```
$ minil new PerlEntrance::OOPTutorial
Writing lib/PerlEntrance/OOPTutorial.pm
Writing Changes
Writing t/00_compile.t
Writing .travis.yml
Writing .gitignore
Writing LICENSE
Writing cpanfile
Initializing git PerlEntrance::OOPTutorial
[PerlEntrance-OOPTutorial] $ git init
Initialized empty Git repository in /Users/username/current_directory/PerlEntrance-OOPTutorial/.git/
Retrieving meta data from lib/PerlEntrance/OOPTutorial.pm.
Name: PerlEntrance::OOPTutorial
Abstract: It's new $module
Version: 0.01
fatal: bad default revision 'HEAD'
[PerlEntrance-OOPTutorial] $ git add .
Finished to create PerlEntrance::OOPTutorial
```

カレントディレクトリに, `PerlEntrance-OOPTutorial`というディレクトリが出来ているはずです.

```
$ ls
PerlEntrance-OOPTutorial
```

これで, ひな形の生成は完了しました.

# ディレクトリ構成

次に, ひな形として生成されたファイルを見ながら, Perlのモジュールのディレクトリ構成について解説します.
生成された`PerlEntrance-OOPTutorial`の中で, `tree`コマンドを実行すると, 次のような出力になるはずです.

```
$ cd PerlEntrance-OOPTutorial
$ tree
.
├── Build.PL
├── Changes
├── LICENSE
├── META.json
├── README.md
├── cpanfile
├── lib
│   └── PerlEntrance
│       └── OOPTutorial.pm
├── minil.toml
└── t
    └── 00_compile.t

3 directories, 9 files
```

## `Build.PL`

モジュールをビルドする為のファイルです. 開くと,

```:Build.PL(抜粋)
# =========================================================================
# THIS FILE IS AUTOMATICALLY GENERATED BY MINILLA.
# DO NOT EDIT DIRECTLY.
# =========================================================================
```

と書いてある通り, 基本的には手動で書き換えることはありません.

## `Changes`

モジュールの更新履歴です. 初期状態では次のようになっています.

```:Changes
Revision history for Perl extension PerlEntrance-OOPTutorial

{{$NEXT}}

    - original version
```

このように書いておくと, モジュールのリリース時に`{{$NEXT}}`の部分が現在の日付に置き換わり,

```:Changes
Revision history for Perl extension PerlEntrance-OOPTutorial

0.01 2015-05-05T09:00:00Z

    - original version
```

のように書き換わってくれます.

## `LICENSE`

このモジュールのライセンスを表すファイルです.
初期状態では, PerlのモジュールはPerlと同じ[Artistic License](http://ja.wikipedia.org/wiki/Artistic_License)か[GPL](http://ja.wikipedia.org/wiki/GNU_General_Public_License)を選択できるライセンスになっています.

よほど特殊な事情ではないかぎり, 書き換える必要はありません.

## `META.json`

モジュールのメタ情報が格納されたファイルです.
こちらも`Build.PL`と同様, Minillaが自動的に生成してくれるので, 手動で書き換える必要はありません.

## `cpanfile`

このモジュールが依存する(必要とする)モジュールを書くファイルです.
初期状態では, 次のようになっています.

```perl:cpanfile
requires 'perl', '5.008001';

on 'test' => sub {
    requires 'Test::More', '0.98';
};
```

`cpanfile`はPerlのDSLとして提供されており, `requires <モジュール名>, <バージョン>;`という記述で, このモジュールが`<モジュール名`に依存しており, その依存モジュールの`<バージョン>`で指定したバージョンが必要である, ということを指定することができます(`<バージョン>`については省略することができます).

```perl
requires 'perl', '5.008001';
```

ここでは, このモジュールがPerlの5.8.1以上を必要としていることを指定しています(基本的に, ここ最近のモジュールはPerl 5.8.1以上で動作するように作ることが一般的です).

例えば, このモジュールが`JSON`に依存しているのであれば,

```perl:cpanfile
requires 'perl', '5.008001';
requires 'JSON'; # 追加

on 'test' => sub {
    requires 'Test::More', '0.98';
}
```

のように記述すると良いでしょう.

なお, `on 'test' => sub { ... };`の部分は, モジュールの動作に必要はないが, テストの為に必要なモジュールを指定する部分です.
後述しますが, Perlにはテストを支援する様々なモジュールが提供されています.
それらを利用するのであれば, このエリアに記述するようにしましょう.

## `minil.toml`

Minillaの設定ファイルです.
基本的には書き換える必要はありません.

## `lib`

モジュール本体のコードを格納するディレクトリです.
詳しくは後述します.

## `t`

モジュールのテストコードを格納するディレクトリです.
こちらも, 詳しくは後述します.

# 名前空間

Perlには, ｢名前空間｣という概念があります.
名前空間は, `package`という構文で指定することができます.

```perl:namespace.pl
use strict;
use warnings;
use utf8;

hoge();

package Hoge;

sub hoge {
    print 'hoge';
}
```

このコードを例に, Perlの名前空間を解説していきます.

まず5行目では, `hoge();`で`hoge`というサブルーチンを呼び出しています.
一方7行目以降では, `package Hoge;`で`Hoge`という名前空間を用意しており, その中で`hoge`というサブルーチンを定義しています.

この状態でスクリプトを実行すると, どうなるでしょうか?

```
$ perl namespace.pl
Undefined subroutine &main::hoge called at namespace.pl line 5.
```

`hoge`というサブルーチンが定義されていない, というエラーが出ました.
ここで注目して欲しいのは, `&main::hoge`の部分です.
実は, Perlでは名前空間が指定されていない場合, 自動的に`main`という名前空間に属している, と処理されるのです.
そして, 特に指定がない場合, Perlは現在の名前空間(この場合, `main`)から変数やサブルーチンを探し出し, 処理しようとします.
そのため, このエラーは`main`という名前空間に`hoge`という関数が定義されていないので, エラーになったという意味になります.

前述の通り, `hoge`というサブルーチンは`Hoge`という名前空間で定義されているので, `main`という名前空間からそのまま利用することができません.
`main`から, `Hoge`という名前空間のサブルーチンを呼ぶ為には, `hoge`というサブルーチンを呼び出す際に, その名前空間も指定しなければなりません.

```perl:namespace.pl
use strict;
use warnings;
use utf8;

Hoge::hoge();

package Hoge;

sub hoge {
    print 'hoge';
}
```

`hoge();`を`Hoge::hoge();`に書き換えました.
スクリプトを実行すると...

```
$ perl namespace.pl
hoge
```

`main`名前空間から, `Hoge`名前空間の`hoge`というサブルーチンを呼び出すことができました.
ここまでの内容をまとめると, 次のようになると思います.

- Perlのサブルーチンや変数は, 必ず何らかの名前空間に属している
    - 名前空間の指定がない場合, `main`という名前空間に属すことになる
- サブルーチンや変数を呼び出す際, 名前空間を省略すると, 現在の名前空間から探しだそうとする
- `Hoge::`のように, プレフィックスに名前空間を付けると, 他の名前空間のサブルーチンを呼び出すことができる

なお, 余談ではありますが, 現在の名前空間は`__PACKAGE__`で確認することができます.

```perl:namespace.pl
use strict;
use warnings;
use utf8;

print __PACKAGE__ . "\n"; # => main
Hoge::hoge();

package Hoge;

sub hoge {
    print __PACKAGE__ . "\n"; # => Hoge
}
```

もし1つの名前空間しかない場合, プログラマは全ての変数名とサブルーチン名が既存のものと重複しないことを確認しながらコーディングをしなければなりません.
しかし, 名前空間を適切に利用すれば, 変数やサブルーチンといったコードの影響範囲を, 名前空間の内部だけに抑えることが出来ます.
更に, 適切な名前空間を与えることで, そのコードの役割(例えば, `MyApp::Model::User`の場合, この名前空間に存在するコードは, `MyApp`というアプリケーションの`User`に関する`Model`についてのコードである, など)を伝えることもできるでしょう.

## 名前空間とファイルの配置

Perlの名前空間とライブラリのファイル配置には, 密接な関係があります.
例えば, あるスクリプトで`Foo::Bar::Baz`という名前空間のコードを利用するために, `use Foo::Bar::Baz;`と記載したとします.
このとき, Perlは`lib`ディレクトリ以下にある`Foo/Bar/Baz.pm`というファイルを探索し, 見つければそのファイルを`Foo::Bar::Baz`のコードとして処理します.
逆に言えば, `Foo::Bar::Baz`という名前空間のコードは, `lib`ディレクトリ以下の`Foo/Bar/Baz.pm`というファイルに記載しなければ, Perlはうまくコードを見つけ出すことが出来ず, 期待する動作をしてくれません.

それではもう一度, `PerlEntrance::OOPTutorial`の内部で`tree`コマンドを実行した時の出力を見てみます.

```
.
├── Build.PL
├── Changes
├── LICENSE
├── META.json
├── README.md
├── cpanfile
├── lib
│   └── PerlEntrance
│       └── OOPTutorial.pm
├── minil.toml
└── t
    └── 00_compile.t

3 directories, 9 files
```

`PerlEntrance::OOPTutorial`という名前空間に属するコードは, `lib`ディレクトリ以下の`PerlEntrance`ディレクトリにある, `OOPTutorial.pm`というファイルに記載するようになっています.
Perlでコードを書く時は, 必ずコードの名前空間とファイル名, ディレクトリ構成を同じにするようにしましょう(このどちらかに誤りがあり, 期待通りにコードが動かない... というのは, Perlでモジュールを作ったり, オブジェクト指向プログラミングをする時によくあるミスです).

# Perlのオブジェクト指向プログラミング

それでは早速, Perlのオブジェクト指向プログラミングについて体験していきます.
誤解を恐れずに一言で言えば, Perlはオブジェクト指向を, ｢名前空間でデータを包む｣ことによって実現している, と言うことができます.

```perl:oop.pl
use strict;
use warnings;
use utf8;

my $papix = Engineer->new(
    name => 'papix',
);

my $hoto = Engineer->new(
    name => 'hoto',
);

print $papix->name . "\n"; # => papix
print $hoto->name  . "\n"; # => hoto

package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}
```

実行結果は, 次のようになります.

```
$ perl oop.pl
papix
hoto
```

コードを詳しく見ていきましょう.
まずは, `Engineer`という名前空間のコードから見ていきます.

## コンストラクタ

```perl:oop.pl(抜粋)
package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}
```

`new`というサブルーチンはコンストラクタです.
`new`でなければならない, というルールはありませんが, Perlではコンストラクタは`new`というサブルーチンを使うのが一般的です.

サブルーチン`new`において, 引数を`my ($class, %params) = @_;`という形で受けています.
一方, このコンストラクタの呼び出し元を見ると, 次のようなコードになっています.

```perl:oop.pl(抜粋)
my $papix = Engineer->new(
    name => 'papix',
);
```

実はPerlは, `Engineer->new( ... );`のような形, つまり`名前空間->サブルーチン名( ... );`というフォーマットで他の名前空間のサブルーチンを呼び出した場合, 呼び出したサブルーチンの第1引数には呼び出した名前空間(つまりは, `->`の左側)が文字列で渡るようになっています.

その為, コンストラクタを次のように書き換えると...

```perl:oop.pl(抜粋)
sub new {
    my ($class, %params) = @_;

    print $class . "\n";

    bless {
        name => $params{name},
    }, $class;
}
```

実行結果は次のようになります.

```
$ perl oop.pl
Engineer
Engineer
papix
hoto
```

`Engineer->new( ... );`で呼び出しているので, コンストラクタの第1引数には`Engineer`という文字列が渡ってきていることがわかります.

...この時点で, ｢何だこれ, 気持ち悪い｣と思う方も多いのではないでしょうか.
しかし**気持ち悪いのはまだまだこれから**です.

### ｢祝福｣

```perl:oop.pl(抜粋)
sub new {
    my ($class, %params) = @_;

    print $class . "\n";

    bless {
        name => $params{name},
    }, $class;
}
```

コンストラクタの最後で, `%params`で受けたパラメータを利用してハッシュリファレンスを生成し, それと引数として受け取ったクラス名(今回の場合は, `Engineer`)を, `bless`という関数に渡しています.
実は, Perlという言語におけるオブジェクトは, このようにして`bless`という関数を使って, リファレンス(大抵の場合, ハッシュリファレンス)と名前空間を｢祝福｣することによって生成されるのです(!?).

これで, 第1引数のハッシュリファレンスで定義したデータと, 第2引数で指定した名前空間のコードが結びつき, 今回の場合はハッシュリファレンスから`Engineer`という名前空間に属するサブルーチンをメソッドとして呼び出すことができるようになります.

## オブジェクトからのメソッド呼び出し

というわけで, コンストラクタを使って,

```
my $papix = Engineer->new(
    name => 'papix',
);
```

のようにして, `{ name => 'papix' }`というデータと`Engineer`という名前空間が紐付いたオブジェクトが生成され,

```
my $hoto = Engineer->new(
    name => 'hoto',
);
```

のようにして, `{ name => 'hoto' }`というデータと`Engineer`という名前空間が紐付いたオブジェクトが生成されました.
この2つのオブジェクトを, Data::Dumperを使って確認してみましょう.

```perl:oop.pl
use strict;
use warnings;
use utf8;

use Data::Dumper;

my $papix = Engineer->new(
    name => 'papix',
);

my $hoto = Engineer->new(
    name => 'hoto',
);

print Dumper $papix;
print Dumper $hoto;

package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}
```

実行結果は, 次の通りになります.

```
$ perl oop.pl
$VAR1 = bless( {
                 'name' => 'papix'
               }, 'Engineer' );
$VAR1 = bless( {
                 'name' => 'hoto'
               }, 'Engineer' );
```

`$papix`と`$hoto`という変数には, それぞれ別のハッシュリファレンスと名前空間(`Engineer`)を祝福(`bless`)したもの(つまりはオブジェクト)が格納されている, という意味です.

それでは次に, オブジェクトからメソッドを呼び出してみます.

```perl:oop.pl
use strict;
use warnings;
use utf8;

my $papix = Engineer->new(
    name => 'papix',
);

my $hoto = Engineer->new(
    name => 'hoto',
);

print $papix->name . "\n";
print $hoto->name  . "\n";

package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}
```

実行結果は次の通りです.

```
$ perl oop.pl
papix
hoto
```

オブジェクトからメソッドを呼び出す場合, `$object->method();`で呼び出すことが出来ます.
このときメソッドは, そのオブジェクトに紐付いた名前空間から検索します.
例えば, `$papix->name();`のように呼び出した場合, `name`というメソッドは, `$papix`に紐付いた名前空間, すなわち`Engineer`という名前空間から検索するのです.

実際, 存在しないメソッドを呼び出した場合... 例えば`$papix->hoge;`のようなコードを書いた場合, Perlは次のようなエラーを出力します.

```
$ perl oop.pl
Can't locate object method "hoge" via package "Engineer" at oop.pl line 15.
```

Perlは, `$papix`に紐付いた`Engineer`という名前空間から`hoge`というメソッドを探したが, 見つからなかった, という意味です.

## メソッドの実装

さて, 呼び出されたメソッドはどのように実装すれば良いのでしょうか.
再び, `Engineer`名前空間のコードを見てみましょう.

```perl:oop.pl
package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}
```

`name`というサブルーチンが, `Engineer`という名前空間に紐ついたオブジェクトの`name`メソッドの実装になります.
ここでの重要なポイントは, サブルーチンの引数を`my ($self) = @_;`で受けている点です.

先程, コンストラクタを呼び出す場面で, `Engineer->new( ... );`のように呼び出すと, 呼び出した`new`というメソッドの第1引数には`Engineer`という文字列, つまり`->`の左側が渡ってくると説明しました.
これと同様, `$papix->name()`のようにオブジェクトから呼び出した場合, 呼び出したメソッドの第1引数には`->`の左側, つまり`$papix`のオブジェクト自身が渡ってきます.
そのため, メソッドの第1引数は自分自身, つまり`$self`で受けることが一般的です.

`name`というメソッドの中で, `$self`の中身をData::Dumperでダンプしてみます.

```
use strict;
use warnings;
use utf8;

my $papix = Engineer->new(
    name => 'papix',
);

my $hoto = Engineer->new(
    name => 'hoto',
);

print $papix->name . "\n";
print $hoto->name  . "\n";

package Engineer;
use Data::Dumper; # Data::DumperはEngineer名前空間で利用しているので, ここでuseしなければならない

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    print Dumper $self;
    return $self->{name};
}
```

実行結果は次のようになります.

```
$ perl oop.pl
$VAR1 = bless( {
                 'name' => 'papix'
               }, 'Engineer' );
papix
$VAR1 = bless( {
                 'name' => 'hoto'
               }, 'Engineer' );
hoto
```

`$papix`から`name`メソッドを呼び出した時の`$self`の中身と, `$hoto`から`name`メソッドを呼び出した時の`$self`の中身は, 先程`main`名前空間で`$papix`及び`$hoto`をダンプした時と同じであることがわかります.

後はメソッドの中で, `$self`で受け取った自分自身のオブジェクトを利用して, メソッドで実現したい操作をしてやれば良いです.
`bless`で祝福されたハッシュリファレンスも, 基本的には普通のハッシュリファレンスのように利用できるので, 例えば`Engineer`名前空間の`name`メソッドのように,

```
sub name {
    my ($self) = @_;
    return $self->{name};
}
```

`$self`というハッシュリファレンスの`name`というキーのデータを返すようにすれば, オブジェクト(に格納されたハッシュリファレンス)から`name`というキーのデータを返すことが出来ますし,

```
sub supernize {
    my ($self) = @_;
    if ($self->{name} eq 'hoto') {
        $self->{name} = 'super hoto';
    }
}
```

のようなメソッドを用意すれば...

```perl:oop.pl
use strict;
use warnings;
use utf8;

my $papix = Engineer->new(
    name => 'papix',
);

my $hoto = Engineer->new(
    name => 'hoto',
);

print $papix->name . "\n";
print $hoto->name  . "\n";
print "--------------------\n";
$papix->supernize;
$hoto->supernize;
print $papix->name . "\n";
print $hoto->name  . "\n";

package Engineer;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}

sub supernize {
    my ($self) = @_;
    if ($self->{name} eq 'hoto') {
        $self->{name} = 'super hoto';
    }
}
```

実行結果は, 次のようになります.

```
papix
hoto
--------------------
papix
super hoto
```

オブジェクト(に格納されたハッシュリファレンス)の中身を, メソッドの中で書き換えることも可能です.

# ファイルの切り出し

ここまで, `oop.pl`というスクリプトを元にして, Perlにおけるオブジェクト指向プログラミングの実装方法について解説してきました.
このスクリプトでは, `main`名前空間と`Engineer`名前空間を同じファイルに記述していましたが, このように同じスクリプトに記述することは滅多になく, `Engineer`名前空間で書いていたコードは`lib`ディレクトリ以下に切り出すことが大半です.

最後に, オブジェクト指向を構成するファイルを別ファイルに切り出し, これを任意のスクリプトから呼び出す方法について解説します.
今回は, `oop.pl`の`Engineer`名前空間で記述したコードを, 先程作ったひな形の`PerlEntrance::OOPTutorial::Engineer`という名前空間として切り出してみることにします.

`PerlEntrance::OOPTutorial::Engineer`は, `lib/PerlEntrance/OOPTutorial/Engineer.pm`として配置しなければなりません.
まずは, `lib/PerlEntrance`ディレクトリに移動し, `OOPTutorial`というディレクトリを作成しましょう.

```
$ ls
Build.PL   Changes    LICENSE    META.json  README.md  cpanfile   lib        minil.toml t

$ cd lib/PerlEntrance
$ ls
OOPTutorial.pm

$ mkdir OOPTutorial
$ ls
OOPTutorial    OOPTutorial.pm
```

それでは, `OOPTutorial`ディレクトリの中に, `Engineer.pm`を作成します.

```
$ vi OOPTutorial/Engineer.pm
```

中身は, 次のようにします.

```perl:lib/PerlEntrance/OOPTutorial/Engineer.pm
package PerlEntrance::OOPTutorial::Engineer;
use strict;
use warnings;
use utf8;

sub new {
    my ($class, %params) = @_;

    bless {
        name => $params{name},
    }, $class;
}

sub name {
    my ($self) = @_;
    return $self->{name};
}

1;
```

末尾の`1;`ですが, とりあえずここではPerlの｢お約束｣だと思って下さい.
Perlのモジュールを構成するファイル(`*.pm`のファイル)は, 必ず最後に真値を返さなければなりません.

それでは, このモジュールを利用したスクリプトを書いてみます.
Perlのモジュールでは, サンプルスクリプトを`eg`というディレクトリに配置することが多いので, `eg`ディレクトリに`oop.pl`というスクリプトを用意することにします.

```
$ ls
Build.PL   Changes    LICENSE    META.json  README.md  cpanfile   lib        minil.toml t

$ mkdir eg
$ cd eg
$ vi oop.pl
```

`oop.pl`の中身は, 次のようにします.

```perl:eg/oop.pl
use strict;
use warnings;
use utf8;

use PerlEntrance::OOPTutorial::Engineer;

my $papix = PerlEntrance::OOPTutorial::Engineer->new(
    name => 'papix',
);

my $hoto = PerlEntrance::OOPTutorial::Engineer->new(
    name => 'hoto',
);

print $papix->name . "\n";
print $hoto->name  . "\n";
```

5行目で, 先程作成した`PerlEntrance::OOPTutorial::Engineer`を読み込み, これを利用してオブジェクトを生成し, メソッドを呼び出しています.

それでは, 早速実行してみましょう.

```
$ cd ../
$ ls
Build.PL	Changes		LICENSE		META.json	README.md	cpanfile	eg		lib		minil.toml	t

$ perl eg/oop.pl
Can't locate PerlEntrance/OOPTutorial/Engineer.pm in @INC (you may need to install the PerlEntrance::OOPTutorial::Engineer module) (@INC contains: /Users/username/.anyenv/envs/plenv/versions/5.18/lib/perl5/site_perl/5.18.2/darwin-2level /Users/username/.anyenv/envs/plenv/versions/5.18/lib/perl5/site_perl/5.18.2 /Users/username/.anyenv/envs/plenv/versions/5.18/lib/perl5/5.18.2/darwin-2level /Users/username/.anyenv/envs/plenv/versions/5.18/lib/perl5/5.18.2 .) at eg/oop.pl line 5.
BEGIN failed--compilation aborted at eg/oop.pl line 5.
```

...実行に失敗しました.
これは, 先程作成した`PerlEntrance::OOPTutorial::Engineer`というモジュールが設置されているディレクトリを, Perlがライブラリを検索する際に検索対象としていないからです.
このモジュールはカレントディレクトリの`lib`以下に設置されているので, このディレクトリを`perl`コマンドの`-I`オプション(検索対象となるディレクトリの追加)で指定すればOKです.

```
$ perl -Ilib eg/oop.pl
papix
hoto
```

# 練習問題

`minil`コマンドでモジュールのひな形が準備が出来たら, このカリキュラムで用意した`PerlEntrance::OOPTutorial::Engineer`を実装してみよう.
また, `PerlEntrance::OOPTutorial::Engineer`以外のクラスを提供してみよう(例えば, `PerlEntrance::OOPTutorial::Chief`など).
`eg`ディレクトリ以下に, これらのクラスを利用した, 適当なスクリプトを書いてみよう.

```perl:例
use strict;
use warnings;
use utf8;

use PerlEntrance::OOPTutorial::Engineer;
use PerlEntrance::OOPTutorial::Chief;

my $hoto = PerlEntrance::OOPTutorial::Engineer->new( name => 'hoto' );
my $higo = PerlEntrance::OOPTutorial::Chief->new( name => 'higo' );

print $hoto->job(); # => 'engineer'
print $higo->job(); # => 'chief'
```
