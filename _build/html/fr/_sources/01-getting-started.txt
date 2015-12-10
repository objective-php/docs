.. The overview file describes the purpose of the specific class
   Added: <date>
   Author: Name <email>

=================
Getting started with Objective PHP
=================

Pre-requisites
""""""""""""""

The most important pre-requisite needed to use Objective PHP is PHP7.

If you don't have it installed yet, please take a lookt at `Official PHP website <http://www.php.net>`_ and read instruction about PHP7 installation on your development machine.


Installation
""""""""""""

The easiest way to start a project with Objective PHP is to use composer's "create-project" feature and get the Objective PHP Starter Kit:

The following command assumes :code:composer is available in your current PATH:

:: 

    composer create-project objective-php/starter-kit op-quick-start

Where *op-quick-start* should be replaced with anything matching your actual project name.


Keep adding methods like this for everything you want to document. It is recommended to only document public methods like this. Private and protected methods should stay isolated and only documented in the source code.


Starting a server
"""""""""""""""""""""""

To quickly run and access the starter application, you can use the internal PHP HTTP server:

::
    
    cd op-quick-start
    php -S localhost:8080 -t public

That's it! If eveerything went fine, you should be able to point a browser to http://localhost:8080 and access the starter kit home page.
