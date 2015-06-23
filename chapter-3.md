# Chapter 3 Mostly static pages

これ以降の章に渡って作っていくアプリをようやく作り始める

まずはシンプルに，静的ページから

> we can easily add just a small amount of dynamic content. In this chapter we’ll learn how.

動的な要素もちょっとだけ入れる (本当にちょっとだけ)

> Along the way, we’ll get our first taste of automated testing, which will help us be more confident that our code is correct  

また，この章からテストを書いていく

> Moreover, having a good test suite will allow us to refactor our code with confidence, changing its form without changing its function.

良いテストを書くと，自信を持ってリファクタリングができる  
きちんとてすとをすれば，書き換えによって機能が望まれないかたちに変化してしまうことがない

## 3.1 Sample app setup

```rails new```してGemfileをいじって```bundle install --without production```する (Chapter 1 参照)

Gemfileのバージョンを今一度指定するために```bundle update```を行うべき  
(Gemfile.lockを無視する)

> Update the gems specified (all gems, if none are specified), ignoring the previously installed gems specified in the Gemfile.lock. In general, you should use bundle install(1) to install the same exact gems and versions across machines.

[bundle-update(1) - Update your gems to the latest available versions](http://bundler.io/v1.5/man/bundle-update.1.html)

```git init```して```git commit```してREADMEを変更して```git commit```して```git push```して```heroku create```して```git ush heroku master```する (Chapter 1 参照)

```heroku logs```

herokuでのログを出力して見る (production環境)
