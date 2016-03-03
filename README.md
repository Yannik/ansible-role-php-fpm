Description
=========

[![Build Status](https://travis-ci.org/Yannik/ansible-role-php-fpm.svg?branch=master)](https://travis-ci.org/Yannik/ansible-role-php-fpm)

This role allows you to to configure php-fpm pools for both PHP 5.6 and 7.0 on the same system.

Requirements
------------

This role has been tested with PHP installed either from dotdeb in Debian or Ondřej Surý's PPA on Ubuntu. Other setups may work aswell.

Role Variables
--------------

* `php_fpm_pools`: The list of pools for php-fpm, each pool is a hash with a name entry (used for filename), and an optional version entry. All the other entries in the hash are pool directives (see http://php.net/manual/en/install.fpm.configuration.php).
    * `version`: the php version this pool should use.
        * Default: `php_fpm_default_version`
    * `php_admin_value[opcache.file_cache]`: Set path to the opcache dir for this pool. Settings this enables opcache automatically when using `php_fpm_secure_opcache`.
        * Example: `/var/www/site1/.opcache`. This folder should not be accessible to the public!
* `php_fpm_pool_defaults`: A list of default directives used for all php-fpm pools
* `php_fpm_ini`: Customization for php-fpm's php.ini as a list of options, each option is a hash using the following structure:
    * `option`: The name of the option
    * `value`: The value of the option
    * `section`: INI section name
    * `versions`: Optional list of versions to apply the ini option on. By default, the option is applied to all php versions in `php_fpm_installed_versions`.
        * Example: `['7.0']`
    * `state`: `present` or `absent`
        * Default: `present`
* `php_fpm_default_version`: The default php version for pools
    * Default: `'5.6'`
* `php_fpm_secure_opcache`: whether to use a secured opcache setup (see below).
    * Default: `True`. You should know what you are doing if you disable this!
* `php_fpm_installed_versions`: This is an list of installed php versions which is used to set php.ini values in all installed versions.
    * Default: `['5.6', '7.0']

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - role: Yannik.php-fpm
           php_ini_options:
             - option: "date.timezone"
               section: "PHP"
               value: "Europe/Berlin"
             - option: "opcache.enable"
               section: "PHP"
               value: "1"
               versions: ['7.0']
             - option: "opcache.enable"
               section: "PHP"
               value: "0"
               versions: ['5.6']
           php_fpm_pools:
             - name: website
               user: website
               group: website
               listen: 127.0.0.1:9105
             - name: website2
               user: website2
               group: website2
               version: "7.0"
             - name: website3
               user: website3
               group: website3
               php_admin_value[opcache.file_cache]: /var/www/website3/.opcache
               version: "7.0"

After Installation
-------
Check your running php-fpm pools using `ps -eH x|grep php`.

What about opcache?
-------
With the default Ubuntu/Debian php-fpm packages, there is one php-fpm masterprocess for each php-version.
The opcache and apc are held by the master process. Due to this  all sites for a certain php version share
the same opcache/apc and the opcache has to have a size big enough for all the sites.
This also has major security implications:
  * https://web.archive.org/web/20150905223439/https://ikanobori.jp/php55-opcache-shared-hosting.html
  * https://bugs.php.net/bug.php?id=69090
  * https://bugs.php.net/bug.php?id=67481 (
  * http://massivescale.blogspot.de/2013/06/zend-opcode-cacher-in-php-55-security.html
  * https://weizenspr.eu/2014/php-fpm-chroot-zend-opcache-problem/

I tested this and it is still possible with PHP 7.0! (Tried without chroot but instead just
included /var/www/site1/secure.php from /var/www/site2/ while only site1 had read permissions.
Was able to extract variables from /var/www/site1/secure.php this way.)

  * https://ma.ttias.be/a-better-way-to-run-php-fpm/
  * https://regilero.github.io/drupal/english/2013/05/16/Warning_chrooted_php_fpm_and_apc/

This is quite cumbersome to do though.

What many shared hosting providers do is disable opcache on php 5.6 and only offer it
on php >= 7.0, which has the `opcache.file_cache_only` and use that. This way the opcache
is file-based and created with the pool user as owner.

The opcache must be enabled in the php.ini of the master process, it is not possible to selectively enable it.
It is however possible (and advised) to disable the opcache using `php_admin_value[opcache.enable] = 0` for
all pools which are not specially configured to use the opcache using a with `file_cache_only`.

This role does this by default when `php_fpm_secure_opcache` is true.

License
-------

GPLv2

Author Information
------------------

Yannik Sembritzki
