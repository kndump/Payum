# 存储

存储允许你保存，获取支付相关的信息。
他们需要被明确的调用，也就是说在需要的时候，你必须主动调用保存或获取方法。
或者，你也可以使用`存储扩展`将存储集成到一个网关中。
`存储扩展`也可以通过它的`身份识别器(Identificator)`来加载模型，这种方式下，你就不必再过多的关心它了。

显式调用的示例：

```php
<?php
use Payum\Core\Storage\FilesystemStorage;

$storage = new FilesystemStorage('/path/to/storage', 'Payum\Core\Model\Payment', 'number');

$order = $storage->create();
$order->setTotalAmount(123);
$order->setCurrency('EUR');

$storage->update($order);

$foundOrder = $storage->find($order->getNumber());
```

隐式调用的示例：

```php
<?php
use Payum\Core\Extension\StorageExtension;
use Payum\Core\Gateway;
use Payum\Core\Storage\FilesystemStorage;

$gateway->addExtension(new StorageExtension(
   new FilesystemStorage('/path/to/storage', 'Payum\Core\Model\Payment', 'number')
));
```

在扩展中使用模型识别器的用法:

```php
<?php
use Payum\Core\Extension\StorageExtension;
use Payum\Core\Storage\FilesystemStorage;
use Payum\Core\Request\Capture;

$storage = new FilesystemStorage('/path/to/storage', 'Payum\Core\Model\Payment', 'number');

$order = $storage->create();
$storage->update($order);

/** @var \Payum\Core\Gateway $gateway */
$gateway->addExtension(new StorageExtension($storage));

$gateway->execute($capture = new Capture(
    $storage->identify($order)
));

echo get_class($capture->getModel());
// -> Payum\Core\Model\Payment
```

## Doctrine ORM

```
php composer.phar install "doctrine/orm"
```

添加令牌和订单类：

```php
<?php
namespace Acme\Entity;

use Doctrine\ORM\Mapping as ORM;
use Payum\Core\Model\Token;

/**
 * @ORM\Table
 * @ORM\Entity
 */
class PaymentToken extends Token
{
}
```

```php
<?php
namespace Acme\Entity;

use Doctrine\ORM\Mapping as ORM;
use Payum\Core\Model\Payment as BasePayment;

/**
 * @ORM\Table
 * @ORM\Entity
 */
class Payment extends BasePayment
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

下一步，需要创建一个入口管理器和Payum的存储：

```php
<?php
use Doctrine\Common\Cache\ArrayCache;
use Doctrine\Common\Persistence\Mapping\Driver\MappingDriverChain;
use Doctrine\ORM\Configuration;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Mapping\Driver\SimplifiedXmlDriver;
use Payum\Core\Bridge\Doctrine\Storage\DoctrineStorage;

$config = new Configuration();
$driver = new MappingDriverChain;

// payum的基础模型
$driver->addDriver(
    new SimplifiedXmlDriver(array('path/to/Payum/Core/Bridge/Doctrine/Resources/mapping' => 'Payum\Core\Model')), 
    'Payum\Core\Model'
);

// 自定义的模型
$driver->addDriver(
    $config->newDefaultAnnotationDriver(array('path/to/Acme/Entity'), false), 
    'Acme\Entity'
);

$config->setAutoGenerateProxyClasses(true);
$config->setProxyDir(\sys_get_temp_dir());
$config->setProxyNamespace('Proxies');
$config->setMetadataDriverImpl($driver);
$config->setQueryCacheImpl(new ArrayCache());
$config->setMetadataCacheImpl(new ArrayCache());

$connection = array('driver' => 'pdo_sqlite', 'path' => ':memory:');

$orderStorage = new DoctrineStorage(
   EntityManager::create($connection, $config),
   'Payum\Entity\Payment'
);

$tokenStorage = new DoctrineStorage(
   EntityManager::create($connection, $config),
   'Payum\Entity\PaymentToken'
);
```

### Doctrine MongoODM.

```
php composer.phar install "doctrine/mongodb": "1.0.*@dev" "doctrine/mongodb-odm": "1.0.*@dev"
```

```php
<?php
namespace Acme\Document;

use Doctrine\ODM\MongoDB\Mapping\Annotations as Mongo;
use Payum\Core\Model\Token;

/**
 * @Mongo\Document
 */
class PaymentToken extends Token
{
}
```

```php
<?php
namespace Acme\Document;

use Doctrine\ODM\MongoDB\Mapping\Annotations as Mongo;
use Payum\Core\Model\Payment as BasePayment;

/**
 * @Mongo\Document
 */
class Payment extends BasePayment
{
    /**
     * @Mongo\Id
     *
     * @var integer $id
     */
    protected $id;
}
```

下一步, 需要创建一个入口管理器和payum的存储：

```php
<?php
use Doctrine\Common\Annotations\AnnotationReader;
use Doctrine\Common\Cache\ArrayCache;
use Doctrine\Common\Persistence\Mapping\Driver\MappingDriverChain;
use Doctrine\Common\Persistence\Mapping\Driver\SymfonyFileLocator;
use Doctrine\ODM\MongoDB\Types\Type;
use Doctrine\ODM\MongoDB\Mapping\Driver\AnnotationDriver;
use Doctrine\ODM\MongoDB\Mapping\Driver\XmlDriver;
use Doctrine\ODM\MongoDB\DocumentManager;
use Doctrine\ODM\MongoDB\Configuration;
use Doctrine\MongoDB\Connection;
use Payum\Core\Bridge\Doctrine\Storage\DoctrineStorage;

Type::addType('object', 'Payum\Core\Bridge\Doctrine\Types\ObjectType');

$driver = new MappingDriverChain;

// payum的基础模型
$driver->addDriver(
    new XmlDriver(
       new SymfonyFileLocator(array(
            '/path/to/Payum/Core/Bridge/Doctrine/Resources/mapping' => 'Payum\Core\Model'
        ), '.mongodb.xml'),
        '.mongodb.xml'
    ), 
    'Payum\Core\Model'
);

// 自定义的模型
AnnotationDriver::registerAnnotationClasses();
$driver->addDriver(
    new AnnotationDriver(new AnnotationReader(), array(
        'path/to/Acme/Document',
    )), 
    'Acme\Document'
);

$config = new Configuration();
$config->setProxyDir(\sys_get_temp_dir());
$config->setProxyNamespace('Proxies');
$config->setHydratorDir(\sys_get_temp_dir());
$config->setHydratorNamespace('Hydrators');
$config->setMetadataDriverImpl($driver);
$config->setMetadataCacheImpl(new ArrayCache());
$config->setDefaultDB('payum_tests');

$connection = new Connection(null, array(), $config);

$orderStorage = new DoctrineStorage(
    DocumentManager::create($connection, $config),
    'Acme\Document\Payment'
);

$tokenStorage = new DoctrineStorage(
    DocumentManager::create($connection, $config),
    'Acme\Document\SecurityToken'
);
```        

## Filesystem.

```php
<?php
use Payum\Core\Storage\FilesystemStorage;

$storage = new FilesystemStorage(
    '/path/to/storage', 
    'Payum\Core\Model\Payment', 
    'number'
);
```

## Propel 2

首先，你需要生成基础模型类。

为此，你还需要创建一个配置文件。
请参阅 [propel的文档](http://propelorm.org/documentation/02-buildtime.html#building-the-model).

接着，执行：
```sh
$ bin/propel --config-dir=path/where/you/created/propel.ext --schema-dir=src/Payum/Core/Bridge/Propel2/Resources/config --output-dir=src/ build
```

接着，你可以把 ```src/Payum/Core/Bridge/Propel2/Resources/install/order.sql``` 和 ```src/Payum/Core/Bridge/Propel2/Resources/install/token.sql``` 导入数据库。

你可以拷贝一份 ```schema.xml``` 文件到自己的项目中并自定义修改它。
如果修改了 ```schema.xml``` 文件，你还需要生成创建表的sql文件。
然后，你只需要执行：
```sh
$ bin/propel --config-dir=your/path/to/propel.xml/directory --schema-dir=your/path/to/schema.xml/directory --output-dir=your-application/resources/ sql:build
```

如果你想要在模型类总添加自己的逻辑，则需要扩展如下类文件：
- ```Payum\Core\Bridge\Propel2\Model\Payment```
- ```Payum\Core\Bridge\Propel2\Model\OrderQuery```
- ```Payum\Core\Bridge\Propel2\Model\Token```
- ```Payum\Core\Bridge\Propel2\Model\TokenQuery```

如果你不需要修改模型类中的逻辑，就可以直接使用了。

接着，你还需要配置一个连接。

这里是一份根据[propel文档](http://propelorm.org/documentation/02-buildtime.html#runtime-connection-settings)做过适配后的代码片段：

```php
<?php

use Propel\Runtime\Propel;
use Propel\Runtime\Connection\ConnectionManagerSingle;
$serviceContainer = Propel::getServiceContainer();
$serviceContainer->setAdapterClass('default', 'mysql');
$manager = new ConnectionManagerSingle();
$manager->setConfiguration(array (
  'dsn'      => 'mysql:host=localhost;dbname=my_db_name',
  'user'     => 'my_db_user',
  'password' => 's3cr3t',
));
$serviceContainer->setConnectionManager('default', $manager);
```

## 自定义

你可以创建自定义的存储方式。为此你只需要实现`StorageInterface`即可。

```php
<?php
use Payum\Core\Storage\StorageInterface;

class CustomStorage implements StorageInterface
{
    // implement all declared methods
}
```

## TODO

* [Pdo](http://php.net/manual/en/book.pdo.php) 存储 - https://github.com/Payum/Payum/issues/205
* [Yii ActiveRecord](http://www.yiiframework.com/doc/guide/1.1/en/database.ar) 存储 - https://github.com/Payum/PayumYiiExtension/pull/4

* [返回首页](index.md).
