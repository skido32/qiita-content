---
title: ファイル操作の基礎：閉じたファイルオブジェクトと再オープン時のエラー
tags:
  - Ruby
  - QiitaEngineerFesta2024
private: false
updated_at: '2024-06-30T11:12:39+09:00'
id: 8ad03fc472d7953b6270
organization_url_name: null
slide: false
ignorePublish: false
---

# 閉じたファイルオブジェクトに対する操作

以下のコードは、ファイルを書き込み用に開き、テキストを書き込む処理です。
最後にファイルを閉じる操作を入れないと、ファイルを開くために確保したメモリが解放されなかったり、整合性と排他制御の問題からファイル操作が正常に行われなかったりします。

```ruby
# File.openでファイルを開き、File.closeで閉じる
file = File.open("example.txt", "w")
file.puts "Hello, world!"
file.close

# もしくはブロックで囲むことで、勝手にファイルが閉じられる
File.open("example.txt", "w").do |file|
  file.puts "Hello, world!"
end
```

ここまでは問題ないですが、一度ファイルを閉じた後に対象のファイルオブジェクトを再利用しようとすると、当然怒られます。

```ruby
file = File.open("example.txt", "w")
file.write("Hello, world!")
file.close

begin
  file.read
rescue IOError => e
  puts "Error: #{e.message}"
end

# Error: closed stream

# File.openに先程のファイルオブジェクトのパスに指定した場合も
# 閉じたファイルオブジェクトを再利用しているのでエラー
begin
  file = File.open(file.path, "r")
rescue IOError => e
  puts "Error: #{e.message}"
end

# Error: closed stream
```

# 解決法

閉じたファイルを操作したいなら、閉じたファイルオブジェクトは使わず、ファイルパスから新しいファイルオブジェクトを再生成することで、エラーを回避できます。

```ruby
file = File.open("example.txt", "w")
file.write("Hello, world!")
file.close

begin
  new_file = File.open("example.txt", "r")
  content = new_file.read
  puts content
  new_file.close
rescue IOError => e
  puts "Error: #{e.message}"
end
```

# 一時ファイルオブジェクトとの挙動の違い

ちなみに Tempfile クラスで作成した一時ファイルオブジェクトの場合は少し挙動が異なります。

```ruby
require 'tempfile'

temp_file = Tempfile.new("example")
temp_file.write("Hello, Tempfile!")
temp_file.close

begin
  new_temp_file = File.open(temp_file.path, "r")
  content = new_temp_file.read
  puts content
  new_temp_file.close
rescue IOError => e
  puts "Error: #{e.message}"
end
```

一時ファイルを閉じた後で再度操作するには、新しいファイルオブジェクトを作成するのは同じですが、Tempfile の場合、閉じた一時ファイルオブジェクトからでもファイルパスを参照できます。
よって、そのパスから File.open で再度ファイルオブジェクトを作成できます。

# まとめ

基本は必要なファイル操作を実施してから、最後にファイルを閉じるべきだと思います。
ただ、どうしても再度ファイル操作が必要になる場合は、閉じたファイルオブジェクトを再利用せずに、新しいファイルオブジェクトを作成することで、エラーを回避できそうです。
