# request spec

## テンプレ
```rb
RSpec.describe 'Follows' do
  desctibe '未ログインの場合' do
    describe 'index' do
      it 'アクセスできないこと' do
        visit follows_path
        expect(response).to redirect_to root_path
        # or
        expect(response.status).to eq(403)
      end
    end
  end

  desctibe 'ログイン済みの場合' do
    let(:login_user) { create(:user, :with_auth) }
    let(:follow_user) { create(:user, :with_auth, name: 'John') }

    before { login(login_user, no_capybara: true) }

    describe 'index' do
      before { create(:artist_follow, artist:, user: follow_user) }

      it do
        visit follows_path
        expect(response.body).to include('John')
      end
    end

    describe 'create' do
      it do
        expect { post follow_path(follow_user) }.to change(Follow, :count).by(1)
      end
    end

    describe 'destroy' do
      before { create(:artist_follow, artist:, user: follow_user) }

      it do
        expect { delete follow_path(follow_user) }.to change(Follow, :count).by(-1)
      end
    end
  end
end
```


## 対象
リクエストが正しく処理されたかをテストする


## チェック内容
* 正しくレスポンス、リダイレクトされたか
* flash 等 controller でセットした内容が response.body に含まれているか


***


## 応用
リクエストから一歩先のデータが正しくできているかや、正しく表示されているかをテストする  
しかしこのテストを request spec で書きすぎると、リクエストの範疇を超えてしまうので書きすぎに注意  

より厳密にテストしたい場合 model spec や view spec でテストするのが良い  
しかし scaffold などでできるレベルのシンプルな構成の場合、下記のような形で request spec で他のテスト範囲をカバーすることもある


### 例
index, show などで正しくデータが渡されているか

```rb
def show
  @book = Book.find(params[:id])
end
```

```haml
%h1= @book.title
```

```rb
describe 'show' do
  let!(:book) { create(:book, title: 'Hoge') }

  it do
    get book_path(book)
    expect(response).to have_http_status(200)
    expect(response.body).to include 'Hoge'
  end
end
```


### 例 2
create, update などで正しくデータが保存されるか

```rb
def create
  @book = Book.new(book_params)
  if @book.save
    redirect_to root_path
  else
    render :new
  end
end
```

```rb
describe 'create' do
  it do
    post book_path, params: { title: 'Hoge' }
    expect(response).to redirect_to root_path
    expect(Book.count).to eq 1
  end
end
```
