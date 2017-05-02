# Capybara

[![endorse](https://api.coderwall.com/willnet/endorsecount.png)](https://coderwall.com/willnet)

## これはなに

[Capybara](https://github.com/jnicklas/capybara) の [README.md](https://github.com/jnicklas/capybara/blob/master/README.md) を翻訳したものです。「意味が通じること」を重視しており、正確な翻訳を意図したものではありません(なるべく正確に訳そうとは思っています)。原文が更新されていたり、翻訳の間違いを見つけたら Pull Request を送っていただけると助かります。

現在の翻訳は、[dbd0d8b3ef7c4f7ce13f929b55d2d456d8033b39 ](https://github.com/jnicklas/capybara/blob/dbd0d8b3ef7c4f7ce13f929b55d2d456d8033b39/README.md)を元に作成されています。

## 協力のお願い

Capybara の README に関連するコミットを [Issues](https://github.com/willnet/capybara-readme-ja/issues) に自動で登録するような仕組みを作りました。([deka](https://github.com/willnet/deka) という gem を使っています)。登録されている Issue は、この訳と翻訳元に対しての差分になります。

この Issue に対応して頂けると大変助かります。もし対応して頂ける場合は、

- 古い Issue から対応する
- 翻訳の編集と同時に、上記の「現在の翻訳は、8100b25952aab2a966552912a1f15d65d9c66b17を元に作成されています。」の部分も編集する

の２つをご協力お願いします。

## 序文

Capybara は、webアプリケーションのテストを補助するライブラリです。ユーザが実際に web アプリを扱うやり方をシミュレートします。Capybara は複数のドライバを切り替えて使うことができます。デフォルトでは Rack::Test と Selenium をサポートしており、外部の gem で Webkit をサポートしています。

誰かの助けが必要なら、メーリングリストで質問してみてください(Github の issue を使うのはご遠慮ください)。 http://groups.google.com/group/ruby-capybara

## 主な利点

- **セットアップいらず** Rails や Rack アプリを使っているならすぐに使えます

- **直感的な API** 私たちが実際に使っている言葉が使えます

- **バックエンドを変更できる** テストを変更することなく、早い headless なブラウザを使ったり実際のブラウザを使ったりができます

## セットアップ

CapybaraにはRuby 1.9.3以上が必要です。インストールするには、次の行をあなたのGemfileに追加して、bundle install を実行してください。

```ruby
gem 'capybara'
```

もし Rails を使っているなら、テストのヘルパファイルに下記の行を追記してください

```ruby
require 'capybara/rails'
```

もし Rails を使っていないなら、Capybara.app に rack アプリを代入してください

```ruby
Capybara.app = MyRackApp
```

もし JavaScript のテストがしたい、もしくはリモートのURLに対してテストをしたいときは、別のドライバを使う必要があるでしょう。

## Capybara を Cucumber と使う

`cucumber-rails` gem はビルトインで Capybara 用のコードを含んでいます。もしあなたが Rails を使っていないなら、`capybara/cucumber` をロードするコードを書く必要があります。

```ruby
require 'capybara/cucumber'
Capybara.app = MyRackApp
```

step 中に Capybara の DSL を下記のように書けます。

```ruby
When /I sign in/ do
  within("#session") do
    fill_in 'Login', :with => 'user@example.com'
    fill_in 'Password', :with => 'password'
  end
  click_link 'Sign in'
end
```

`@javascript`タグを使うことで、ドライバを`Capybara.javascript_driver`(:selenium がデフォルト)に切り替えることができます。

```ruby
@javascript
Scenario: do something Ajaxy
  When I click the Ajax link
  ...
```

明示的にドライバを指定する`@selenium`や`@rack_test`タグも用意されています。

## Capybara を RSpec と共に使う

下記の上を追加して(通常は`spec_helper.rb`ファイル)、RSpec 2.x 用のコードをロードしましょう。

```ruby
require 'capybara/rspec'
```

もしあなたが Rails を使っているならば、Capybara の spec は `spec/features` に置きましょう。

もしあなたが Rails を使っていないのならば、Capybara を使いたい全ての example groups に `:type => :feature` を追加する必要があります。

下記のように spec を書けます。

```ruby
describe "the signin process", :type => :feature do
  before :each do
    User.make(:email => 'user@example.com', :password => 'caplin')
  end

  it "signs me in" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Login', :with => 'user@example.com'
      fill_in 'Password', :with => 'password'
    end
    click_link 'Sign in'
    expect(page).to have_content 'Success'
  end
end
```

`:js => true` を使うことで、ドライバを`Capybara.javascript_driver`(:selenium がデフォルト)に切り替えることができます。もしくは`:driver`オプションを使うと特定のドライバに切り替えることができます。

例

```
describe 'some stuff which requires js', :js => true do
  it 'will use the default js driver'
  it 'will switch to one specific driver', :driver => :webkit
end
```

最後に、Capybara には受け入れテスト記述用のDSLもあります。

```ruby
feature "Signing in" do
  background do
    User.make(:email => 'user@example.com', :password => 'caplin')
  end

  scenario "Signing in with correct credentials" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Login', :with => 'user@example.com'
      fill_in 'Password', :with => 'caplin'
    end
    click_link 'Sign in'
    expect(page).to have_content 'Success'
  end

  given(:other_user) { User.make(:email => 'other@example.com', :password => 'rous') }

  scenario "Signing in as another user" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Login', :with => other_user.email
      fill_in 'Password', :with => other_user.password
    end
    click_link 'Sign in'
    expect(page).to have_content 'Invalid email or password'
  end
end
```


`feature` は実は単なる `describe ... :type => :feature` のエイリアスです。`background` は `before` 、 `scenario` は `it`、`given` と `given!` は それぞれ `let` と `let!` のエイリアスです。

## Capybara を Test::Unit で使う

もしあなたが Rails を使っているなら、下記のコードを `test_helper.rb` に追加しましょう。`ActionDispatch::IntegrationTest`を継承している全てのテストで Capybara が利用可能になります。

```ruby
class ActionDispatch::IntegrationTest
  # Capybara DSL をすべての integration テストで利用可能にする
  include Capybara::DSL
end
```

もしあなたが Rails を使っていないのなら、Capybara テスト用のベースとなるクラスを下記のように定義します。

```ruby
class CapybaraTestCase < Test::Unit::TestCase
  include Capybara::DSL

  def teardown
    Capybara.reset_sessions!
    Capybara.use_default_driver
  end
end
```

`teardown` をオーバライドする全てのサブクラスで `super` を呼ぶことを忘れないでください。

ドライバを変更するには、`Capybara.current_driver` に代入しましょう。

例

```ruby
class BlogTest < ActionDispatch::IntegrationTest
  setup do
    Capybara.current_driver = Capybara.javascript_driver # デフォルト :selenium
  end

  test 'shows blog posts' do
    # ... このテストは Seleniumu によって実行される ...
  end
end
```

## Capybara を MiniTest::Spec で使う

ベースとなるクラスを Test::Unit と同じように設定しましょう。(Rails では、ベースとなるクラスが ActionDispatch::IntegrationTest 以外ということもありえます)

capybara_minitest_spec という gem ([GitHub](https://github.com/ordinaryzelig/capybara_minitest_spec), [rubyGems.org](https://rubygems.org/gems/capybara_minitest_spec)) が、 Capybara 用の MiniTest::Spec expectations を提供しています。例

```ruby
page.must_have_content('Important!')
```

## ドライバ

Capybara では、同じ DSL で様々なブラウザや headless なドライバを扱うことができます。

### ドライバを選択する

デフォルトでは、Capybara は `:rack_test` ドライバを使います。`:rack_test` は速いけれど、JavaScript をサポートしていないのと、Rack アプリの外側から HTTP リソースにアクセス出来ないという制限があります。これらの制限を回避するには、欲しい機能に合わせて別のデフォルトドライバを設定します。例えばもし Selenium で全てのテストを走らせたければ、下記のように書けます。

```ruby
Capybara.default_driver = :selenium
```

しかし、もしあなたが Rspec か Cucumber を使っているのなら、早い `:rack_test` を default_driver のままにしておいて、JavaScript を扱う必要のあるテストにだけ それぞれ `:js => true` もしくは `@javascript` でマークを付けることもできます。デフォルトでは、JavaScript のテストは `:selenium` ドライバが使われます。`Capybara.javascript_driver`に代入することで変更可能です。

ドライバを一時的に変更することもできます(一般的に Before/setup と After/teardown ブロックで使われます)

```ruby
Capybara.current_driver = :webkit # 一時的に別のドライバを設定
... tests ...
Capybara.use_default_driver       # デフォルトドライバに戻す
```

**注釈:** ドライバの変更は新しい session を作ります。なのでテストの途中で変更することはできません。

### RackTest

RackTest は Capybara のデフォルトドライバです。これは pure Ruby で書かれていて、JavaScript の実行はサポートしていません。RackTest ドライバは直接 Rack のインタフェースに作用するので、テスト用のサーバーを立ち上げる必要はありません。つまり Rack アプリ(Rails や Sinatra、他のほとんどの Ruby フレームワークは Rack アプリです)でなければ、このドライバを使うことは出来ないということです。さらには、 RackTest ドライバはリモートのアプリのテストにも使えませんし、アプリが扱うリモートの URL にアクセスすることもできません(例: 外部サイトへのリダイレクト、外部 API、OAuth サービス)

[capybara-mechanize](https://github.com/jeroenvandijk/capybara-mechanize) は、RackTest に似ていて、リモートのサーバにアクセス可能なドライバを提供しています。

RackTest はこのように設定することが出来ます。

```ruby
Capybara.register_driver :rack_test do |app|
  Capybara::RackTest::Driver.new(app, :headers => { 'HTTP_USER_AGENT' => 'Capybara' })
end
```

詳しくは "ドライバを設定、追加する" のセクションを見てください。

### Selenium

現在、Capybara は [Selenium 2.0(Webdriver)](http://docs.seleniumhq.org/docs/01_introducing_selenium.jsp#selenium-2-aka-selenium-webdriver) をサポートしており、Selenium RC はサポートしていません。Selenium を利用するためには、`selenium-webdriver` gem をインストールする必要があります。bundler を使っているのなら Gemfile にそれを追加しましょう。Firefox がインストールされていた場合、設定は全てすんでおり、すぐに Selenium を使い始めることが出来ます。

**注釈:** 異なるスレッドでサーバを動かす種類のドライバは、テスト中で同一のトランザクションを共有することができません。それによりテストとテストサーバ感でデータを共有できなくなります。Transaction Fixtures の章を読んでください。

### Capybara-webkit

[capybara-webkit](https://github.com/thoughtbot/capybara-webkit) は、headless なテスト用のドライバです。QtWebKit をレンダリングエンジン用に使っています。capybara-webkit は JavaScript を実行出来ます。ブラウザを全てロードすることはないので、Selenium のようなドライバよりかなり早いです。

このようにインストールして

```sh
gem install capybara-webkit
```

このように設定すると使えます。

```ruby
Capybara.javascript_driver = :webkit
```

### Poltergeist

[Poltergeist](https://github.com/jonleighton/poltergeist) は、Capybara と [PhantomJS](http://phantomjs.org/) とを統合する、もう一つの headless なドライバです。完全に headless なので、Xvfb が必要ありません。さらに、ページ中で起きた Javascript のエラーを検知してレポートしてくれます。

## DSL

完全なリファレンスは [rubydoc.info](http://rubydoc.info/github/jnicklas/capybara/master) で利用可能です。

**注釈:** Capybara での全ての検索は case sensitive です。これは、case insensivity をサポートしていない XPath を大量に使っているためです

### Navigating

他のページに遷移するメソッドとして<tt>[visit](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Session#visit-instance_method)</tt>が使えます。

```ruby
visit('/projects')
visit(post_comments_path(post))
```

visit メソッドは引数を一つだけ取り、リクエストに使用するメソッドは **常に** GET です。

テストのアサーション用に[カレントパス](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Session#current_path-instance_method)を取得することができます。

```ruby
expect(current_path).to eq(post_comments_path(post))
```

### Clicking links and buttons

フルのリファレンスはこちら: [Capybara::Node::Actions](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)

リンクやボタンを押してwebアプリと通信することが出来ます。Capybara は自動でリダイレクトに対応し、ボタンに関連するフォームをサブミットします。

```ruby
click_link('id-of-link')
click_link('Link Text')
click_button('Save')
click_on('Link Text') # リンクかボタンどちらかをクリック
click_on('Button Value')
```

### Interacting with forms

フルのリファレンスはこちら: [Capybara::Node::Actions](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Actions)

フォーム要素を操作するツールがたくさんあります。

```ruby
fill_in('First Name', :with => 'John')
fill_in('Password', :with => 'Seekrit')
fill_in('Description', :with => 'Really Long Text...')
choose('A Radio Button')
check('A Checkbox')
uncheck('A Checkbox')
attach_file('Image', '/path/to/image.jpg')
select('Option', :from => 'Select Box')
```

### Querying

フルのリファレンスはこちら: [Capybara::Node::Matchers](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Matchers)

Capybara はページに特定の要素が存在しているかを調べることが出来るオプションを豊富に持っており、かつそれらの要素の操作もできます。

```ruby
page.has_selector?('table tr')
page.has_selector?(:xpath, '//table/tr')

page.has_xpath?('//table/tr')
page.has_css?('table tr.foo')
page.has_content?('foo')
```

**注釈:** `has_no_selector?` のような否定形は、`not has_selector?`とは異なります。詳しくは asynchronous JavaScript の章を読みましょう。

Rspecの魔法のマッチャによって下記のように書けます。

```ruby
expect(page).to have_selector('table tr')
expect(page).to have_selector(:xpath, '//table/tr')

expect(page).to have_xpath('//table/tr')
expect(page).to have_css('table tr.foo')
expect(page).to have_content('foo')
```

### Finding

フルのリファレンスはこちら: [Capybara::Node::Finders](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Node/Finders)

特定の要素を探して操作することが出来ます。

```ruby
find_field('First Name').value
find_link('Hello').visible?
find_button('Send').click

find(:xpath, "//table/tr").click
find("#overlay").find("h1").click
all('a').each { |a| a[:href] }
```

*注釈:* find は、Ajax の章で説明されているように、要素がページに表示されるのを待ちます。もし要素が現れなければエラーを吐きます。

これらの要素は Capybara の DSL メソッドを全て使え、ページの箇所を特定して DSL の作用する場所を制限することが出来ます。

```ruby
find('#navigation').click_link('Home')
expect(find('#navigation')).to have_button('Sign out')
```

### Scoping

Capybara は form 操作やリンクやボタンのクリックなどの特定のアクションを、ページの特定のエリア内で行うように制限することが可能です。within メソッドを使うことでそれができ、オプションでセレクタの種類を特定することが出来ます。

```ruby
within("li#employee") do
  fill_in 'Name', :with => 'Jimmy'
end

within(:xpath, "//li[@id='employee']") do
  fill_in 'Name', :with => 'Jimmy'
end
```

fieldset や table 用の特別なメソッドがあります。fieldset は legend タグ内の id かテキスト、table は caption タグの id かテキストを見ます。

```ruby
within_fieldset('Employee') do
  fill_in 'Name', :with => 'Jimmy'
end

within_table('Employee') do
  fill_in 'Name', :with => 'Jimmy'
end
```

### Scripting

ドライバがサポートしていれば、簡単に JavaScript を実行できます。

```ruby
page.execute_script("$('body').empty()")
```

簡単な式であれば、script の結果を受け取ることが出来ます。複雑な式だとうまくいかないかもしれません。

```ruby
result = page.evaluate_script('4 + 4');
```

### Debugging

下記のメソッドで、現在の状況をスナップショットとして撮って見れます。便利です。

```ruby
save_and_open_page
```

現在の DOM の状態を、 `page.html` を使用することで文字列として取得することができます。

```ruby
print page.html
```

これは主にデバッグ時に有用なものです。`page.html` に対してテストをするのは避けて、より表現力のある finder メソッドを代わりに使いましょう。

最後に、ドライバがサポートしていればスクリーンショットを保存することができます。

```ruby
page.save_screenshot('screenshot.png')
```

また、スクリーンショットを保存し、自動的に開くことができます。

```ruby
save_and_open_screenshot
```

## Matching

Capybara の要素の見つけ方をカスタマイズすることができます。`Capybara.exact` と `Capybara.match` の二つの選択肢があります。

### Exactness

`Capybara.exact` と `exact` オプションは XPath gem の内部で使われている `is` に影響します。`exact` が true のとき、全ての `is` 式が完全一致になります。 false のときは部分一致を許可します。この方法で部分一致を許可するかどうか指定できます。デフォルトでは `Capybara.exact` は false です。

例:

```ruby
click_link("Password") # "Password confirmation" にもマッチする
Capybara.exact = true
click_link("Password") # "Password confirmation" にはマッチしない
click_link("Password", exact: false) # オーバライドできる
```

### Strategy

`Capybara.match` と `match` オプションを使うことで、複数の要素が問合せにマッチしたときに Capybara がどのように振る舞うかをコントロールできます。現在 Capybara では4つの振る舞い方が定義されています。

1. **first:** 最初にマッチした要素を採用する
2. **one:** 二つ以上の要素がマッチしたら例外を発生させる
3. **smart:** もし `exact` が `true` なら、`one` と同じように二つ以上の要素がマッチしたら例外を発生させる。もし `exact` が `false` なら、まず最初に完全一致で試してみて、二つ以上の要素がマッチしたら例外を発生させる。もし要素が見つからなかったら、部分一致で再度要素を探す。ここで複数の要素がマッチしたら例外を発生させる。
4. **prefer_exact:** もし複数の要素がマッチし、そのうちのいくつかが完全一致でいくつかが部分一致だったら、最初の完全一致の要素を返す。

`Capybara.match` のデフォルトは `:smart` です。Capybara 2.0.x の挙動にしたいのならば、`Capybara.match` を `:one` に設定しましょう。Capybara 1.x の挙動にしたいのならば、`Capybara.match` を `:prefer_exact` に設定しましょう。

## Transactions and database setup

いくつかの Capybara のドライバは実際の HTTP サーバに対して動きます。Capybara はこれに対応しており、テストプロセスと同一のプロセス(ただし別スレッド)で HTTP サーバを走らせます。Selenium は HTTP サーバを使うドライバの一つです。RackTest は違います。

もしあなたが SQL データベースを使っているのなら、それぞれのテストをトランザクション内で実行し、テストの終わりにロールバックするやり方がよく使われます。例えば rspec-rails はそれをデフォルトで実行しています。トランザクションはスレッドをまたがって共有できないので、もし HTTP サーバを使用するドライバを使っているのなら、あなたがデータベースに入れたデータを Capybara からは確認することができません。

Cucumber はトランザクションの代わりに truncation を使うことでこの問題に対応しています。すなわち、それぞれのテストの後に全てのデータベースを空にしているのです。[database_cleaner](https://github.com/bmabey/database_cleaner) のような gem を使うことで、同じような振る舞いをさせることができます。

ORM に、全てのスレッドで同じトランザクションを使わせることも可能です。これはスレッドセーフ的な課題があり、奇妙な現象が起こる可能性があります。この方法は気をつけて使いましょう。下記のモンキーパッチを使うと ActiveRecord でスレッド間で同一トランザクションができます。

```ruby
class ActiveRecord::Base
  mattr_accessor :shared_connection
  @@shared_connection = nil

  def self.connection
    @@shared_connection || retrieve_connection
  end
end
ActiveRecord::Base.shared_connection = ActiveRecord::Base.connection
```

## Asynchronous JavaScript (Ajax and friends)

Ajax を使っている部分をテストするときに、まだページ上に現れていない DOM 要素を扱おうとしてしまうケースがあるはずです。Capybara はページ上の要素が現れるのを待つことで、自動でこの問題に対応します。

下記のような DSL を書いたとき

```ruby
click_link('foo')
click_link('bar')
expect(page).to have_content('baz')
```

もし *foo* というリンクをクリックすることが ajax のトリガーになっていて、完了したときに *bar* というリンクがページに追加されるとしたら、リンクが表示されないために *bar* リンクをクリックするのに失敗するでしょう。しかし Capybara は諦めてエラーを投げる前に、少しの間待ってリトライします。次の行でb*baz* を探すのも同様です。少しの間待ってリトライします。どれくらいの時間待つかを調整することもできます(デフォルトは2秒です)。

```ruby
Capybara.default_wait_time = 5
```

この振る舞いがあることによって気をつけて欲しいことがあります。下記の2つの命令は同じでは **ありません** 。 **常に** 後者を使うようにしてください！

```ruby
!page.has_xpath?('a')
page.has_no_xpath?('a')
```

前者は、コンテンツがまだ削除されていない場合すぐに失敗します。後者だけが非同期プロセスによってコンテンツがページから削除されるのを待ちます。

しかし、 Capybara の Rspec マッチャは賢くてどちらの書き方もうまく扱えます。下記の二つの命令は機能的に同じです。

```ruby
expect(page).not_to have_xpath('a')
expect(page).to have_no_xpath('a')
```

Capybara の待つ振る舞いはとても革新的で、下記のような書き方も扱えます。

```ruby
expect(find('#sidebar').find('h1')).to have_content('Something')
```

もし JavaScript によって `#sidebar` がページから見えなくなった場合、Capybara は自動で `#sidebar` とそれが含む全ての要素をリロードします。そして AJAX リクエストによって `#sidebar` の内容が変化(`h1` のテキストが "Something" に)した場合、テストは通ります。もしこの挙動をさせたくなかったら、`Capybara.automatic_reload` を `false` にセットしてください。

## Using the DSL elsewhere

`Capybara::DSL` を include することで、どんなコンテキストでも capybara の DSL を使うことができるようになります。

```ruby
require 'capybara'
require 'capybara/dsl'

Capybara.default_driver = :webkit

module MyModule
  include Capybara::DSL

  def login!
    within("//form[@id='session']") do
      fill_in 'Login', :with => 'user@example.com'
      fill_in 'Password', :with => 'password'
    end
    click_link 'Sign in'
  end
end
```

これは、Capybara がサポートしていないテストフレームワークや、テスト以外の通常のスクリプティングでも使えます。

## Calling remote servers

通常、Capybara は同一プロセス内の Rack アプリをテストすることを想定しています。ただ、`app_host` に値を設定することで、ネット上の web サーバとやりとりすることもできます。

```ruby
Capybara.current_driver = :selenium
Capybara.app_host = 'http://www.google.com'
...
visit('/')
```

**注釈:** デフォルトドライバ(rack_test)はリモートサーバに対応していません。対応しているドライバであれば、指定したURLに直接アクセスすることができます。

```ruby
visit('http://www.google.com')
```

デフォルトでは、Capybara は自動的に Rack アプリをブートしようとしてしまいます。リモートのアプリをテストするなら、Capybara の Rack サーバをオフにしたくなるかもしれません。

```ruby
Capybara.run_server = false
```

## Using the sessions manually


[Session](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Session) を手動でインスタンス化して扱うことができます。

```ruby
require 'capybara'

session = Capybara::Session.new(:webkit, my_rack_app)
session.within("//form[@id='session']") do
  session.fill_in 'Login', :with => 'user@example.com'
  session.fill_in 'Password', :with => 'password'
end
session.click_link 'Sign in'
```

## XPath, CSS and selectors

Capybara は入力したセレクタの種類を推測するようなことはしません。デフォルトでは常に CSS を使用します。もし XPath が使いたいのであれば、下記のようにする必要があります。

```ruby
within(:xpath, '//ul/li') { ... }
find(:xpath, '//ul/li').text
find(:xpath, '//li[contains(.//a[@href = "#"]/text(), "foo")]').value
```

もしくはデフォルトのセレクタを XPath にすることもできます。

```ruby
Capybara.default_selector = :xpath
find('//ul/li').text
```

カスタムセレクタを追加することもできます。これは同じようなセレクタをよく使っている場合に便利です。

```ruby
Capybara.add_selector(:id) do
  xpath { |id| XPath.descendant[XPath.attr(:id) == id.to_s] }
end

Capybara.add_selector(:row) do
  xpath { |num| ".//tbody/tr[#{num}]" }
end

Capybara.add_selector(:flash_type) do
  css { |type| "#flash.#{type}" }
end
```

xpath メソッドに渡したブロックは、XPath 形式の String か、もしくは XPath gem を通して生成されたものを返さなければいけません。上記の設定により下記のようにカスタムセレクタを使うことができるようになります。

```ruby
find(:id, 'post_123')
find(:row, 3)
find(:flash_type, :notice)
```

## Beware the XPath // trap

XPath での // は特別で、あなたが思ってるような意味ではないかもしれません。通常の認識とは違い、// は "ドキュメント全体のどこか" であって "今のコンテキスト内でのどこか" ではありません。例えば

```ruby
page.find(:xpath, '//body').all(:xpath, '//script')
```

これは全ての script タグを body の中から探すと思うかもしれません。でも実際には、これはドキュメント全体の script タグを探します！ あなたが探しているのは .// で、これは "カレントノードの配下" を表します。

```ruby
page.find(:xpath, '//body').all(:xpath, './/script')
```

within で同じようにやるとこうなります。

```ruby
within(:xpath, '//body') do
  page.find(:xpath, './/script')
  within(:xpath, './/table/tbody') do
  ...
  end
end
```

## Configuring and adding drivers

Capybara はドライバを切り替えることが簡単にできます。また、ドライバを設定するためのAPIが用意されているし、独自のドライバを追加することが出来ます。下記は selenium ドライバの設定を上書きして chrome を使う方法です。

```ruby
Capybara.register_driver :selenium do |app|
  Capybara::Selenium::Driver.new(app, :browser => :chrome)
end
```

この設定に違う名前を付けることもできます。

```ruby
Capybara.register_driver :selenium_chrome do |app|
  Capybara::Selenium::Driver.new(app, :browser => :chrome)
end
```

これにより、簡単にブラウザを切り替えてテストすることができます。

```ruby
Capybara.current_driver = :selenium_chrome
```


ブロックの戻り値は Capybara::Driver::Base の記述する API に従わなければいけませんが、Capybara::Driver::Base を継承している必要はありません。gem はこの API を、自分独自のドライバを Capybara に加えるために使えます。


[selenium wiki](http://code.google.com/p/selenium/wiki/RubyBindings)にはドライバ設定の情報が詳しく書かれています。


## Gotchas

* テストスレッドから session と request にアクセスすることはできません。response へのアクセスは限定されています。いくつかのドライバは response header と HTTP status code にアクセスできますが、アクセスできないドライバ(例: selenium)もあります。
* Rails の統合テストを使っていないので、Rails 特有のもの(例: controller)へアクセスすることは出来ません。
* 時間の固定: 現在時刻に依存したフィーチャをうまく動かすために、モックを使うのは一般的なやりかたです。しかし問題が起こることがあります。それは Capybara の Ajax のタイミングがシステムの時間を使っているせいで、failure な時に Capybara はタイムアウトせずにハングってしまいます。時間を止める機能でなく、時間を移動させる機能であればうまくいきます。[Timecop](https://github.com/travisjeffery/timecop) は両方の機能が使えます。
* Rack::Test を使っているときは、URL で visit しているかどうかチェックするべきです。たとえば、session は posts_path と posts_url で別々になります。もし Action Mailer の中で URL を使っているとしたら、default_url_options を Rails のデフォルトの www.example.com に設定しましょう。

## Development

開発環境をセットアップするには、下記のようにします。

```sh
bundle install
bundle exec rake  # run the test suite
```

issue や pull request の送り方は [CONTRIBUTING.md](https://github.com/jnicklas/capybara/blob/master/CONTRIBUTING.md) に書いてあります。
