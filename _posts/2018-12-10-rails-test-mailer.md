---
layout: post
title: "Rails: test mailer"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/12/10/
---

Basic version:
~~~ruby
describe 'notify_withdrawal' do
  let(:mail) { described_class.notify_withdrawal(user, email: user.email) }

  it 'renders the headers' do
    expect(mail.subject).to eq(
      I18n.t('user_mailer.notify_withdrawal.subject', locale: locale)
    )
    expect(mail.to).to eq([user.email])
    expect(mail.from).to eq(['support@example.com'])
  end

  it 'renders the body' do
    expect(mail.body.encoded).to match(user.name)
  end
end
~~~

[ref1](https://blog.morizyun.com/blog/action-mailer-rails-rspec-test/index.html)

When having multipart templates:
~~~ruby
expect(mail.text_part.body.encoded).to match(expire_date)
expect(mail.html_part.body.encoded).to match(@user.name)​
~~~

[ref2](https://stackoverflow.com/questions/29454102/in-rails-why-is-my-mail-body-empty-only-in-my-test)


In test environment, writing `XXXMailer.send_mail(aug)` seems not really executing the content in send_mail action, for example, if you update a column value in this action and wanting to rspec test it. You will need to add `deliver` action by writing `expect { XxxMailer.send_mail(aug).deliver }.to change { column }.from(0).to(1)`

[ref3](https://stackoverflow.com/questions/8339957/actionmailer-test-failed-but-works-in-development-environment)

When test deliver_now:
~~~ruby
expect { subject }.to change { ActionMailer::Base.deliveries.count }.by(1)
~~~

When test deliver_later:
~~~ruby
expect {
YourMailer.your_method.deliver_later
}.to have_enqueued_job.on_queue('mailers')
~~~
or
~~~ruby
expect { YourMailer.your_method.deliver_later }
  .to have_enqueued_job(ActionMailer::DeliveryJob).with(
    'YourMailer', 'your_method', 'deliver_now',
    aug1, aug2, aug3
  )
~~~
