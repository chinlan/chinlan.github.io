---
layout: post
title: "Rails: message text encryption"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/09/10/
---

Use case:
- The message text needs to be encrypted before saving into database, and decrypted when loading from database.
- The column name is text, but in the API response, 'body' is the preferred key name.

Plan A:
Using Type Casting
[ref](https://metova.com/rails-5-attributes-api/)
Define the `cast` method to decide what to return, the default behavior is to call `cast` on both setting and getting.
If you only need to customize the setting part, override `deserialize` for the getting part, and leave `cast` to setting (`serialize`).

/initializers/active_record/encoded_string_type.rb
~~~ruby
class EncodedStringType < ActiveRecord::Type::String
  def serialize(value)
    encode(value)
  end

  def deserialize(value)
    decode(value)
  end

  private

  def decode(value)
    return if value.nil?
    cipher = OpenSSL::Cipher.new('aes-256-cbc')
    cipher.decrypt
    cipher.pkcs5_keyivgen(Message::ENCRYPT_KEY)
    result =
    cipher.update(Base64.decode64(value)) + cipher.final
    result.force_encoding('UTF-8')
  end

  def encode(value)
    return if value.nil?
    cipher = OpenSSL::Cipher.new('aes-256-cbc')
    cipher.encrypt
    cipher.pkcs5_keyivgen(Message::ENCRYPT_KEY)
    result = cipher.update(value) + cipher.final
    Base64.encode64(result)
  end
end

ActiveRecord::Type.register(:encoded_string, EncodedStringType)
~~~

/models/message.rb
~~~ruby
attribute :text, :encoded_string
alias_attribute :body, :text
~~~

Plan B:
Callback

/models/message.rb
~~~ruby
attr_accessor :body

after_find :decode_and_assign_body
before_save :encode_and_assign_text, if: -> { body }

def encode_and_assign_text
  cipher = OpenSSL::Cipher.new('aes-256-cbc')
  cipher.encrypt
  cipher.pkcs5_keyivgen(ENCRYPT_KEY)
  result = cipher.update(body) + cipher.final
  self.text = Base64.encode64(result)
end

def decode_and_assign_body
  cipher = OpenSSL::Cipher.new('aes-256-cbc')
  cipher.decrypt
  cipher.pkcs5_keyivgen(ENCRYPT_KEY)
  result =
  cipher.update(Base64.decode64(text)) + cipher.final
  self.body = result.force_encoding('UTF-8')
end
~~~


-----------------------------------------------
Another thing to look at for encryption:
`ActiveSupport::MessageEncryptor`

[doc](https://api.rubyonrails.org/classes/ActiveSupport/MessageEncryptor.html)





