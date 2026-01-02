# 不可变性

不可变性意味着对象的状态在创建后无法更改。
你不是修改实例，而是创建一个具有所需更改的新实例。
这种方法对于值对象（例如 Money、ID 和 DTO）很常见。它有助于避免意外的副作用：
方法不能静默地更改共享状态，这使得代码更容易推理。

## 可变陷阱（我们要避免的）

```php
// 构建一次并重用的共享基本查询：
$base = Post::find()->where(['status' => Post::STATUS_PUBLISHED]);

// 在代码深处，我们只需要一篇文章：
$one = $base->limit(1)->one(); // 修改底层构建器（粘性限制！）

// 后来我们重用相同的 $base 期望一个完整列表：
$list = $base->orderBy(['created_at' => SORT_DESC])->all();
// 糟糕：仍然限制为 1，因为之前的 limit(1) 修改了 $base。
```

## 在 PHP 中创建不可变对象

没有直接的方法来修改实例，但你可以使用 clone 来创建具有所需更改的新实例。
这就是 `with*` 方法的作用。

```php
final class Money
{
    public function __construct(
        private int $amount,
        private string $currency,
    ) {
        $this->validateAmount($amount);
        $this->validateCurrency($currency);
    }

    private function validateAmount(string $amount)
    {
     if ($amount < 0) {
            throw new InvalidArgumentException('金额必须为正数。');
        }
    }

    private function validateCurrency(string $currency)
    {
        if (!in_array($currency, ['USD', 'EUR'])) {
            throw new InvalidArgumentException('无效货币。仅支持 USD 和 EUR。');
        }
    }

    public function withAmount(int $amount): self
    {
        $this->validateAmount($amount);

        if ($amount === $this->amount) {
            return $this;
        }

        $clone = clone $this;
        $clone->amount = $amount;
        return $clone;
    }

    public function withCurrency(string $currency): self
    {
        $this->validateCurrency($currency);

        if ($currency === $this->currency) {
            return $this;
        }

        $clone = clone $this;
        $clone->currency = $currency;
        return $clone;
    }

    public function amount(): int
    {
        return $this->amount;
    }

    public function currency(): string
    {
        return $this->currency;
    }

    public function add(self $money): self
    {
        if ($money->currency !== $this->currency) {
            throw new InvalidArgumentException('货币不匹配。不能添加不同货币的钱。');
        }
        return $this->withAmount($this->amount + $money->amount);
    }
}

$price = new Money(1000, 'USD');
$discounted = $price->withAmount(800);
// $price 仍然是 1000 USD，$discounted 是 800 USD
```

- 我们将类标记为 `final` 以防止子类突变；或者，仔细设计扩展。
- 在构造函数和 `with*` 方法中验证，以便每个实例始终有效。

> [!TIP]
> 如果你定义一个简单的 DTO，你可以使用现代 PHP `readonly` 并将属性保留为 `public`。`readonly` 关键字
> 将确保在创建对象后无法修改属性。

## 使用 clone（以及为什么它很便宜）

PHP 的 clone 执行对象的浅拷贝。对于仅包含标量
或其他不可变对象的不可变值对象，浅克隆就足够了并且很快。在现代 PHP 中，克隆小的值对象在时间和内存上都很便宜。

如果你的对象持有必须复制的可变子对象，请实现 `__clone` 以深度克隆它们：

```php
final class Order
{
    public function __construct(
        private Money $total
    ) {}

    public function total(): Money
    {
        return $this->total;
    }

    public function __clone(): void
    {
        // Money 在我们的示例中是不可变的，因此不需要深度克隆。
        // 如果它是可变的，你可以这样做：$this->total = clone $this->total;
    }

    public function withTotal(Money $total): self
    {
        $clone = clone $this;
        $clone->total = $total;
        return $clone;
    }
}
```

## 使用风格

- 构建一次值对象并传递它。如果你需要更改，请使用返回新实例的 `with*` 方法。
- 在不可变对象中首选标量/不可变字段；如果字段可以突变，请隔离它并在需要时在 `__clone` 中深度克隆它。

不可性与 Yii 对可预测的、无副作用代码的偏好非常一致，并使服务、缓存
和配置更加健壮。
