---
layout: post
title: "simple form"
description: ""
categories: [rails]
tags: [form]
redirect_from:
  - /2016/12/07/
---

## f.input

example:
~~~
f.input :bathroom, collection: 1..12, wrapper_html: { class: 'col-sm-3'}, include_blank: I18n.t('simple_form.placeholders.house.not_show_hint')
~~~
會包含一個空白選項，可以塞自己想放的內容

如果不要包含空白選項，

就把include_blank: 改成 prompt:

一樣後面可塞自己想放的內容

~~~
input_html: { disabled: (@house.family? ? false : true) }
~~~
還可加上這樣的東西，來依據條件disable某個input

要另外加上css
~~~
<%= f.text_field :at_floor, label: false, :style => ’width: 35%’ %>
~~~

# simple_field
## f.fields_for
用在巢狀表單或多對多關係的情況的helper
~~~
<%= simple_form_for @house, url: render_form_url(@house), wrapper: :table_form do |f| %>
...
  <%= f.simple_fields_for :rooms do |room| %>
    <%= render ’room_fields’, f: room %>
  <% end %>
<% end %>
~~~
原本像上面這樣寫即可針對到@house的nested attribute :rooms，

但要是想對:rooms做一些排序之類的設定，

而在controller寫了

~~~
@rooms = @house.rooms.order(:order)
~~~
可將helper做以下改寫（只是在:rooms後面加上@rooms）
~~~
<%= f.simple_fields_for :rooms, @rooms do |room| %>
  <%= render ’room_fields’, f: room %>
 <% end %>
~~~
就可以達到想要的排序結果了
