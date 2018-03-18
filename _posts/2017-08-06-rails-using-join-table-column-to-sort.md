---
layout: post
title: "Rails: 使用join table的欄位排序"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/06/
---

使用情境：

Plan has_many :tags, through: :tags_plans, -> { joins(:tags_plans).where(deleted: false).order(“tags_plans.position”) }

這裡要使用join還是includes取決於使用情境，includes會將關聯records 全部載入，joins不會。

所以如果只是在query的時候要用到關聯model的欄位，那就不需要用到includes，

includes會載入所有records，joins只載入那些和被join的model有關聯的records。

所以Plan.joins(:tags_plans)只會載入那些和tags_plans有關聯的plans

Plan.includes(:tags_plans)會載入全部的plans
