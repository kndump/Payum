# 处理敏感信息

所有的敏感信息（信用卡号，cvv，持卡人姓名等）应该被直接提交给网关。
不允许保存这些信息，甚至临时保存也不允许。
如果你想要保存这些敏感信息，必须参照[PCI SSC 数据安全准则](https://www.pcisecuritystandards.org/security_standards/)。
这是一个很有挑战性的任务，而且超出了本章节的范围。
我们会在这里描述一些帮助你避免无意保存了敏感信息的实战经验。

所有的信息，像信用卡，必须被`SensitiveValue`类进行封装。

```php
<?php

use Payum\Core\Security\SensitiveValue;

$cardNumber = new SensitiveValue('theCreditCardNumber');

serialize($cardNumber);
//会返回null

clone $cardNumber;
// 会抛出异常

$cardNumber->erase();
// 永远移除该值.

$cardNumber->get();
// 获取敏感值并擦除它

$cardNumber->peek();
// 获取敏感值，但不擦除它。应该小心使用该方法。

(string) $cardNumber;
// 会返回空字符串

json_encode($cardNumber);
// 会返回{}

var_dump($cardNumber);
// 不会打印出敏感数据
```

所有已被支持的网关都会意识到这个类，并且会安全的处理它。

返回 [首页](index.md).
