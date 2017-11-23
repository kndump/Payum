# 开发一个自定义的Payum网关

本章节对于想要基于payum开发一个自定义网关的开发者是很有帮助的。
Payum提供了一个骨架式的项目帮助我们了解如何实现。

1. 创建一个新的项目

```bash
$ composer create-project payum/skeleton
```

2. 把所有出现 `payum` 的地方替换为你自己的vendor名称。这可能是你的github用户名，我们先假设你选用的是 `acme`。
3. 把所有出现 `skeleton` 的地方替换为网关名称，例如 Stripe，Paypal等。我们先假设你选用的是 `paypal`。
4. 向payum的构建器注册一个网关工厂，并创建一个网关。

```php
<?php

use Payum\Core\PayumBuilder;

$defaultConfig = [];

$payum = (new PayumBuilder)
    ->addGatewayFactory('paypal', new \Acme\Paypal\PaypalGatewayFactory($defaultConfig))

    ->addGateway('paypal', [
        'factory' => 'paypal',
        'sandbox' => true,
    ])

    ->getPayum()
;
```

5. 自定义的网关要实现所有接口中定义的方法，否则在使用时会抛出 `Not implemented` 异常：

```php
<?php

use Payum\Core\Request\Capture;

/** @var \Payum\Core\Payum $payum */
$paypal = $payum->getGateway('paypal');

$model = new \ArrayObject([
  // ...
]);

$paypal->execute(new Capture($model));
```

返回 [首页](index.md).