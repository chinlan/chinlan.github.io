---
layout: post
title: "Upgrade terraform"
description: ""
categories: [devops]
tags: [Terraform]
redirect_from:
  - /2021/03/23/
---

~~~
Error: Error loading state: state snapshot was created by Terraform v0.13.5, which is newer than current v0.12.28; upgrade to Terraform v0.13.5 or greater to work with this state
~~~
上記エラーがあるので、ローカルのバーションを0.13.5以上にあげることが必要です。

`versions.tf`について：
- 0.13の新しい仕様です。
- `terraform 0.13upgrade`後versions.tfが自動作成されます
https://www.terraform.io/docs/cli/commands/0.13upgrade.html

### プロセス記録:
（将来アップグレードのときに参考にできます）
brew upgrade terraform
最新は0.14なので、いきなり0.14.8にあげてしまいました。（公式サイトではこんなスキップのやり方は非推薦です）
下記みたいに、先に0.13に戻ってから、0.14にupgradeするとのエラーメッセージが出ました。
<img src="https://user-images.githubusercontent.com/14109108/112117334-3ad22b80-8bf6-11eb-92cb-e411fbadcdc5.png" width=500 />
なので、0.13をもダウンロードして：
brew install terraform@0.13
<img src="https://user-images.githubusercontent.com/14109108/112117596-7967e600-8bf6-11eb-9bf0-8831f92f7e43.png" width=500 />
0.14のバーションをunlinkして：
brew unlink terraform
<img src="https://user-images.githubusercontent.com/14109108/112117762-a5836700-8bf6-11eb-8878-193379d33a88.png" width=400 />
0.13のバーションにlinkする：
brew link terraform@0.13
<img src="https://user-images.githubusercontent.com/14109108/112117878-c9df4380-8bf6-11eb-986a-1fed0162e204.png" width=400 />
これでterraform --versionのときに0.13.6で示すようになりました。
試しにstateについてリクエスト見ます：
<img src="https://user-images.githubusercontent.com/14109108/112118216-19257400-8bf7-11eb-895a-555a5a8b98d8.png" width=400 />
<img src="https://user-images.githubusercontent.com/14109108/112118281-2b9fad80-8bf7-11eb-94bb-c1a5b8c9ae7b.png" width=400 />
警告メッセージについて修正した後：
<img src="https://user-images.githubusercontent.com/14109108/112118419-512cb700-8bf7-11eb-808b-dfc4a2518aea.png" width=400 />
<img src="https://user-images.githubusercontent.com/14109108/112118477-61449680-8bf7-11eb-870e-568cbb754bff.png" width=400 />
<img src="https://user-images.githubusercontent.com/14109108/112118557-74576680-8bf7-11eb-875e-4cf400f4b5a9.png" width=400 />
<img src="https://user-images.githubusercontent.com/14109108/112118628-85a07300-8bf7-11eb-8029-01d93e745598.png" width=400 />


[オフィシャルアップグレード指南](https://github.com/hashicorp/terraform/blob/main/website/upgrade-guides/0-13.html.markdown)

`tfenv`や`tfswitch`とかを使って、複数のバーションを同時に違うプロジェクトで使えるようにするのもいいかもしれません：
https://stackoverflow.com/questions/56283424/upgrade-terraform-to-specific-version
