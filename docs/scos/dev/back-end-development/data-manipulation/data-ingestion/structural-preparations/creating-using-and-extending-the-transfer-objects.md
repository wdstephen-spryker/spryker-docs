---
title: Creating, using, and extending the transfer objects
description: The article provides information on creation and usage of the Transfer objects.
last_updated: Jun 16, 2021
template: howto-guide-template
originalLink: https://documentation.spryker.com/2021080/docs/ht-use-transfer-objects
originalArticleId: f3c1c5d9-348c-4d02-9d4b-f0cede048566
redirect_from:
  - /2021080/docs/ht-use-transfer-objects
  - /2021080/docs/en/ht-use-transfer-objects
  - /docs/ht-use-transfer-objects
  - /docs/en/ht-use-transfer-objects
  - /v6/docs/ht-use-transfer-objects
  - /v6/docs/en/ht-use-transfer-objects
  - /v5/docs/ht-use-transfer-objects
  - /v5/docs/en/ht-use-transfer-objects
  - /v4/docs/ht-use-transfer-objects
  - /v4/docs/en/ht-use-transfer-objects
  - /v2/docs/ht-use-transfer-objects
  - /v2/docs/en/ht-use-transfer-objects
---

Transfer objects are simple data containers. Their purpose is to retrieve a standardized way to access data and get more expressive method signatures. Transfer objects are available everywhere in the system.

This article will teach you how to create and use transfer objects.

## Creating transfer objects
Transfer objects are defined in XML. The specific classes are generated by an internal script.

### XML definition
The following example describes a Customer with email, first name, last name, and the `isGuest` flag:

```xml
<?xml version="1.0"?>
<transfers xmlns="spryker:transfer-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:transfer-01 http://static.spryker.com/transfer-01.xsd">

    <transfer name="Customer">
        <property name="email" type="string"/>
        <property name="firstName" type="string"/>
        <property name="lastName" type="string"/>
        <property name="isGuest" type="bool"/>
    </transfer>

</transfers>
```

### Available types
You can use any name for your transfer objects. However, make sure that the names start with a small letter and use the camelCase format.
As for the types, you can use PHP native types: `int`, `string`, `bool`, and `array`. To create a nested transfer object, use the name of the transfer object as the type. You can also define collections of objects with the [] symbols.

```xml
<transfer name="MyTransfer">
    <property name="foo"    type="int" />
    <property name="bar"    type="string" />
    <property name="baz"    type="bool" />
    <property name="bat"    type="array" />
    <property name="item"   type="Foo" /> <!-- Foo is the name of another transfer object-->
    <property name="items"  type="Foo[]" />
</transfer>
```

The transfer object associative property attribute allows working with associative arrays and collections. An *associative array* is an array with a string index where instead of linear storage, each value can be assigned a specific key.

The associative attribute can be used with all PHP native data types or collections.

Schema generation example:

```xml
<?xml version="1.0"?>
<transfers xmlns="spryker:transfer-01"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="spryker:transfer-01 http://static.spryker.com/transfer-01.xsd" >

    <transfer name="Foo">
        <property name="bars" type="Bar[]" singular="bar" associative="true" />
        <property name="bazs" type="array" singular="baz" associative="true" />
    </transfer>

    <transfer name="Bar">
        <property name="test" type="string"/>
    </transfer>

</transfers>
```

Examples of associative arrays usage:

```php
$fooTransfer = new FooTranfer();

// adding transfers with associated keys
$fooTransfer->addBar('a', new Bar());
$fooTransfer->addBar('b', new Bar());

// setting a transfer object with associated keys
$bars = new \ArrayObject([
    'a' => new Bar(),
    'b' > new Bar(),
]);
$fooTransfer->setBars($bars);

// adding associated array items
$fooTransfer->addBaz('x', 'X');
$fooTransfer->addBaz('y', 'Y');

// setting an associated array
$fooTransfer->setBazs([
    'x' => 'X',
    'y' => 'Y',
]);
```

## Using the transfer objects

The example below is a typical use case for a transfer object. The customer object is created and the setters are used to set an email, first, and last name:

```php
<?php
$customerTransfer = new CustomerTransfer();
$customerTransfer
    ->setEmail('john.doe@spryker.com')
    ->setFirstName('John')
    ->setLastName('Doe');

echo $customerTransfer->getFirstName(); // echos 'John'
```

## Object nesting
Transfer objects can be nested. For instance, a cart object contains several items like this:

```php
<?php
// first cart item
$cartItem1 = new ItemTransfer();
$cartItem1->setSku('123abc')->setQuantity(1);

// second cart item
$cartItem2 = new ItemTransfer();
$cartItem2->setSku('888abc')->setQuantity(1);

// a cart with two items
$cartTransfer = new CartTransfer();
$cartTransfer->addItem($cartItem1)->addItem($cartItem2);

// returns ItemTransfer[] as ArrayObject
$items = $cartTransfer->getItems();
```

## Checking the required fields

{% info_block warningBox %}
*Require* methods have been deprecated in version 3.25.0 of the Transfer module and replaced by the [get-or-fail](#get-or-fail) methods.
{% endinfo_block %}
In general, a transfer object must not know which fields are required, as it can be used for different use cases. However, when you use a transfer object, you always expect the existence of specific parameters. This can be checked with a special require-method for each property:

```php
<?php
// This throws a RequiredTransferPropertyException if the first name is not set:
$customerTransfer->requireFirstName()->getFirstName();
```

<a name="get-or-fail"></a>

### Using the get-or-fail methods
Starting from version 3.25.0 of the [Transfer](https://github.com/spryker/transfer) module, transfers expose the new *get-or-fail* methods that are meant to replace the deprecated *require* methods. Here is how it works: for each nullable property, an extra *getter* method is generated, along with the regular one. This new *getter* will throw an exception when trying to get the value of a property that was not previously set. The name of the method is composed of the `get` prefix, the name of the property, and the `OrFail` suffix.
```php
$fooBarTransfer = new FooBarTransfer();

// happy case
$fooBarTransfer->setFoo('foo');
$fooBarTransfer->getFooOrFail();

// unhappy case
$fooBarTransfer->getBarOrFail(); // exception will be thrown
```
Keep in mind that these new methods are not generated for the array and collection transfer properties, as these properties cannot be nullable by design.

### Property constants
The transfer object exposes all properties as constants which can be used in forms and tables:

```php
<?php
ItemTransfer::SKU; // = 'sku'
CustomerTransfer::FIRST_NAME; // = 'firstName'
```

## File location
Most of the modules define transfer objects in a dedicated XML file: `(ModuleNamespace)/Shared/Module/Transfer/module.transfer.xml`.

Therefore, you can find the XML definition, for example, for the `CustomerGroup` module in `vendor/spryker/spryker/Bundles/CustomerGroup/src/Spryker/Shared/CustomerGroup/Transfer/customer_group.transfer.xml`.

### Adding more file locations
If you have third-party modules using our transfer objects, you can easily add additional source directories in your projects. To do so, extend `Spryker\Zed\Transfer\TransferConfig` and return all additional *glob* patterns from `getAdditionalSourceDirectoryGlobPatterns()`.

{% info_block warningBox "Glob patterns" %}
The Transfer module uses PHP's `glob()` function to resolve paths.</br>For more information, see [PHP documentation](https://php.net/manual/en/function.glob.php).
{% endinfo_block %}

Let’s say you have a custom extension package called `my-vendor/my-package` that uses transfer objects. Using Composer, by default, this package will be installed under `vendor/my-vendor/my-package`.
If you want your transfer objects to be created from definitions stored under `vendor/my-vendor/my-package/src/Transfer`, provide the necessary *glob* pattern like this:

```php
<?php

namespace Pyz\Zed\Transfer;

use Spryker\Shared\Application\ApplicationConstants;
use Spryker\Shared\Config\Config;
use Spryker\Zed\Transfer\TransferConfig as SprykerTransferConfig;

class TransferConfig extends SprykerTransferConfig
{

    /**
     * @return string[]
     */
    protected function getAdditionalSourceDirectoryGlobPatterns()
    {
        return [
            APPLICARION_ROOT_DIR . '/vendor/my-vendor/my-package/src/Transfer/',
        ];
    }

}
```

{% info_block infoBox "Naming" %}
Make sure your transfer object definition files end with `.transfer.xml` (even for your custom packages).
{% endinfo_block %}

## Transfer object generation
To generate the objects, run:

```bash
vendor/bin/console transfer:generate
```

This command retrieves all `*.transfer.xml` files from the project- and core-level, merges them, and generates PHP classes: `src/Generated/Shared/Transfer/(Name)Transfer.php`.

## Transfer file expansion
Transfer objects can be expanded from different bundles. Any other module can add properties to the existing transfer objects. For instance, the [Tax](https://github.com/spryker/tax) module may expect a *customer tax-id*. So in the `tax.transfer.xml`, you can add the required properties to the customer. But keep in mind that it is not possible to remove existing properties or to change their type.

```xml
<?xml version="1.0"?>
<transfers xmlns="spryker:transfer-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:transfer-01 http://static.spryker.com/transfer-01.xsd">

    <transfer name="Customer">
        <property name="taxId" type="int"/>
    </transfer>

</transfers>
```

After the transfers have been generated, you can set and get the customer’s *taxId* like this:

```php
<?php
use \Generated\Shared\Transfer\CustomerTransfer;

$customerTransfer = new CustomerTransfer();
$customerTransfer->setTaxId(54321);
$taxId = $customerTransfer->getTaxId();
```

## Transfer strict types

Starting from version 3.27.0 of the [Transfer](https://github.com/spryker/transfer) module, transfers support the so-called strict mode. In the strict mode, all the relevant *get-*, *set-* and *add-* methods are generated with PHP data types in place of their arguments and return values. The strict mode is disabled by default for backward compatibility reasons and can be enabled for specific transfer properties or for an entire transfer. To enable the strict mode, you need to use the new `strict` XML attribute while defining a transfer. If the `strict` attribute is applied to a property, then only that property will be considered as a strict property while applying the attribute to a `<transfer/>` element means that the whole transfer is supposed to be strict:

```xml
<?xml version="1.0"?>
<transfers xmlns="spryker:transfer-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:transfer-01 http://static.spryker.com/transfer-01.xsd">

    <transfer name="Foo" strict="true">
    </transfer>

     <transfer name="Bar">
        <property name="test" type="string" strict="true"/>
    </transfer>

    <transfer name="Buzz">
        <property name="foo" type="Foo" strict="true"/>
    </transfer>

</transfers>
```

In the example above, the `Foo` transfer is  fully strict, meaning that all of its getters, setters, and adders have data types where possible. The only acceptable value of this new attribute is `true`.

Also, in the example above, the `Buzz` transfer has a nested transfer `Foo` declared as a strict property. In case of trying to put to the `foo` property something other than `Foo` transfer object, an exception is thrown.

{% info_block warningBox %}

There is a limitation to keep in mind here: the usage of the `strict` attribute must be consistent for the same transfer/property across different definitions. In other words, if some transfer or property is defined as strict in one module, you cannot have it defined as non-strict in another module or at the project level. Otherwise, an exception will be thrown when merging the transfer definition.

{% endinfo_block %}

### Nullable arrays vs. collections
In the strict mode, there is a concept of nullable arrays. Each strict transfer property of the `array` type without the `singular` or `associative` attribute can accept `null` as its value. This differs from the non-strict mode, where array transfer properties cannot be nullable by design. All the nullable arrays have the get-or-fail method in place, the same as with all the other non-array nullable transfer properties.

### Singular name for associative array properties
Associative array properties in the strict mode must have the `singular` attribute defined, and the value of this attribute must not match the value of the `name` attribute. Otherwise, an exception is thrown during the transfer generation.

### Get-collection-item methods
Transfers expose the new *get-collection-item* methods for all the `associative` collection transfer properties. For example, for this property definition

```xml
…
<property name=“items” type=“string[]” singular=“item” associative=“true”/>
```

{% info_block warningBox %}
If you extend the existing transfer object, don't copy over all fields, but only the fields you want to add on top.
{% endinfo_block %}

## Transfer definition validation
Starting from version 3.28.0 of the [Transfer](https://github.com/spryker/transfer) module, you can validate transfer XML definition files against a configurable XSD schema. This validation is run as part of the general transfer validation process triggered by the `transfer:validate` command. You can enable or disable the validation by overriding the `\Spryker\Zed\Transfer\TransferConfig::isTransferXmlValidationEnabled()` method. Spyker provides default XSD schema for validation that can be found in `vendor/transfer/data/definition/transfer-01.xsd`. You can change this schema by overriding `\Spryker\Zed\Transfer\TransferConfig::getXsdSchemaFilePath()`.

Under the hood, the validation tool uses regular `\DOMDocument::schemaValidate()` to validate the transfer definition file against the provided schema, so see the documentation for this method to find out more about possible validation errors in case there are any.

{% info_block warningBox "The attribute value of the *root* element" %}

The only valid attribute value of the root `<transfers></transfers>` element is `spryker:transfer-01 http://static.spryker.com/transfer-01.xsd`, even though now the validation also allows `spryker:transfer-01 https://static.spryker.com/transfer-01.xsd` (`https` vs. `http`). This is done for backward compatibility reasons. Pay attention to this when creating transfer definition files and always use the valid value.

{% endinfo_block %}

## Related Spryks

You can use the following definitions to generate the related code:

* Add shared transfer schema.

See the [Spryk](/docs/scos/dev/sdk/{{site.version}}/development-tools/spryk-code-generator.html) documentation for details.
 