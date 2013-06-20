---
layout: post
title: "Adding CakePHP as composer package"
date: 2013-06-18 12:20
comments: true
categories: [CakePHP, composer]
published: true
---

Here is a quick note on how to add CakePHP 2.x as a dependency for your cake project
using composer.
The root `composer.json` file would look something like this:

```
{
    "name": "vendorname/appname",
    "description": "Addding cake as composer package",
    "repositories": [
        {
            "type": "package",
            "package": {
                "name": "cakephp/cakephp",
                "version": "2.3.6",
                "dist": {
                    "url": "https://github.com/cakephp/cakephp/archive/2.3.6.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "https://github.com/cakephp/cakephp/",
                    "type": "git",
                    "reference": "tree/2.3.6/"
                }
            }
        }
    ],
    "require":{
        "cakephp/cakephp": "~2.3"
    },
    "config":{
        "vendor-dir":"Vendor"
    }
}
```

As you can see, we don't specify autoload or include-dirs, we let CakePHP's own
autoloader still handle that part.

Next you need to update your `app/webroot/index.php` (and `test.php`)

```
define('CAKE_CORE_INCLUDE_PATH', ROOT . DS . APP_DIR . DS . 'Vendor' . DS . 'cakephp' . DS . 'cakephp' . DS . 'lib');
```

And also update your `app/Console/cake.php`

```
$root = dirname(dirname(__FILE__)) . $ds . 'Vendor' . $ds . 'cakephp' . $ds . 'cakephp';
ini_set('include_path', $root . $ds . 'lib' . PATH_SEPARATOR . ini_get('include_path'));
```