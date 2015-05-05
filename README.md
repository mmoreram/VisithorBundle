Visithor Bundle for Symfony
===========================

[![Build Status](https://travis-ci.org/Visithor/VisithorBundle.png?branch=master)](https://travis-ci.org/Visithor/VisithorBundle)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/Visithor/VisithorBundle/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/Visithor/VisithorBundle/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/visithor/visithor-bundle/v/stable.png)](https://packagist.org/packages/visithor/visithor-bundle)
[![Latest Unstable Version](https://poser.pugx.org/visithor/visithor-bundle/v/unstable.png)](https://packagist.org/packages/visithor/visithor-bundle)

Symfony Bundle for PHP Package [visithor](http://github.com/visithor/visithor),
a library that provides you a simple and painless way of testing your 
application routes with specific HTTP Codes.

Please read [Visithor](http://github.com/visithor/visithor) documentation in 
order to understand the final purpose of this project and how you should create
your config files.

## Installation

All you need to do is add this package in your composer under `require-dev` 
block and you will be able to test your application.

``` yaml
'require-dev':
    ...
    
    'visithor/visithor-bundle': '~0.1'
```

Then you have to update your dependencies.

``` bash
php composer.phar update
```

## Integration

This Bundle integrates the project in your Symfony project. This means that adds
all the commands in your project console, so when you do `app/console` you will
see the `visithor:*` commands.

``` bash
php app/console visithor:go
```

## Config

This Bundle provides you some extra features when defining your urls. Now you
can define your routes using the route name and an array of parameters.

``` yml
defaults:
    #
    # This value can be a simple HTTP Code or an array of acceptable HTTP Codes
    # - 200
    # - [200, 301]
    #
    http_codes: [200, 302]

urls:
    #
    # By default, is there is no specified HTTP Code, then default one is used
    # as the valid one
    #
    - http://google.es
    - http://elcodi.io
    
    #
    # There are some other formats available as well
    #
    - [http://shopery.com, 200]
    - [http://shopery.com, [200, 302]]
    
    #
    # This Bundle adds some extra formats
    #
    - [store_homepage, 200]
    - [[store_category_products_list, {'slug': 'women-shirts', 'id': 1}], 200]
    - [[store_category_products_list, {'slug': 'another-name', 'id': 1}], 302]
    - [[store_homepage, {_locale: es}]]
```

## Environment

Maybe you need to prepare your environment before Visithor tests your routes,
right? Prepare your database, load your fixtures, and whatever you need to make
your test installation works.

For this reason, you can define a service called `visitor.environment_builder`
than implements the interface 
`Visithor\Bundle\Environment\Interfaces\EnvironmentBuilderInterface`.

> You don't need to define such service because is injected if exists

If you take a look at this interface, you will se that you need to define two 
methods. The first one is intended to setUp your environment and will be called 
just once at the beginning of the suite. The second one will tear down such 
environment (for example removing database).

``` php
use Symfony\Component\HttpKernel\KernelInterface;
use Visithor\Bundle\Environment\Interfaces\EnvironmentBuilderInterface;

/**
 * Class EnvironmentBuilder
 */
class EnvironmentBuilder implements EnvironmentBuilderInterface
{
    /**
     * Set up environment
     *
     * @param KernelInterface $kernel Kernel
     *
     * @return $this Self object
     */
    public function setUp(KernelInterface $kernel)
    {
        //
    }

    /**
     * Tear down environment
     *
     * @param KernelInterface $kernel Kernel
     *
     * @return $this Self object
     */
    public function tearDown(KernelInterface $kernel)
    {
        //
    }
}
```

You can call some commands just creating a new Application given the Kernel is
passed as parameter and calling the commands with their parameters.

``` php
/**
 * Set up environment
 *
 * @param KernelInterface $kernel Kernel
 *
 * @return $this Self object
 */
public function setUp(KernelInterface $kernel)
{
    $application = new Application($kernel);
    $application->setAutoExit(false);
    
    $application->run(new ArrayInput(
        'command' => 'doctrine:database:create',
        '--env' => 'test',
        '--quiet' => true,
    ));
}
```

## Roles

You will, for sure, have the need to test your private routes. Of course, this
is a common need and this bundle satisfies it :)

Let's check our simple security file.

``` yaml
security:

    providers:
        in_memory:
            memory: ~

    firewalls:
        default:
            provider: in_memory
            http_basic: ~
            anonymous: ~

    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
        - { path: ^/superadmin, roles: ROLE_SUPERADMIN }
```

Then, let's see our Visithor configuration.

``` yaml
urls:
    - ['/', 200]
    - ['/admin', 200, {'role': 'ROLE_ADMIN', 'firewall': 'default'}]
    - ['/superadmin', 403, {'role': 'ROLE_ADMIN', 'firewall': 'default'}]
```

In this case, all routes are under the default firewall, called `default`.

Route `admin_route` is protected by the access role `ROLE_ADMIN`, and because we 
are testing against this role, then we'll receive a 200.

Route `superadmin_route` is protected by the access role `ROLE_SUPERADMIN`, but
in this case we are testing again using role `ROLE_ADMIN`, so we'll receive a
403 code.

Of course, you can define your firewall as a global option. Your routes will 
apply security only if both role and firewall options are defined.

``` yaml
defaults:
    options:
        firewall: default
urls:
    - ['/', 200]
    - ['/admin', 200, {'role': 'ROLE_ADMIN'}]
    - ['/superadmin', 403, {'role': 'ROLE_ADMIN'}]
```