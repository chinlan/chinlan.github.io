---
layout: post
title: "Rails: stripe webhook"
description: ""
categories: [rails]
tags: [stripe]
redirect_from:
  - /2019/04/08/
---


要用`POST`，不能用PUT

在stripe的dashboard做設定 （Add endpoint)

[stripe web hooks在收到200之前會連續三天每小時重試一次](https://stripe.com/ja-JP/blog/webhooks)

在完成購買的時候引發以下callback（需要檢查是新的購入還是由系統引發的自動更新，只有自動更新的情況才執行）
~~~ruby
module StripeWebhooks
  class AutorenewalController < ApplicationController
    include PaymentErrorHandlingConcern
    before_action :set_event, :prepare_params

    def create
      operator = Payments::Platforms::StripeOperator.new(
        :renew, @user, Settings.devices.pc, I18n.locale, @params
      )
      raise Payments::Errors::InvalidInputError unless operator.verify
      result = operator.execute
      if result
        render json: result, serializer: PaymentSerializer, root: 'payment'
      else
        render_error json: operator, status: :bad_request
      end
    end

    private

    def set_event
      event_json = JSON.parse(request.body.read)
      @event = Stripe::Event.retrieve(event_json['id'])
      head :no_content unless event_json.dig('request', 'id').nil? # Check if it's a renew request
    end

    def prepare_params
      subscription_id = @event.data.object.subscription
      subscription = Stripe::Subscription.retrieve(subscription_id)
      expire_at = Time.zone.at(subscription.current_period_end)
      sale = StripeSale.find_by(subscription_id: subscription_id)
      @user = User.find(sale.user_id)
      @params = {
      sale: sale,
      expire_at: expire_at,
      receipt: subscription,
      product_id: sale.product_id
      }
    end
  end
end
~~~

￼
