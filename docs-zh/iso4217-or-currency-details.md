# ISO4217 或 币种细节 

Payum 提供了获取在[ISO4217](http://en.wikipedia.org/wiki/ISO_4217)规范中被列出的币种细节的能力。
为了获取信息，你需要执行一个针对具体币种代码的 GetCurrency 请求。


```php
<?php

use Payum\Core\Request\GetCurrency;

$factory = new \Payum\Offline\OfflineGatewayFactory();
$gateway = $factory->create();

$gateway->execute($currency = new GetCurrency('USD'));

echo $currency->alpha3;  // USD
echo $currency->name;    // US Dollar
echo $currency->exp;     // 2
echo $currency->country; // US

// 等等信息...
```

或者在其他行为内部实现：

```php
<?php

use Payum\Core\Action\ActionInterface;
use Payum\Core\GatewayAwareInterface;
use Payum\Core\GatewayAwareTrait;
use Payum\Core\Request\GetCurrency;

class FooAction implements ActionInterface, GatewayAwareInterface
{
    use GatewayAwareTrait;
    
    public function execute($request)
    {
        $this->gateway->execute($currency = new GetCurrency('USD'));
        
        echo $currency->alpha3;  // USD
        echo $currency->name;    // US Dollar
        echo $currency->exp;     // 2
        echo $currency->country; // US
    }
    
    public function supports($request) {
        // TODO: Implement supports() method.
    }
}
```

再或者直接使用 ISO4217 服务:

```php
<?php

use Payum\ISO4217\ISO4217;

$iso4217 = new ISO4217();

$currency = $iso4217->findByAlpha3('USD');

echo $currency->getAlpha3();  // USD
echo $currency->getName();    // US Dollar
echo $currency->getExp();     // 2
echo $currency->getCountry(); // US
```

## 继续阅读 

* [应用架构](the-architecture.md).
* [已支持的网关](supported-gateways.md).
* [存储](storages.md).

返回 [首页](index.md).
