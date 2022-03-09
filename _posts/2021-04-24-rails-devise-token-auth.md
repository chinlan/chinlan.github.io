---
layout: post
title: "Rails: devise_token_auth"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2021/04/24/
---

Try using devise_token_auth with an API only Rails application.

### Routes:
When adding namespace to default devise_token_auth routes like below:
~~~ rb
  namespace :api do
    mount_devise_token_auth_for 'User', at: 'user_auth', controllers: {
      registrations: 'api/user_auth/registrations'
    }
    mount_devise_token_auth_for 'AdminUser', at: 'admin_user_auth', controllers: {
      registrations: 'api/admin_user_auth/registrations' # Maybe not needed
    }
    mount_devise_token_auth_for 'CompanyAdminUser', at: 'company_admin_user_auth', controllers: {
      registrations: 'api/company_admin_user_auth/registrations', # Maybe not needed
      sessions: 'api/company_admin_user_auth/sessions'
    }
  end
~~~

The authenticate methods will become: `authenticate_api_user!`, `authenticate_api_admin_user!`, `authenticate_api_company_admin_user!`.

If want to use `authenticate_user!`, `authenticate_admin_user!`, `authenticate_company_admin_user!` we need to write the routes like below:
~~~ rb
  mount_devise_token_auth_for 'User', at: 'api/user_auth', controllers: {
    registrations: 'api/user_auth/registrations',
    omniauth_callbacks: "api/user_auth/omniauth_callbacks",
    sessions: 'api/user_auth/sessions'
  }
  mount_devise_token_auth_for 'AdminUser', at: 'api/admin_user_auth', controllers: {
    registrations: 'api/admin_user_auth/registrations', # Maybe not needed
    sessions: 'api/admin_user_auth/sessions'
  }
  mount_devise_token_auth_for 'CompanyAdminUser', at: 'api/company_admin_user_auth', controllers: {
    registrations: 'api/company_admin_user_auth/registrations', # Maybe not needed
    sessions: 'api/company_admin_user_auth/sessions'
  }
~~~
And add below too.
/app/controllers/application_controller.rb
~~~ rb
class ApplicationController < ActionController::API
  include DeviseTokenAuth::Concerns::SetUserByToken
  before_action :configure_permitted_parameters, if: :devise_controller?

  # https://github.com/lynndylanhurley/devise_token_auth/issues/1165
  # or change route mounting and use authenticate_api_admin_user! in Api::Manager::ApplicationController
  def configure_permitted_parameters
    self.class.serialization_scope :view_context
  end
end
~~~

### Omniauth:
Though we are using Rails api only mode, but to add omniauth we need to add back below:

/config/application.rb
~~~ rb
# https://edgeguides.rubyonrails.org/api_app.html#using-session-middlewares
# https://github.com/lynndylanhurley/devise_token_auth/issues/183
# https://github.com/omniauth/omniauth#integrating-omniauth-into-your-rails-api
config.session_store :cookie_store, key: '_interslice_session'
config.middleware.use ActionDispatch::Cookies # Required for all session management
config.middleware.use ActionDispatch::Session::CookieStore, config.session_options
~~~

The order of the routes being called is:
`GET /api/user_auth/line` -> This will show the line login permission page
`GET /omniauth/line/callback` -> The callback url which needs to be set in line developer console
The routes above points to this controller action: `/api/user_auth/omniauth_callbacks#redirect_callbacks` and in this action, it will redirect to `GET /api/user_auth/:provider/callback` action.

Example of RSpec test:
~~~ rb
require "rails_helper"

RSpec.describe "Api::UserAuth::OmniauthCallbacks", type: :request do
  # /api/user_auth/omniauth_callbacks#redirect_callbacks
  # The callback url which needs to be set in line developer console: '/omniauth/line/callback'
  describe "GET|POST /omniauth/:provider/callback" do
    subject do
      get "/omniauth/line/callback", params: params
    end

    let(:params) do
      {
        code: "aezhFGC0tnynANAllyrd",
        state: "c982d7d95c93311b7e7c2709d724602396e471758aeb72de"
      }
    end

    before do
      Rails.application.env_config["devise.mapping"] = Devise.mappings[:user]
      Rails.application.env_config["omniauth.params"] = { 'namespace_name' => nil, 'resource_class' => 'User' }
      Rails.application.env_config["omniauth.auth"] = OmniAuth.config.mock_auth[:line]
    end

    it 'redirects to "/api/user_auth/line/callback"' do
      subject
      expect(response).to redirect_to("/api/user_auth/line/callback")
    end

  end

  # /api/user_auth/omniauth_callbacks#omniauth_success
  describe "GET /api/user_auth/:provider/callback" do
    subject do
      get "/api/user_auth/line/callback"
    end

    before do
      Rails.application.env_config["devise.mapping"] = Devise.mappings[:user]
      Rails.application.env_config["omniauth.params"] = { 'namespace_name' => nil, 'resource_class' => 'User' }
      Rails.application.env_config["omniauth.auth"] = OmniAuth.config.mock_auth[:line]
      allow_any_instance_of(Api::UserAuth::OmniauthCallbacksController).to receive(:auth_hash).and_return(
        OmniAuth::AuthHash.new({
          provider: 'line',
          uid: "U7ff0a35417f9561d2c777a49811cee41",
          info: OmniAuth::AuthHash::InfoHash.new({
            description: nil, image: "https://profile.line-scdn.net/xxxxxxxx", name: 'test'
          }),
          credentials: OmniAuth::AuthHash.new({
            expires: true,
            expires_at: 1621503793,
            refresh_token: 'O15mfUjnB8Skrat1clH1" token="eyJhbGciOiJIUzI1NiJ9._unXaPIdwNh45xWA6m2Fx6ORhK4CqxROPmlpIsRGjA7Rt5OCLIZpUTKMVlvgx4827kkt3B6LJGrOm4UgALA3BCxRJa6846PyweiGx1CJey4UoE-_NrHDADaZBJRgNpT9tgLr6W6NOO2Fhstg18xGLTWrvcCw-TrTK28qGsqKgWU.9pMoia89D8yMtpE70VgWF2P5--42sTkaByGxU00D2zo'
          })
        })
      )
    end

    it 'creates or updates a user, sets auth header (signin) and renders user with auth data' do
      expect do
        subject
      end.to change{
        User.count
      }.by(1)
      user = User.last
      expect(response.status).to eq(200)
      expect(response.headers["uid"]).to be_present
      expect(response.headers["access-token"]).to be_present
      expect(response.headers["client"]).to be_present
      result = JSON.parse(response.body)
      expect(result["user"]["id"]).to eq(user.id)
      expect(result["user"]["provider"]).to eq("line")
      expect(result["user"]["uid"]).to eq("U7ff0a35417f9561d2c777a49811cee41")
      expect(result["user"]["name"]).to eq("test")
      expect(result["user"]["email"]).to eq(nil)
      expect(result["user"]["auth_token"]).not_to be_blank
      expect(result["user"]["client_id"]).not_to be_blank
      expect(result["user"]["expiry"]).not_to be_blank
      expect(result["user"]["config"]).to be(nil)
      expect(result["user"]["oauth_registration"]).to be_truthy
    end
  end

  # /api/user_auth/omniauth_callbacks#omniauth_failure
  describe "GET /api/user_auth/failure" do
    subject do
      get "/api/user_auth/failure"
    end

    it 'renders error message' do
      subject
      expect(response.status).to eq(401)
      result = JSON.parse(response.body)
      expect(result['errors']).to eq(I18n.t('errors.messages.line_login_error'))
    end
  end
end
~~~

### Override the default controllers:
For several reasons we have to override some of the default controllers:
- As an API only rails application, we only want it to render json result.
- We want a customized error messages format.
- To add some status management for admin_users ('invited'/'enabled'/'disabled'), only 'enabled' ones can login, others will see customized error messages, etc.

Example of overriding 1 of the SessionsController: (We don't use confirmable and lockable therefore those parts are commented out.)
~~~ rb
module Api
  module AdminUserAuth
    class SessionsController < DeviseTokenAuth::SessionsController
      # Check if status is 'enabled' before sign_in
      def create
        field = (resource_params.keys.map(&:to_sym) & resource_class.authentication_keys).first

        @resource = nil
        if field
          q_value = get_case_insensitive_field_from_resource_params(field)

          @resource = find_resource(field, q_value)
        end

        if @resource&.enabled? && valid_params?(field, q_value) && (!@resource.respond_to?(:active_for_authentication?) || @resource&.active_for_authentication?)
          valid_password = @resource.valid_password?(resource_params[:password])
          if (@resource.respond_to?(:valid_for_authentication?) && !@resource.valid_for_authentication? { valid_password }) || !valid_password
            return render_create_error_bad_credentials
          end
          @token = @resource.create_token
          @resource.save

          sign_in(:user, @resource, store: false, bypass: false)

          yield @resource if block_given?

          render_create_success
        elsif @resource&.invited?
          render_invited_error
        elsif @resource&.disabled?
          render_disabled_error
        # Confirmable and lockable are not used
        # elsif @resource && @resource.enabled? && !(!@resource.respond_to?(:active_for_authentication?) || @resource.active_for_authentication?)
        #   if @resource.respond_to?(:locked_at) && @resource.locked_at
        #     render_create_error_account_locked
        #   else
        #     render_create_error_not_confirmed
        #   end
        else
          render_create_error_bad_credentials
        end
      end

      private

      def render_create_error_bad_credentials
        render json: {
          success: false,
          errors: { messages: [I18n.t('devise_token_auth.sessions.bad_credentials')] }
        }, status: :unauthorized
      end

      def render_disabled_error
        render json: {
          success: false,
          errors: { messages: [I18n.t('devise_token_auth.sessions.disabled_account')] }
        }, status: :unauthorized
      end

      def render_invited_error
        render json: {
          success: false,
          errors: { messages: [I18n.t('devise_token_auth.sessions.not_enabled', email: @resource.email)] }
        }, status: :unauthorized
      end
    end
  end
end
~~~

### Validations:
To add custom validations like scoped uniqueness of email, we need to Stop using devise_token_auth UserOmniauthCallbacks because it requires unique email with no scope, will need to copy only what each needs in User, AdminUser, CompanyAdminUser models.

/config/initializers/devise_token_auth.rb
~~~ rb
...
config.default_callbacks = false
...
~~~

