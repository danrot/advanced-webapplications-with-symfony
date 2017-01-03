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

[^1]: <http://php.net/manual/en/language.basic-syntax.phptags.php>
[^2]: <http://php.net/manual/en/language.oop5.php>
[^3]: <http://php.net/manual/en/language.namespaces.php>
[^4]: <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md>
[^5]: <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md>
[^6]: <http://php.net/manual/en/>
[^7]: <https://getcomposer.org/>
[^8]: <https://getcomposer.org/doc/articles/versions.md>
