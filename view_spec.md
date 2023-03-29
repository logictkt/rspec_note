# view spec

## テンプレ
```rb
describe 'books/show.html.haml' do
  describe '表示項目' do
    let(:book) { create(:book, title: 'お粥食べたい', isbn: 'ISBN111-1-1111111-1-1') }

    before do
      assign(:book, book)

      render template: 'books/show'
    end

    context '販売中の場合' do
      it 'すべての項目が表示されること' do
        expect(rendered).to include 'お粥食べたい'
        expect(rendered).to include 'ISBN111-1-1111111-1-1'
        expect(rendered).not_to include '品切れ'
      end
    end

    context '品切れの場合' do
      before { allow(book).to recieve(:sinagire?).and_return(true) }

      it '品切れ表記されること' do
        expect(rendered).not_to include '品切れ'
      end
    end
  end
end
```

## 対象
view ファイルを対象としたテスト  
表示されるべき項目のテストがメイン  

if などで表示の切り分けを行っている箇所や、表示はしないけれども view ファイルで定義しているもの ( meta tag など) のテストが主な対象
