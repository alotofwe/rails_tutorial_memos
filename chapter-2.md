# Chapter 2 A toy app

> As discussed in Box 1.2, the rest of the book will take the opposite approach, developing a full sample application incrementally and explaining each new concept as it arises, but for a quick overview (and some instant gratification) there is no substitute for scaffolding.

> the whole point of this tutorial is to take you beyond this superficial, scaffold-driven approach to achieve a deeper understanding of Rails.

ばさっと概要を把握して見た目も簡単に確認するために，この章ではscaffoldを使ってたくさんの機能を簡単に作る (scaffold-driven)  
この章以降では他の方法を使う

## 2.1 Planning the application

```rails new```してGemfileをいじって```bundle install --without production```してgitに上げてherokuにも上げる (Chapter 1参照)

アプリを作る準備完了だ

> The typical first step when making a web application is to create a data model, which is a representation of the structures needed by our application.

とりあえず最初はデータモデルを作ってアプリの構成を示すのが典型

### 2.1.1 A toy model for users

とりあえず最小限に，これから先に作るアプリのためのデータモデルを決めておく

Usersは以下のとおり

![Figure2.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_user_model.png)

今は最小限の定義だけで，Section 6.1からDBにテーブルを作っていく

### 2.1.2 A toy model for microposts

Micropostsは以下のとおり

![Figure2.3](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_micropost_model.png)

> There’s an additional complication, though: we want to associate each micropost with a particular user.

特定のユーザに各マイクロソフトを紐付けるためにuser_idカラムを定義する  
Section 2.3.3やChapter 11から，どういうふうに紐付けを行っていくかを見ていく
