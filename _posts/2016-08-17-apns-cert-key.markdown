---
layout: post
title: "Apns-cert-key"
date: 2016-08-17T13:57:50+08:00
categories: iOS,备忘,Tips
---

在生成APNS证书的时候，不要Key与Cert分开来生成，直接从keychain中选中cert和私钥，一起导出为p12，比如开发证书命名为apns_dev.p12

~~~
openssl pkcs12 -in apns_dev.p12 -out apns_dev.pem -nodes -clcerts
~~~

生成之后，向apple apns服务器发送命令验证证书是否正确：

~~~
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert apns_dev.pem -key apns_dev.pem
~~~

开发证书向开发证书发送，正式证书向正式环境发送。