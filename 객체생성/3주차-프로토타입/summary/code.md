## Java
```java
public class Order implements Cloneable{
    private Orderer orderer;
    private long orderIdx;
    private long totalAmount;
    private long lastAmount;
    private long giftCouponPrice;
    private long giftCouponUsedAmount;
    private int discountRate;
    private LocalDateTime expiryDt;
    private LocalDateTime earliesstDt;

    @Override
    public Order clone() throws CloneNotSupportedException {
        return (Order) super.clone();   //Object의 clone()이용해서 복제후 Object에서 Order로 클래스 형변환
    }
}

public class Client {
    public static void main(String[] args) throws CloneNotSupportedException {
        Order order1 = new Order();
        Order order2 = order1.clone();

        System.out.println(order1 == order2);           //false
        System.out.println(order1.equals(order2));      //false
    }
}
```
자바에서는 객체를 복제할 수 있도록 최상위 클래스인 Object에서 `clone()`를 지원한다. 하지만 Object의 clone을 이용하기 위해서는 Cloneable을 implements해야만 사용이 가능하다.

또한, 객체간의 동등성을 판단하기 위해 String의 equals() 처럼 object의 equals()를 이용하면 기대했던 값과는 다르게 `false` 가 반환된다. <br>이는 Object의 equals()는 `==`를 통해 비교하기 때문에 동일성비교와 같은 결과를 보여주는 것이고, String의 equals()는 값만을 비교하도록 재정의 했기 때문에 동등성 비교가 가능한 것이다.

따라서 객체의 동등성을 비교하고 싶다면 `equals()`를 재정의 해주면 된다.

```java
public class Order implements Cloneable{
    private Orderer orderer;
    private long orderIdx;
    private long totalAmount;
    private long lastAmount;
    private long giftCouponPrice;
    private long giftCouponUsedAmount;
    private int discountRate;
    private LocalDateTime expiryDt;
    private LocalDateTime earliesstDt;

    @Override
    public Order clone() throws CloneNotSupportedException {
        return (Order) super.clone();   //Object의 clone()이용해서 복제후 Object에서 Order로 클래스 형변환
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order)) return false;
        Order order = (Order) o;
        return orderIdx == order.orderIdx && totalAmount == order.totalAmount && lastAmount == order.lastAmount && giftCouponPrice == order.giftCouponPrice && giftCouponUsedAmount == order.giftCouponUsedAmount && discountRate == order.discountRate && Objects.equals(orderer, order.orderer) && Objects.equals(expiryDt, order.expiryDt) && Objects.equals(earliesstDt, order.earliesstDt);
    }

    @Override
    public int hashCode() {
        return Objects.hash(orderer, orderIdx, totalAmount, lastAmount, giftCouponPrice, giftCouponUsedAmount, discountRate, expiryDt, earliesstDt);
    }
}

public class Client {
    public static void main(String[] args) throws CloneNotSupportedException {
        Order order1 = new Order();
        Order order2 = order1.clone();

        System.out.println(order1 == order2);           //false
        System.out.println(order1.equals(order2));      //true
    }
}
```

<br>

## PHP
```php
class Order{
    private $orderer;
    private $orderIdx;
    private $totalAmount;
    private $lastAmount;
    private $giftCouponPrice;
    private $giftCouponUsedAmount;
    private $discountRate;
    private $expiryDt;
    private $earliesstDt;
}

class client extends PHPUnit\Framework\TestCase{
    function test_clone객체비교() {
        $order1 = new Order();
        $order2 = clone $order1;

        $this->assertFalse($order == $order2);
        $this->assertTrue($order === $order3);
    }
}
```
php는 `clone`이라는 키워드로 객체 복제를 지원하고 있으며, `==`,`===` 연산자로 동일성, 동등성을 비교해보면 복제된 객체와 동등하지만 동일하지는 않게 복제가 되는 것을 볼 수 있다.