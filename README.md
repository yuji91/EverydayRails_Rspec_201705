Sample Rails 4.1.x application for [Everyday Rails Testing with RSpec: A Practical Approach to Test-driven Development](https://leanpub.com/everydayrailsrspec) by Aaron Sumner. This repository demonstrates incremental testing of an existing application, starting with an untested codebase and working through model, controller, and feature specs.

Each chapter's progress has a specific branch in this repository. See chapter 1 of the book for details.

Using Git, you can check out each version by name. See details in the book.

If you're not comfortable with Git, you can also use GitHub's handy branch/tag filter to select a specific tag and browse the source code online. To learn more about Git, I recommend the free resources [Git Immersion](http://gitimmersion.com/) or [Try Git](http://www.codeschool.com/courses/try-git).

---
以下、Qiitaへの投稿
##概要
Cloud9でEveryday Rails Rspecの開発環境を構築する

##目標
rspecテストを実行したときに failures が 0 件になること

##環境
- ruby  2.3.3p222
- rails 4.1.1
- rspec 3.1.0
- rake 12.1.0

##注意点
ruby 2.4.0, Selenium-WebDriverは使わない
feature specではphantomjsを使用する

##手順
1. Cloud9でWorkSpace作成
2. SetUp - bundle update, rspec実行
3. warning解消
4. failure解消
5. rails server

###1. Cloud9でWorkSpace作成、rspec実行
[Clone from Git or Mercurial URL]
https://github.com/everydayrails/rails-4-1-rspec-3-0

[Template]
Ruby

※最新のリポジトリは下記 (2017年時点)
https://github.com/everydayrails/everydayrails-rspec-2017

###2.  SetUp - bundle update, rspec実行

```bash:環境のSetUp
１．Cloud9のWorkspace起動後に下記の警告文が表示される（2017/10現在）
ruby-2.1.1 is not installed.
To install do: 'rvm install ruby-2.1.1'

そのまま2.1.1をインストールしてもよいが、Verが古いので2.3.3を入れる
#rvmでインストール可能なVer一覧を確認
rvm list known
rvm install ruby-2.3.3
rvm --default use ruby-2.3.3

２．bundler（Rubyのライブラリ管理ツール）をインストールする
gem install bundler

この時点で bundle install/update すると下記エラーが発生するのでruby_depを入れる
Gem::InstallError: ruby_dep requires Ruby version >= 2.2.5, ~> 2.2.
An error occurred while installing ruby_dep (1.5.0), and Bundler cannot continue.
Make sure that `gem install ruby_dep -v '1.5.0'` succeeds before bundling.

gem install ruby_dep -v '1.5.0'
bundle update

#rvmの切り替えや必要なgemをインストール後、rspecテストプログラムを実行
rspec
```
```bash:rspec実行結果（修正前）
:~/workspace (master) $ rspec

/usr/local/rvm/gems/ruby-2.3.0/gems/activesupport-4.1.1/lib/active_support/values/time_zone.rb:285: warning: circular argument reference - now
RubyDep: WARNING: Your Ruby is outdated/buggy.
RubyDep: WARNING: Your Ruby is: 2.3.0 (buggy). Recommendation: upgrade to 2.3.1.
RubyDep: WARNING: (To disable warnings, see:http://github.com/e2/ruby_dep/wiki/Disabling-warnings )

ContactsController
  administrator access
    behaves like public access to contacts
      GET #index
        with params[:letter]
/usr/local/rvm/gems/ruby-2.3.0/gems/activerecord-4.1.1/lib/active_record/associations/has_many_association.rb:74: warning: circular argument reference - reflection
/usr/local/rvm/gems/ruby-2.3.0/gems/activerecord-4.1.1/lib/active_record/associations/has_many_association.rb:78: warning: circular argument reference - reflection
/usr/local/rvm/gems/ruby-2.3.0/gems/activerecord-4.1.1/lib/active_record/associations/has_many_association.rb:82: warning: circular argument reference - reflection
/usr/local/rvm/gems/ruby-2.3.0/gems/activerecord-4.1.1/lib/active_record/associations/has_many_association.rb:101: warning: circular argument reference - reflection
：
（中略）
：
About BigCo modal

  An error occurred in an after hook
    Selenium::WebDriver::Error::WebDriverError: Could not find Firefox binary (os=linux). Make sure Firefox is installed or set the path manually with Selenium::WebDriver::Firefox::Binary.path=
    occurred at /usr/local/rvm/gems/ruby-2.3.0/gems/selenium-webdriver-2.43.0/lib/selenium/webdriver/firefox/binary.rb:127:in `path`

  toggles display of the modal about display (FAILED - 3)
：
：
：
Failures:

  1) ContactsController administrator access behaves like full access to contacts PATCH #update invalid attributes does not change the contact's attributes
     'Failure/Error: expect(assigns(:contact).reload.attributes).to eq contact.attributes'
：
：
：
'82 examples, 5 failures, 1 pending'
```
複数のwarningとfailureが出てしまう
この記事では上記問題の解消手順を記載する


###3. warning解消

3-1. `warning: circular argument reference - now` の修正
3-2. `warning: circular argument reference - reflection` の修正

###3-1. `warning: circular argument reference - now` の修正
/usr/local/rvm/gems/ruby-2.3.3/gems/activesupport-4.1.1/lib/active_support/values/time_zone.rb

```ruby:time_zone.rbの修正
vi /usr/local/rvm/gems/ruby-2.3.3/gems/activesupport-4.1.1/lib/active_support/values/time_zone.rb

[Line285] def parse(str, now=now)
      --> def parse(str, now=now())
```

###3-2. `warning: circular argument reference - reflection` の修正

```ruby:has_many_association.rbの修正
vi /usr/local/rvm/gems/ruby-2.3.3/gems/activerecord-4.1.1/lib/active_record/associations/has_many_association.rb


[Line 74] def has_cached_counter?(reflection = reflection)
      --> def has_cached_counter?(reflection = reflection())

[Line 78] def cached_counter_attribute_name(reflection = reflection)
      --> def cached_counter_attribute_name(reflection = reflection())

[Line 82] def update_counter(difference, reflection = reflection)
      --> def update_counter(difference, reflection = reflection())

[Line101] def inverse_updates_counter_cache?(reflection = reflection)
      --> def inverse_updates_counter_cache?(reflection = reflection())
```

###4. failuer解消
4-1. `Selenium::WebDriver::Error::WebDriverError `
4-2. `expect(assigns(:contact).reload.attributes).to eq contact.attributes `

###4-1.`Selenium::WebDriver::Error::WebDriverError `
```bash:実行コマンド
gem uninstall selenium-webdriver
```
```ruby:GemFileを修正
group :test do
  gem "faker", "~> 1.4.3"
  gem "capybara", "~> 2.4.3"
  gem "database_cleaner", "~> 1.3.0"
  gem "launchy", "~> 2.4.2"
  #下記のgemを削除
  #gem "selenium-webdriver", "~> 2.43.0"
  #下記のgemを追加
  gem 'poltergeist'
  gem 'shoulda-matchers', '~> 2.6.2'
end
```



Selenium::WebDriver::Error::WebDriverError: Could not find Firefox binary (os=linux). Make sure Firefox is installed or set the path manually with Selenium::WebDriver::Firefox::Binary.path=

```bash:実行コマンド
sudo apt-get update
sudo apt-get install xvfb -y
sudo apt-get install firefox xvfb -y

sudo npm install -g phantomjs
```

上記PackageをInstallするとSeleniumが起動できるようになるが、
Rspec実行中にブラウザテストの段階で処理が止まってしまう

```ruby:rails_helper.rb
c9 open /home/ubuntu/workspace/spec/rails_helper.rb  

require 'capybara/poltergeist'
Capybara.javascript_driver = :poltergeist
```
[参考]https://github.com/teampoltergeist/poltergeist

###4-2.`expect(assigns(:contact).reload.attributes).to eq contact.attributes `

```ruby:rspec実行結果
Failures:

  1) ContactsController user access behaves like full access to contacts PATCH #update invalid attributes does not change the contact's attributes
     Failure/Error: expect(assigns(:contact).reload.attributes).to eq contact.attributes
       
       expected: {"id"=>2, "firstname"=>"Lawrence", "lastname"=>"Smith", "email"=>"jailyn@lemke.org", "created_at"=>Sun, 04 Jun 2017 11:55:02 UTC +00:00, "updated_at"=>Sun, 04 Jun 2017 11:55:02 UTC +00:00}
            got: {"id"=>2, "firstname"=>"Lawrence", "lastname"=>"Smith", "email"=>"jailyn@lemke.org", "created_at"=>Sun, 04 Jun 2017 11:55:02 UTC +00:00, "updated_at"=>Sun, 04 Jun 2017 11:55:02 UTC +00:00}
```
期待値と結果は同一に見えるが、timestampがmicrosecond以下で異なっている

```ruby:contacts_controller.rbの修正
vi /home/ubuntu/workspace/spec/controllers/contacts_controller_spec.rb

[Line 198] 
 it "does not change the contact's attributes" do
          expect(assigns(:contact).reload.attributes).to eq contact.attributes
FIX->     expect(assigns(:contact).reload.attributes).to eq contact.reload.attributes

```


[参考]https://github.com/everydayrails/rails-4-1-rspec-3-0/issues/48

###5. rspec実行
```bash:実行コマンド
cd ~/workspace
bundle update
rspec
```
```bash:rspec実行結果（修正後）
Finished in 8.17 seconds (files took 2.52 seconds to load)
82 examples, 0 failures, 1 pending
```
###6. 番外編、Gitでの操作
EverydayRailsの各単元は下記コマンドで確認できる
`git branch -l`
特定の単元についてのソースコードを取得したい場合は下記コマンドを実行する
`git pull [branch name]`

```bash:GitHubでリポジトリを作成後、下記コマンドを実行
#現在のRemoteBranchを確認
git remote -v
origin  git@github.com:everydayrails/rails-4-1-rspec-3-0.git (fetch)
origin  git@github.com:everydayrails/rails-4-1-rspec-3-0.git (push)

#現在のRemoteBranchを削除
git remote rm origin

#自分のRemoteBranchを追加
git remote add origin git@github.com:[YourGitName]/[GitRepoName].git
git remote -v
origin  git@github.com:[YourGitName]/[GitRepoName].git (fetch)
origin  git@github.com:[YourGitName]/[GitRepoName].git (push)
git add -A
git commit -m 'Fix Failuer with EverydayRails-Rspec'
git push -u origin master
```