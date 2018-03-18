---
layout: post
title: "為放在s3的靜態網站設定CloudFront https連線"
description: ""
categories: [AWS]
tags: [s3, CloudFront]
redirect_from:
  - /2017/08/07/
---

使用情境：

主要網站：www.example.com

新建了一個靜態宣傳網站：beginner.example.com

### 1. S3設定：
將這個網站放到AWS S3，bucket name: beginner

選到beginner bucket，照以下的選單順序選下去

`s3 bucket -> properties -> static website hosting`

選擇`Use this bucket to host a website`

設定`index document: index.html`

    `Error document: error.html`

（確定本來靜態網站的source裡有這兩個頁面）

這裡要記錄下頁面上出現的Endpoint url內容

`http://beginner.example.co.jp.s3-website-ap-northeast-1.amazonaws.com`

選單的permissions的地方設定Public access為Everyone


### 2. Route 53設定

`Route 53 -> Hosted zones-> Create Record Set` （如果還沒有建立的話）

已經設定好的話就直接選到原本的主網站example 

然後在其下建立sub domain: beginner.example.com


### 3. 接下來做CloudFront的設定：

切換到CloudFront

Create Distribution -> web

origin設定為剛剛在S3那邊的endpoint位址，

behavior選擇Redirect Http to Https,

CNAMES填上beginner.example.com

(這邊如果不設定好CNAME, Route 53那邊要選alias target時這個distribution就不會被列在選單裡）

Custom SSL Certificate: 如果之前有import到ACM裏的話，可以直接選

（有時候剛import需要等一陣子才會生效），或者如果手邊沒有現成的話，也可以在ACM裡產生一個

root object: index.html（指定靜態網頁的首頁檔案）

Route 53選到剛建立的beginner.example.com，在alias target的地方選到剛剛的distribution domain name後存檔


### 4. S3 bucket policy那邊要做這樣的修改：

root object: index.html（指定靜態網頁的首頁檔案）

產生出新的Distribution後，記下他的domain name


### 5. Route 53 選到剛作成的beginner.yamap.co.jp，在alias target的地方選到剛剛的distribution domain name後存擋


### 6. s3 bucket policy那邊要做這樣的修改(主要是第二段Sid: "2"那邊）：
~~~~~~~~~
{

    "Version": "2012-10-17",
    "Id": "Policy1499743576985",
    "Statement": [
        {
            "Sid": "Stmt1499743572498",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::270040163539:user/xxxxxx"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::beginner.example.co.jp"
        },
        {
            "Sid": "2",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::beginner.example.co.jp/*"
        }
    ]
}
~~~~~~~~~~

### 7. 最後用以下兩個路徑去access放在S3裡面的圖片確認是否設定成功，

（CloudFront Distribution的deploy需要比較長時間，每次修改完設定都要等到狀態變成deployed之後才算修改完成）

- s3-ap-northeast-1.amazonaws.com/beginner.example.co.jp/images/image.jpg

- (Your-CloudFront-Distribution-Domain)/images/image.jpg


# 關於如何refresh CloudFront cache:

有兩種方式：

- Object invalidation

    進入CloudFront主控台，選到對的distribution, create new invalidation

    不管是invalid單一個物件，還是米字號一整個資料夾含好幾個物件

    一條invalidation都只算一件

    一個月可免費建立千件，超過就須計費了

- Versioned object names

    在更新內容時，為物件改名，加上版本號，例如：image -> imagev2

     AWS推薦的方法是第二種，因為沒有需計費的可能，又方便管理內容的差異
