# 便捷的支付通知

通知就是一个回调。
网关会回调通知你，以便让你知道支付状态的变化。
它可能是由于一笔退款或支付被处理完成。
该流程图展示了两个便捷通知的示例：

![notification](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=cGFydGljaXBhbnQgUGF5cGFsCgAHDGNhcHR1cmUucGhwAAsNbm90aWZ5ABIFCgAZCy0-KwA_BjogYSBwdXJjYWhzZQoAUgYtPi0AQws6IHBlbmRpbmcAFggtPgBKCjogc3VjY2VzcwBiBmljYXRpb24AMTkARgcAVBZjYW5jZWxlZCAodXNlciB2b2lkIG9uIHAAggcFIHNpZGUp&s=default)

如果你参照 [开始章节](get-it-started.md)的描述使用payum构建器创建了一个paypal网关，
你就不需要再关心回调地址了，payum已经为你实现了。
你只需要保证 [回调脚本](examples/notify-script.md)是对外网公开的就可以。

一旦接收到通知，模型也会自动更新。
你所需要做的就是添加一个扩展来检测支付状态的变更，并对此做出合适的处理。

这是一个扩展的示例：

```php
<?php

use Payum\Core\Extension\Context;
use Payum\Core\Extension\ExtensionInterface;
use Payum\Core\Model\PaymentInterface;
use Payum\Core\Request\Generic;
use Payum\Core\Request\GetHumanStatus;
use Payum\Core\Request\GetStatusInterface;

class PaymentStatusExtension implements ExtensionInterface
{
    /**
     * {@inheritDoc}
     */
    public function onPostExecute(Context $context)
    {
        $request = $context->getRequest();
        if (false == $request instanceof Generic) {
            return;
        }
        if ($request instanceof GetStatusInterface) {
            return;
        }

        $payment = $request->getModel();
        if (false == $payment instanceof PaymentInterface) {
            return;
        }

        $context->getGateway()->execute($status = new GetHumanStatus($payment));

        // 检查状态并做出合适的处理
    }

    /**
     * {@inheritDoc}
     */
    public function onPreExecute(Context $context)
    {
    }

    /**
     * {@inheritDoc}
     */
    public function onExecute(Context $context)
    {
    }
}
```

返回 [首页](index.md).
