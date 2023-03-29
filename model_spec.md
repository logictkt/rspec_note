# model spec

## テンプレ
```rb
RSpec.describe Hoge do
  desctibe 'validates' do
    it { expect(build(:hoge).valid?).to be true }
  end

  describe 'scope' do
    describe 'division' do
      subject { Hoge.division(type) }

      let!(:fizz) { create(:hoge, :fizz) }
      let!(:buzz) { create(:hoge, :buzz) }

      context 'fizz の場合' do
        let(:type) { :fizz }

        it { is_expected.to contain_exactly fizz }
      end

      context 'buzz の場合' do
        let(:type) { :buzz }

        it { is_expected.to contain_exactly buzz }
      end
    end
  end

  describe 'クラスメソッドメソッド' do
  end

  describe 'インスタンスメソッド' do
  end
end
```

## 対象
モデルに定義しているバリデーションなどの設定や、メソッドの振る舞いをテストする  
モデルに定義している大まかな区分でまず describe を区切ると見やすい  

ポイント  
1 メソッド 1 describe  
1 設定 1 describe  


## 個人的なおすすめ
シンプルで何も中身が無いようなモデルで、下記のようなものだけは書いておく  
なにかイレギュラーなマージミス等でモデルがファイルごと壊れていることを検知できる

```rb
desctibe 'validates' do
  it { expect(build(:hoge).valid?).to be true }
end
```
