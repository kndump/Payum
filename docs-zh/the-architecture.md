# 应用架构

下面展示的代码片段仅用于演示目的（伪代码）。
它们是为了说明各种任务的一般实现方式。
如果要了解更实用的例子，可以参考在它们下面给出的链接。
一般来说，你需要创建一个_[请求][base-request]_，为了知道该如何处理这些请求还需要实现_[行为][action-interface]_。
使用实现了_[网关接口][gateway-interface]_的_网关_，所有的事情都是经由网关处理的。
网关接口强制要求我们指定执行`请求`所使用的所有可能的`行为`。
所以说，网关是`请求`和`行为`交汇的场所。

_**说明**: 如果你想了解更实用的示例，我们提供了沙箱环境: [在线演示][sandbox-online], [代码][sandbox-code]._

```php
<?php
use Payum\Core\Gateway;
use Payum\Core\Request\Capture;

$gateway = new Gateway;
$gateway->addAction(new CaptureAction);

//CaptureAction 真正完成相应工作
$gateway->execute($capture = new Capture(array(
    'amount' => 100,
    'currency' => 'USD'
)));

var_export($capture->getModel());
```

```php
<?php
use Payum\Core\Action\ActionInterface;
use Payum\Core\Request\Capture;

class CaptureAction implements ActionInterface
{
    public function execute($request)
    {
       $model = $request->getModel();

       //在这里实现真正`捕获`支付的逻辑

       $model['status'] = 'success';
       $model['transaction_id'] = 'an_id';
    }

    public function supports($request)
    {
        return $request instanceof Capture;
    }
}
```

上面展示的是一个大致的流程。现在我们来讨论一下细节：

_**链接**: 参阅一个更实用的例子: [CaptureController][capture-controller]._

## 子请求

一个`行为`并不想单独完成所有的工作，因此它会委托一些任务给其他`行为`。
为了达到这种目的，这个`行为`必须是一个_有网关意识的(gateway aware)_`行为`。
只有这样，它才能创建出一个子请求继续传递给网关。

```php
<?php
use Payum\Core\Action\ActionInterface;
use Payum\Core\GatewayAwareInterface;
use Payum\Core\GatewayAwareTrait;

class FooAction implements ActionInterface, GatewayAwareInterface
{
    use GatewayAwareTrait;
    
    public function execute($request)
    {
        //do its jobs

        // delegate some job to bar action.
        $this->gateway->execute(new BarRequest);
    }
    
    public function supports($request) {
        // TODO: Implement supports() method.
    }
}
```

_**链接**: 参阅 paypal [CaptureAction][paypal-capture-action]._

## 响应

重定向或者信用卡提交表单又是怎么样的呢？
一些网关，例如Paypal ExpressCheckout，需要在它们自己的页面做认证。
Payum可以使用被我们称为_[响应][base-reply]_的东西来处理这种情况。
这是一个继承自异常的特殊对象，可以像异常一样被抛出。
你可以在任何时候抛出一个http重定向`响应`，并在最顶层捕获它。

```php
<?php
use Payum\Core\Action\ActionInterface;
use Payum\Core\Reply\HttpRedirect;

class FooAction implements ActionInterface
{
    public function execute($request)
    {
        throw new HttpRedirect('http://example.com/auth');
    }
    
    public function supports($request) {
        // TODO: Implement supports() method.
    }
}
```

上面我们看到了抛出了一个`响应`的`行为`。
这个`响应可以把用户重定向到一个新的链接。
下面的代码示例就演示了我们怎么捕获并处理这个响应。

```php
<?php

use Payum\Core\Reply\HttpRedirect;

try {
    /** @var \Payum\Core\Gateway $gateway */
    $gateway->addAction(new FooAction);

    $gateway->execute(new FooRequest);
} catch (HttpRedirect $reply) {
    header( 'Location: '.$reply->getUrl());
    exit;
}
```

_**链接**: 了解更实用的示例请参阅: [AuthorizeTokenAction][paypal-authorize-token-action]._

## 管理状态

一个好的状态处理是很重要的。
状态不应该被硬编码，且应该很容易被复用。为此，我们使用了_[请求状态接口][status-request-interface]_。
在我们的库中默认提供了一套[请求状态][status-request]，你也可以通过实现状态接口(status interface)来自由的使用你自己定义的状态。

```php
<?php
use Payum\Core\Action\ActionInterface;
use Payum\Core\Request\GetStatusInterface;

class FooAction implements ActionInterface
{
    public function execute($request)
    {
        if ('success condition') {
           $request->markCaptured();
        } else if ('pending condition') {
           $request->markPending();
        } else {
           $request->markUnknown();
        }
    }

    public function supports($request)
    {
        return $request instanceof GetStatusInterface;
    }
}
```

```php
<?php

use Payum\Core\Request\GetHumanStatus;

/** @var \Payum\Core\Gateway $gateway */
$gateway->addAction(new FooAction);

$gateway->execute($status = new GetHumanStatus);

$status->isCaptured();
$status->isPending();

// or

$status->getValue();
```

_**链接**: 状态逻辑可以像[paypal one][paypal-status-action]一样复杂，也可以像[authorize.net one][authorize-status-action]一样简单._

## 扩展

肯定应该有一种实现自有逻辑的网关的方式。
_[扩展][extension-interface]_就是用来干这个的。
我们来一起看下面的例子。
假设在用户要`捕获`一笔支付前，你想要检查一下用户权限：

```php
<?php
use Payum\Core\Extension\ExtensionInterface;
use Payum\Core\Extension\Context;

class PermissionExtension implements ExtensionInterface
{
    public function onPreExecute(Context $context)
    {
        $request = $context->getRequest();
        
        if (false == in_array('ROLE_CUSTOMER', $request->getModel()->getRoles())) {
            throw new Exception('The user does not have the required roles.');
        }

        // congrats, user has enough rights.
    }
    
    public function onExecute(Context $context) {
        // TODO: Implement onExecute() method.
    }
    
    public function onPostExecute(Context $context) {
        // TODO: Implement onPostExecute() method.
    }
}
```

```php
<?php

/** @var \Payum\Core\Gateway $gateway */
$gateway->addExtension(new PermissionExtension);

// here is the place where the exception may be thrown.
$gateway->execute(new FooRequest);
```

_**链接**: [存储扩展][storage-extension-interface] 是一个内建的扩展._

## 持久化模型

在被重定向到网关之前，你可能想要在某些地方存储下一些相关信息，对吧？
我们也考虑到了这种情况。
_[存储][storage-interface]_ 和它的 _[相关扩展][storage-extension-interface]_ 就是用来处理这种情况的。
存储扩展可以用来解决两种情况。
首先，它可以在请求被处理之后储存下该模型。
其次，它可以在请求被处理之前查询到相关模型。
当前已被支持的储存有[Doctrine][doctrine-storage] [Zend Table Gateway][zend-table-gateway] 和 [文件系统][filesystem-storage]（仅用于测试用途）

```php
<?php
use Payum\Core\Gateway;
use Payum\Core\Extension\StorageExtension;

/** @var \Payum\Core\Storage\StorageInterface $storage */
$storage = new FooStorage;

$gateway = new Gateway;
$gateway->addExtension(new StorageExtension($storage));
```

## 关于API

网关API有不同的版本？或者，网关本身提供了官方的SDK？
你知道吗，我们也已经考虑到了这些问题。

假设网关确实存在不同的版本，分别是：1.0和2.0。
我们想在`FooAction`中使用1.0的版本，而在`BarAction`中想要使用2.0的版本。
为了解决这个问题，我们需要在这两个行为中实现_有API意识的行为(API aware action)_。
当这种有API意识的行为被加入到网关中，它会一个接着一个的去尝试设置API，直到该行为接受了其中一个。

```php
<?php
use Payum\Core\ApiAwareInterface;
use Payum\Core\ApiAwareTrait;
use Payum\Core\Action\ActionInterface;
use Payum\Core\Exception\UnsupportedApiException;

class FooAction implements ActionInterface, ApiAwareInterface
{
    use ApiAwareTrait;
    
    public function __construct() 
    {
        $this->apiClass = Api::class;    
    }    
    
    
    public function execute($request) 
    {
        $this->api; // Api::class 
    }
    
    public function supports($request) {
        // TODO: Implement supports() method.
    }
}

class BarAction implements ActionInterface, ApiAwareInterface
{
    use ApiAwareTrait;
    
    public function __construct() 
    {
        $this->apiClass = AnotherApi::class;    
    }    
    
    
    public function execute($request) 
    {
        $this->api; // AnotherApi::class 
    }
    
    public function supports($request) {
        // TODO: Implement supports() method.
    }
}
```

```php
<?php
use Payum\Core\Gateway;

$gateway = new Gateway;
$gateway->addApi(new FirstApi);
$gateway->addApi(new SecondApi);

// 在这里 ApiVersionOne 会被注入到 FooAction
$gateway->addAction(new FooAction);

// 在这里 ApiVersionTwo 会被注入到 BarAction
$gateway->addAction(new BarAction);
```

_**链接**: 参阅 authorize.net [capture action][authorize-capture-action]._

## 总结

基于如上我们描述的应用架构，我们最终得到了一个完全解耦，方便扩展和重用的库。
比如，你可以添加自定义的行为或日志扩展。
感谢它的灵活性，让我们可以实现任何任务。

下一章节 [订单集成](your-order-integration.md).

回到 [首页](index.md).

[sandbox-online]: http://sandbox.payum.forma-dev.com
[sandbox-code]: https://github.com/Payum/PayumBundleSandbox
[base-request]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Request/Generic.php
[status-request-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Request/GetStatusInterface.php
[status-request]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Request/GetHumanStatus.php
[base-reply]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Reply/Base.php
[action-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Action/ActionInterface.php
[extension-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Extension/ExtensionInterface.php
[storage-extension-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Extension/StorageExtension.php
[storage-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Storage/StorageInterface.php
[doctrine-storage]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Bridge/Doctrine/Storage/DoctrineStorage.php
[zend-table-gateway]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Bridge/Zend/Storage/TableGatewayStorage.php
[filesystem-storage]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/Storage/FilesystemStorage.php
[gateway-interface]: https://github.com/Payum/Payum/blob/master/src/Payum/Core/GatewayInterface.php
[capture-controller]: https://github.com/Payum/PayumBundle/blob/master/Controller/CaptureController.php
[paypal-capture-action]:https://github.com/Payum/PaypalExpressCheckoutNvp/blob/master/Action/CaptureAction.php
[paypal-authorize-token-action]: https://github.com/Payum/Payum/blob/master/src/Payum/Paypal/ExpressCheckout/Nvp/Action/Api/AuthorizeTokenAction.php
[paypal-status-action]: https://github.com/Payum/Payum/blob/master/src/Payum/Paypal/ExpressCheckout/Nvp/Action/PaymentDetailsStatusAction.php
[authorize-capture-action]: https://github.com/Payum/Payum/blob/master/src/Payum/AuthorizeNet/Aim/Action/CaptureAction.php
[authorize-status-action]: https://github.com/Payum/Payum/blob/master/src/Payum/AuthorizeNet/Aim/Action/StatusAction.php
[omnipay]: https://github.com/adrianmacneil/omnipay
[omnipay-example]: https://github.com/Payum/PayumBundleSandbox/blob/master/src/Acme/PaymentBundle/Controller/SimplePurchasePaypalExpressViaOmnipayController.php
[bundle-doc]: index.md#symfony-payum-bundle
[payum-bundle]: https://github.com/Payum/PayumBundle/blob/master/PayumBundle.php
