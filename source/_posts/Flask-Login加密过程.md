---
title: Flask-Login加密过程
date: 2018-09-21 15:12:15
tags:
---

草稿:

flask-login类库涉及到flask的session对象，flask`session`的源码在`flask.sessions`中，其中最重要的是`open_session`,`save_session`函数。顾名思义，`open_session`是解密的函数，`save_session`是加密的函数, 加解密使用的是`itsdangerous`库，使用到的是`URLSafeTimedSerializer`类。`URLSafeTimedSerializer`中重要的函数可以从以下几个开始看:

* `load_payload`: decode的数据
* `dump_payload`：encode数据
*  `make_signer`: 创建一个签名类实例，调用`sign`和`unsign`方法来给字符串签名，以及验证签名合法，raise `BadSignature`，默认会以`b'.'`来将encode的数据和签名连接起来作为加密后的值。

### `dump_payload`

源码位于`URLSafeSerializerMixin.dump_payload`, 具体过程如下:

* 调用超类的`dump_payload`即json.dumps(obj, separators=(',', ':'))
* 得到json再调用`zlib.compress`进行压缩， 如果压缩后的长度比原来小就替换
* 调用`itsdangerous.base64_encode`函数将json字符串encode，该函数主要是调用`base64.urlsafe_b64encode(b'xxx').strip(b'=')
* 如果json是被压缩过的，在json前加一个b'.', base64d = b'.' + base64d，然后将字符串return

### `load_payload`

是`dupm_payload`的反过程

### `sign`

### `unsign`





