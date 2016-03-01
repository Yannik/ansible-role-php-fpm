Description
=========

[![Build Status](https://travis-ci.org/Yannik/ansible-role-php-fpm.svg?branch=master)](https://travis-ci.org/Yannik/ansible-role-php-fpm)

This role allows you to to configure php-fpm pools for both PHP 5.6 and 7.0 on the same system.

Requirements
------------

This role has been tested with PHP installed either from dotdeb in Debian or Ondřej Surý's PPA on Ubuntu. Other setups may work aswell.

Role Variables
--------------


`php_fpm_pools` is a list of dicts whose keys/values will be set in a pool config file. Use `version`, to specify the php version.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - role: php-fpm
           php_fpm_pools:
             - name: website
               user: website
               group: website
               listen: 127.0.0.1:9105
             - name: website2
               user: website2
               group: website2
               version: 7.0

License
-------

GPLv2

Author Information
------------------

Yannik Sembritzki
