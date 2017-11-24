# Event Dispatcher

事件分发扩展提供了一种桥接到 [Symfony EventDispatcher 组件](http://symfony.com/doc/current/components/event_dispatcher/index.html)的方式。
EventDispatcherComponent使得我们可以不用修改Payum就为它添加行为。

## 启用事件分发扩展

```php
<?php

use Payum\Core\Bridge\Symfony\Extension\EventDispatcherExtension;

/** @var \Payum\Core\Gateway $gateway */
/** @var \Symfony\Component\EventDispatcher\EventDispatcherInterface $eventDispatcher */

$gateway->addExtension(
    new EventDispatcherExtension($eventDispatcher)
);
```

## 监听事件

```php
<?php

use Payum\Core\Bridge\Symfony\Event\ExecuteEvent;
use Payum\Core\Bridge\Symfony\PayumEvents;

/** @var \Symfony\Component\EventDispatcher\EventDispatcherInterface $eventDispatcher */

$eventDispatcher->addListener(
    PayumEvents::GATEWAY_EXECUTE,
    function(ExecuteEvent $event) {
        // 做一些处理
    }
);
```

| 事件名称 | `PayumEvents` 常量 | 传入监听器的参数 |
| --- | --- | ---|
| payum.gateway.pre_execute | `PayumEvents::GATEWAY_PRE_EXECUTE` | `ExecuteEvent` |
| payum.gateway.execute | `PayumEvents::GATEWAY_EXECUTE` | `ExecuteEvent` |
| payum.gateway.post_execute | `PayumEvents::GATEWAY_POST_EXECUTE` | `ExecuteEvent` |

## PayumBundle的优势

如果你使用的是Symfony全栈框架和PayumBundle，那你就完全可以通过配置文件来新增事件分发扩展。

```yaml
services:
    app.payum.extension.event_dispatcher:
        class: Payum\Core\Bridge\Symfony\Extension\EventDispatcherExtension
        arguments: ["@event_dispatcher"]
        tags:
            - { name: payum.extension, all: true, prepend: false }
```

然后添加监听器：

```yaml
services:
    app.payum.listener.render_template:
        class: AppBundle\EventListener\RenderTemplateListener
        tags:
            - { name: kernel.event_listener, event: payum.gateway.execute }
```

返回 [首页](index.md).
