# 在数据库中加密存储网关配置

为了加密（和稍后解密）敏感的配置信息（比如支付提供者凭证），我们需要做两件事情：

* 保证模型实现了 `CryptedInterface`。`GatewayConfig` 类以及实现了该接口。
* 创建一个加密器的实例。
* 将存储包装到 `CryptoStorageDecorator` 装饰器中。
 
首先，我们要安装一个用于加密的库 `defuse/php-encryption`。

```bash
$ composer require defuse/php-encryption:^2
```

加密库安装好之后，我们就可以配置存储了：

```php
<?php
namespace Acme;

use Payum\Core\Storage\CryptoStorageDecorator;
use Payum\Core\PayumBuilder;
use Payum\Core\Payum;

/** @var \Payum\Core\Storage\StorageInterface $realStorage */

// 加密参数需要被保存下来，在以后会被用到
$secret = \Defuse\Crypto\Key::createNewRandomKey()->saveToAsciiSafeString();
$cypher = new \Payum\Core\Bridge\Defuse\Security\DefuseCypher($secret);

$gatewayConfigStorage = new CryptoStorageDecorator($realStorage, $cypher);

/** @var Payum $payum */
$payum = (new PayumBuilder())
    ->addDefaultStorages()
    ->setGatewayConfigStorage($gatewayConfigStorage)

    ->getPayum()
;
```

返回 [首页](index.md).
