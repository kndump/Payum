# 日志

由于我们正在处理支付业务，因此需要记录一些重要的细节。
如果出现了什么问题，那么一个好的日志文件，可以帮助我们很容易发现问题。
这个库提供对[遵循PSR-3规范的日志](http://www.php-fig.org/psr/psr-3/)的支持。

你需要先创建一个日志记录器，注入进去，然后将该日志扩展添加进网关。

```php
<?php
use Payum\Core\Bridge\Psr\Log\LoggerExtension;
use Payum\Core\Tests\Mocks\Action\LoggerAwareAction;
use Payum\Core\Gateway;

/** @var \Psr\Log\LoggerInterface $logger */

$gateway = new Gateway;
$gateway->addExtension(new LoggerExtension($logger));
$gateway->addAction(new LoggerAwareAction);

$gateway->execute('a request');
```

完成这些之后，你只需要简单的在你想要记录日志的`行为`中实现 `LoggerAwareInterface` 接口。
它会被扩展注入。

```php
<?php
namespace App\Payum\Action;

use Payum\Core\Action\ActionInterface;
use Psr\Log\LoggerAwareInterface;
use Psr\Log\LoggerInterface;

class LoggerAwareAction implements ActionInterface, LoggerAwareInterface
{
    /** @var \Psr\Log\LoggerInterface $logger */
    protected $logger;

    /**
     * {@inheritDoc}
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    /**
     * {@inheritDoc}
     */
    public function execute($request)
    {
        if ($this->logger) {
            $this->logger->debug('I can log something here');
        }
    }

    /**
     * {@inheritDoc}
     */
    public function supports($request)
    {
        return $request == 'a request';
    }
}
```

返回 [首页](index.md).