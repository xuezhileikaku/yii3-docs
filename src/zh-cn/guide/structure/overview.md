# 概述

Yii 应用程序代码通常按上下文分组为模块。在每个模块中，可以按类型进行分组。

例如，如果应用程序是在线商店，上下文可能是：

- 客户
  - 个人资料
  - 产品列表
  - 结账
- 物流
- 配送
  - 地址
- 服务台
  - 支持
  - 索赔
  - 退货
- 会计
  - 退货
  - 交易
  - 税务

对于 "客户" 上下文，位于 `App\Customer` 命名空间下，结构将是：

```
App/
  Customer/                          <-- 模块命名空间
    Entity/
      Customer.php                   <-- "个人资料" 和 "结账" 共享的实体
    Profile/
      Widget/
        Gravatar.php
      ProfileRepository.php          <-- 仓储通常特定于上下文
      ProfileController.php          <-- "Customer\Profile" 入口点
    ProductList/                     <-- 模块命名空间
      Entity/                        <-- 特定于 "Customer\ProductList" 的实体
        Category.php
        Product.php
      ProductsListController.php     <-- "Customer\ProductList" 入口点
    Checkout/                        <-- 模块命名空间
      CheckoutController.php
```

上下文可能包含子上下文。如果一个类被多个上下文共享，它将被移动到这两个上下文的祖先中。

上下文可能具有[称为 "action" 或 "controller" 的入口点](action.md)。它的工作是接收[请求实例](../runtime/request.md)，以适当的格式将其传递给[领域层](domain.md)，并根据领域层的返回创建[响应](../runtime/response.md)。

此外，Yii 应用程序还具有以下内容：

* [入口脚本](entry-script.md)：它们是最终用户可以直接访问的 PHP 脚本。
  它们负责启动请求处理周期。通常，单个入口脚本处理整个应用程序。
* [服务](service.md)：它们通常是在依赖容器中注册的无状态对象，
  并提供各种操作方法。
* [中间件](middleware.md)：它们代表需要在动作处理程序实际处理每个请求之前和之后调用的代码。
