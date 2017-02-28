% Advanced Web Applications with Symfony
% Daniel Rotter

# Introduction

## PHP

PHP is a general-purpose programming language, which was primarily designed for
programming server-side web applications.

PHP files end usually with the file extension `.php` and mark the start of PHP
code with the tag `<?php`. It is recommended not to use the ending tag `?>` in
PHP only files, since they can introduce problems[^1]. So a simple "Hello
World"! program can look like this:

```php
<?php
echo 'Hello world';
```

The OOP part of PHP is heavily inspired by Java[^2]. The next listing shows a
simple class with a method call.

```php
<?php
class Output
{
    protected $value;

    public function __construct($value)
    {
        $this->value = $value;
    }

    public function print()
    {
        echo $this->value;
    }
}

$output = new Output('This is my value');
$output->print();
```

The most visible difference to Java is that variables are prefixed with a `$`,
and that methods on objects are called with `->` instead of `.`.

Classes can extend other classes with the `extends` keyword.

```php
class LogOutput extends Output
{
    public function print()
    {
        echo 'DEBUG: ';
        parent::print();
    }
}
```

Interfaces can be created with the `interface` keyword and are implemented by
classes with the `implements` keyword. All method signatures from the interface
must be implemented by implementations of that interface, otherwise there will
be an error.

```php
interface Output
{
    public function print();
}

class DefaultOutput implements Output
{
    public function print()
    {
        echo 'Print something!';
    }
}
```

All these elements are located in some namespace[^3]. By default this is the
global namespace `\`. Subnamespaces are divided by `\`. Elements from other
namespaces must either be prefixed with the namespace it is contained in, or
must be imported with the `use` keyword first. Namespaces are defined with the
`namespace` keyword and are valid for the rest of the file.

```php
namespace My\Great\Application\Console;

use Monolog\Logger;

$logger = new Logger();
```

There are recommendations on the coding standard in PHP, named PSR-1[^4] and
PSR-2[^5]. The code examples above are following these standards.

For further information have a look at the exhaustive documentation[^6].

## Composer

The dependency managament tool for PHP is called Composer[^7]. It allows to
reuse libraries or packages other people have written. Therefore every project
has a `composer.json` file in its root directory. The file consists of an JSON
object like in the following example:

```json
{
    "require": {
        "monolog/monolog": "^1.22.0"
    }
}
```

The `require` key is the most important one, and describes the dependencies of
the project. In this case it depends on monolog, a logging library. The name
consists of a vendor and project part, which is divided by a slash. This key is
representing the name of the project, whereby the value defines the versions
which can be used. The `^` means that every version starting with the given one
and up to the next major version (2.0 in this case) might be used. There is a
reference on the Composer documentation about all the available version
operators[^8].

In addition to installing the dependencies Composer is also responsible for
generating the autoloader, which allows us to use components based on the full
qualified name consisting of the namespace and the class name with the `use`
statement we have already seen before.

## Symfony

Symfony is a framework for building web applications with PHP. There is a
standard edition already defining the folder structure. The Composer command
`create-project` can be used to create a new application built on top of the
Symfony Standard Edition[^9]:

```bash
composer create-project symfony/framework-standard-edition
```

This command will download the standard edition and install all required
dependencies. It also asks for a few parameters being installation specific,
like the parameters for the database connection. You can leave the defaults,
unless you have reason to change them. Check the Symfony documentation[^10] for
more details about the installation process.

One of the most important files in the standard directory is `bin/console`. It
allows to run tasks defined by the Symfony application. E.g. `server:run`
starts a webserver we can use for development. This makes it easier to run this
application, since no separate webserver has to be set up. Also, since the
webserver is running under the same user, there won't be any permission issues.

So the next line on the terminal should run a webserver on
<http://localhost:8000>, where a welcome screen should be shown for now.

```bash
bin/console server:run
```

# Routing & Controller

In order to return a page two things are needed: a route and a controller.

* The controller is a PHP function, which gets a `Request` object and returns a
`Response` object.
* The route maps a URL to a specific controller.

Since these two tasks are related to HTTP, it is a good idea to think of
Symfony as a HTTP framework.

## Controller

Symfony comes with a base class called `Controller`, which adds a few
convenience methods making it easier to use. This couples the controller to the
Symfony Framework, which is fine for a custom client applications, since
changing the underlying framework will probably result in a complete rewrite
anyway.

Each controller can contain multiple actions, being methods suffixed with
`Action` by convention. Every action must return a `Response` object
representing a HTTP response.

So a simple "Hello World" action could be placed in a controller located at
`src/AppBundle/Controller/IndexController` and look like this:

```php
<?php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class IndexController extends Controller
{
    public function indexAction(Request $request, $name = 'World')
    {
        return new Response(sprintf('Hello %s!', $name));
    }
}
```

Note that the `$request` parameter is not used in this example. It is only
there to see how to get access to the sent request, in case something must be
read from there. This parameter could also be omitted here, because access to
the request is not required in this example. The Symfony documentation explains
how to use the `Request` and `Response` classes[^11].

The `indexAction` takes a `$name` parameter, which is used when generating a
HTTP response with a simple string as a body. This parameter is passed via the
routing mechanism of Symfony and set to `World` in case it is not given.

## Routing

The action shown previously is not accessible as it is, since no route is using
it. Therefore a route using this action must be created. The Symfony routing
documentation[^12] shows that this is possible via annotations, YAML, XML and
plain PHP.

Which of these should be used are again depending on the use case. For a custom
client project annotations are a good fit, since the fact that they couple the
class to the framework is negligible. But for a library, which might even be
used in different frameworks, this is not acceptable, leaving YAML or XML as
the better fit.

Since this guide concentrates on developing individual applications the example
is showing how to handle routing with annotations based on the previous
controller.

```php
<?php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class IndexController extends Controller
{
    /**
     * @Route("/{name}", name="homepage")
     */
    public function indexAction($name = 'World')
    {
        return new Response(sprintf('Hello %s!', $name));
    }
}

```

The `@Route` annotation marks the method as the controller for a route. The
first argument of the annotation is the pattern for the route. Placeholders are
enclosed in curly brackets and passed to the same named parameters of the
method. The `name` argument of the annotation is used to identify the route.
This name can be used to generate a URL[^13] or to find the route when using
the `debug:route` command on the command line:

```
 $ bin/console debug:route
 ---------- -------- -------- ------ ---------
  Name       Method   Scheme   Host   Path
 ---------- -------- -------- ------ ---------
  homepage   ANY      ANY      ANY    /{name}
 ---------- -------- -------- ------ ---------
```

However, the controller file must still be registered `app/config/routing.yml`
as a source for the route configuration, otherwise the system does not know
about the containing routes.

```yaml
app:
    resource: "@AppBundle/Controller/"
    type:     annotation
```

This configuration described that every file in the `Controller` folder of the
`AppBundle` might contain `@Route` annotations, which must be considered during
the routing process.

# Templating

Manually composing a string as seen in the previous example when creating the
`Response` object works, but will get very cumbersome when returning complex
HTML structures. The goal of template languages is to make working with such
structures more enjoyable. Therefore Symfony is primarily used with Twig[^14].

Think of the templates as the view part of the application, which only covers
presentation, but not business logic. That means instead the controller is
composing the HTML itself it is delegating this task to the template engine.
The base controller of Symfony has the `render` method for that:

```php
<?php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class IndexController extends Controller
{
    /**
     * @Route("/{name}", name="homepage")
     */
    public function indexAction($name = 'World')
    {
        return $this->render('index.html.twig', [
            'name' => $name,
        ]);
    }
}
```

The `render` method returns a `Response` object with the content of the
template, so we can simply return its return value instead of creating our own
`Response` object.

The first argument `index.html.twig` is the name of the template. It is
locating the path to the template starting from the `app/Resources/views`
directory. Mind that the templates might also be located in other places, which
might be useful when developing a reusable library. Again, the Symfony
documentation[^15] explains how to access the templates in this case.

The second argument is an array containing the data for the template. The array
defines the name of the variables in the twig template and its value.

A simple twig template rendering the name in the title and in the body should
therefore be located at `app/Resources/views/index.html.twig` and can look like
this:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Hello {{name}}!</title>
    </head>
    <body>
        <h1>Hello {{name}}, welcome on our Homepage!</h1>
    </body>
</html>
```

`{{name}}` is the way you print the content of the `name` variable. But twig
can do a lot more than only replacing placeholders, it has also control
structures that help implementing presentation logic in twig templates, like
`if` to e.g. display certain data only on a specific condition or `for` to loop
over an array of data. The twig documentation[^16] has a page explaining all
the methods important for developers implementing templates. This page also
mentions template inheritance[^17], which is very useful for almost every real life
web application, since it allows to define a base frame for every page.

# Translations

Symfony comes with a translation component[^18], which can be used to translate
static text in your application. The first thing that has to be done is to
activate the translator in the configuration:

```yaml
framework:
    translator: { fallbacks: ["%locale%"] }
```

There is already a commented line like that in the standard edition of Symfony.

There are many ways to put the translations files, but the one recommended by
the Symfony Best Practices is `app/Resources/translations`. This is also the
one with the highest priority, meaning they take precedence over all the other
locations. There are different formats to use, but XLIFF is the most popular
one. So a file with the translation which was used in the previous template
example should be named `app/Resources/translations/messages.en.xlf` might look
like this:

```xliff
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" target-language="en"
        datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="welcome">
                <source>welcome</source>
                <target>Hello %name%, welcome on our homepage</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

The translation can be used in PHP, e.g. when used in the Controller with
`$this->get('translator')->trans()`:

```php
<?php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class IndexController extends Controller
{
    /**
     * @Route("/{_locale}/{name}", name="homepage")
     */
    public function indexAction($name = 'World')
    {
        return $this->render('index.html.twig', [
            'name' => $name,
            'greeting' => $this->get('translator')->trans(
                'welcome',
                ['%name%' => $name]
            ),
        ]);
    }
}
```

Mind the `{_locale}` parameter in the route. This means that the locale is
included in the URL, so calling `/en/max` will set the locale to English,
whereby `/de/max` will set the locale to German. This way the locale can be
easily switched by using different URLs. The translator will use the other
localization file if it exists and fall back to English as specified in the
configuration.

So adding German as another language is as easy as adding another
`app/Resources/translations/messages.de.xlf` file, and changing the
translations.

But quite often the translation is used in the twig template. Therefore some
twig methods are available. So instead of using the translator in the 
controller the method can be called from the twig template:

```jinja
<!DOCTYPE html>
<html>
    <head>
        <title>Hello {{name}}!</title>
    </head>
    <body>
        <h1>{{ 'welcome'|trans({'%name%': name}) }}</h1>
    </body>
</html>
```

# Database

Almost every reasonable real life web application has to store data somewhere.
A quite common approach is to use a relational database for that. MySQL works
pretty good with PHP and is also OpenSource, so these two technologies are a
very popular combination.

There are different ways to access a MySQL database in PHP. There is the mysqli
extension[^19], which solely allows to work with MySQL. A second option is the
PDO extension[^20], which is also a data access abstraction layer. That means
the same methods are used for every database, but the SQL sent to these
databases have still to be adapted.

As the documentation says Symfony is not coupled to any of these methods[^21],
but it provides a tight integration for doctrine[^22], which was heavily
inspired by the Java ORM Hibernate.

## Configuration

When creating a new project with Symfony Composer will already ask for some
parameters. This also includes the database connection parameters:

* `database_host`
* `database_port`
* `database_name`
* `database_user`
* `database_password`

These parameters are stored in the `app/config/parameters.yml` file, and can
be adapted later as well.

There is a simple command to create the configured database:

```bash
bin/console doctrine:database:create
```

## Entities

Doctrine is an object relational mapper, which means that it is persisting
objects into a relational database. Therefore it needs some objects to do so.
These objects are called entities, and they are defined by a class. There are
no rules where to put the class, but it makes sense to have a `Entity` folder
in your Bundle.

The given example uses a `Product` and `Purchase` entity for illustration
purposes.

```php
// src/AppBundle/Entity/Product.php
<?php

namespace AppBundle\Entity;

class Product
{
    /**
     * @var string
     */
    private $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setName($name)
    {
        $this->name = $name;
    }
}
```

The `Product` entity consists of only a name, and has no relations to other
entities.

```php

// src/AppBundle/Entity/Purchase.php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Product;

class Purchase
{
    /**
     * @var Product[]
     */
    private $products;

    public function __construct(array $products = [])
    {
        $this->products = $products;
    }

    public function getProducts()
    {
        return $this->products;
    }

    public function addProduct(Product $product)
    {
        $this->products[] = $product;
    }
}
```

The `Purchase` entity is only a list of products, for a more complete and complex
example more information would be needed.

It is very important to see here, that the entities have no dependencies on
doctrine at all. This is the most important characteristics of the data mapper
pattern[^23], which doctrine is implementing.

## Mapping

In order to correctly handle the entities doctrine needs some metadata about
them. As with the routes there are three possibilities to define this metadata:
Annotations, XML and YAML. Annotations are easy to use, because they are
directly connected to your code. For this reason they are the first choice of
the Symfony Best Practices[^24].

However, keep in mind that the direct connection is also their biggest
weakness, since the metadata information is tightly coupled to the code. So for
reusable libraries XML or YAML are the preferable way to specify the metadata.

Since this guide is about building individual applications it will still use
annotations for the mapping. Therefore the mapping namespace must be included
and used for the annotations, as in the following example in the Product:

```php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Purchase;
use Doctrine\ORM\Mapping as ORM;
use Ramsey\Uuid\Uuid;

/**
 * @ORM\Entity
 */
class Product
{
    /**
     * @ORM\Id
     * @ORM\Column(type="guid")
     * @var string
     */
    private $uuid;

    /**
     * @ORM\Column(type="string")
     * @var string
     */
    private $name;

    /**
     * @ORM\ManyToOne(targetEntity="Purchase", inversedBy="products")
     * @ORM\JoinColumn(name="purchase_uuid", referencedColumnName="uuid")
     * @var Purchase
     */
    private $purchase;

    public function __construct($name)
    {
        $this->uuid = Uuid::uuid4();
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setName($name)
    {
        $this->name = $name;
    }
}
```

Mind that there have been two additions, which are somehow database related.
The first one is the UUID, which allows us to identify a product. Having such
a value is a good idea anyway. The example makes use of a UUID because of
various reasons[^25].

The other thing is a relation to an order, although we might not want to have
that in our domain model. This is because of a technical limitation: The
foreign key in a relational model has to be written on the many-side, although
in our use case it might be more important to have a relation from the order to
the product. The `JoinColumn` annotation describes the name of the column
storing the foreign key (`purchase_uuid`) and the name of the column in the other
entity (`uuid`). The `ManyToOne` annotation defines the other side of the
relation and the entity variable containing the reference to the product.

```php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Product;
use Doctrine\ORM\Mapping as ORM;
use Ramsey\Uuid\Uuid;

/**
 * @ORM\Entity
 */
class Purchase
{
    /**
     * @ORM\Id
     * @ORM\Column(type="guid")
     * @var string
     */
    private $uuid;

    /**
     * @ORM\OneToMany(targetEntity="Product", mappedBy="purchase")
     * @var Product[]
     */
    private $products;

    public function __construct(array $products = [])
    {
        $this->uuid = Uuid::uuid4();
        $this->products = $products;
    }

    public function getProducts()
    {
        return $this->products;
    }

    public function addProduct(Product $product)
    {
        $this->products[] = $product;
    }
}

```

The purchase entity also has an UUID and the other side of the order-product
relation. It does not include the `JoinColumn` annotation, since this field is
not represented in the database. But doctrine needs this metadata in order to
make the relation work in both directions.

After the metadata mapping has been created the following command will create
the database schema:

```bash
bin/console doctrine:schema:create
```

If a schema was already existing before, the following command will update the
schema:

```bash
bin/console doctrine:schema:update --force
```

Be aware that this command should not be used in production, since the doctrine
migrations[^26] are doing a much better job.

## Usage

The following code shows how to insert data into the database. This code can be
used in a Symfony command[^27], which is a nice place to play around with new
stuff, because they are very easy to setup.

```php
$entityManager = $this->getContainer()
    ->get('doctrine.orm.entity_manager');

$product = new Product('Toaster');

$entityManager->persist($product);
$entityManager->flush();
```

The `EntityManager` is the public API offered by doctrine to work with
entities. When a new `Product` is created doctrine does not know about it yet.
Therefore the `persist` method has to be called. However, this only means that
the data is handled by doctrine from now on, but there has no insertion
happened yet. The data will be written to the database when the `flush` method
is called.

For reading values from the database there are various opportunities[^28]. The
easiest one is to make use of a `Repository`. Every entity has its own
repository, which can be retrieved from the `EntityManager` by passing a
combination of the bundle and entity name to the `getRepository` method. The
repository has various methods to load entities it is responsible for, whereby
`find` is the simplest one, since it simply takes the ID of the entity and
returns a matching object.

This object can then be altered in PHP, and the values in the database will be
adjusted as soon as `flush` is again called on the `EntityManager`. Mind that
it is not necessary anymore to call `persist`, because doctrine has already
loaded the entity and therefore knows about it.

```php
$entityManager = $this->getContainer()
    ->get('doctrine.orm.entity_manager');

$product = $entityManager->getRepository('AppBundle:Product')
    ->find('some-uuid');
$product->setName('Roaster');

$entityManager->flush();
```

Entities can also be deleted by passing the reference to the `remove` method of
the `EntityManager` and calling `flush` afterwards:

```php
$entityManager = $this->getContainer()
    ->get('doctrine.orm.entity_manager');

$product = $entityManager->getRepository('AppBundle:Product')
    ->find('some-uuid');

$entityManager->remove($product);
$entityManager->flush();
```

## Optimizations

Reading data from the database as explained above can have a serious perfomance
impact, especially when associations are included. One problem is overfetching,
meaning that more data than needed is loaded. However, the queries above have a
different problem. When loading a `Purchase` object and accessing the
association containing the `Product`, the `Product` entities will be lazily
instantiated, and that includes another database call.

### DQL

The Doctrine Query Language[^29] is one way to tackle this problem. It is a
language similar to SQL, but works with objects instead of database tables and
rows.

It can solve problems, which occur when loading entities in the following way:

```php
$purchaseRepository = $this->getDoctrine()
    ->getRepository('AppBundle:Purchase');

$purchase = $purchaseRepository->find($id);
$products = $purchase->getProducts();
```

This code will create database queries when calling the `find` method on the
`$purchaseRepository` and when lazily loading the products when calling
`getProducts` on `$purchase`. The latter one can be easily avoided with a DQL
query:

```php
$query = $this->getDoctrine()
    ->getManager()
    ->createQuery(
        'SELECT pu, pr'
        . ' FROM AppBundle:Purchase pu'
        . ' JOIN pu.products pr'
        . ' WHERE pu.uuid = :uuid'
    );
$query->setParameter('uuid', $id);

$purchase = $query->getSingleResult();
$products = $purchase->getProducts();
```

In this case `getProducts` will not query the database another time, because
the `JOIN` in the DQL already loads the products eagerly. If only the products
are needed the `JOIN` can be avoided in general, making the query even faster:

```php
$query = $this->getDoctrine()
    ->getManager()
    ->createQuery(
        'SELECT pr'
        . ' FROM AppBundle:Product pr'
        . ' WHERE pr.purchase = :uuid'
    );
$query->setParameter('uuid', $id);

$products = $query->getResult();
```

In case the object graph is not needed the performance can also be increased by
using a different hydration mode[^30].

There is also a QueryBuilder[^31], which uses an object-oriented interface to
create query.

### Custom repositories

DQL queries are a great way to improve the performance of a web application,
but it would be very bad for maintainability to spread them all over the
application.

For this reason doctrine has introduced custom repositories[^32], which allow
to group all the DQL queries in a meaningful way. The first thing to do is to
implement a repository class:

```php
<?php

namespace AppBundle\Repository;

use Doctrine\ORM\EntityRepository;

class ProductRepository extends EntityRepository
{
    public function getProductsByPurchaseUuid($purchaseUuid)
    {
        return $this->createQueryBuilder('p')
            ->where('p.purchase = :purchaseUuid')
            ->setParameter('purchaseUuid', $purchaseUuid)
            ->getQuery()
            ->getResult();
    }
}
```

The class has to inherit from the `EntityRepository` base implementation of
doctrine. The example also uses the previously mentioned `QueryBuilder`, whose
initialization is handled by the parent class.

Then doctrine has to be configured in a way that it knows about the custom
repository class. This is done in the mapping configuration, in the following
example annotations are used, but the same is possible with YAML and XML:

```php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Purchase;
use Doctrine\ORM\Mapping as ORM;
use Ramsey\Uuid\Uuid;

/**
 * @ORM\Entity(repositoryClass="AppBundle\Repository\ProductRepository")
 */
class Product
{
    // ...
}
```

Afterwards the new repository can be used to e.g. load all the products for a
given purchase in the controller:

```php
$productRepository = $this->getDoctrine()
    ->getRepository('AppBundle:Product');

return $this->render('purchase.html.twig', [
    'products' => $productRepository->getProductsByPurchaseUuid($id)
]);
```

### Caching

Doctrine has three different caching possibilities to improve performance: The
metadata cache, the query cache and the result cache[^33]. Symfony allows to
easily set these caches in its configuration[^34].

Doctrine has adapter to different cache systems. APCu[^35] is one of them, and
will serve as an example. All the different adapters can be used for all three
types of caches.

Using the metadata and query cache is always highly recommended in a production
environment. The metadata cache will avoid reading the mapping metadata from
annotations, XML or YAML again for every request, and the query cache does the
same for the transition from DQL to SQL queries.

The result cache is a little bit different. It caches the results from the
queries, so that no database call is made for the configured duration. That
means that a change in the database is not immediately reflected in the web
application. Although it is a huge performance boost it might not always be an
acceptable tradeoff. That is also the reason this cache type has to be
activated for each query.

These caches should be activated in the `app/config/config_prod.yml` file,
which is only loaded in a production environment.

```yaml
doctrine:
    orm:
        metadata_cache_driver: apcu
        query_cache_driver: apcu
        result_cache_driver: apcu
```

The caches for metadata and queries are immediately active for the entire
application. Enabling the result cache for certain queries is just a
`useResultCache(true)` call on the query. The following snippet uses the
`QueryBuilder` from the previous `ProductRepository` example.

```php
$this->createQueryBuilder('p')
    ->where('p.purchase = :purchaseUuid')
    ->setParameter('purchaseUuid', $purchaseUuid)
    ->getQuery()
    ->useResultCache(true)
    ->getResult();
```

### Native Queries

Native queries[^36] should only be used if the previous optimizations are not
good enough, because they bypass the doctrine mapping and have to be mapped
once more, which is a tedious task. Therefore they allow to add vendor-specific
optimizations and stored procedures to the query.

The query is created using the `createNativeQuery` method of the
`EntityManager`, which takes the SQL query and a `ResultSetMapping`, which
describes how the data should be mapped to the object. This looks like the
following example when implemented in the `ProductRepository`:

```php
$resultSetMappingBuilder = $this->createResultSetMappingBuilder('p');

$sql = sprintf(
    'SELECT %s FROM product p WHERE p.purchase_uuid = ?',
    $resultSetMappingBuilder->generateSelectClause(['p' => 'p'])
);

$query = $this->getEntityManager()
    ->createNativeQuery($sql, $resultSetMappingBuilder);
$query->setParameter(1, $purchaseUuid);

return $query->getResult();
```

Implementing the `ResultSetMapping` can get quite complicated for big queries
and adds mental overhead, for these reasons it should really be the last
resort.

# Forms & Validation

A very important part of a web application is the interaction with the user.
This interaction can be handled with forms[^37], which is handled by a separate
component in Symfony. There is another component for validations. These two
components are a powerful combination together.

## Forms

Forms can be build using a `FormBuilder`, which combines multiple `FormType`s
into a form, which can be rendered to HTML and convert data from a request to
an object.

Therefore it is important that all accessed properties are either public or
have getters and setters. As an example the `Purchase` class is used:

```php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Product;
use Doctrine\ORM\Mapping as ORM;
use Ramsey\Uuid\Uuid;

/**
 * @ORM\Entity
 */
class Purchase
{
    /**
     * @ORM\Id
     * @ORM\Column(type="guid")
     * @var string
     */
    private $uuid;

    /**
     * @ORM\Column(type="datetime")
     * @var \DateTime
     */
    private $date;

    /**
     * @ORM\OneToMany(targetEntity="Product", mappedBy="purchase")
     * @var Product[]
     */
    private $products;

    public function __construct(\DateTime $date = null, array $products = [])
    {
        $this->uuid = Uuid::uuid4();
        $this->date = $date ?: new \DateTime();
        $this->products = $products;
    }

    public function getUuid()
    {
        return $this->uuid;
    }

    public function getDate()
    {
        return $this->date;
    }

    public function setDate(\DateTime $date)
    {
        $this->date = $date;
    }

    public function setProducts($products)
    {
        $this->products = $products;
    }

    public function getProducts()
    {
        return $this->products;
    }

    public function addProduct(Product $product)
    {
        $this->products[] = $product;
    }
}
```

The mapping from a form to the class is done in a `FormType`:

```php
<?php
namespace AppBundle\Form;

use AppBundle\Entity\Purchase;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\OptionsResolver\OptionsResolver;

class PurchaseType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('date', DateTimeType::class)
            ->add('products', EntityType::class, [
                'class' => 'AppBundle:Product',
                'choice_label' => 'name',
                'multiple' => true,
            ])
            ->add('save', SubmitType::class);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Purchase::class,
        ]);
    }
}
```

The `FormBuilder` passed to the `buildForm` method can be used to build the
form. With the `add` method new properties are added, which have to match the
name of the properties of the described class. The second argument describes
what kind of value that property holds. The Symfony documention[^38] contains a
list of available built-in form types.

The `EntityType` takes additional parameters, which are passed as third
parameter to the `add` method. They describe which entity is loaded for a
select.

Finally the form has to be rendered in a controller. The form will be sent to
the same action using `POST` as method. The form component knows if the form is
already submitted and if the submitted data is valid. Hence the controller has
to consider three different paths:

* The form has not been submitted yet
* The form has been submitted with invalid data
* The form has been submitted with valid data

```php
<?php
namespace AppBundle\Controller;

use AppBundle\Entity\Product;
use AppBundle\Entity\Purchase;
use AppBundle\Form\ProductType;
use AppBundle\Form\PurchaseType;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class IndexController extends Controller
{
    /**
     * @Route("purchases/new", name="purchase_create")
     */
    public function newPurchaseAction(Request $request)
    {
        $purchase = new Purchase();
        $form = $this->createForm(PurchaseType::class, $purchase);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $purchase = $form->getData();

            foreach ($purchase->getProducts() as $product) {
                $product->setPurchase($purchase);
            }

            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($purchase);
            $entityManager->flush();

            return $this->redirectToRoute('purchase', [
                'id' => $purchase->getUuid(),
            ]);
        }

        return $this->render('purchase.form.html.twig', [
            'form' => $form->createView(),
        ]);
    }

    /**
     * @Route("/purchases/{id}", name="purchase")
     */
    public function purchaseAction($id)
    {
        $productRepository = $this->getDoctrine()
            ->getRepository('AppBundle:Product');

        return $this->render('purchase.html.twig', [
            'products' => $productRepository->getProductsByPurchaseUuidNative($id),
        ]);
    }
}
```

The `createForm` takes the previously defined `PurchaseType`. The created form
will handle the given `Request`. The next `if` checks if the form has been
submitted and is valid. In case it is the `Purchase` object will be returned
from the `getData` method of the form and persist and flush it. In this special
case all the `Product`s from the Purchase must be iterated, so that the
`Purchase` object can be set on the `Product`. This is necessary because the
`Product` is the owning side of the association between the `Product` and
`Purchase`, and the data would not be saved otherwise.

Afterwards the `redirectToRoute` method is called to redirect ths user to the
just created product. It takes the name of the route and the passed parameters.

In case the form was not submitted the form will be rendered using a template.
When the case has already been submitted with invalid data, the validation
errors will also be passed. Therefore the correct methods in twig have to be
used:

```jinja
{{ form_start(form) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```

`form_start` renders the start tag of the form, `form_widget` renders all the
fields of the form and `form_end` render the end tag and hidden form fields.

## Validation

An important part of form management is the validation. Actually validation is
a completely separate component in Symfony[^39]. However, the validation
component is used when the `$form->isValid()` method is called. This call is
actually not validating the form, but the object created by the form.

Therefore the metadata about the validation has to be added to the actual
object. The most simple validation to use is the `NotBlank` assertion, which
looks something like this:

```php
<?php

namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Product
{
    // ...

    /**
     * @ORM\Column(type="string")
     * @Assert\NotBlank()
     * @var string
     */
    private $name;

    // ...
}
```

There are of course a lot other assertions available. The documentation has a
good overview of them [^40]. If none of them match the requirements it is also
possible to develop an own validator[^41].

[^1]: <http://php.net/manual/en/language.basic-syntax.phptags.php>
[^2]: <http://php.net/manual/en/language.oop5.php>
[^3]: <http://php.net/manual/en/language.namespaces.php>
[^4]: <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md>
[^5]: <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md>
[^6]: <http://php.net/manual/en/>
[^7]: <https://getcomposer.org/>
[^8]: <https://getcomposer.org/doc/articles/versions.md>
[^9]: <https://github.com/symfony/symfony-standard>
[^10]: <http://symfony.com/doc/current/setup.html>
[^11]: <http://symfony.com/doc/current/components/http_foundation.html>
[^12]: <http://symfony.com/doc/current/routing.html>
[^13]: <http://symfony.com/doc/current/routing.html#generating-urls>
[^14]: <http://twig.sensiolabs.org/>
[^15]: <http://symfony.com/doc/current/templating.html#template-naming-and-locations>
[^16]: <http://twig.sensiolabs.org/doc/1.x/templates.html>
[^17]: <http://twig.sensiolabs.org/doc/1.x/templates.html#template-inheritance>
[^18]: <http://symfony.com/doc/current/translation.html>
[^19]: <http://php.net/manual/en/book.mysqli.php>
[^20]: <http://php.net/manual/en/book.pdo.php>
[^21]: <http://symfony.com/doc/current/doctrine.html>
[^22]: <http://www.doctrine-project.org/>
[^23]: <https://martinfowler.com/eaaCatalog/dataMapper.html>
[^24]: <http://symfony.com/doc/current/best_practices/business-logic.html#doctrine-mapping-information>
[^25]: <https://philsturgeon.uk/http/2015/09/03/auto-incrementing-to-destruction/>
[^26]: <http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html>
[^27]: <http://symfony.com/doc/current/console.html>
[^28]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/working-with-objects.html#querying>
[^29]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html>
[^30]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html#hydration-modes>
[^31]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html>
[^32]: <http://symfony.com/doc/current/doctrine/repository.html>
[^33]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/caching.html#integrating-with-the-orm>
[^34]: <https://symfony.com/doc/current/reference/configuration/doctrine.html#caching-drivers>
[^35]: <http://php.net/manual/en/book.apcu.php>
[^36]: <http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/native-sql.html>
[^37]: <http://symfony.com/doc/current/forms.html>
[^38]: <http://symfony.com/doc/current/forms.html#built-in-field-types>
[^39]: <https://symfony.com/doc/current/validation.html>
[^40]: <https://symfony.com/doc/current/validation.html#constraints>
[^41]: <https://symfony.com/doc/current/validation/custom_constraint.html>
