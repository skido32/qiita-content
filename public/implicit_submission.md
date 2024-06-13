---
title: 暗黙的送信（Implicit submission）について
tags:
  - JavaScript
  - QiitaEngineerFesta2024
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

最近触っていたコードの中で以下のような処理を見つけました。

```javascript
$("input[type='text']").on("keypress", (e) => {
  if (e.keyCode === 13) {
    $("form").valid();
    return false;
  }
});
```

パッと実装意図が分からず、調べる中で学びがあったので、メモとして残そうと思います。

# 該当コードの実装意図

結論からいくと、Enter キーによる form 送信を防ぐためでした。
input 要素にフォーカスしている時に、Enter キーを押すと送信されてしまう場合があります。
この挙動はユーザーの意図しない送信を誘発する可能性があるため、
送信ボタンをクリックしたときだけ submit する形にしていると思われます。

# Enter キーによる送信

Enter キーによる送信は`暗黙的送信（Implicit submission）`と呼ばれる仕様であり、
WHATWG の仕様書には以下のような記載があります。

https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#implicit-submission

> 4.10.21.2 Implicit submission
>
> A form element's default button is the first submit button in tree order whose form owner is that form element.
>
> If the user agent supports letting the user submit a form implicitly (for example, on some platforms hitting the "enter" key while a text control is focused implicitly submits the form), then doing so for a form, whose default button has activation behavior and is not disabled, must cause the user agent to fire a click event at that default button.
>
> > Note：There are pages on the web that are only usable if there is a way to implicitly submit forms, so user agents are strongly encouraged to support this.
>
> If the form has no submit button, then the implicit submission mechanism must perform the following steps:
>
> - If the form has more than one field that blocks implicit submission, then return
> - Submit the form element from the form element itself with userInvolvement set to "activation"
>
> For the purpose of the previous paragraph, an element is a field that blocks implicit submission of a form element if it is an input element whose form owner is that form element and whose type attribute is in one of the following states: Text, Search, Telephone, URL, Email, Password, Date, Month, Week, Time, Local Date and Time, Number

上記の内容を整理すると、

- form には default ボタンという概念があり、form 内の最初に登場する submit ボタンが default ボタンとして扱われる
- 暗黙的送信が起こると、（disable でない限り）default ボタンの click イベントが発火
- form 内に submit ボタンが存在しない場合も、input 要素が 1 つなら、暗黙的送信が起こる

つまり、form 内に input 要素が複数あり、submit タイプのボタンが存在しない場合以外のフォームでは暗黙的送信が起こるということになります。

# 改めて該当処理の必要性を検討

上記を踏まえて、改めて該当処理の必要性を検討します。
submit タイプの代わりに button タイプのボタンを置き、js でバリデーションチェック後に submit させる作りであれば、暗黙的送信は起こらない可能性が高そうです。
ただ、画面の input 要素が 1 つになる場合もあるし、submit タイプのボタンが置かれる可能性もあります。
よって、該当処理は必要なのだと判断しました。
