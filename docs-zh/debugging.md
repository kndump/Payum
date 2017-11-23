# 调试

我必须承认，payum的架构致使它比较难调试（每个行为自己决定它是否支持请求，一个行为可以委托一些工作给其他行为，等等）。
为了解决难以调试的问题，我们实现了 `LogExecutedActionsExtension`。
它会记录所有被执行的行为的一些细节。
你只需要遵循PSR-3规范添加该扩展，稍后再检查日志文件。

_**注意**: 你可以筛选 `[Payum]` 的日志，比如使用 `grep` 工具._

```php
<?php
use Payum\Core\Bridge\Psr\Log\LogExecutedActionsExtension;
use Payum\Core\Tests\Mocks\Action\CaptureAction;
use Payum\Core\Gateway;
use Payum\Core\Request\Capture;

/** @var \Psr\Log\LoggerInterface $logger */

$gateway = new Gateway;
$gateway->addExtension(new LogExecutedActionsExtension($logger));
$gateway->addAction(new CaptureAction);

$gateway->execute(new Capture($model = new \stdClass));
```

这里是一个日志可能包含的内容的示例：

```
DEBUG - [Payum] 1# Payum\Core\Action\StatusDetailsAggregatedModelAction::execute(GetHumanStatus{model: Token})
DEBUG - [Payum] 2# Payum\Payex\Action\PaymentDetailsStatusAction::execute(GetHumanStatus{model: PaymentDetails})
DEBUG - [Payum] 1# Payum\Core\Action\CaptureDetailsAggregatedModelAction::execute(Capture{model: Token})
DEBUG - [Payum] 2# Payum\Payex\Action\PaymentDetailsCaptureAction::execute(Capture{model: PaymentDetails})
DEBUG - [Payum] 3# Payum\Payex\Action\Api\InitializeOrderAction::execute(InitializeOrder{model: ArrayObject})
DEBUG - [Payum] 3# InitializeOrderAction::execute(InitializeOrder{model: ArrayObject}) throws reply HttpRedirect{url: https://test-confined.payex.com/PxOrderCC.aspx?orderRef=7cbefc70ff294fd194d2411f457423d6}
DEBUG - [Payum] 2# PaymentDetailsCaptureAction::execute(Capture{model: PaymentDetails}) throws reply HttpRedirect{url: https://test-confined.payex.com/PxOrderCC.aspx?orderRef=7cbefc70ff294fd194d2411f457423d6}
DEBUG - [Payum] 1# CaptureDetailsAggregatedModelAction::execute(Capture{model: PaymentDetails}) throws reply HttpRedirect{url: https://test-confined.payex.com/PxOrderCC.aspx?orderRef=7cbefc70ff294fd194d2411f457423d6}
```

如你所见，它列出了被执行行为的调用栈。
同时，它也列出了请求的一些细节。
例如，它可以列出与请求相关的模型类。

返回 [首页](index.md).