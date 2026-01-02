# 010 — 代码风格

Yii 3 包中使用的代码格式基于 [PSR-1](https://www.php-fig.org/psr/psr-1/) 和
[PSR-12](https://www.php-fig.org/psr/psr-12/),并在其上添加了额外规则。

## 名称

- 仅使用英语。
- 使用 camelCase 表示法,包括缩写(例如,`enableIdn`)。
- 使用尽可能短但解释性的名称。
- 永远不要修剪或缩写名称。
- 为类、接口、特征和变量添加后缀 `Collection`,这是一个[集合](https://en.wikipedia.org/wiki/Collection_(abstract_data_type))。

## 类型

- 在可能的情况下声明[参数和返回类型](https://www.php.net/manual/en/migration70.new-features.php)。
- [为属性使用类型](https://wiki.php.net/rfc/typed_properties_v2)。
- 使用严格类型。尽可能避免混合和联合类型,除了兼容类型,如 `string|Stringable`。

## 注释

应该避免内联注释,除非没有注释就无法理解代码。
一个很好的例子是对某些 PHP 版本中的错误的变通方法。

方法注释是必要的,除非它没有添加方法名和签名已经具有的内容。

类注释应该描述类的目的。

[参见 PHPDoc](https://github.com/yiisoft/docs/blob/master/014-docs.md#phpdoc)。

## 格式化

### 无对齐

属性、变量和常量值赋值不应该对齐。
这同样适用于 phpdoc 标签。原因是对齐的语句经常导致更大的差异甚至冲突。

```php
final class X
{
    const A = 'test';
    const BBB = 'test';

    private int $property = 42;
    private int $test = 123;

    /**
     * @param int $number 只是一个数字。
     * @param array $options 嗯...选项!
     */
    public function doIt(int $number, array $options): void
    {
        $test = 123;
        $anotherTest = 123;
    }
}
```

### 链式调用

链式调用应该格式化以提高可读性。
如果是一个不适合 120 个字符行长度的长链,那么每个调用应该在一个新行上:

```php
$object
    ->withName('test')
    ->withValue(87)
    ->withStatus(Status::NEW)
    ->withAuthor($author)
    ->withDeadline($deadline);
```

如果是一个短链,可以在单行上:

```php
$object = $object->withName('test');
```

## 字符串

- 当没有变量时,使用 `'Hello!'`
- 要将变量放入字符串中,首选 `"Hello, $username!"`

## 类和接口

### 默认为 final

类应该默认为 `final`。

### 默认为 private

常量、属性和方法应该默认为 private。

### 组合优于继承

首选[组合到继承](guide/en/concept/di-container.md)。

### 属性、常量和方法顺序

顺序应该如下:

- 常量
- 属性
- 方法

在每个类别中,项目应该按可见性排序:

- public
- protected
- private

### 抽象类

抽象类*不应该*以 `Abstract` 为前缀或后缀。

#### 不可变方法

不可变方法约定如下:

```php
public function withName(string $name): self
{
    $new = clone $this;
    $new->name = $name;
    return $new;
}
```

1. 克隆对象名称是 `$new`。
2. 返回类型是 `self`。

#### 布尔检查方法

用于检查某些内容是否为真的方法应该命名如下:

```php
public function isDeleted(): bool;
public function hasName(): bool;
public function canDoIt(): bool;
```

#### 方法中的标志

方法中的布尔标志最好避免。这表明该方法可能做得太多,应该有两个方法而不是一个。

```php
public function login(bool $refreshPage = true): void;
```

最好有两个方法:

```php
public function login(): void;
public function refreshPage(): void;
```

## 变量

为未使用的变量添加下划线(`_`)前缀。例如:

```php
foreach ($items as $key => $_value) {
    echo $key;
}
```

## 导入

首选导入类和函数,而不是使用完全限定名称:

```php
use Yiisoft\Arrays\ArrayHelper;
use Yiisoft\Validator\DataSetInterface;
use Yiisoft\Validator\HasValidationErrorMessage;
use Yiisoft\Validator\Result;

use function is_iterable;
```

## 其他约定

- [命名空间](004-namespaces.md)
- [异常](007-exceptions.md)
- [接口](008-interfaces.md)
