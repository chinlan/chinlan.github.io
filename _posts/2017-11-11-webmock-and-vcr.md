---
layout: post
title: "VCR vs Webmock stub"
description: ""
categories: [rails]
tags: [test]
redirect_from:
  - /2017/11/11/
---

webmock的stub不知道怎麼寫的時候（ex: 不確定外部服務的response內容是什麼），可以用VCR先把request和response內容搞清楚（查看存成的檔案即可）

這是在VCR不好用的時候才堅持用stub每次跑測試，比如說那種有截止日期的測試，如果沒有這種要素的話，其實用VCR真的輕鬆很多，只是檔案會增加，還有檔案中要是不小心印出敏感資料要記得遮擋，vcr.rb要記得gitignore掉。

只用webmock的寫法：
~~~ ruby
RSpec.shared_context 'Get subscription from Google' do
  before do
    stub_request(:post, 'https://www.googleapis.com/oauth2/v4/token').with(
      body: {
        "assertion": /.+/,
        "grant_type": 'urn:ietf:params:oauth:grant-type:jwt-bearer'
      },
      headers: {
        "content-type": 'application/x-www-form-urlencoded'
      }
    ).to_return(
        body: {
          access_token: /.+/,
          token_type: 'Bearer',
          expires_in: 3600
        }.to_json,
        headers: {
          "content-type": 'application/x-www-form-urlencoded'
        }
    )

    stub_request(:get, 'https://www.googleapis.com/androidpublisher/v2/'\
      'applications/xxxxkjfjkldajddkjsljdlfjlksjdkljfkljkljklsjdlkfjkdlseijroj\
      'zlxdkjfj.sjdifjidjlsje.siijds.jcjdl.sijeifjijiojjs;fsjdijfilslj).with(
        headers: {
          "content-type": 'application/x-www-form-urlencoded'
        }
      ).to_return(
        body: {
          kind: 'androidpublisher#subscriptionPurchase',
          startTimeMillis: Time.current.to_i * 1000,
          expiryTimeMillis: (Time.current.to_i + 12.months.to_i) * 1000,
          autoRenewing: true,
          priceCurrencyCode: 'JPY',
          priceAmountMicros: 480.00,
          countryCode: 'JP',
          developerPayload: 'string',
          paymentState: 1,
          cancelReason: nil,
          userCancellationTimeMillis: nil,
          orderId: 'GPA.3334-2753-3846-64353'
        }.to_json,
        headers: {
          "content-type": 'application/json; charset=UTF-8'
        }
      )
    end
  end
end
~~~

~~~ ruby
RSpec.shared_context 'Verify subscription from appstore' do
  before do
    data = YAML.safe_load(File.read('./spec/fixtures/ios.yml'))
    body = data['response']
    body['latest_receipt_info'][0]['expires_date_ms'] = (Time.current.to_i + 12.months.to_i) * 1000
    receipt64 = data['data'][0]['receipt']
    stub_request(:post, 'https://buy.itunes.apple.com/verifyReceipt').with(
      body: "{ \"receipt-data\": \"#{receipt64}\", \"password\": \"#{Rails.application.secrets.appstore_secret}\" }",
      headers: {
       'Accept' => '*/*',
       'Accept-Encoding' => 'gzip;q=1.0,deflate;q=0.6,identity;q=0.3',
       'User-Agent' => 'Ruby'
      }
    ).to_return(status: 200, body: body.to_json)
  end
end
~~~

~~~ ruby
require 'rails_helper'

RSpec.describe 'Sales API' do
  let(:user) { create(:user) }
  let(:android_data) do
    YAML.safe_load(File.read('./spec/fixtures/android.yml'))
  end
  let(:ios_data) do
    YAML.safe_load(File.read('./spec/fixtures/ios.yml'))
  end

  describe 'POST /sales', autodoc: true do
    context 'when user is signed in' do
      include_context 'login'

      context 'when device is android' do
        include_context 'Get subscription from Google'

        let(:user_agent) { 'MYAPP/4.0.10 (KYV41; Android 7.1.1; Locale/ja_JP)' }

        it 'creates an android_sale' do
          params = android_data['data'][1].to_json
          post '/sales', params: params, headers: http_headers
          expect(response.status).to eq(201)
          sale = AndroidSale.last
          expect(sale.product_id).to eq('jp.co.myapp.autorenew.supporter.loyal.1year')
          result = JSON.parse(response.body)
          expect(result['user']['supporter_type']).to eq('loyal')
        end
      end

      context 'when device is ios' do
        context 'when autorenew is not cancelled' do
          include_context 'Verify subscription from appstore'
          it 'creates an ios_sale' do
            params = ios_data['data'][0].to_json
            post '/sales', params: params, headers: http_headers
            expect(response.status).to eq(201)
            sale = IosSale.last
            expect(sale.product_id).to eq('jp.co.myapp.autorenew.supporter.1year')
            result = JSON.parse(response.body)
            expect(result['user']['supporter_type']).to eq('standard')
          end
        end
      end
    end
  end
end
~~~
