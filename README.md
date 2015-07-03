# Symfony FsbProxyBundle
Authenticated symfony bundle: provides an authentication layer on top of a PHP proxy.

This bundle provides a User model and a UserProvider, to authenticate users based on a yml file.
It also provides a Symfony command to add users with encrypted passwords.

Once the user is connected, it provides a PHP proxy thanks to [8p/Guzzle-Bundle][1] and the [Guzzle PHP library][2], currently v6.

### Why?
This bundle is usefull if you want to proxy an HTTP application, with an authentication layer stronger than http standards such as HTTP_BASIC.

### Requirements
 - Symfony 2.7 or above
 - 8p/GuzzleBundle (included by composer)

### Installation
To install this bundle, run the command below and you will get the latest version by [Packagist][3].

``` bash
composer require fsb/proxy-bundle
```

### Usage
Load required bundles in AppKernel.php:
``` php
// app/AppKernel.php
public function registerBundles()
{
  $bundles = array(
    // [...]
    new EightPoints\Bundle\GuzzleBundle\GuzzleBundle()
    new Fsb\Bundle\ProxyBundle\FsbProxyBundle()
  );
}
```

Set up configuration

``` yaml
# app/config/config.yml
twig:
  # [...]
  globals:
    proxy_title: Your Proxy HTML title

# FsbProxyBundle uses assetic, and for the login page,
# Both CSS, SASS and JS resources.
# However, you can override the templates and manage it yourself.
assetic:
  # [...]
  bundles: [ FsbProxyBundle ]
  filters:
    # [...]
    scss:
      # Requires ruby-sass
      apply_to: "\.scss$"
    # OPTIONALLY, you can install uglifyjs and uglifycss to minify the assets
    uglifyjs2:
      bin: "path/of/your/node_modules/.bin/uglifyjs"
    uglifycss:
      bin: "path/of/your/node_modules/.bin/uglifycss"

guzzle:
  base_url: http://localhost:8888/
  
fsb_proxy:
  users_provider_file_path: "path/of/users.yml"
```

Set up routing
# app/config/routing.yml or any other routing file
``` yaml
fsb_proxy:
  resource: "@FsbProxyBundle/Resources/config/routing.yml"
  prefix:   /
```

Set up security
``` yaml
# app/config/security.yml
security:
  encoders:
    # The FsbProxyBundle User model class, you can choose your favorite encoder
    Fsb\Bundle\ProxyBundle\Model\User:
      algorithm:            pbkdf2
      hash_algorithm:       sha512
      iterations:           1000000
      encode_as_base64:     true

  providers:
    # [...]
    proxy_users:
      id: fsb_proxy.provider.yaml_user_provider

  firewalls:
    # [...]
    fsb_proxy_app_login:
      pattern:  ^/login$
      security: false
  
    fsb_proxy_app:
      pattern:  ^/
      provider: proxy_users
      form_login:
        check_path: fsb_proxy_login_check_page
        login_path: fsb_proxy_login_page
        always_use_default_target_path: true
        default_target_path: /
      logout:
        path:   fsb_proxy_logout_page
        target: /

  access_control:
    - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
    - { path: ^/, roles: ROLE_USER }
```

### Authentication
Here we are!
Now why not create a new user access ?
Do not worry about the users.yml, as long as the path is correct, if the file does not exists,
The application will {try} to create it for you.

``` bash
php app/console fsb-proxy:users:create [username] [password]
```

Both username and password arguments are optionnal, the command will ask for it if you do not provide it.
You can also manually edit the users.yml file with a user information as an array:

``` yml
# /path/of/your/users.yml
username:
  salt: # the salt used to encrypt the password
  password: # the encrypted password
```

### Extends
The Bundle itself provides the security layer, with login and logout routes ;
And a default login page, before "proxying" routes through the Guzzle client.

As a symfony bundle, you can extend it, to benefits of [Symfony inheritance][4],
Then override resources and / or controllers:


```
Controller/
  RestController.php --> Manage PHP proxy once authenticated
  Security/
    AuthenticationController.php --> Manage authentication
Resources/
  views/
    layout.html.twig --> Base template with HTML doctype
    Security/
      login.html.twig --> Login page template
```

All you need to do is setting up your own bundle as child of FsbProxyBundle:

``` php
// src/You/YourBundleName/YouYourBundleName.php
namespace You\YourBundleName;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class YouYourBundleName extends Bundle
{
  public function getParent()
  {
    return 'FsbProxyBundle';
  }
}
```

### Authors
 - Florent Schildknecht ([Portfolio][5])

### License
This bundle is released under the [MIT license](Resources/LICENSE)

 [1]: https://github.com/8p/GuzzleBundle
 [2]: http://docs.guzzlephp.org/en/v6/
 [3]: https://packagist.org/
 [4]: http://symfony.com/doc/2.7/cookbook/bundles/inheritance.html
 [5]: http://floschild.me
