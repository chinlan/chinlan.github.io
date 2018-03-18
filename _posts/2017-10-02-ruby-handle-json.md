---
layout: post
title: "Ruby: 處理JSON相關methods"
description: ""
categories: [ruby]
tags: [json]
redirect_from:
  - /2017/10/02/
---

# 讀取JSON形式的檔案然後轉換成hash
`JSON.load()`

sample.json
~~~ json
{
  "Ocean": {
    "Squid":10,
    "Octopus":8
  },
  "Sky": {
    "Swallow":2,
    "Crow":2
  }
}
~~~

讀取上面的sample.json檔案後轉換成hash。
~~~ ruby
require 'json'

File.open("sample.json") do |file|
  hash = JSON.load(file)
  p hash
end
~~~

執行結果
~~~
{"Ocean"=>{"Squid"=>10, "Octopus"=>8}, "Sky"=>{"Swallow"=>2, "Crow"=>2}}
~~~

# JSON形式的string轉換成hash
`JSON.parse()`
`JSON.load()`

~~~ ruby
require 'json'

$str = '{ "Ocean": { "Squid":10, "Octopus":8 }'
JSON.parse($str);
p hoge
~~~

執行結果
~~~
{"Ocean"=>{"Squid"=>10, "Octopus"=>8}}
~~~

# hash轉換成JSON形式的string
`JSON.generate()`

~~~ ruby
require 'json'

str = JSON.generate({ "Ocean" => { "Squid" => 10, "Octopus" =>8 }})
puts str
~~~

# hash轉成JSON檔案
`JSON.dump()`
JSON.dump()內部會呼叫`JSON.generate()`

~~~ ruby
require 'json'

File.open("sample2.json", 'w') do |file|
  hash = { "Ocean" => { "Squid" => 10, "Octopus" =>8 }}
  str = JSON.dump(hash, file)
end
~~~

sample2.json
~~~
{"Ocean":{"Squid":10,"Octopus":8}}
~~~

# hash轉成pretty format的JSON形式
`JSON.pretty_generate()`

~~~ ruby
require 'json'

hash = { "Ocean" => { "Squid" => 10, "Octopus" =>8 }}
json_str = JSON.pretty_generate(hash)
puts json_str
~~~

執行結果
~~~
{
  "Ocean": {
    "Squid": 10,
    "Octopus": 8
  },
  "Sky": {
    "Swallow": 2,
    "Crow": 2
  }
}
~~~

[ref](http://uxmilk.jp/13387)
[ref](http://www.sejuku.net/blog/16196)


## json使用 “  雙引號
~~~
hash = {enable: false}

hash.to_json
 => "{\"enable\":false}"

hash.as_json
 => {"enable"=>false}

JSON.parse(hash.to_json)
 => {"enable"=>false}

hash.stringify_keys
 => {"enable"=>false}

~~~
