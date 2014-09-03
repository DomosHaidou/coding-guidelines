# Demac Media Coding Standards

- **Section 1**
    - [What is this document?](#what-is-this-document)
    - [Audience and prerequisites](#audience-and-prerequisites)
    - [Why should we care?](#why-should-we-care)
    - [Maintainability and Technical debt](#maintainability-and-technical-debt)
- **Section 2: The Tools**
    - [Stinky code and PHP_CodeSniffer](#stinky-code-and-codesniffer)
    - [Installation and Configuration](#installation-and-configuration)
- **Section 3: Things to Avoid**
    - [Raw Sql Queries](#raw-sql-queries)
    - [Queries inside a loop](#queries-inside-a-loop)
    - [Direct class instantiation](#direct-class-instantiation)
- **Section 4: Design Patterns**
    - [Adapter Pattern](#adapter-pattern)

### What is this document?

This is an introduction to the ideas and approaches that motivate and faciliate
writting readable and maintainable code that follows both Demac Media's and
Magento's coding standards.

We'll walk through concrete examples on why the usage of these code standards
can prevent future problems bith in terms of maintainability and performance.

### Audience and prerequisites

This document is intended for any Magento Developer working at Demac Media.

You are expected to be familiar with Magento, the way the code is structured,
the development workflow and the deployment process at Demac Media.

### Why should we care?

Much can be said about the importance of coding standards and coding styles, but
the main advantage that coding standards and styles bring to the table is
consistency.

Consistency is crucial, specially as we grow and have more developers working on
the same codebase. Coding standards make code easier to read and thus share.

As a Magento Developer at Demac Media you should not only strive to write code
that gets the job done but also to write code that easy to **add to**,
**maintain** and **debug**.

Furthermore, coding standards can also be powerful tools; with the right set of
rules we can prevent some of nasty performance problems that are very common
among **Magento** extensions.

### Maintainability and Technical Debt

At this point you probably realized that writing maintainable code keeps getting
mentioned as one of the main reasons of why coding standards are important.

> The more readable the source code is, the easier it is for someone to mantain
> that code.

Maintainable code is one of the ways we can mitigate the cost of technical debt;
maintainable code allow you to quickly and easily:

- Find and Fix bugs
- Add new features
- Improve usability
- Increase performance
- Make it easier for other to mantain the code

> Technical debt (also known as design debt or code debt) is a
> neologistic metaphor referring to the eventual consequences of poor system
> design, software architecture or software development within a codebase.
> [Technical Debt](http://en.wikipedia.org/wiki/Technical_debt)

The truth is that technical debt will never fully go away, it is a fact of
a developers live; sometimes compromises have to be made. That being said, there
is no reason to reduce the impact of technical debt and improve how we deal with
it.

Proper coding standards and coding styles are the key tools at our disposal for
improving our code, and preventing (or at least reducing) technical debt; and at
the end making life for everyone involved a little easier.

## The Tools

[//]: # (Do a recap of what we have learned so far regarding coding standards)

So now that we know why having and properly using **coding standards** is an
important part of our development practices; we can go over how to actually
make and apply those coding standards.

As part of our workflow we will be using serveral tools to check and identify
problems in our code. The most important and powerful tool in our toolbox right
now is [**PHP CodeSniffer**](https://github.com/squizlabs/PHP_CodeSniffer), so let's go ahead and take a closer look at
codesniffer.

### Stinky code and Codesniffer

> PHP_CodeSniffer is a PHP5 script that tokenises PHP, JavaScript and CSS files
> to detect violations of a defined coding standard. It is an essential
> development tool that ensures your code remains clean and consistent. It can
> also help prevent some common semantic errors made by developers.

PHP_CodeSniffer true power comes from the flexibility of using multiple coding
standards and even defining our own set of coding standards. PHP_CodeSniffer has
the following coding standards available out of the box:

- Zend
- PEAR (default)
- PHPCS
- Squiz

However, since Magento is a unique and special snowflake (or at least some of it
was coded that way) we have to use **Magento's own coding standard**, created by
Magento ECG (Expert Consulting Group).

This standard is actually a great tool and it checks against the following
common problems:

- raw SQL queries.
- SQL queries inside a loop.
- direct instantiation of Mage and Enterprise classes.
- unncessary collection loading.
- excesive code complexity.
- use of dangerous functions.
- use of PHP superglobals.

However when we originally started to implement PHP_Codesniffer with this
particular standard we found a critical problem, the standard doesn't exclude or
threat the Phtml files differently.

This resulted on a lot of noise and false positivies been generated for the
frontend developers, for that reason we forked the standard and released our own
patched version that will handle phtml files differently and with a different
set of rules.

### Installation and Configuration

For details on how to install and configure PHPCodeSniffer and the Demac/ECG
coding standard please see the specific guides:

- [PHP_CodeSniffer
  Installation](coding_standards/php_codesniffer/installation.md)
- [ECG Coding Standard
  Installation](coding_standard/php_codesniffer/demac_ecg.md)
- [PHPStorm and PHP_CodeSniffer
  Setup](coding_standard/php_codesniffer/phpstorm_setup.md)


## Things to avoid

Now that we are fully setup, let's go over of some of the most common mistakes
and code smells that we want to avoid.

### Raw SQL Queries

There are several ways I have seen this one done, the most common one would be
to manually build a query using raw SQL and then pass the query to the Magento
database adapter.

#### Example code

```php
<?php

  $query =
   ' SELECT'
   . '   stores.store_id as store_id,'
   . '   product_entity.entity as product_id'
   . ' FROM ' . $coreTableVariableName . ' as product_entity'
   . ' JOIN ' . $coreTableSecondVariableName . ' as stores'
   . '   ON product_id.store_id = store.entity_id'
   . ' WHERE'
   . ' product.entity_id = 1';

   $this->_getWriteAdapter()->query($query);


```

#### Explanation

Is easy to forget that Magento is more than a simple ecommerce application, at
its core Magento its a full application framework build on top of the Zend Framework.

This means that Magento has a set of ORM classes that can be used to access the
database regardless of the backend that is being run (MySQL, MSSQL, Postgress,
etc).

Most of the time we can get the information that we need by just modifying the
collection that we are working on, it would be rare for a developer to be in
need of building a query by hand.

However from time to time we will be required to create our own queries, in
that case we should always try to use the database adapters, for example:

```php
<?php

  $adapter = $this->_getReadAdapter();
  $where   = $this->adapter->quoteInto('product.entity_id = ?', $productId);
  $from    = array('product' => Mage::getModel('core/resource')->getTableName('catalog/product')); 

  $storesTable    = array('product' => Mage::getModel('core/resource')->getTableName('catalog/product')); 


  $select = $adapter->select()
    ->from($from)
    ->join(
        array('stores' => $storesTable), 
        'product_id.store_id = store.entity_id',
    )
    ->where($where);

  $data = $adapter->fetchRow($select);

  // Do something with the data.

```

#### Guidelines

- Avoid writing Raw SQL queries when possible
- Never hardcode a table name inside a query, use the getTableName function
  instead
- If you current solution requires Raw SQL talk it over with another developer,
  preferably a Sr. Dev

---

### Queries inside a loop

This is another common mistake, and one that can be particularly hard to catch
when working locally or even on a staging environment.

This issue is not unique to Magento and is due a well know issue with the PHP
garbage collector and [circular references](coding_standards/references/circular-references.md).

#### Example code

```php
<?php
  
  $productCollection = Mage::getModel('catalog/product')->getCollection();

  foreach($productCollection as $product)
  {
    $product->load(); // Here is our problem 
    echo $product->getName();
  }

```

#### Explanation

On the previous code example there are two problems that can cause performance
issues and straight memory leaks.

The first issue arises due the way the collection is being iterated, since we
are passing the product collection as the first argument of the foreach loop
Magento will internally call load on that collection.

Resulting on high memory consumption and poor performance, normally the issue
can be avoided by using either a resource iterator or paginating the collection.

The second problem with that example is the use of the load function inside the
loop; **Load**, **Save** and **Delete** also commonly referred as **LSD**
operations (seriously who names this things) are particularly problematic when
called inside loops.

Instead of saving the products inside the foreach loop, we have several options
available, one of the is the usage of an iterator.

#### Guidelines

- **Load**, **Save** and **Delete** operations should be avoided inside loops.
- A **collection iterator** should be used instead of the LSD operation.

---

### Direct class instantiation 

> We unfortunately see this kind of code more often that we should when doing
> third party audits on sites and extensions.

#### Example code

```php
<?php

  // Instantiating a new model directly
  $directModel = new Demac_Example_Model_Something;

```

#### Explanation

The previous code is very common to see on developers that are either new to
Magento or that never learned how to use the factory methods and how they work.

Instantiating classes like that breaks one of the Magento design patterns, which
is the use of factories for class instantiation, for example:

```php
<?php 

  // Creating a new model
  $newModel = Mage::getModel('demac_example/something');

  // Creating a new helper
  $newHelper = Mage::helper('demac_example');
```

Magento 1 uses this design pattern combine with its module structure to provide
a way for developers to easily extend a class functionalty without directly
having to modify that, by simply overloading the original class.

#### Guidelines

- **Always** avoid instantiating Magento classes directly.

## Design Patterns

On this section we are going to review useful patterns that can help developers
write clean and maintainable code.

### Adapter Pattern

[//]: # (Describe the adapter pattern first introduced by james cowie and demonstrated in the chase extension)

> In software engineering, the adapter pattern is a software design pattern
> that allows the interface of an existing class to be used from another
> interface. It is often used to make existing classes work with others
> without modifying their source code. [Source:
> Wikipedia](http://en.wikipedia.org/wiki/Adapter_pattern)

#### Intent

The Adapter pattern mains goals are:

- Wrap an existing class with a new interface.
- Match and old component to a new system.
- Convert the interface of a class into a another interface the client expects.

There are multiple ways we can make use of the adapter pattern, from writing an
api wrapper that can be used by our extensions to separating the Magento
specific logic for our extension code.

The **Magento Adapter Pattern** was first introduced to me by [James
Cowie](http://jcowie.co.uk/);
and it provides an easy and simple way to decouple our extension code from the
Magento core code.

The main idea behind this pattern is to encapsulate the Magento specific code
inside an adapter class, making it testable and not as tightly couple to the
base Magento code.

Take a look at the following class: 

```php
<?php

/**
 * Class Demac_Chase_Model_Paymentech_Api_Adapter_Abstract
 *
 * This class contains the generic methods for any chase api adapter, specifically methods
 * that are relate to pulling information and values from the configuration.
 */
class Demac_Chase_Model_Paymentech_Api_Adapter_Abstract extends Mage_Core_Model_Abstract
{
...
    protected function getConfig($field, $storeId=null)
    {
        $path = 'payment/'.$this->_code.'/'.$field;
        return Mage::getStoreConfig($path, $storeId);
    }
...

```


