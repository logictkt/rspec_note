# mailer spec

## テンプレ
```rb
RSpec.describe HogeMailer do
  describe 'hello' do
    let(:mail) { HogeMailer.with(to: 'hoge@example.com', name: 'Okada').hello }

    it do
      expect(mail.subject).to eq 'Mail subject'
      expect(mail.to).to eq ['hoge@example.com']
      expect(mail.body).to include 'Hello Okada san!'
    end
  end
end
```

## 対象
メーラーを対象としたテスト  
シンプル
