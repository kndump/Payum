# 如何为子仓库贡献代码

有时你并不需要Payum所有的东西，可能只需要安装某一个单独的网关。
我们假设你需要安装 `payum/stripe`。
随后你发现了一个bug，并修复了它，然后你想再把它贡献给payum主仓库。
这个补丁是不会被 payum/stripe 库接受的，因为它是一个只读库。
我会在这里向你展示该如何把你的修复推送给 payum/payum 。

1. 在github上 Fork [payum/payum](https://github.com/Payum/Payum) 库.
2. 本地 clone forked 库.

```bash
$ git clone git@github.com:Foo/Payum.git Payum
$ cd Payum
$ git checkout -b patch1
```

3. 把你fork的 `payum/stripe` 库设置为远程库，并拉取更新

```bash
$ git remote add foo git@github.com:foo/Stripe.git
$ git fetch foo
```

4. 从子仓库中合并更新

```bash
$ git subtree pull --prefix=src/Payum/Stripe/ foo patch1
```

5. 把修改后的代码推送到远程，然后开启一个push request

```bash
$ git push origin patch1
```

返回 [首页](index.md).
