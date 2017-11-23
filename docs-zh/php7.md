# PHP7

Payum可以运行在PHP7环境中，但是有一些特性还不能正常使用：

* Doctrine MongoODM 存储还不支持，因为它依赖于PHP扩展mongo。这个扩展中针对PHP7的正式版还在计划中。
* Authorize.NET AIM 官方扩展还不支持PHP7。
* KlarnaInvoice 使用了 XMLRPC 库. 这个库还不兼容PHP7。

* [返回首页](index.md).
