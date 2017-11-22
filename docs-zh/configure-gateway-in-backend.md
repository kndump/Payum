# 在后端配置网关

在 [开始章节](get-it-started.md) 我们已经展示了如何在代码中配置网关。
有时候，你可能会想要在其他地方比如数据库中存储网关（大部分情况下是网关凭据）。
以便管理员可以在后台对它们进行编辑。
这里就展示了如何实现这种效果的基础示例。
   
## 配置

首先，我们需要创建一个存储网关信息的实体。
该模型必须必须实现 `Payum\Core\Model\GatewayConfigInterface`。

_**说明**: 在本章节，我们使用 DoctrineStorage._

```php
<?php
namespace Acme\Payment\Entity;

use Doctrine\ORM\Mapping as ORM;
use Payum\Core\Model\GatewayConfig as BaseGatewayConfig;

/**
 * @ORM\Table
 * @ORM\Entity
 */
class GatewayConfig extends BaseGatewayConfig
{
    /**
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="IDENTITY")
     *
     * @var integer $id
     */
    protected $id;
}
```

现在，我们需要创建一个存储，并基于此构建payum。

```php
<?php
//config.php

use Payum\Core\Bridge\Doctrine\Storage\DoctrineStorage;
use Payum\Core\PayumBuilder;
use Payum\Core\Payum;
use Payum\Core\Registry\DynamicRegistry;

// $objectManager 是一个doctrine对象管理的实例.

$gatewayConfigStorage = new DoctrineStorage($objectManager, 'Acme\Payment\Entity\GatewayConfig');

/** @var Payum $payum */
$payum = (new PayumBuilder())
    ->addDefaultStorages()
    ->setGatewayConfigStorage($gatewayConfigStorage)

    ->getPayum()
;
```

## 保存网关配置

```php
<?php
//create_config.php

include __DIR__.'/config.php';

/** @var \Payum\Core\Storage\StorageInterface $gatewayConfigStorage */

$gatewayConfig = $gatewayConfigStorage->create();
$gatewayConfig->setGatewayName('paypal');
$gatewayConfig->setFactoryName('paypal_express_checkout_nvp');
$gatewayConfig->setConfig(array(
    'username' => 'EDIT ME',
    'password' => 'EDIT ME',
    'signature' => 'EDIT ME',
    'sandbox' => true,
));

$gatewayConfigStorage->update($gatewayConfig);
```

## 使用网关

```php
<?php
// prepare.php

include __DIR__.'/config.php';

/** @var \Payum\Core\Payum $payum */
$gateway = $payum->getGateway('paypal');
```

返回 [首页](index.md).

 
 

