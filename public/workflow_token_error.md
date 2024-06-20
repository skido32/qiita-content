---
title: refusing to allow a Personal Access Token to create or update workflowエラーの解決法
tags:
  - GitHub
  - Workflow
  - GitHubActions
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

https://qiita.com/Qiita/items/32c79014509987541130

最近 Qiita CLI を使って記事を管理し始めたのですが、
以下の token エラーに遭遇して push できなかったので、対処法を備忘録として残そうと思います。

```
$ git push origin main
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 26.54 KiB | 8.85 MiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/skido32/qiita-content.git
 ! [remote rejected] main -> main (refusing to allow a Personal Access Token to create or update workflow `.github/workflows/publish.yml` without `workflow` scope)
error: failed to push some refs to 'https://github.com/skido32/qiita-content.git'
```

# 対処法

`Settings`→`Developer settings`→`personal access tokens`からアクセストークン設定を確認

![スクリーンショット 2024-06-19 23.28.02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2518928/ac230a5b-67a7-632f-8896-de5b8973a611.png)

私の場合は`Fine-grained personal access tokens`を利用していたので、
対象のアクセストークン設定画面を開き、`Access on`→`Edit`

![スクリーンショット 2024-06-19 23.37.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2518928/079385f8-e274-e912-9d44-db4559cfc80d.png)

`Permissions`→`Repository permissions`→`Workflows`
`Access: Read and write`に変更し、`update`をクリックで設定完了

![スクリーンショット 2024-06-19 23.46.06.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2518928/97173727-86ba-ff5b-91d1-ba9386778342.png)

無事 push することができました！

```
$ git push origin main
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 26.54 KiB | 13.27 MiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/skido32/qiita-content.git
   c69fcf3..e8cf83e  main -> main
```
