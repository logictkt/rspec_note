# rspec_note

## descrive, context
* descrive → テスト対象
* context → パターン


## let, let!
どっちを使っても構わないが、let! を使用する方が脳内メモリーを使わなくて済む  
テスト対象及び、テスト中にデータを切り替えてテストしたい項目のみを定義する

```rb
# テスト対象を明確にする let
descrive 'validate' do
  let(:user) { build(:user) }

  it { expect(user).to be_valid }
end
```

```rb
# テストデータの切り替え
descrive 'validate' do
  let(:user) { build(:user, name:) }

  context 'name がない場合' do
    let(:name) { nil }

    it do
      expect(user).not_to be_valid
      expect(user.errors).to be_of_kind(:name, :blank)
    end
  end

  context 'name がある場合' do
    let(:name) { 'Tanaka' }

    it { expect(user).to be_valid }
  end
end

```


## subject
圧倒的に見やすい場合を除いて使用しない  
subject を利用する方が短くて済む場合もあるが、見やすさ読みやすさの観点から、subject を使用するより let を使用した方が良いコードになることが多い  
上記 let の上側の例は subject でも見やすいので可


## before
テスト的に必要な初期化処理や変数の定義などを行う

```rb
descrive 'ダウンロード履歴' do
  let(:user) { create(:user) }

  context '上限に達していない場合' do
    it { expect(user.download_ok?).to be_true }
  end

  context '上限に達している場合' do
    before { create_list(:download_log, 10, user:) }

    it 'ダウンロードが有効にならないこと' do
      expect(user.download_ok?).to be_false
    end
  end
end
```


## 値の共通化
let を使用した値の共通化は行わない  
before とインスタンス変数を使用する  
テスト対象でない場合に let を使用しない

before
```rb
descrive '' do
  let(:user) { create(:user) }

  descrive '' do
    let(:book) { create(:book, user:)

    it {}
  end

  descrive '' do
    let(:book) { create(:book, user:)

    it {}
  end
end
```

after
```rb
descrive '' do
  before { @user = create(:user) }

  descrive '' do
    let(:book) { create(:book, user: @user)
    it {}
  end

  descrive '' do
    let(:book) { create(:book, user: @user)
    it {}
  end
end
```


## it
descrive, context のみでテスト対象が明確である場合、it の名称は両略可とする

## factory
### セットする値
ユーザー名などのユニーク制限がかかっていないが固有名詞的なものは `sequence` を用いる

### association
belongs_to で宣言されているものは、初期から定義する

### 値
DB で null: false になっているものを定義する

### trait
特定の条件がある場合 trait を用いて宣言する

```rb
factory :user do
  name { 'Ebisu' }
  age { 20 }

  trait :miseinen do
    age { 10 }
  end
end
```

### transient
has_many 等の先の値を定義したい時などに使用すると便利

```ruby
factory :user do
  trait :with_book do
    transient do
      sequence(:title) { |n| "title_#{n}" }
    end

    after(:build) do |user, evaluator|
      build(:book, user:, title: evaluator.title)
    end
  end
end
```


## ポイント
### 無理な再利用しない
テスト内容的に再利用可能（共通定義）であっても、テスト対象・内容が明確に異なる場合（descriveが異なる）に、値の再利用を行わない

```rb
let(:user) { create(:user) )

descrive 'AAA' do
end

descrive 'BBB' do
end
```


### テスト区分を超えたテストをしない
例 : ユーザー名は大文字にコンバートして保存されるという仕様がある場合のそれぞれのテスト  
本来はモデルのインスタンスの処理 （セッター） とかに追加すればいいがサンプルなので許して

前提
```ruby
class User
  def self.name_upper name
    name.upcase
  end
end

class UsersController
  def create
    user = User.new(name: User.name_upper(params[:name))
    if user.save
      redirect_to root_path, notice: 'welcome!'
    else
      render :new
    end
  end
end
```

だめ
```rb
Rspec.system do
  descrive 'create' do
    before { post aaa_path(name: 'aaa') }

    it do
      expect(response).to redirect_to root_path
      expect(flash[:notice]).to eq 'welcome!'
      expect(User.last.name).to eq 'AAA'
    end
  end
end
```

いい
```rb
Rspec.system do
  descrive 'create' do
    before do
      allow(User).to recieve(name_upper).and_return(true)
      post aaa_path(name: 'aaa')
    end

    it do
      expect(response).to redirect_to root_path
      expect(flash[:notice]).to eq 'welcome!'
      expect(User).to have_received(:name_upper).once
    end
  end
end

Rspec.model do
  desctive 'name_upper' do
    it do
      expect(User.name_upper('hoge')).to eq 'HOGE'
    end
  end
end
```
