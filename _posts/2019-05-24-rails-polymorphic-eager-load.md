---
layout: post
title: "Rails: polymorphic eager load"
description: ""
categories: [rails]
tags: [polymorphic eager-load]
redirect_from:
  - /2019/05/24/
---

Context:
The website will held `Campaign`,
users can entry their `Image` or `Activity` to the `Campaign` as a `CampaignEntryContent`.

When trying to search a column from a polymorphic association (search by activities.title on CampaignEntryContent#index):
~~~ruby
CampaignEntryContent.joins(:content).where(content: { deleted: false }).count
~~~
will cause below error:
~~~
`ActiveRecord::EagerLoadPolymorphicError: Cannot eagerly load the polymorphic association :content`
~~~

If adding below setting in addition to the conventional polymorphic setting in CampaignEntryContent model:
~~~ruby
belongs_to :activity, -> { where(campaign_entry_contents: {content_type: 'Activity'}) }, foreign_key: 'content_id'

def activity
return unless content_type == "Activity"
   super
end
~~~

Then `CampaignEntryContent.joins(:activity).where(activities: { deleted: false }).count` can work.

But if we want to do something like:
~~~ruby
CampaignEntryContent.joins(:activity).where('activities.title like ?', "%#{params[:keyword]}%").count
~~~
It fails again.
~~~
missing FROM-clause entry for table "campaign_entry_contents"
~~~
To make it work, will have to do a join like below:
~~~ruby
CampaignEntryContent.where(content_type: "Activity").joins("INNER JOIN activities ON campaign_entry_content.content_id = activities.id").where('activities.title like ?', "%#{params[:keyword]}%").count
~~~



[ref](https://stackoverflow.com/questions/680141/activerecord-querying-polymorphic-associations)
