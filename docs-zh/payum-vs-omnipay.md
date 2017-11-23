# Payum 和 Omnipay 对比

简短的回答是Payum提供了和Ominipay相同的功能，以及一些额外的特性。
支付模型并不是只能使用[Payum 提供的](https://github.com/Payum/Payum/blob/master/src/Payum/Core/Model/Payment.php)。
我(作者)鼓励你使用你自己的模型，或者[电子商务平台](https://github.com/Sylius/Sylius/blob/master/src/Sylius/Component/Payment/Model/Payment.php)已经提供的模型。
理由也很简单：你给Payum发送一个请求去捕获你的模型。
在行为中，你[把支付模型转换成网关需要的特殊格式](https://github.com/Payum/Payum/blob/master/src/Payum/Paypal/ExpressCheckout/Nvp/Action/ConvertPaymentAction.php)，很大可能是一个数组。
这种方法的优点就是你的代码永远不需要修改。
示例如下：

    $gateway->execute(new Capture($payment));

所有网关的不同之后都被封装在网关内部了。
当然，payum支持特定格式的网关，也支持Payum的支付模型。
在Omnipay中，你不可能简单的实现把stripe的网关适配到paypal网关，
因为他们的表现不尽相同，而且更重要的是他们所需要的数据也都是不同的。
Stripe需要提供一个信用卡，而Paypal并不关心信用卡，而是需要配置返回和取消的链接。
你不得不在代码里反映出这些不同之处，这是抽象吗？
相反的，Payum会为你生成取消和返回的链接，而且它们也是安全的（我们稍后再继续讨论这个）。

有时你必须获取关于付款交易或付款人的更多细节，或者一个错误的更多信息。
Payum允许你访问代码和支付网关之间通信的所有数据。
数据是一笔支付的特定格式，因此，如果你对[Paypal 协议 (举例)](https://developer.paypal.com/docs/classic/express-checkout/gs_expresscheckout/)熟悉的话，那就可以很容易理解这期间发生了什么。
另外一个比较好的例子是 Klarna Checkout 。它会返回邮寄/账单地址，性别和出生日期。
使用Payum你可以在你需要的时候，很方便的从支付中获取这些数据。

Payum为你提供更好的状态处理机制。
Omnipay只提供了两种状态成功和失败，但这还不够。
例如，Paypal有时会因为[多货币问题](http://stackoverflow.com/questions/19864511/paypal-sandbox-pending-multicurrency)返回挂起状态。
在这种情况下，omnipay认为付款失败了，但实际上并没有失败。
或者用户可以在paypal取消该笔支付，Omnipay会告诉你失败，但事实也并非如此。
如果需要使用Payum默认未提供的状态，你也可以轻松地添加。
或许你的[电子商务平台](https://github.com/Sylius/Sylius/blob/master/src/Sylius/Bundle/PayumBundle/Payum/Request/GetStatus.php#L24)已经提供了一些状态，而你想继续使用这些状态，
那也是完全没有问题的，Payum可以做到去适配这些状态。

有时用户可能想要欺骗你或者少付一些钱。
作为开发者，你必须考虑到这些，并妥善处理。
你向用户公开什么数据？
这可能被错误的运用吗？
在你验证之前，你不能信任出现在url里的金额。
例如，Paypal会向你推送一个通知到你之前发送给他们的通知链接上。
Payum会自动为你生成这样一个链接，当通知返回时，payum也会自动验证它。
你会从中拿到一个唯一且安全的链接。
一旦你将这个链接移除或置为失效，用户将再不会访问到与该链接关联的支付交易。
Omnipay没有提供任何可以帮你解决这些安全问题的策略。
这些安全链接还有一个较好的边际效应。
一旦它们不再被需要，就会被置为失效或被移除。
例如，当用户在浏览器中点击"返回"按钮时，他不能再对该笔购买进行二次支付。
因为购买链接已经不存在了，取而代之的是，他会看到一个404错误。

将信用卡信息保存在你自己的系统不是一个好的实践，对吗？
你没有理由说意外保存了这些信息，甚至仅仅是临时保存。
Payum提供了一个[敏感值](https://github.com/Payum/Payum/blob/master/src/Payum/Core/Security/SensitiveValue.php)对象来帮你避免意外保存了这些数据。
当然你还是可以保存这些信息的，但是你最好清楚知道这样做的原因和后果。

Omnipay仅支持需要重定向到网关侧或需要信用卡的网关。
但实际上还有其他不同形式的网关。
例如，Klarna Checkout [需要渲染一个代码片段(iframe)](https://developers.klarna.com/en/se+php/kco-v2/checkout/2-embed-the-checkout),
Stripe.Js 需要在付款页面 [执行他们的JS代码](https://stripe.com/docs/stripe.js?)，
Stripe Checkout [需要渲染他们自己的弹窗](https://stripe.com/docs/checkout)。
Payum 支持[所有这些网关](supported-gateways.md)。
正如我们一开始说的那样，你可以从一个网关切换到另一个网关，而无需改动任何代码。
我并不认为重复造轮子(重新实现每一个网关)是一个好的做法。
这也就是[桥接Omnipay网关](index.md#omnipay-bridge-external)存在的原因。
它允许你以Payum的方式使用Omnipay网关。

Payum尝试去[标准化支付流程](get-it-started.md)。
这需要三步：`准备`，`捕获/授权`和`完成`。
第一步是“准备”，在这一步你必须准备付款，计算总价格，税收，获取用户或邮寄信息等。
一旦完成了这些，你可以将用户重定向到`捕获\授权`的步骤。
在这里，用户可能被重定向到网关侧或输入信用卡信息，或其他东西。
而这则取决于你使用的网关。
在“完成”这一步，您必须获得支付状态并做出相应的处理。
Omnipay只是解决了这其中的部分问题。

通过网关工厂，你可以轻松地重写|替换网关的部分功能，或添加自定义行为，扩展和API。


Payum 为大部分现代化的框架提供了官方扩展：
[Symfony](https://github.com/Payum/PayumBundle)，
[Laravel](https://github.com/Payum/PayumLaravelPackage)，
[Silex](https://github.com/Payum/PayumSilexProvider)，
[Yii](https://github.com/Payum/PayumYiiExtension)，
[Zend](https://github.com/Payum/PayumModule)。

最后让我们来比较一下 Payum 和 Omnipay 的具体接口实现：

* Payum [GatewayInterface](https://github.com/Payum/Payum/blob/master/src/Payum/Core/GatewayInterface.php) vs Omnipay [GatewayInterface](https://github.com/thephpleague/omnipay-common/blob/master/src/Common/GatewayInterface.php)
* Payum [GatewayFactoryInterface](https://github.com/Payum/Payum/blob/master/src/Payum/Core/GatewayFactoryInterface.php) vs Omnipay [GatewayFactory](https://github.com/thephpleague/omnipay-common/blob/master/src/Common/GatewayFactory.php) 工厂类并没有接口.

返回 [首页](index.md).
