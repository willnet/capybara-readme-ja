# Capybara

## これはなに

[Capybara](https://github.com/teamcapybara/capybara) の [README.md](https://github.com/teamcapybara/capybara/blob/master/README.md) を翻訳したものです。「意味が通じること」を重視しており、正確な翻訳を意図したものではありません(なるべく正確に訳そうとは思っています)。原文が更新されていたり、翻訳の間違いを見つけたら Pull Request を送っていただけると助かります。

現在の翻訳は、[331c8d1811e637d793ee5557c70a0db66cad8217](https://github.com/teamcapybara/capybara/blob/331c8d1811e637d793ee5557c70a0db66cad8217/README.md) を元に作成されています。新しい翻訳に追従するPull Requestをお待ちしています。

## 序文

Capybara は、webアプリケーションのテストを補助するライブラリです。ユーザが実際に web アプリを扱うやり方をシミュレートします。Capybara は複数のドライバを切り替えて使うことができます。デフォルトでは Rack::Test と Selenium をサポートしており、外部の gem で WebKit をサポートしています。

### Capybara をサポートする

もしあなたやあなたの会社が Capybara に価値を感じて、継続的なメンテナンスや開発を財政的に支援してくださるのでしたら、[Patreon](https://www.patreon.com/capybara)を見てください。

誰かの助けが必要なら、メーリングリストで質問してみてください(Github の issue を使うのはご遠慮ください)。 http://groups.google.com/group/ruby-capybara

## 目次

- [主な利点](#主な利点)
- [セットアップ](#セットアップ)
- [Capybara を Cucumber と使う](#capybara-を-cucumber-と使う)
- [Capybara を RSpec と使う](#capybara-を-rspec-と使う)
- [Capybara を Test::Unit と使う](#capybara-を-testunit-と使う)
- [Capybara を Minitest と使う](#capybara-を-minitest-と使う)
- [Capybara を Minitest::Spec と使う](#capybara-を-minitestspec-と使う)
- [ドライバ](#ドライバ)
    - [ドライバを選択する](#ドライバを選択する)
    - [RackTest](#racktest)
    - [Selenium](#selenium)
    - [Apparition](#apparition)
- [DSL](#dsl)
    - [Navigating](#navigating)
    - [Clicking links and buttons](#clicking-links-and-buttons)
    - [Interacting with forms](#interacting-with-forms)
    - [Querying](#querying)
    - [Finding](#finding)
    - [Scoping](#scoping)
    - [Working with windows](#working-with-windows)
    - [Scripting](#scripting)
    - [Modals](#modals)
    - [Debugging](#debugging)
- [Matching](#matching)
    - [Exactness](#exactness)
    - [Strategy](#strategy)
- [Transactions and database setup](#transactions-and-database-setup)
- [Asynchronous JavaScript (Ajax and friends)](#asynchronous-javascript-ajax-and-friends)
- [Using the DSL elsewhere](#using-the-dsl-elsewhere)
- [Calling remote servers](#calling-remote-servers)
- [Using sessions](#using-sessions)
    - [Named sessions](#named-sessions)
    - [Using sessions manually](#using-sessions-manually)
- [XPath, CSS and selectors](#xpath-css-and-selectors)
- [Beware the XPath // trap](#beware-the-xpath--trap)
- [Configuring and adding drivers](#configuring-and-adding-drivers)
- [Gotchas:](#gotchas)
- ["Threadsafe" mode](#threadsafe-mode)
- [Development](#development)

## 主な利点

- **セットアップいらず** Rails や Rack アプリを使っているならすぐに使えます

- **直感的な API** 私たちが実際に使っている言葉が使えます

- **バックエンドを変更できる** テストを変更することなく、速い headless なブラウザを使ったり実際のブラウザを使ったりができます

- **パワフルな同期機能** 非同期プロセスの完了を待つためのコードを書く必要はありません。

## セットアップ

CapybaraにはRuby 2.7.0以上が必要です。インストールするには、次の行をあなたのGemfileに追加して、bundle install を実行してください。

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
もし Rails 5.0 以上を使っているにもかかわらず、Rails 5.1 から導入されたシステムテストを使っていないときは、 Rails のデフォルトに合わせてサーバーを Puma に変更した方が良いでしょう。

```ruby
Capybara.server = :puma # セットアップはこれで完了
Capybara.server = :puma, { Silent: true } # テストで Puma 起動時のログを出力をしないオプション
```

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

`@javascript`タグを使うことで、ドライバを`Capybara.javascript_driver`(`:selenium` がデフォルト)に切り替えることができます。

```ruby
@javascript
Scenario: do something Ajaxy
  When I click the Ajax link
  ...
```

明示的にドライバを指定する`@selenium`や`@rack_test`タグも用意されています。

## Capybara を RSpec と使う

下記の上を追加して(通常は`spec_helper.rb`ファイル)、RSpec 3.5 以上用のコードをロードしましょう。

```ruby
require 'capybara/rspec'
```

もしあなたが Rails を使っているならば、Capybara の spec は（[RSpec を使っている場合]((https://relishapp.com/rspec/rspec-rails/v/4-0/docs/directory-structure))） `spec/features` か `spec/system` に置きましょう。
もしあなたが別のディレクトリに Capybara の spec を置いているなら、書いているテストの種類に応じて example groups に `type: :feature` または `type: :system` を追加してください。

もしあなたが Rails の system spec を使っているなら、使用するドライバを選ぶために[ドキュメント](https://relishapp.com/rspec/rspec-rails/docs/system-specs/system-spec#system-specs-driven-by-selenium-chrome-headless)を見てください。
もしあなたが Rails を使っていないのならば、Capybara を使いたい全ての example groups に `type: :feature` を追加する必要があります。

下記のように spec を書けます。

```ruby
describe "the signin process", type: :feature do
  before :each do
    User.create(email: 'user@example.com', password: 'password')
  end

  it "signs me in" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', :with => 'user@example.com'
      fill_in 'Password', :with => 'password'
    end
    click_button 'Sign in'
    expect(page).to have_content 'Success'
  end
end
```

`js: true` を使うことで、ドライバを`Capybara.javascript_driver`(`:selenium` がデフォルト)に切り替えることができます。もしくは`:driver`オプションを使うと特定のドライバに切り替えることができます。

例

```ruby
describe 'some stuff which requires js', js: true do
  it 'will use the default js driver'
  it 'will switch to one specific driver', driver: :apparition
end
```

Capybara には受け入れテスト記述用のDSLもあります。

```ruby
feature "Signing in" do
  background do
    User.create(:email => 'user@example.com', :password => 'caplin')
  end

  scenario "Signing in with correct credentials" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', :with => 'user@example.com'
      fill_in 'Password', :with => 'caplin'
    end
    click_link 'Sign in'
    expect(page).to have_content 'Success'
  end

  given(:other_user) { User.create(:email => 'other@example.com', :password => 'rous') }

  scenario "Signing in as another user" do
    visit '/sessions/new'
    within("#session") do
      fill_in 'Email', :with => other_user.email
      fill_in 'Password', :with => other_user.password
    end
    click_link 'Sign in'
    expect(page).to have_content 'Invalid email or password'
  end
end
```


`feature` は実は単なる `describe ... type: :feature` のエイリアスです。`background` は `before` 、 `scenario` は `it`、`given` と `given!` は それぞれ `let` と `let!` のエイリアスです。

最後に、view spec 用の Capybara マッチャ もあります。

```ruby
RSpec.describe "todos/show.html.erb", type: :view do
  it "displays the todo title" do
    assign :todo, Todo.new(title: "Buy milk")

    render

    expect(rendered).to have_css("header h1", text: "Buy milk")
  end
end
```

**注釈: 「capybara/rspec」を require する場合、Capybara::DSL メソッドである `all`/`within` と同じ名前のビルトイン RSpec マッチャ間の名前衝突を避けるために、いくつかのプロキシメソッドがインストールされます。**
**「capybara/rspec」を require しないのなら、RSpec と「capybara/dsl」を require した後に「capybara/rspec/matcher_proxies」を require することで、名前衝突回避用のプロキシメソッドをインストールできます。**

## Capybara を Test::Unit と使う

`Test::Unit` を使っているなら、Capybara テスト用のベースとなるクラスを下記のように定義します。

```ruby
class CapybaraTestCase < Test::Unit::TestCase
  include Capybara::DSL

  def teardown
    Capybara.reset_sessions!
    Capybara.use_default_driver
  end
end
```

## Capybara を Minitest と使う

* もしあなたが Rails のシステムテストを使っているなら、使用するドライバを選ぶためにドキュメントを見てください。

* もしあなたが Rails を使っているにもかかわらず、Rails のシステムテストを使っていないのなら、下記のコードを `test_helper.rb` に追加しましょう。
`ActionDispatch::IntegrationTest`を継承している全てのテストで Capybara が利用可能になります。


```ruby
require 'capybara/rails'
require 'capybara/minitest'

class ActionDispatch::IntegrationTest
  # 全てのテストで Capybara DSL を利用可能にします
  include Capybara::DSL
  # `assert_*` メソッドが Minitest assertions のように振る舞うようにします
  include Capybara::Minitest::Assertions

  # テスト間のセッションとドライバをリセットします
  teardown do
    Capybara.reset_sessions!
    Capybara.use_default_driver
  end
end
```


* もしあなたが Rails を使っていないのなら、Capybara のテスト用のベースとなるクラスを下記のように定義します。

```ruby
require 'capybara/minitest'

class CapybaraTestCase < Minitest::Test
  include Capybara::DSL
  include Capybara::Minitest::Assertions

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

## Capybara を Minitest::Spec と使う

Minitest 用の上記手順に従った上で、さらに capybar/minitest/spec を require します。

```ruby
page.must_have_content('Important!')
```

## ドライバ

Capybara では、同じ DSL で様々なブラウザや headless なドライバを扱うことができます。

### ドライバを選択する

デフォルトでは、Capybara は `:rack_test` ドライバを使います。`:rack_test` は速いけれど、JavaScript をサポートしていないのと、Rack アプリの外側からリモート API や OAuth サービスのような HTTP リソースにアクセス出来ないという制限があります。これらの制限を回避するには、欲しい機能に合わせて別のデフォルトドライバを設定します。例えばもし Selenium で全てのテストを走らせたければ、下記のように書けます。

```ruby
Capybara.default_driver = :selenium # :selenium_chrome と :selenium_chrome_headless も登録されます
```

しかし、もしあなたが Rspec か Cucumber を使っているのなら（そして JS なしであなたのアプリが正しく動作しているのなら）、より速い `:rack_test` を default_driver のままにしておいて、JavaScript を扱う必要のあるテストにだけ それぞれ `js: true` もしくは `@javascript` でマークを付けることもできます。デフォルトでは、JavaScript のテストは `:selenium` ドライバが使われます。`Capybara.javascript_driver`に代入することで変更可能です。

ドライバを一時的に変更することもできます(一般的に Before/setup と After/teardown ブロックで使われます)

```ruby
Capybara.current_driver = :apparition # 一時的に別のドライバを設定
# ここでテストする
Capybara.use_default_driver       # デフォルトドライバに戻す
```

**注釈:** ドライバの変更は新しい session を作ります。なのでテストの途中で変更することはできません。

### RackTest

RackTest は Capybara のデフォルトドライバです。これは pure Ruby で書かれていて、JavaScript の実行はサポートしていません。RackTest ドライバは直接 Rack のインタフェースに作用するので、テスト用のサーバーを立ち上げる必要はありません。つまり Rack アプリ(Rails や Sinatra、他のほとんどの Ruby フレームワークは Rack アプリです)でなければ、このドライバを使うことは出来ないということです。さらには、 RackTest ドライバはリモートのアプリのテストにも使えませんし、アプリが扱うリモートの URL にアクセスすることもできません(例: 外部サイトへのリダイレクト、外部 API、OAuth サービス)

[capybara-mechanize](https://github.com/jeroenvandijk/capybara-mechanize) は、RackTest に似ていて、リモートのサーバにアクセス可能なドライバを提供しています。

RackTest はこのように設定することが出来ます。

```ruby
Capybara.register_driver :rack_test do |app|
  Capybara::RackTest::Driver.new(app, headers: { 'HTTP_USER_AGENT' => 'Capybara' })
end
```

詳しくは "Configuring and adding drivers" のセクションを見てください。

### Selenium

現在、Capybara は [Selenium 3.5以上(Webdriver)](http://docs.seleniumhq.org/docs/01_introducing_selenium.jsp#selenium-2-aka-selenium-webdriver) をサポートしています。Selenium を利用するためには、`selenium-webdriver` gem をインストールする必要があります。bundler を使っているのなら Gemfile にそれを追加しましょう。

Capybara は Selenium を使用する名前付きのドライバを数多く用意しています。それらが以下です:

* :selenium                 => Firefox を動作させる Selenium
* :selenium_headless        => headless な設定で Firefox を動作させる Selenium
* :selenium_chrome          => Chrome を動作させる Selenium
* :selenium_chrome_headless => headless な設定で Chrome を動作させる Selenium

これらはローカルデスクトップにおける設定 (関連するソフトウェアのインストールを含む) では動作するはずですが、追加のオプションをブラウザに渡す必要のある CI 環境で使用する場合、カスタマイズが必要になることがあります。

**注釈:** 異なるスレッドでサーバを動かす種類のドライバは、テスト中で同一のトランザクションを共有することができません。それによりテストとテストサーバ感でデータを共有できなくなります。[トランザクションとデータベースのセットアップ](#transactions-and-database-setup)の章を読んでください。

### Apparition

[Apparition ドライバ]((https://github.com/twalpole/apparition))は、ヘッドレスな設定でもそうでない設定でも Chrome を使ってテストを実行できる新しいドライバです。モダン JS/CSS を使えるようにしつつ、[Poltergeist ドライバ API](https://github.com/teampoltergeist/poltergeist) および [capybara-webkit API](https://github.com/thoughtbot/capybara-webkit)との下位互換性を提供しています。CDP を使用して Chrome と通信するため、chromedriver が不要になります。このドライバは Capybara の現在の開発者によって開発されており、最新のカピバラのリリースに追従するよう努められています。v1.0になったら teamcapybara のリポジトリにおそらく移動されるでしょう。

## DSL

完全なリファレンスは [rubydoc.info](http://rubydoc.info/github/jnicklas/capybara/master) で利用可能です。

**注釈: デフォルトでは、Capybara は visible 要素のみを検索します。実際のユーザーは目に見えない要素を操作することができないためです。**

**注釈:** Capybara での全ての検索は case sensitive です。これは、case insensivity をサポートしていない XPath を大量に使っているためです

### Navigating

他のページに遷移するメソッドとして<tt>[visit](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Session#visit-instance_method)</tt>が使えます。

```ruby
visit('/projects')
visit(post_comments_path(post))
```

visit メソッドは引数を一つだけ取り、リクエストに使用するメソッドは **常に** GET です。

**注釈**: `current_path` の値を直接テストすることで、現在のパスを assert することもできます。しかし、 `have_current_path` マッチャを使う方が、前のアクション (`click_link` など) が完了していることを保証するCapybara の[待機動作](#asynchronous-javascript-ajax-and-friends)を利用するため、より安全です。

```ruby
expect(current_path).to eq(post_comments_path(post))
```

### Clicking links and buttons

フルのリファレンスはこちら: [Capybara::Node::Actions](http://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Actions)

リンクやボタンを押してwebアプリと通信することが出来ます。Capybara は自動でリダイレクトに対応し、ボタンに関連するフォームをサブミットします。

```ruby
click_link('id-of-link')
click_link('Link Text')
click_button('Save')
click_on('Link Text') # リンクかボタンどちらかをクリック
click_on('Button Value')
```

### Interacting with forms

フルのリファレンスはこちら: [Capybara::Node::Actions](http://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Actions)

フォーム要素を操作するツールがたくさんあります。

```ruby
fill_in('First Name', :with => 'John')
fill_in('Password', :with => 'Seekrit')
fill_in('Description', :with => 'Really Long Text...')
choose('A Radio Button')
check('A Checkbox')
uncheck('A Checkbox')
attach_file('Image', '/path/to/image.jpg')
select('Option', from: 'Select Box')
```

### Querying

フルのリファレンスはこちら: [Capybara::Node::Matchers](http://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers)

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

フルのリファレンスはこちら: [Capybara::Node::Finders](http://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Finders)

特定の要素を探して操作することが出来ます。

```ruby
find_field('First Name').value
find_field(id: 'my_field').value
find_link('Hello', :visible => :all).visible?
find_link(class: ['some_class', 'some_other_class'], :visible => :all).visible?

find_button('Send').click
find_button(value: '1234').click

find(:xpath, "//table/tr").click
find("#overlay").find("h1").click
all('a').each { |a| a[:href] }
```

追加の属性/プロパティで要素を検索する必要がある場合は、フィルター ブロックを渡すこともできます。これは、通常の待機動作内でチェックされます。
これを頻繁に使用する必要がある場合は、[カスタム セレクター](http://www.rubydoc.info/github/teamcapybara/capybara/Capybara#add_selector-class_method) を追加するか、[既存のセレクタにフィルターを追加する](http://www.rubydoc.info/github/teamcapybara/capybara/Capybara#modify_selector-class_method)のが良いでしょう。

```ruby
find_field('名'){ |el| el['データ-xyz'] == '123'}
find("#img_loading"){ |img| img['complete'] == true }
```

**注釈:** `find` は、Ajax の章で説明されているように、要素がページに表示されるのを待ちます。もし要素が現れなければエラーを吐きます。

これらの要素は Capybara の DSL メソッドを全て使え、ページの箇所を特定して DSL の作用する場所を制限することが出来ます。

```ruby
find('#navigation').click_link('Home')
expect(find('#navigation')).to have_button('Sign out')
```

### Scoping

Capybara は form 操作やリンクやボタンのクリックなどの特定のアクションを、ページの特定のエリア内で行うように制限することが可能です。<tt>[within](http://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session#within-instance_method)</tt> メソッドを使うことでそれができ、オプションでセレクタの種類を特定することが出来ます。

```ruby
within("li#employee") do
  fill_in 'Name', with: 'Jimmy'
end

within(:xpath, "//li[@id='employee']") do
  fill_in 'Name', with: 'Jimmy'
end
```

fieldset や table 用の特別なメソッドがあります。fieldset は legend タグ内の id かテキスト、table は caption タグの id かテキストを見ます。

```ruby
within_fieldset('Employee') do
  fill_in 'Name', with: 'Jimmy'
end

within_table('Employee') do
  fill_in 'Name', with: 'Jimmy'
end
```

### Working with windows

Capybara には、ウィンドウの検索と切り替えを簡単にする方法がいくつか用意されています。

```ruby
facebook_window = window_opened_by do
  click_button 'Like'
end
within_window facebook_window do
  find('#login_email').set('a@example.com')
  find('#login_password').set('qwerty')
  click_button 'Submit'
end
```

### Scripting

ドライバがサポートしていれば、簡単に JavaScript を実行できます。

```ruby
page.execute_script("$('body').empty()")
```

簡単な式であれば、script の結果を受け取ることが出来ます。

```ruby
result = page.evaluate_script('4 + 4');
```

より複雑な script の場合は、それらを 1 つの式として記述する必要があります。

```ruby
result = page.evaluate_script(<<~JS, 3, element)
  (function(n, el){
    var val = parseInt(el.value, 10);
    return n+val;
  })(arguments[0], arguments[1])
JS
```

### Modals

ドライバーがサポートしていれば、アラート、確認、プロンプトに対して、受け入れ(accept)、無視(dismiss)、応答(respond)することができます。

アラートを生成するコードをブロックでラップすることにより、アラート メッセージを受け入れ(accept)、無視(dismiss)できます。

```ruby
accept_alert do
  click_link('Show Alert')
end
```

確認のモーダルを生成するコードをブロックでラップすることで、確認を受け入れ(accept)、無視(dismiss)できます。

```ruby
dismiss_confirm do
  click_link('Show Confirm')
end
```

プロンプトを受け入れ(accept)、無視(dismiss)できます。また、応答のモーダルに入力するテキストを提供することもできます。

```ruby
accept_prompt(with: 'Linus Torvalds') do
  click_link('Show Prompt About Linux')
end
```

すべてのモーダルメソッドは、提示されたメッセージを返します。したがって、戻り値を変数に代入することによりプロンプトメッセージにアクセスできます:

```ruby
message = accept_prompt(with: 'Linus Torvalds') do
  click_link('Show Prompt About Linux')
end
expect(message).to eq('Who is the chief architect of Linux?')
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

スクリーンショットは、アプリのディレクトリからの相対パスである `Capybara.save_path` に保存されます。
`capybara/rails` を require している場合、 `Capybara.save_path` はデフォルトで `tmp/capybara` になります。

## Matching

Capybara の要素の見つけ方をカスタマイズすることができます。`Capybara.exact` と `Capybara.match` の二つの選択肢があります。

### Exactness

`Capybara.exact` と `exact` オプションは XPath gem の内部で使われている `is` に影響します。`exact` が true のとき、全ての `is` 式が完全一致になります。 false のときは部分一致を許可します。Capybaraに組み込まれている多くのセレクタが `is` を使用しています。 この方法で部分一致を許可するかどうか指定できます。デフォルトでは `Capybara.exact` は false です。

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

**注釈:** Rails 5.1 以上では、アプリとテストスレッドの間でデータベース接続を「安全に」共有します。したがって、もしあなたが Rails 5.1 以降を使用しているのなら、このセクションを無視してよいはずです。

いくつかの Capybara のドライバは実際の HTTP サーバに対して動きます。Capybara はこれに対応しており、テストプロセスと同一のプロセス(ただし別スレッド)で HTTP サーバを走らせます。Selenium は HTTP サーバを使うドライバの一つです。RackTest は違います。

もしあなたが SQL データベースを使っているのなら、全てのテストを1つのトランザクション内で実行し、そのトランザクションをテストの終わりにロールバックする方法が一般的です。例えば rspec-rails では、デフォルトでこの方法が採用されています。トランザクションは通常、スレッド間で共有できないので、もし HTTP サーバを使用するドライバを使っているのなら、テストコード上であなたがデータベースに入れたデータは Capybara から見ることができません。

Cucumber はトランザクションの代わりに truncation を使うことでこの問題に対応しています。すなわち、それぞれのテストの後に全てのデータベースを空にしているのです。[database_cleaner](https://github.com/DatabaseCleaner/database_cleaner) のような gem を使うことで、同じような振る舞いをさせることができます。

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
Capybara.default_max_wait_time = 5
```

この振る舞いがあることによって気をつけて欲しいことがあります。下記の2つの命令は同じでは **ありません** 。 **常に** 後者を使うようにしてください！

```ruby
# `visit` の結果ページが読み込まれるドライバを使っているとします
# Capybara.predicates_wait は `true` で
# 1秒後に AJAX によって `a` タグが削除されるページを考えてみてください
visit(some_path)
!page.has_xpath?('a')  # is false
page.has_no_xpath?('a')  # is true
```

最初の式:

* `has_xpath?('a')` は、`visit` が戻り値を返した直後に呼び出されます。リンクはまだ削除されていないため、`true` になります。
* Capybara は成功した predicates/assertions を待たないため、**`has_xpath?` はすぐに `true` を返します**。
* 式は `false` を返します (先頭の `!` で否定されているため)。

2 番目の式:

* `has_no_xpath?('a')` は、`visit` が戻り値を返した直後に呼び出されます。リンクがまだ削除されていないため、`false`になります。
* Capybara は失敗した predicates/assertions を待つため、**`has_no_xpath?` はすぐに `false` を返しません**。
* Capybara は、定義された `default_max_wait_time` の秒数の間、 predicate/assertion を繰り返し再チェックします。
* 1秒後、predicate（この場合 `has_no_xpath?('a')` ）は `true` に変わります (リンクが削除されたため)。
* 式は `true` を返します。

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

通常、Capybara は同一プロセス内の Rack アプリをテストすることを想定しています。ただ、`app_host` に値を設定することで、インターネット上の別の場所にある web サーバとやりとりすることもできます。

```ruby
Capybara.current_driver = :selenium
Capybara.app_host = 'http://www.google.com'
...
visit('/')
```

**注釈:** デフォルトドライバ(`:rack_test`)はリモートサーバに対応していません。対応しているドライバであれば、指定したURLに直接アクセスすることができます。

```ruby
visit('http://www.google.com')
```

デフォルトでは、Capybara は自動的に Rack アプリを起動しようとします。リモートでアプリを動かしているなら、Capybara の Rack サーバをオフにしたくなるかもしれません。

```ruby
Capybara.run_server = false
```

## Using the sessions manually

Capybara は名前付きセッション (指定されていない場合は `:default`) を扱うことができ、複数のセッションが同じドライバとテストアプリのインスタンスを使って通信できるようにします。もし今使用しているドライバとテストアプリのインスタンスを使用している特定の名前のセッションが見つからない場合、今使用しているドライバを使用して新しいセッションが作成されます。

### Named sessions

別のセッションで操作を行って、前のセッションに戻ってくるには下記のようにします。

```ruby
Capybara.using_session("Bob's session") do
  # Bob のブラウザセッションで何かする
end
# 前のセッションに戻ってくる
```

永久的に現在のセッションを別のセッションに切り替えるには下記のようにします。

```ruby
Capybara.session_name = "some other session"
```

### Using sessions manually

[Session](http://rubydoc.info/github/jnicklas/capybara/master/Capybara/Session) を手動でインスタンス化して扱うことができます。

```ruby
require 'capybara'

session = Capybara::Session.new(:webkit, my_rack_app)
session.within("form#session") do
  session.fill_in 'Email', :with => 'user@example.com'
  session.fill_in 'Password', :with => 'password'
end
session.click_button 'Sign in'
```

## XPath, CSS and selectors

Capybara は入力したセレクタの種類を推測するようなことはしません。デフォルトでは常に CSS を使用します。もし XPath が使いたいのであれば、下記のようにする必要があります。

```ruby
within(:xpath, './/ul/li') { ... }
find(:xpath, './/ul/li').text
find(:xpath, './/li[contains(.//a[@href = "#"]/text(), "foo")]').value
```

もしくはデフォルトのセレクタを XPath にすることもできます。

```ruby
Capybara.default_selector = :xpath
find('.//ul/li').text
```

Capybara には、他にも多くのビルトインセレクタ型が用意されています。ビルトインセレクタの全一覧とそれに適用できるフィルタは、[build-in selectors](https://www.rubydoc.info/github/teamcapybara/capybara/Capybara/Selector)で確認できます。

Capybara はカスタムセレクタを追加することもできます。これは同じようなセレクタをよく使っている場合に便利です。下記の例はとてもシンプルですが、ここに示している以外にも多くの機能が利用できます。より詳しい例については Capybara のビルトインセレクタの定義を参照してください。

```ruby
Capybara.add_selector(:my_attribute) do
  xpath { |id| XPath.descendant[XPath.attr(:my_attribute) == id.to_s] }
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
find(:my_attribute, 'post_123') # マッチする属性を持つ要素を見つける
find(:row, 3) # table row の body の 3番目の row を見つける
find(:flash_type, :notice) # 'flash' という id を持ち、 'notice' という class を持つ要素を見つける
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
# 注釈: Capybara はこれをデフォルトで登録します。
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
* Rails の統合テストを使っていないので、Rails 特有のもの(例: `controller`)へアクセスすることは出来ません。
* 時間の固定: 現在時刻に依存したフィーチャをうまく動かすために、モックを使うのは一般的なやりかたです。しかし、単調なプロセスの時刻へのアクセスをサポートがない ruby と platform の組み合わせにおいては問題が起こることがあります。それは Capybara の Ajax のタイミングがシステムの時間を使っているせいで、failure な時に Capybara はタイムアウトせずにハングってしまいます。時間を止める機能でなく、時間を移動させる機能であればうまくいきます。[Timecop](https://github.com/travisjeffery/timecop) は両方の機能が使えます。
* Rack::Test を使っているときは、URL で visit しているかどうかチェックするべきです。たとえば、session は `posts_path` と `posts_url` で別々になります。もし Action Mailer の中で絶対 URL を使っているとしたら、`default_url_options` を Rails のデフォルトの `www.example.com` に設定しましょう。
* サーバーのエラーは、サーバースレッドを開始するセッションでのみ発生します。もしあなたが複数のセッションを使用して特定のサーバーエラーをテストしているのなら、最初のセッションを使用したエラー (通常は `:default`)に対するテストになっているかどうかを確認してください。
* WebMock が有効になっている場合、「開いているファイルが多すぎます」というエラーが表示されることがあります。単純な `page.find` 呼び出しの結果、タイムアウトが起こるまで何千もの HTTP リクエストが発生するかもしれません。デフォルトでは、WebMock は新しいコネクションを生成するごとにリクエストを送信します。この問題を回避するには、[WebMock の `net_http_connect_on_start: true` パラメータを有効にする](https://github.com/bblimke/webmock/blob/master/README.md#connecting-on-nethttpstart)必要があるかもしれません。

## "Threadsafe" mode

通常モードでは、Capybara の設定オプションのほとんどはグローバルに設定されるため、複数のセッションを使用していて、1つのセッションのみの設定を変更したいような場合に、問​​題を引き起こす可能性があります。提供する
このような例をサポートするため、 Capybara は、"スレッドセーフ"モードを提供するようになりました。下記のようにすることで有効になります。

```ruby
Capybara.threadsafe = true
```

この設定は、セッションが作られる前にのみ変更できます。"スレッドセーフ"モードでは、下記のように Capybara の振る舞いが変化します。

* ほとんどのオプションはセッションごとに設定できるようになりました。これらはセッションの作成時または作成後に設定できます。セッション作成時点でのグローバルな各オプションはデフォルトのままです。セッション固有 **ではない** オプションは次のとおりです。`app`、`reuse_server`、`default_driver`、`javascript_driver`、`threadsafe`。`register_driver` と `register_server` によって登録されたドライバとサーバーもグローバルになります。

```ruby
my_session = Capybara::Session.new(:driver, some_app) do |config|
  config.automatic_label_click = true # my_session のみに設定
end

my_session.config.default_max_wait_time = 10 # my_session のみに設定
Capybara.default_max_wait_time = 2 # my_session の default_max_wait を変更しない
```

* `current_driver` と `session_name` はスレッド固有です。つまり、 `using_session` と
 `using_driver` も現在のスレッドにのみ影響するオプションということになります。

## Development

開発環境をセットアップするには、下記のようにします。

```sh
bundle install
bundle exec rake  # Firefox でテストスイートを実行します - `geckodriver` をインストールする必要があります
bundle exec rake spec_chrome # Chrome でテストスイートを実行します - `chromedriver` をインストールする必要があります
```

issue や pull request の送り方は [CONTRIBUTING.md](https://github.com/jnicklas/capybara/blob/master/CONTRIBUTING.md) に書いてあります。
