# 信用卡号码打码

在处理信用卡的方式上，会经常需要对卡号或持卡人姓名进行打码。
这里有一个类可以实现此目的。
它也可以配置打码的标识和展示的长度。

```php
<?php

use Payum\Core\Security\Util\Mask;

echo Mask::mask("3456-7890-1234-5678");
// 3XXX-XXXX-XXXX-5678

echo Mask::mask("4567890123456789", "*");
// 4***********6789

echo Mask::mask("4928-9012-abcd-3456", null, 8);
// 4XXX-XXXX-abcd-3456
```

返回 [首页](index.md).