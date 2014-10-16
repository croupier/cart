# Cart
[![Build Status](https://secure.travis-ci.org/mike182uk/cart.png)](http://travis-ci.org/mike182uk/cart)
[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/mike182uk/cart/badges/quality-score.png?s=400a4f03f3d494434d9240123de352edd89eb52d)](https://scrutinizer-ci.com/g/mike182uk/cart/)
[![Code Coverage](https://scrutinizer-ci.com/g/mike182uk/cart/badges/coverage.png?s=c7dd3fffa6ed075c7e28d9af54eb88373ba314fd)](https://scrutinizer-ci.com/g/mike182uk/cart/)
[![Total Downloads](https://poser.pugx.org/mike182uk/cart/downloads.png)](https://packagist.org/packages/mike182uk/cart)
[![License](https://poser.pugx.org/mike182uk/cart/license.png)](https://packagist.org/packages/mike182uk/cart)

A flexible and modern shopping cart package.

## Prerequisites

- PHP >=5.3.0

## Installation

### Composer

Add this package as a dependency in your `composer.json`.

```js
{
    "require" : {
        "mike182uk/cart" : "v2.1.*"
    }
}
```

## Usage

- [Cart](#cart)
- [Cart Item](#cart-item)
- [Cart Storage Implementation](#cart-store)

### <a id="cart"></a>Cart

#### Create a new cart

To create a new cart instance you must pass an id and a storage implementation to the cart constructor:

```php
use Cart;
use Cart\Storage\SessionStore;

$id = 'cart-01';
$cartSessionStore = new SessionStore();

$cart = new Cart($id, $cartSessionStore);
```

The storage implementation must implement `Cart\Storage\Store`.

The id is used for saving / restoring cart state via the storage implementation.

#### Add an item to the cart

Use the `add` method to add an item to the cart. A valid `Cart\CartItem` must be passed to this method.

```php
use Cart\CartItem;

$item = new CartItem;
$item->name = 'Macbook Pro';
$item->sku = 'MBP8GB';
$item->price = 1200;
$item->tax = 200;

$cart->add($item);
```

If the item already exists in the cart, the quantity of the existing item will be updated to include the quantity of the item being added.

#### Remove an item from the cart

Remove an item from the cart by passing the item id to the `remove` method.

```php
$cart->remove('e4df90d236966195b49b0f01f5ce360a356bc76b');
```

#### Update an item in the cart

To update a propery of an item in the cart use the `update` method. You will need to pass the cart item id, the name of the property to update and the new value. This method will return the item id (in case it has changed due to the update).

```php
$newId = $cart->update('e4df90d236966195b49b0f01f5ce360a356bc76b', 'price', 959.99);
```

If you try and update an item that does not exist in the cart a `InvalidArgumentException` will be thrown.

#### Retrieve an item in the cart

Retrieve an item from the cart by its id use the `get` method. If the item does not exist `null` is returned.

```php
$item = $cart->get('e4df90d236966195b49b0f01f5ce360a356bc76b');

if ($item) {
    // ...
}
```

#### Retrieve all items in the cart

Retrieve all items in the cart using the `all` method. This will return an array of all the items in the cart.

```php
$cartItems = $cart->all();

if (count($cartItems) > 0) {
    foreach ($cartItems as $item) {
        // ...
    }
}
```

#### Determine if an item exists in the cart

Determine if an item exists in the cart using the `has` method. Returns `true` or `false`.

```php
if ($cart->has('e4df90d236966195b49b0f01f5ce360a356bc76b')) {
    // ...
}
```

#### Clear The Cart

Clear the cart using the `clear` method.

```php
$cart->clear();
```
This will also clear the saved state for this cart in the store.

#### Save / restore cart state

Save the cart using the `save` method.

```php
$cart->save();
```

This will save the current cart items and cart id to the store.

Restore the cart using the `restore` method.

```php
$cart->restore();
```

This will add any stored cart items back to the cart and set the cart id. If there is a problem restoring the cart a `Cart\CartRestoreException` will be thrown. This will only happen if:

- the saved data is unserializable
- the unserialized data is invalid (not an array)
- the cart id is not present in the unserialized data
- the cart items are not present in the unserialized data
- the cart id is invalid (not a string)
- the cart items are invalid (not an array)

#### Other Cart Methods

##### totalUniqueItems

Get the total number of unique items in the cart.

```php
$cart->totalUniqueItems();
```

##### totalItems

Get the total number of items in the cart.

```php
$cart->totalItems();
```

##### total

Get the total price of all the cart items including tax.

```php
$cart->total();
```

You can get the total excluding tax by using the `totalExcludingTax` method.

```php
$cart->totalExcludingTax();
```

##### tax

Get the total tax of all the cart items.

```php
$cart->tax();
```

##### toArray

Export the cart to an array.

```php
$cartData = $cart->toArray();
```

Array will be structured like:

```php
array(
    'id' => 'cart-01', // cart id
    'items' => array(
        // cart items as array
    )
)
```

##### getId

Get the id of the cart.

```php
$cart->getId();
```

##### getStore

Get the cart storage implementation.

```php
$cart->getStore();
```

### <a id="cart-item"></a>Cart Item

#### Create a new Cart Item

```php
use Cart\CartItem;

$item = new CartItem;

$item->name = 'Macbook Pro';
$item->sku = 'MBP8GB';
$item->price = 1200;
$item->tax = 200;
$item->options = array(
    'ram' => '8 GB',
    'ssd' => '256 GB'
);
```

`Cart\CartItem` implements `ArrayAccess` so properties can be assigned to the cart item as if accessing an array:

```php
$item = new CartItem;

$item['name'] = 'Macbook Pro';
$item['sku'] = 'MBP8GB';
$item['price'] = 1200;
$item['tax'] = 200;
$item['options'] = array(
    'ram' => '8 GB',
    'ssd' => '256 GB'
);
```

An array of data can also be passed to the cart item constructor to set the cart item properties:

```php
$itemData = array(
    'name' => 'Macbook Pro';
    'sku' => 'MBP8GB';
    'price' => 1200;
    'tax' => 200;
    'options' => array(
        'ram' => '8 GB',
        'ssd' => '256 GB'
    )
);

$item = new CartItem($itemData);
```

If no quantity is passed to the cart item constructor, the quantity is set to `1` by default.

If no price is passed to the cart item constructor, the price is set to `0.00` by default.

If no tax is passed to the cart item constructor, the tax is set to `0.00` by default.

#### Cart Item ID

Each cart has a unique ID. This ID is generated using the properties set on the cart item. You can get the cart item ID using the method `getId` or by accessing the property `id`.

```php
$id = $item->getId();
```

```php
$id = $item->id;
```

```php
$id = $item['id'];
```

**Changing a property on the cart item will change its ID.**

Cart item properties can be ommitted from the cart item id generation process. By default `quantity` is the only property that is ommitted - this means that 2 of the same items, that only vary by quantity, will still be treated as the same item (this is how the cart knows whether to add a new item or update the quantity of an existing one).

You can see what properties are omitted by using the `getOmittedHashProperties` method.

```php
$omittedProperties = $item->getOmittedHashProperties();
```

You can also override this method in a subclass if you have need to change what properties are omitted.

#### Cart Item Methods

##### get

Get a piece of data set on the cart item.

```php
$name = $item->get('name');
```

This is the same as doing:

```php
$name = $item['name'];
```

```php
$name = $item->name;
```

You can also define a custom getter for a cart item property on a subclass.

```php
use Cart\CartItem;

class MyCartItem extends CartItem
{
    const SKU_PREFIX = 'ABC_';

    public function getSku()
    {
        return self::SKU_PREFIX . $this->get('sku');
    }
}

$item = new MyCartItem;
$item->sku = 123;

$sku = $item->getSku(); // will return ABC_123 instead of 123
```

The custom getter will only be used when accessing a property using array access (`$item['sku']`) or direct access (`$item->sku`). Alternatively the the custom getter can be called directly.

`CartItem::get()` will not use the custom getter. This will always return the original value of the property.

Inside the custom getter `CartItem::get()` should be used to retrieve a cart item property.

##### set

Set a piece of data on the cart item.

```php
$item->set('name', 'Macbook Pro');
```

This is the same as doing:

```php
$item['name'] = 'Macbook Pro';
```

```php
$item->name = 'Macbook Pro';
```

If you are setting the item quantity, the value must be an integer otherwise an `InvalidArgumentException` is thrown.

```php
$item->quantity = 1; // ok

$item->quantity = '1' // will throw exception
```

If you are setting the item price or tax, the value must be numeric otherwise an `InvalidArgumentException` is thrown.

```php
$item->price = 10.00; // ok

$item->price = '10' // ok

$item->price = 'ten' // will throw exception
```

You can also define a custom setter for a cart item property on a subclass.

```php
use Cart\CartItem;

class MyCartItem extends CartItem
{
    const SKU_PREFIX = 'ABC_';

    public function setSku($sku)
    {
       $this->set('sku', self::SKU_PREFIX . $sku);
    }
}

$item = new MyCartItem;
$item->sku = 123;

$sku = $item->getSku(); // will return ABC_123 instead of 123
```

The custom setter will only be used when setting a property using array access (`$item['sku'] = '123'`) or direct access (`$item->sku = '123'`). Alternatively the the custom setter can be called directly.

##### getTotalPrice

Get the total price of the cart item including tax `((item price + item tax) * quantity)`.

```php
$item->getTotalPrice();
```

You can also get the total price excluding tax `(item price * quantity)` using the `getTotalPriceExcludingTax` method.

```php
$item->getTotalPriceExcludingTax();
```

##### getSinglePrice

Get the single price of the cart item including tax `(item price + item tax)`

```php
$item->getSinglePrice();
```

You can also get the single price excluding tax by using the `getSinglePriceExcludingTax` method.

```php
$item->getSinglePriceExcludingTax();
```

##### getTotalTax

Get the total tax of the cart item `(item tax * quantity)`.

```php
$item->getTotalTax();
```

##### getSingleTax

Get the single tax value of the cart item.

```php
$item->getSingleTax();
```

##### toArray

Export the item to an array.

```php
$itemArr = $item->toArray();
```

Array will be structured like:

```php
array(
    'id' => 'e4df90d236966195b49b0f01f5ce360a356bc76b', // cart item unique id
    'data' => array(
        'name' => 'Macbook Pro',
        'sku' => 'MBP8GB',
        'price' => 1200,

        // ... other cart item properties
    )
)
```

### <a id="cart-store"></a>Cart Storage Implementation

A cart storage implementation must implement `Cart\Storage\Store`.

This package provides 2 basic storage implementations: `Cart\Storage\SessionStore` and `Cart\Storage\CookieStore`.

When the `save` method of the cart is called, the cart id and serialized cart data is passed to the `put` method of the storage implementation.

When the `restore` method of the cart is called, the cart id is passed to the `get` method of the storage implementation.

When the `clear` method of the cart is called, the cart id is passed to the `flush` method of the storage implementation.

An example session storage implementation may look like:

```php
use Cart\Store;

class SessionStore implements Store
{
    /**
     * {@inheritdoc}
     */
    public function get($cartId)
    {
        return isset($_SESSION[$cartId]) ? $_SESSION[$cartId] : array();
    }

    /**
     * {@inheritdoc}
     */
    public function put($cartId, $data)
    {
        $_SESSION[$cartId] = $data;
    }

    /**
     * {@inheritdoc}
     */
    public function flush($cartId)
    {
        unset($_SESSION[$cartId]);
    }
}
```
