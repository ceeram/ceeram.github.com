---
layout: post
title: "Opauth 1.0-alpha arriving soon"
date: 2013-06-22 12:59
comments: true
categories: [opauth, oauth]
published: false
---

Lately i have been hacking on [Opauth](http://opauth.org) as i blogged about
earlier already. Most things are now in place for what will be released as
Opauth version 1.0. Currently its still a work in progress, but if you like, you
can already play with it and provide some feedback before we freeze the api.

### Changes ###

Here a quick overview of things that have changed since 0.x versions:

#### Opauth no longer makes an additional request to your application ####

In the past, Opauth used `callback_transport` config and `shipToCallback()`
method to pass the authentication data to your application. This has all been
removed as you will now receive a `Response` object directly from `Opauth->run()`
This also requires less to none framework specific code around Opauth.

#### Http requests are now handled by individual transport classes ####

We still support non-curl users with the included File transport class, which
uses the `file_get_contents()` method. Also there is a built in Curl transport
class which is the default transport. Additionally you can add your own
transport classes to make the requests with tools like Guzzle.

#### No more PHP 5.2 support ####

We have dropped support for this very outdated PHP version. The minimum required
version is now PHP 5.3. The code has been refactored into several classes and
namespaces have been added.

#### Map response data ####

Response mapping was add, so you can add your own mapping in each strategy
config array to map raw data to the response object in your preferred structure

#### What's left? ####

In order to push to alpha state, we need to update all documentation and help
3rd party strategy maintainers to upgrade to 1.0 version.

We have moved the github repo away from uzyn's github account into the [opauth](https://github.com/opauth)
organisation. Feel free to checkout the `wip/1.0` branch of Opauth and the
strategies. Feedback or help out updating the docs for Opauth and all the
strategies will be highly appreciated.


### Example ###

Here is a simple example on how to use Opauth 1.0. Create a directory with the
following composer.json file and then run `composer install`

    {
        "require": {
            "opauth/opauth": "dev-wip/1.0",
            "opauth/twitter": "dev-wip/1.0"
        }
    }

Next create `index.php`

    <?php
    require 'vendor/autoload.php';
    $config = array(
            'Strategy' => array(
                    'Twitter' => array(
                            'key' => 'your_key',
                            'secret' => 'your_secret'
                    ),
            ),
            'path' => '/opauth/'
    );
    $Opauth = new Opauth\Opauth($config);
    $response = $Opauth->run();
    echo "Authed as " . $response->name . " with uid" . $response->uid;

Fill in the key and secret in `index.php` then add this `.htaccess`

    <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*)$ index.php [QSA,L]
    </IfModule>

Set `DocumentRoot` of your web server to this directory, or create a vhost as this
example does not work when opauth is in a subdirectory

Now point the browser to http://localhost/opauth/twitter to see it in action.

### Example in CakePHP application###

Here is an example on using Opauth 1.0 in a CakePHP application. Create a fresh
application (or use an existing application), add the following composer.json
file and then run `composer install`

    {
        "require": {
            "opauth/opauth": "dev-wip/1.0",
            "opauth/twitter": "dev-wip/1.0"
        },
		"config":{
			"vendor-dir":"Vendor"
		}
    }

In your `app/Config/bootstrap.php` add the following lines

 	App::import('Vendor', array('file' => 'autoload'));
    Configure::write('Opauth', array(
		'Strategy' => array(
			'Twitter' => array(
				'key' => 'your_key',
				'secret' => 'your_secret'
				),
			),
		'path' => '/opauth/'
    );

In your `app/Config/routes.php` add the following line

	Router::connect('/opauth/*', array('controller' => 'users', 'action' => 'opauth'));

In your `UsersController.php` add the following action

	public function opauth() {
		$Opauth = new Opauth\Opauth($config);
		$response = $Opauth->run();
		//use the Response object as you like
		//debug($response); to see what it looks like
	}

Now point the browser to http://pathtoyour/application/opauth/twitter to see it in action.

Alternatively you could use an Authenticate adapter

create `app/Controller/Component/Auth/OpauthAuthenticate.php`

	<?php

	class OpauthAuthenticate extends BaseAuthenticate {

		public function authenticate(CakeRequest $request, CakeResponse $response) {
			$config = Configure::read('Opauth');
			try {
				$Opauth = new Opauth\Opauth($config);
				$result = $Opauth->run();
				return $result->info;
			} catch (Exception $e) {}
			return false;
		}

	}


and your UsersController opauth action would then look like:

	public function opauth() {
		$this->Auth->authenticate = 'Opauth';
		if ($this->Auth->login()) {
			return $this->redirect($this->Auth->loginRedirect);
		}
		$this->Session->setFlash('Social login failed');
		$this->redirect('/');
	}
