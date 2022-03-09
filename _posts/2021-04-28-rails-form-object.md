---
layout: post
title: "Rails: form object"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2021/04/28/
---

app/forms/application_form.rb
~~~ rb
class ApplicationForm
  include ActiveModel::Model

  def model
    raise 'need to be override if using uniqueness validator'
  end
end
~~~

app/validators/uniqueness_validator.rb
~~~ rb
class UniquenessValidator < ActiveRecord::Validations::UniquenessValidator
  def validate_each(record, attribute, value)
    if record.is_a?(ApplicationForm)
      model = record.model
      # search for models with attribute equals to form field value
      query = if options[:case_sensitive] == false && value
                model.class.where("lower(#{attribute}) = ?", value.downcase)
              else
                model.class.where(attribute => value)
              end
      # if model persisted, query should bypass model
      if model.persisted?
        query = query.where("#{model.class.primary_key} != ?", model.id)
      end
      # apply scope if options has been declared
      Array(options[:scope]).each do |field|
        # add condition to only check unique value with the same scope
        query = query.where(field => record.send(field))
      end

      record.errors.add(attribute, :taken) if query.count.positive?
    else
      super
    end
  end
end
~~~

app/forms/companies/update_form.rb
~~~ rb
module Companies
  class UpdateForm < ApplicationForm
    attr_accessor :name, :address, :phone_number, :currency, :status,
                  :supported_languages, :contract_start_date,
                  :contract_expiration_date, :company_admin_users

    validates :name, presence: true, if: -> { @params[:name] }
    validates :address, presence: true, if: -> { @params[:address] }
    validates :phone_number, presence: true, if: -> { @params[:phone_number] }
    validates :contract_start_date, presence: true, if: -> { @params[:contract_start_date] }
    validates :status, inclusion: { in: ::Company.status.values }, allow_nil: true
    validate :valid_supported_languages, if: -> { @params[:supported_languages] }
    validate :check_company_admin_users, if: -> { @params[:company_admin_users] }

    def initialize(params, company)
      @company_admin_users_params = params["company_admin_users"]
      @company_params = params.except("company_admin_users")
      super(params)
      @company = company
      @params = params
    end

    def submit!
      @company.assign_attributes(@company_params)
      @company.company_admin_users << company_admin_users_attributes if @company_admin_users_params.present?
      @company.save!
    end

    private

    def valid_supported_languages
      return unless supported_languages.detect { |language| ::Company::SUPPORTED_LANGUAGES.exclude?(language.to_sym) }
      errors.add(:supported_languages, I18n.t('errors.messages.inclusion'))
    end

    def company_admin_user_forms
      @company_admin_user_forms ||= @company_admin_users_params.map do |company_admin_user_params|
        if company_admin_user_params[:id]
          ::CompanyAdminUsers::UpdateForm.new(company_admin_user_params, @company)
        else
          ::CompanyAdminUsers::CreateForm.new(company_admin_user_params, @company)
        end
      end
    end

    def check_company_admin_users
      company_admin_user_forms.each_with_index do |company_admin_user_form, index|
        next if company_admin_user_form.valid?
        errors.add(:"company_admin_users[#{index}]", [company_admin_user_form.errors])
      end
    end

    # company_admin_usersはdisabledができるので、destroyの処理がありません、もしくは必要になった時DELETE company_admin_users/:idで処理
    def company_admin_users_attributes
      # @company_admin_users_params.map do |company_admin_user_param|
      #   company_admin_user = @company.company_admin_users.find_or_initialize_by(id: company_admin_user_param[:id])
      #   company_admin_user.assign_attributes(company_admin_user_param)
      #   company_admin_user
      # end
      company_admin_user_forms.map do |company_admin_user_form|
        company_admin_user_form.build
      end
    end
  end
end
~~~

app/forms/company_admin_users/create_form.rb
~~~ rb
module CompanyAdminUsers
  class CreateForm < ApplicationForm
    attr_accessor :name, :email, :locale, :status, :role, :company_id

    validates :name, presence: true
    validates :email, presence: true, uniqueness: { scope: :company_id }
    validates :email, 'valid_email_2/email': { disposable: true, mx: true }
    validates :role, inclusion: { in: ::CompanyAdminUser.role.values }, allow_nil: true

    def initialize(params, company)
      create_params = params.merge(company_id: company.id)
      super(create_params)
      @company_admin_user = CompanyAdminUser.new(create_params)
    end

    def model
      @company_admin_user
    end

    def build
      @company_admin_user
    end

    def submit!
      @company_admin_user.save!
    end
  end
end
~~~

app/forms/company_admin_users/update_form.rb
~~~ rb
module CompanyAdminUsers
  class UpdateForm < ApplicationForm
    attr_accessor :name, :email, :locale, :status, :role, :company_id, :id

    validates :name, presence: true, if: -> { @params[:name] }
    validates :email, presence: true, uniqueness: { scope: :company_id }, if: -> { @params[:email] }
    validates :email, 'valid_email_2/email': { disposable: true, mx: true }, if: -> { @params[:email] }
    validates :role, inclusion: { in: ::CompanyAdminUser.role.values }, allow_nil: true

    def initialize(params, company)
      super(params)
      @params = params
      @company = company
      @company_admin_user = @company.company_admin_users.find_or_initialize_by(id: params[:id]) # findだけでもいいかも
    end

    def model
      @company_admin_user
    end

    def build
      @company_admin_user.assign_attributes(@params)
      @company_admin_user
    end

    def submit!
      @company_admin_user.update!(@params)
    end
  end
end
~~~

app/forms/hotels/update_form.rb
~~~ rb
module Hotels
  class UpdateForm < ApplicationForm
    attr_accessor :name, :address, :phone_number, :status, :hp_url, :logo_image, :hotel_translations

    validates :name, presence: true, if: -> { @params.key?(:name) }
    validates :address, presence: true, if: -> { @params.key?(:address) }
    validates :phone_number, presence: true, if: -> { @params.key?(:phone_number) }
    validates :hp_url, presence: true, if: -> { @params.key?(:hp_url) }
    validates :status, inclusion: { in: ::Hotel.status.values }, allow_nil: true
    validate :check_hotel_translations, if: -> { @params.key?(:hotel_translations) }

    def initialize(params, hotel)
      @hotel_translations_params = params[:hotel_translations]
      @hotel_params = params.except(:hotel_translations)
      super(params)
      @hotel = hotel
      @params = params
    end

    def submit!
      @hotel.assign_attributes(@hotel_params)
      build_hotel_translations
      @hotel.save!
    end

    private

    def hotel_translation_forms
      @hotel_translation_forms ||= @hotel_translations_params.map do |hotel_translation_params|
        if hotel_translation_params[:id]
          ::HotelTranslations::UpdateForm.new(hotel_translation_params, @hotel)
        else
          ::HotelTranslations::CreateForm.new(hotel_translation_params, @hotel)
        end
      end
    end

    def check_hotel_translations
      return if @hotel_translations_params.blank?
      hotel_translation_forms.each_with_index do |hotel_translation_form, index|
        next if hotel_translation_form.valid?
        errors.add(:"hotel_translations[#{index}]", [hotel_translation_form.errors])
      end
    end

    def hotel_translations_attributes
      hotel_translation_forms.map do |hotel_translation_form|
        hotel_translation_form.build
      end
    end

    def hotel_translation_ids
      default_locale_translation_id = @hotel.hotel_translations.where(locale: 'ja').take.id
      return [default_locale_translation_id] if @hotel_translations_params.blank?
      @hotel_translations_params.pluck(:id).compact.push(default_locale_translation_id)
    end

    def build_hotel_translations
      # delete
      to_delete_ids = @hotel.hotel_translation_ids - hotel_translation_ids
      HotelTranslation.delete(to_delete_ids) if to_delete_ids.present?
      # create / update
      @hotel.hotel_translations << hotel_translations_attributes if @hotel_translations_params.present?
    end
  end
end
~~~
