# request spec

## テンプレ
```rb
RSpec.describe 'Books' do
  describe 'index' do
    it do
      visit books_path
      expect(response.status).to eq(200)
    end
  end

  describe 'create' do
    desctibe '未ログインの場合' do
      it 'アクセスできないこと' { }
    end

    desctibe 'ログイン済みの場合' do
      before do
        login(create(:user), no_capybara: true)

        # 単純な controller なら別にモックしないが、複雑な処理が行われる場合は、そこをテストをするわけでないのでモックする
        allow_any_instance_of(Book).to receive(:save).and_return(true)
      end

      context '保存成功' do
        it do
          post book_path, params: { title: 'xxx' }
          expect(response).to redirect_to book_path
          expect(Book).to have_received(:save).once # モックする場合コールされたかを検証する
        end
      end

      context '保存失敗' do
        it do
          post book_path, params: { title: '' }
          expect(response.body).to include('Book has 1 error.') # render :new されることを検証するので new であることを示す内容
        end
      end
    end
  end
end
```


## 対象
リクエストが正しく処理されたかをテストする  
request に対する応答をテストするので、以下のチェック内容に限定してテストすると見通しがよい


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
