---
layout: post
title: "Rails: using savon to deal with SOAP API"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2021/06/30/
---

Goal: calling TlLincoln Soap API in a Rails application.

* A good tool for examining the xml request and response body: SoapUI
  Import the WSDL files needed, and the request examples will be generated automatically.
  If proxy is needed, it can also be set easily.

`Gemfile`:
```
gem 'savon'
```

`/app/lib/tl_lincoln/client.rb`:
~~~ruby
module TlLincoln
  class Client
    TEST_DOMAIN = 'https://test472.tl-lincoln.net'.freeze
    PROD_DOMAIN = 'https://www.tl-lincoln.net'.freeze
    DOMAIN = Rails.env.production? ? PROD_DOMAIN : TEST_DOMAIN

    HOTEL_SEARCH_WSDL = "#{DOMAIN}/agtapi/v1/crs/CrsHotelInfoService?wsdl".freeze
    HOTEL_AVAIL_WSDL = "#{DOMAIN}/agtapi/v1/crs/CrsAvailableInquiryService?wsdl".freeze
    HOTEL_RES_WSDL = "#{DOMAIN}/agtapi/v1/crs/CrsReservationService?wsdl".freeze
    PING_WSDL = "#{DOMAIN}/agtapi/v1/crs/CrsOtherService?wsdl".freeze
    HOTEL_DETAIL_WSDL = "#{DOMAIN}/agtapi/v1/crs/CrsHotelDetailService?wsdl".freeze

    HOTEL_SEARCH_OPERATIONS = %i[ota_hotel_search].freeze
    HOTEL_AVAIL_OPERATIONS = %i[ota_hotel_avail].freeze
    HOTEL_RES_OPERATIONS = %i[ota_cancel ota_hotel_res ota_read ota_hotel_res_modify].freeze
    PING_OPERATIONS = %i[ota_ping].freeze
    HOTEL_DETAIL_OPERATIONS = %i[ota_hotel_descriptive_info].freeze

    OPERATIONS = (HOTEL_SEARCH_OPERATIONS + HOTEL_AVAIL_OPERATIONS + HOTEL_RES_OPERATIONS + PING_OPERATIONS + HOTEL_DETAIL_OPERATIONS).freeze

    def initialize(wsdl)
      @request = Request.new(wsdl)
    end

    OPERATIONS.each do |operation|
      define_method(operation) do |message, attributes = {}|
        retries = 5
        begin
          Response.new(@request.call(operation, message, attributes))
        rescue Timeout::Error, Net::OpenTimeout, Errno::ECONNREFUSED
          Rails.logger.info "Timeout while calling #{operation}"
          retries -= 1
          if retries.positive?
            retry
          else
            return nil
          end
          # rescue Savon::Error => error
          #   Rails.logger.info "Savon Error while calling #{operation} with message #{message}, attributes #{attributes}. Error code: #{error.http.code}"
        end
      end
    end
  end
end

~~~

`/app/lib/tl_lincoln/request.rb`:
~~~ruby
module TlLincoln
  class Request
    NAMESPACES = { "xmlns:head": "http://www.seanuts.co.jp/ota/header" }.freeze

    def initialize(wsdl)
      @savon_client = Savon.client(
        wsdl: wsdl,
        proxy: ENV['PROXY'], # savon does not support https proxy
        open_timeout: 300,
        read_timeout: 300,
        pretty_print_xml: true,
        env_namespace: :soapenv,
        namespaces: NAMESPACES,
        namespace: "http://www.opentravel.org/OTA/2003/05",
        namespace_identifier: :ns,
        soap_header: soap_header,
        filters: %w[head:Password head:Username]
      )
    end

    def call(operation, message, attributes = {})
      default_attributes = { 'Version' => '1.0', 'PrimaryLangID' => 'jpn' }
      # Below is for debugging, print `request.body` and know what xml contents it will generate
      # request = @savon_client.build_request(operation, message: message, attributes: default_attributes.merge(attributes))
      @savon_client.call(operation, message: message, attributes: default_attributes.merge(attributes))
    end

    private

    # Known from SoapUI automatically generated xml request example after import the WSDL files
    def soap_header
      {
        "head:Interface" => {
          "head:PayloadInfo" => {
            "head:CommDescriptor" => {
              "head:Authentication" => {
                "head:Username" => ENV['TLLINCOLN_USERNAME'],
                "head:Password" => ENV['TLLINCOLN_PASSWORD']
              }
            }
          }
        }
      }
    end
  end
end

~~~

`/api/lib/tl_lincoln/response.rb`:
~~~ruby
module TlLincoln
  class Response
    def initialize(response)
      @response = response
    end

    def body
      @body ||= @response.body
    end

    def tl_room_stay
      base = hotel_avail_result || hotel_reservation_result
      @tl_room_stay ||= base&.dig(:room_stays, :room_stay) || {}
    end

    def api_rooms_result
     @api_rooms_result ||= [tl_room_stay.dig(:room_types, :room_type)].flatten.compact
    end

    def api_plans_result
      # booking系のroom_rateレスポンスにrate_plan_codeがない
      @api_plans_result ||= if api_room_plans_result[0]&.dig(:@rate_plan_code).nil?
        [tl_room_stay.dig(:rate_plans, :rate_plan)].flatten.compact
                            else
        [tl_room_stay.dig(:rate_plans, :rate_plan)].flatten.compact.map do |rate_plan|
          room_rate = api_room_plans_result.find{ |rate| rate[:@rate_plan_code] == rate_plan[:@rate_plan_code] }
          # 1個目を代表に
          charge_type = [room_rate.dig(:rates, :rate)].flatten.compact.first&.dig(:base, :@type)
          rate_plan.merge!(charge_type: charge_type)
          rate_plan
        end
                            end
    end

    def api_room_plans_result
      @api_room_plans_result ||= [tl_room_stay.dig(:room_rates, :room_rate)].flatten.compact
    end

    # For Avail =============

    def hotel_avail_result
      Rails.logger.info avail_error_result and raise ::TlLincoln::Error.new(avail_error_result) if avail_error_result.present?
      @hotel_avail_result ||= body[:ota_hotel_avail_rs]
    end

    def avail_error_result
      [body.dig(:ota_hotel_avail_rs, :errors, :error)].flatten.compact
    end

    # For Booking ===========

    def hotel_reservation_result
      Rails.logger.info create_booking_error_result and raise ::TlLincoln::Error.new(create_booking_error_result) if create_booking_error_result.present?
      Rails.logger.info modify_booking_error_result and raise ::TlLincoln::Error.new(modify_booking_error_result) if modify_booking_error_result.present?
      @hotel_reservation_result ||= body.dig(:ota_hotel_res_rs, :hotel_reservations, :hotel_reservation) || body.dig(:ota_hotel_res_modify_rs, :hotel_res_modifies, :hotel_res_modify) || {}
    end

    def create_booking_error_result
      [body.dig(:ota_hotel_res_rs, :errors, :error)].flatten.compact
    end

    def modify_booking_error_result
      [body.dig(:ota_hotel_res_modify_rs, :errors, :error)].flatten.compact
    end

    def cancel_booking_error_result
      [body.dig(:ota_cancel_rs, :errors, :error)].flatten.compact
    end
  end
end
~~~

`/api/lib/tl_lincoln/error.rb`:
~~~ruby
module TlLincoln
  class Error < StandardError
    def initialize(error_result_array)
      super
      @error_array = error_result_array
      # @error_hash = error_result_hash[:error]
    end

    def error_codes
      @error_array.pluck(:@code)
      # @error_hash[:@code]
    end

    def message
      @error_array.pluck(:@short_text).join("\n")
      # @error_hash[:@short_text]
    end

    def error_types
      @error_array.pluck(:@type)
      # @error_hash[:@type]
    end

    def is_system_limit_error?
      error_codes.include?("737")
    end
  end
end
~~~


`/app/models/tl_lincoln/param_builders/booking.rb`:
~~~ruby
module TlLincoln
  module ParamBuilders
    class Booking
      # request_type: ['create', 'update', 'cancel']
      def initialize(params, request_type='create')
        @params = params
        @request_type = request_type
      end

      def build_params
        default = { 'ns:POS' => { 'ns:Source' => { 'ns:RequestorID' => { :@Type => "5", 'ns:CompanyName' => 'NIN-NIN' } } } }
        default.merge!(hotel_reservation_hash) if @request_type == 'create'
        default.merge!(hotel_res_modify_hash) if @request_type == 'update'
        default.merge!(hotel_res_cancel_hash) if @request_type == 'cancel'
        default
      end

      private

      def hotel_reservation_hash
        { 'ns:HotelReservations' => { 'ns:HotelReservation' => reservation_hash } }
      end

      def hotel_res_modify_hash
        { 'ns:HotelResModifies' => { 'ns:HotelResModify' => reservation_hash } }
      end

      def hotel_res_cancel_hash
        result = {}
        result.merge!(unique_id_hash)
        result.merge!(verification_hash)
        result.merge!(extension_hash) if @params[:cancellation_charge]
        result
      end

      def reservation_hash
        result = {}
        result.merge!(unique_id_hash) if @request_type == 'update' && reservation_code
        result.merge!(room_stay_hash)
        result.merge!(guest_profile_hash)
        result.merge!(global_info_hash)
        result.merge!(extension_hash)
        result
      end

      def reservation_code
        return @params[:booking_channel_manager_code] if @params[:booking_channel_manager_code].present?
        return nil if @params[:booking_id].nil?
        booking = ::Booking.find(@params[:booking_id])
        @reservation_code ||= booking.channel_manager_code
      end

      def unique_id_hash
        { 'ns:UniqueID' => { :@Type => "14", :@ID => reservation_code, :@ID_Context => "CrsConfirmNumber" } }
      end

      def plan_code
        return @params[:plan_channel_manager_code] if @params[:plan_channel_manager_code].present?
        return nil if @params[:hotel_plan_ids].nil?
        plan = ::HotelPlan.find(@params[:hotel_plan_ids][0])
        @plan_code ||= plan.channel_manager_code
      end

      def guest_hash_array(room_param)
        result = []
        # ※10:大人、8:子供、7幼児のコードを指定した場合でも、レスポンスはそれぞれ51、53、58が返されます。
        result.push({ :@AgeQualifyingCode => '51', :@Count => room_param[:adults].to_i }) if room_param[:adults].present? && (room_param[:adults]).positive?
        # 食事・寝具利用
        result.push({ :@AgeQualifyingCode => '53', :@Count => room_param[:children_a].to_i }) if room_param[:children_a].present? && (room_param[:children_a]).positive?
        # キッズミール・寝具利用
        result.push({ :@AgeQualifyingCode => '55', :@Count => room_param[:children_b].to_i }) if room_param[:children_b].present? && (room_param[:children_b]).positive?
        # 添い寝（食事・寝具なし）
        result.push({ :@AgeQualifyingCode => '58', :@Count => room_param[:children_d].to_i }) if room_param[:children_d].present? && (room_param[:children_d]).positive?
        result
      end

      def hotel_rooms
        @hotel_rooms ||= ::HotelRoom.where(id: @params[:hotel_rooms].pluck(:hotel_room_id))
      end

      def get_room_code(room_param)
        return room_param[:hotel_room_channel_manager_code] if room_param[:hotel_room_channel_manager_code].present?
        hotel_room = hotel_rooms.find{ |room| room.id == room_param[:hotel_room_id] }
        hotel_room.channel_manager_code
      end

      def accommodation_period_dates
        date_array = (@params[:checkin_date]...@params[:checkout_date]).to_a
        date_array.map(&:to_s)
      end

      def room_rate_hash_array
        @params[:hotel_rooms].map do |room_param|
          room_code = get_room_code(room_param)
          accommodation_period_dates.map do |date|
            {
              :@EffectiveDate => date, :@ExpireDate => date,
              :@RoomTypeCode => room_code, :@NumberOfUnits => 1,
              'ns:GuestCounts' => { 'ns:GuestCount' => guest_hash_array(room_param) }
            }
          end
        end.flatten
      end

      def room_stay_hash
        { 'ns:RoomStays' => { 'ns:RoomStay' => { 'ns:RatePlans' => { 'ns:RatePlan' => { :@RatePlanCode => plan_code } }, 'ns:RoomRates' => { 'ns:RoomRate' => room_rate_hash_array } } } }
      end

      def customer_hash(profile_param)
        fullname_hankaku = (profile_param[:first_name_hiragana] + profile_param[:last_name_hiragana]).katakana.zen_to_han
        result = {'ns:PersonName' => { 'ns:GivenName' => profile_param[:first_name], 'ns:Surname' => profile_param[:last_name], 'ns:TPA_Extensions' => { 'ns:LastNameZenkakuHurigana' => profile_param[:last_name_hiragana], 'ns:GivenNameZenkakuHurigana' => profile_param[:first_name_hiragana], 'ns:FullNameHankaku' => fullname_hankaku } } }
        result.merge!({ 'ns:Telephone' => { :@PhoneNumber => profile_param[:phone_number] } }) if profile_param[:phone_number]
        result.merge!({ 'ns:Email' => profile_param[:email] }) if profile_param[:email]
        result.merge!({ 'ns:Address' => { 'ns:AddressLine' => profile_param[:address] } }) if profile_param[:address]
        result
      end

      def guest_profile_hash
        guest_profile_array = @params[:guests].map do |profile_param|
          {
            # :@PrimaryIndicator => 'true', # 代表者の資料だけ送りしますので
            :@PrimaryIndicator => profile_param[:representative] ? profile_param[:representative].to_s : 'false',
            'ns:Profiles' => {
              'ns:ProfileInfo' => {
                'ns:Profile' => { :@ProfileType => "1", 'ns:Customer' => customer_hash(profile_param) }
              }
            }
          }
        end
        { 'ns:ResGuests' => { 'ns:ResGuest' => guest_profile_array } }
      end

      def special_requests_hash
        { 'ns:SpecialRequests' => { 'ns:SpecialRequest' => { 'ns:Text' => @params[:special_requests] } } }
      end

      def reservation_ids_hash
        { 'ns:HotelReservationIDs' => { 'ns:HotelReservationID' => { :@ResID_Type => "14", :@ResID_Value => @params[:unique_id] } } }
      end

      def company_profile_hash
        # TODO: revise company code
        { 'ns:Profiles' => { 'ns:ProfileInfo' => { 'ns:Profile' => { :@ProfileType => "4", 'ns:CompanyInfo' => { 'ns:CompanyName' => 'NIN-NIN', :attributes! => { 'ns:CompanyName' => { 'Code' => "001" } } } } } } }
      end

      def hotel
        return nil if @params[:hotel_id].nil? && @params[:hotel_channel_manager_code].nil?
        @hotel ||= ::Hotel.find_by(id: @params[:hotel_id]) || ::Hotel.find_by(channel_manager_code: @params[:hotel_channel_manager_code])
      end

      def hotel_code
        @hotel_code ||= @params[:hotel_channel_manager_code] || hotel&.channel_manager_code
      end

      def hotel_name
        @hotel_name ||= hotel&.name
      end

      def basic_property_info_hash
        { 'ns:BasicPropertyInfo' => { :@HotelCode => hotel_code, :@HotelName => hotel_name } }
      end

      def global_info_hash
        duration = "P#{@params[:checkout_date].to_date.mjd - @params[:checkin_date].to_date.mjd}N"
        default = { 'ns:TimeSpan' => { :@Start => @params[:checkin_date], :@End => @params[:checkout_date], :@Duration => duration } }
        default.merge!(special_requests_hash) if @params[:special_requests]
        default.merge!(reservation_ids_hash) if @request_type == "create"
        default.merge!(company_profile_hash)
        default.merge!(basic_property_info_hash)
        { 'ns:ResGlobalInfo' => default }
      end

      def pay_type_code
        @pay_type_code ||= ::TlLincoln::PayType.new.pay_type_code(@params[:pay_type])
      end

      def extension_hash
        basic_info_hash = {}
        basic_info_hash.merge!({ 'ns:SettlementDiv' => pay_type_code }) if @request_type != 'cancel'
        basic_info_hash.merge!({ 'ns:CancellationCharge' => @params[:cancellation_charge] }) if @request_type != 'create' && @params[:cancellation_charge]
        # cancelの時にcancellation_chargeがあれば、cancellationNotice（取消料補足の説明）が必須です、nin-ninでは使わなさそうなので、仮の内容を一旦入れます
        basic_info_hash.merge!({ 'ns:CancellationNotice' => 'Cancellation Notice' }) if @request_type == 'cancel' && @params[:cancellation_charge]
        { 'ns:TPA_Extensions' => { 'ns:BasicInfo' => basic_info_hash } }
      end

      def verification_hash
        { 'ns:Verification' => { 'ns:TPA_Extensions' => { 'ns:BasicPropertyInfo' => { 'ns:HotelCode' => hotel_code } } } }
      end
    end
  end
end
~~~

`/app/models/tl_lincoln/request_client.rb`:
~~~ruby
module TlLincoln
  class RequestClient
    def initialize(hotel_code=nil)
      @hotel_code = hotel_code
    end

    def request_for_saving_to_db
      message = { 'ns:AvailRequestSegments' => { 'ns:AvailRequestSegment' => { :@AvailReqType => 'Both', 'ns:HotelSearchCriteria' => { 'ns:Criterion' => { 'ns:HotelRef' => { :@HotelCode => @hotel_code } } } } } }
      response = avail_client.ota_hotel_avail(message, { 'RateDetailsInd' => true })
      Rails.logger.info response.avail_error_result and raise if response.avail_error_result.present?
      response
    end

    # 特定プラン・部屋の詳細（カレンダー含む）：<部屋・プラン詳細、空室・料金カレンダー取得> (AvailRatesOnly => true)
    # 現在、hotel_id以外必須なparamありません
    # def request_avail_hotel_rooms(params)
    #   message = ::TlLincoln::ParamBuilders::Availability.new(params).build_params
    #   response = avail_client.ota_hotel_avail(message, { 'RateDetailsInd' => true, 'AvailRatesOnly' => true })
    #   if response.avail_error_result.present?
    #     raise ::TlLincoln::Error.new(response.avail_error_result)
    #   else
    #     build_avail_result_entities(response)
    #   end
    # end

    # 特定プラン・部屋の詳細（カレンダー含む）：<部屋・プラン詳細、空室・料金カレンダー取得> (AvailRatesOnly => false)
    def request_avail_prepare_booking(params)
      message = ::TlLincoln::ParamBuilders::RoomAvailability.new.build(params)
      response = avail_client.ota_hotel_avail(message, { 'RateDetailsInd' => true, 'AvailRatesOnly' => false })
      if response.avail_error_result.present?
        raise ::TlLincoln::Error.new(response.avail_error_result)
      else
        check_can_book(params, build_rate_entities(response))
      end
    end

    # カレンダー：<部屋・プラン詳細、空室・料金カレンダー取得>
    def request_calendar(params)
      messages = ::TlLincoln::ParamBuilders::Calendar.new.build(params)

      messages.map do |message|
        response = avail_client.ota_hotel_avail(message, { 'RateDetailsInd' => true, 'AvailRatesOnly' => true })
        if response.avail_error_result.present?
          raise ::TlLincoln::Error.new(response.avail_error_result)
        else
          TlLincoln::EntityBuilders::Calendar.new(response).build
        end
      end.flatten
    end

    # 特定プラン1x特定部屋1の宿泊詳細料金取得: <部屋プランの宿泊詳細料金取得>
    # required params: hotel_room_id, hotel_plan_id, hotel_id, guests: [adults, children_a, children_b, children_d](一組だけ for this 1 room)
    # checkin_date, checkout_date -> optional, 無いなら今日からこの組み合わせ（部屋・プラン・ゲスト人数）の一ヶ月の料金を返します
    def request_avail_rate_detail(params)
      message = ::TlLincoln::ParamBuilders::RateDetail.new.build(params)
      response = avail_client.ota_hotel_avail(message, { 'AvailRatesOnly' => true })
      if response.avail_error_result.present?
        raise ::TlLincoln::Error.new(response.avail_error_result)
      else
        build_avail_result_entities(response)
      end
    end

    # 複数の施設コードでリクエストできる: <施設空室、部屋・プラン・在庫・料金一覧取得>
    # ！cancel_penaltyは含まれていません！
    # 一旦複数施設の検索を使わない？tl_lincoln試験環境でも複数の施設を作ることができません。使うときはデータベースのtl_lincolnホテルコードを全部codeをpluckしてリクエストメッセージの複数ns:HotelRefを組み合わせます
    # 複数施設の検索をしない限りあまり必要ないと感じてます、カレンダータイプの代わりにrate_detailの一ヶ月範囲で使うなら、これでカレンダータイプを代わるのもいいかもしれません
    # 現在、hotel_id以外必須なparamありません
    def request_avail_hotels_rooms(params)
      message = ::TlLincoln::ParamBuilders::List.new.build(params)
      response = avail_client.ota_hotel_avail(message, { 'AvailRatesOnly' => true, 'HotelStayOnly' => true, 'PricingMethod' => 'Lowest' })
      if response.avail_error_result.present?
        raise ::TlLincoln::Error.new(response.avail_error_result)
      else
        build_avail_result_entities(response)
      end
    end

    def request_create_booking(params, unique_booking, request_id)
      message = ::TlLincoln::ParamBuilders::Booking.new(params).build_params
      response = booking_client.ota_hotel_res(message, { 'ResStatus' => "Commit" })
      unique_booking.unique_booking_logs.create!(log_type: 'create_booking', request: params, response: response.body, rails_request_id: request_id)
      if response.create_booking_error_result.present?
        raise ::TlLincoln::Error.new(response.create_booking_error_result)
      else
        build_booking_result_entity(response)
      end
    end

    def request_modify_booking(params, unique_booking, request_id)
      message = ::TlLincoln::ParamBuilders::Booking.new(params, 'update').build_params
      response = booking_client.ota_hotel_res_modify(message, { 'ResStatus' => "Commit" })
      unique_booking.unique_booking_logs.create!(log_type: 'update_booking', request: params, response: response.body, rails_request_id: request_id)
      if response.modify_booking_error_result.present?
        raise ::TlLincoln::Error.new(response.modify_booking_error_result)
      else
        build_booking_result_entity(response)
      end
    end

    def request_cancel_booking(params, unique_booking, request_id)
      message = ::TlLincoln::ParamBuilders::Booking.new(params, 'cancel').build_params
      response = booking_client.ota_cancel(message, { 'CancelType' => "Commit" })
      unique_booking.unique_booking_logs.create!(log_type: 'cancel_booking', request: params, response: response.body, rails_request_id: request_id)
      if response.cancel_booking_error_result.present?
        raise ::TlLincoln::Error.new(response.cancel_booking_error_result)
      else
        response.body.dig(:ota_cancel_rs, :@status)
      end
    end

    private

    def avail_client
      @avail_client ||= ::TlLincoln::Client.new(TlLincoln::Client::HOTEL_AVAIL_WSDL)
    end

    def booking_client
      @booking_client ||= ::TlLincoln::Client.new(TlLincoln::Client::HOTEL_RES_WSDL)
    end

    def build_avail_result_entities(response)
      hotel_room_entities = build_hotel_room_entities(response)
      hotel_plan_entities = build_hotel_plan_entities(response)
      rate_entities = build_rate_entities(response)
      { hotel_room_entities: hotel_room_entities, hotel_plan_entities: hotel_plan_entities, rate_entities: rate_entities }
    end

    def build_hotel_room_entities(response)
      api_rooms_result = response.api_rooms_result
      room_codes = api_rooms_result.pluck(:@room_type_code)
      # when request before pre_booking, do not need to eager load, but anyway add to the whitelist
      hotel_rooms = ::HotelRoom.with_translations.with_images.where(channel_manager_code: room_codes)
      api_rooms_result.map do |room_type|
        hotel_room = hotel_rooms.find{ |room| room.channel_manager_code == room_type[:@room_type_code] }
        next if hotel_room.nil? # dbにない部屋は略して表示に出さない
        tl_room = ::TlLincoln::HotelRoom.new(room_type)
        ::Entity::HotelRoom.new(hotel_room, tl_room)
      end
    end

    def build_hotel_plan_entities(response)
      api_plans_result = response.api_plans_result
      plan_codes = api_plans_result.pluck(:@rate_plan_code)
      # when request before pre_booking, do not need to eager load, but anyway add to the whitelist
      hotel_plans = ::HotelPlan.with_translations.with_images.where(channel_manager_code: plan_codes)
      api_plans_result.map do |rate_plan|
        hotel_plan = hotel_plans.find{ |plan| plan.channel_manager_code == rate_plan[:@rate_plan_code] }
        next if hotel_plan.nil? # dbにないプランは略して表示に出さない
        tl_plan = ::TlLincoln::HotelPlan.new(rate_plan)
        ::Entity::HotelPlan.new(hotel_plan, tl_plan)
      end
    end

    def build_rate_entities(response)
      api_room_plans_result = response.api_room_plans_result
      plan_codes = api_room_plans_result.pluck(:@rate_plan_code).uniq
      room_codes = api_room_plans_result.pluck(:@room_type_code).uniq
      hotel_plans = ::HotelPlan.where(channel_manager_code: plan_codes)
      hotel_rooms = ::HotelRoom.where(channel_manager_code: room_codes)
      api_room_plans_result.map do |room_rate|
        tl_rate = ::TlLincoln::Rate.new(room_rate)
        hotel_plan = hotel_plans.find{ |plan| plan.channel_manager_code == room_rate[:@rate_plan_code] }
        hotel_room = hotel_rooms.find{ |room| room.channel_manager_code == room_rate[:@room_type_code] }
        next if hotel_plan.nil? || hotel_room.nil? # dbにない部屋・プランは略して表示に出さない
        ::Entity::Rate.new(tl_rate, hotel_room, hotel_plan)
      end
    end

    def check_can_book(params, rate_entities)
      needed_room_ids = params[:hotel_rooms].pluck(:hotel_room_id)
      result_room_ids = rate_entities.map(&:hotel_room_id).compact
      (needed_room_ids - result_room_ids).blank? && rate_entities.map(&:availability_status).exclude?('ClosedOut')
    end

    def build_booking_result_entity(response)
      tl_booking = ::TlLincoln::Booking.new(response)
      ::Entity::Booking.new(tl_booking)
    end
  end
end
~~~

`/app/models/tl_lincoln/xxx.rb` are to deal with the response contents from tl_lincoln api response.
`/app/models/entity/xxx.rb` are to provide a mutual interface between different external api responses, which will be used directly in controller/serializer for client-side displaying.


References:
https://www.savonrb.com/version2/
https://download.oracle.com/otn_hosted_doc/jdeveloper/1012/web_services/ws_wsdlstructure.html
https://stackoverflow.com/questions/4394620/how-do-i-use-savon-nested-attributes-hash
https://crunchify.com/basic-wsdl-structure-understanding-wsdl-explained/
https://qiita.com/shibuchaaaan/items/0b8005310a13c641d089
