# system spec

## テンプレ
```rb
RSpec.describe 'Users', js: true do
  describe 'modal' do
    before do
      visit root_path
      click_link 'Sign Up' # Sign Up リンクをクリックすることでモーダルで Sign Up 画面が表示される
    end

    context '通常' do
      it '登録できること' do
        fill_in 'Email', with: 'hoge@example.com'
        fill_in 'Password', with: 'password'
        click_button 'Sign Up'
        expect(page).to have_content('welcome!')
      end
    end

    context 'すでに登録されているメールアドレスの場合' do
      before { create(:use, email: 'hoge@example.com') }

      it 'メールアドレス登録重複のエラーが表示されること' do
        fill_in 'Email', with: 'hoge@example.com'
        fill_in 'Password', with: 'password1'
        click_button 'Sign Up'
        expect(page).to have_content('Email has already been taken') # モーダル内にエラー文言が表示される
        # より厳密にする場合 have_selector などで modal 内を探索する
      end
    end
  end
end
```

サンプルなのでセキュリティ云々については触れないで下さい


## 対象
UI を対象としたテストを行う  
ブラウザの挙動をシミュレートさせるため、人間から見た振る舞いをテストしたい場合に使用する  

例
* ボタンを押したら開くアコーディオンメニューが、規定通り動くかどうか
* 特定条件化のみ画面に表示されるよう css で制御された項目が、規定通り動くかどうか
* ajax 等を用いて非同期に画面が描画されるもの
