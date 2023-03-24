# factory

## テンプレ
```rb
FactoryBot.define do
  factory :user do
    sequence(:name) { |n| "name_#{n}" }
    sex { 1 } # model では enum で管理. User.sexes[:man] でも可

    trait :man do
      sex { 1 }
    end

    trait :woman do
      sex { 2 }
    end

    trait :not_known_sex do
      sex { 0 }
    end

    trait :with_taikai do # 退会時に退会レコードが紐づく設計
      transient do
        taikai_at { Time.current }
      end

      after(:build) do |user, evaluator|
        user.user_taikai = build(:user_withdraw, taikai_at: evaluator.taikai_at)
      end
    end
  end
end
```


## セットする値
ユーザー名などのユニーク制限がかかっていないが固有名詞的なものは `sequence` を用いる


## association
belongs_to で宣言されているものは、初期から定義する  
has_xxx 系は後述の trait を使用する


## 値
DB で null: false になっているものを定義する


## trait
特定の条件がある場合 trait を用いて宣言する


## transient
has_many 等の子要素先の値を定義したい時などに使用すると便利  
