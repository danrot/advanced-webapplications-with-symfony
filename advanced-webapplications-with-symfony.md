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

# Database

Almost every reasonable real life web application has to store data somewhere.
A quite common approach is to use a relational database for that. MySQL works
pretty good with PHP and is also OpenSource, so these two technologies are a
very popular combination.

There are different ways to access a MySQL database in PHP. There is the mysqli
extension[^18], which solely allows to work with MySQL. A second option is the
PDO extension[^19], which is also a data access abstraction layer. That means
the same methods are used for every database, but the SQL sent to these
databases have still to be adapted.

As the documentation says Symfony is not coupled to any of these methods[^20],
but it provides a tight integration for doctrine[^21], which was heavily
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

The given example uses a `Product` and `Order` entity for illustration
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

// src/AppBundle/Entity/Order.php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Product;

class Order
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

The `Order` entity is only a list of products, for a more complete and complex
example more information would be needed.

It is very important to see here, that the entities have no dependencies on
doctrine at all. This is the most important characteristics of the data mapper
pattern[^22], which doctrine is implementing.

## Mapping

In order to correctly handle the entities doctrine needs some metadata about
them. As with the routes there are three possibilities to define this metadata:
Annotations, XML and YAML. Annotations are easy to use, because they are
directly connected to your code. For this reason they are the first choice of
the Symfony Best Practices[^23].

However, keep in mind that the direct connection is also their biggest
weakness, since the metadata information is tightly coupled to the code. So for
reusable libraries XML or YAML are the preferable way to specify the metadata.

Since this guide is about building individual applications it will still use
annotations for the mapping. Therefore the mapping namespace must be included
and used for the annotations, as in the following example in the Product:

```php
<?php

namespace AppBundle\Entity;

use AppBundle\Entity\Order;
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
     * @ORM\ManyToOne(targetEntity="Order", inversedBy="products")
     * @ORM\JoinColumn(name="order_uuid", referencedColumnName="uuid")
     * @var Order
     */
    private $order;

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
various reasons[^24].

The other thing is a relation to an order, although we might not want to have
that in our domain model. This is because of a technical limitation: The
foreign key in a relational model has to be written on the many-side, although
in our use case it might be more important to have a relation from the order to
the product. The `JoinColumn` annotation describes the name of the column
storing the foreign key (`order_uuid`) and the name of the column in the other
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
class Order
{
    /**
     * @ORM\Id
     * @ORM\Column(type="guid")
     * @var string
     */
    private $uuid;

    /**
     * @ORM\OneToMany(targetEntity="Product", mappedBy="order")
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

The Order entity also has an UUID and the other side of the order-product
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
migrations[^25] are doing a much better job.

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
[^18]: <http://php.net/manual/en/book.mysqli.php>
[^19]: <http://php.net/manual/en/book.pdo.php>
[^20]: <http://symfony.com/doc/current/doctrine.html>
[^21]: <http://www.doctrine-project.org/>
[^22]: <https://martinfowler.com/eaaCatalog/dataMapper.html>
[^23]: <http://symfony.com/doc/current/best_practices/business-logic.html#doctrine-mapping-information>
[^24]: <https://philsturgeon.uk/http/2015/09/03/auto-incrementing-to-destruction/>
[^25]: <http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html>
