---
layout: post
title: "Using composer with CakePHP 2.x"
date: 2013-02-22 00:21
comments: true
categories:
published: true
---

Lately i have been hacking on [Opauth](http://opauth.org) and finally was
forced to learn more about composer.
Within the CakePHP community composer has not been adopted really,
altough `composer-installer` supports an easy way for installing `cakephp-plugin` already.
Many plugin maintainers have not added `composer.json` files to their repositories, as CakePHP itself is not using it as well.
There is no need to stop you from using composer with your CakePHP application. You can install both plugins and 3rd party libraries easily.
Currently you would clone/download required libraries to the `vendors/` or `app/Vendor/` directory.
<!--more-->

For libraries that support composer install, here is the way to go:

```
$ cd app/
$ curl -s https://getcomposer.org/installer | php
$ php composer.phar require --no-update opauth/opauth:dev-wip/1.0 opauth/twitter:dev-wip/1.0
$ php composer.phar config vendor-dir Vendor
$ php composer.phar install
```

Note: the following would also work, but would not add the vendor-dir to `composer.json`
```
$ COMPOSER_VENDOR_DIR=Vendor php composer.phar install
```

This will create a `composer.json` file with the required dependencies for your application.
`--no-update` is needed here to avoid installing directly to the default directory for composer, which is `vendor/`
You can also use the init command for creating the composer.json, only make sure you have configured the `vendor-dir`
before running `install` or `require`

When running install it would produce the following output
```
Loading composer repositories with package information
Installing dependencies
  - Installing themattharris/tmhoauth (0.7.5)
    Cloning 71e2594f5a6b720a309c627bd457926e1287c1c2

  - Installing opauth/opauth (dev-wip/1.0 09547ca)
    Cloning 09547ca9620a2f34ea5472fad4c7787d0538981e

  - Installing opauth/twitter (dev-wip/1.0 a849f80)
    Cloning a849f8048042a2b49ed5b0291363fd60d46b5258

opauth/opauth suggests installing opauth/facebook (Allows Facebook authentication)
opauth/opauth suggests installing opauth/google (Allows Google authentication)
Writing lock file
Generating autoload files
```

The opauth/twitter package had its own dependency on tmhOAuth, which will automatically get installed as well.
Besides installing the libraries to the `app/Vendor/` directory it also has generated `app/Vendor/autoload.php`

In your `app/Config/bootstrap.php` you will import the autoload file so you can instantiate all vendor classes directly
``` phpinline
App::import('Vendor', array('file' => 'autoload'));
```

Opauth would be instantiated in your UsersController.php
``` phpinline
$Opauth = new Opauth\Opauth($config);
$data = $Opauth->run();
```

Install cakephp-plugin with composer
```
$ php composer.phar require ceeram/cakepdf:dev-master
composer.json has been updated
Loading composer repositories with package information
Updating dependencies
  - Installing ceeram/cakepdf (dev-master 1.0)
    Cloning 1.0

Writing lock file
Generating autoload files
```
Now the CakePdf plugin has been installed in app/Plugin/CakePdf

You can also run an interactive composer console
```
$ php composer.phar require
Search for a package []: shama/ftp

Found 1 packages matching shama/ftp

   [0] shama/ftp dev-master

Enter package # to add, or a "[package] [version]" couple if it is not listed []: 0
Search for a package []:
composer.json has been updated
Loading composer repositories with package information
Updating dependencies
  - Installing shama/ftp (dev-master a84483c)
    Cloning a84483ccff167c63c0b69fcf55b365b87972d951

Writing lock file
Generating autoload files
```

Check [Packagist](http://packagist.org) for more composer packages

One day you can't imagine how you ever could live without a decent dependency manager for your PHP projects.
I'm totally hooked on [composer](https://getcomposer.org)