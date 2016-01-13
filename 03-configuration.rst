.. The overview file describes the purpose of the specific class
   Added: <date>
   Author: Name <email>

=============
Configuration
=============


Concept
"""""""

In many frameworks, developer has to deal with usually huge structured data to represent the application and application
components configuration. This approach has several major drawbacks: it leads to very hard to read and maintain files, and
mostly, it needs the developer to know by heart all configuration directives and options.

For Objective PHP, we tried hard to find another way to handle configuration directives, a way that would address those problems.
After several experiments, we came up with an objective approach: all configuration directives are exposed as objects, and
configuration files only contain one-level arrays filled with instances of ``ObjectivePHP\Config\DirectiveInterface``.

Objective PHP exposes this interface, but also several abstract classes to ease Directives conception. There a three kind of
directives at this time: ``SingleValueDirective``, ``StackedValueDirective`` and ``SingleValueDirectiveGroup``. Their behavior is
described later in this chapter.

All directives of a project configuration are imported in an ``ObjectivePHP\Confi\Config`` instance, available by default through
``Application::getConfig()``, making it available almost everywhere in the application.

Config object
"""""""""""""

The Config object extends ``ObjectivePHP\Primitives\Collection\Collection``, and as such offers lots of high-level manipulation
methods on the configuration data. This object can import ``DirectiveInterface`` instances through either ``Config::import()`` (one by one)
or ``Config::fromArray()`` (an array of directives at once).

Once imported, the DirectiveInterface object is not kept as is. It is used by the Config object to create or merge entries in the configuration data.
The configuration directive value can then be fetched using the ``Config::get($key, $default = null)`` method, as one would do on
any ``Collection``.

.. note:: Directive key (or prefix when working with group) are stored as constant in the Directive classes, so that it is not needed to remember all keys, and prevent from making typos when referring a given directive in the Config object.


Directive types
""""""""""""""""

Single Value
^^^^^^^^^^^^

A SingleValueDirective can contain only one value, which can be scalar or structured (object, array). In case the same Directive is
imported twice or more, the latest imported overwrites the previous one by default:

.. code-block:: php

    class Single extends ObjectivePHP\Config\SingleValueDirective
    {
        const DIRECTIVE = 'stack';
    }

    $config->import(new Single('x'))
           ->import(new Single('y'));
    $config->get(Single::DIRECTIVE) == "y";

This behaviour can be altered by changing the DirectiveMerge policy:

.. code-block:: php

    $config->import(new Single('x'))
           ->setMergePolicy(MergePolicy::COMBINE)
           ->import(new Single('y'));
    $config->get(Single::DIRECTIVE) == ["x", "y"];



Stacked Value
^^^^^^^^^^^^^

A StackedValueDirective is pretty close to the SingleValue used with the COMBINE merge policy, but with a major difference:
it's value **always** is an array, even if only one directive of that kind is imported into the Config object:

.. code-block:: php

    class Stacked extends ObjectivePHP\Config\StackedValueDirective
    {
        const DIRECTIVE = 'stack';
    }

    $config->import(new Stacked('x'));
    $config->get(Stacked::DIRECTIVE) == ["x"];

    $config->import(new Stacked('y'));
    $config->get(Stacked::DIRECTIVE) == ["x", "y"];


Single Value Group
^^^^^^^^^^^^^^^^^^

With this directive type, values will be handled just like with Single Value, especially the merge policy, but with a main
difference: each single entry in a Single Value Group has an individual identifier, that will be concatenated to the group
prefix:

.. code-block:: php

    class Grouped extends ObjectivePHP\Config\SingleValueDirectiveGroup
    {
        const PREFIX = 'group';
    }

    $config->import(new Grouped('first', 'first value');
    $config->import(new Grouped('second', 'second value');

    $config->get(Grouped::PREFIX . '.first') == 'first value';

    // all grouped directives can be fetched as new Config object using subset()
    $config->subset(Grouped::PREFIX)->toArray() == ['first' => 'first value', 'second' => 'second value'];

.. note:: While fetching syntax might not be as intuitive as one could expect, remember that the idea behind all this is that application developers should only deal with directives instantiation, since configuration directives are exepexted to be used by the framework itself and components. All other, arbitrary, application (especially business) parameters should be handled using Application::setParam() and Application::getParam(), not Config.

Default directives
""""""""""""""""""

Objective PHP and its packages comes with a few directives:

ObjectivePHP\\Application\\Config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======================== =============== =============================================================================
Class                    Type            Description
======================== =============== =============================================================================
**ActionNamespace**      Stack           Namespace prefixes where to search for action classes
**ApplicationName**      Single          Application name
**LayoutsLocations**     Stack           Paths where to search for layout scripts
**Route**                Group           Simple Router route definitions
**ViewsLocations**       Stack           Paths where to search for view scripts (optional)
======================== =============== =============================================================================

ObjectivePHP\\ServicesFactory\\Config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======================== =============== =============================================================================
Class                    Type            Description
======================== =============== =============================================================================
**Service**              Group           Service specification
======================== =============== =============================================================================

ObjectivePHP\\EloquentPackage\\Config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======================== =============== =============================================================================
Class                    Type            Description
======================== =============== =============================================================================
**EloquentCapsule**      Group           Eloquent ORM Capsule DB connection configuration
======================== =============== =============================================================================

ObjectivePHP\\DoctrinePackage\\Config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======================== =============== =============================================================================
Class                    Type            Description
======================== =============== =============================================================================
**EntityManager**        Group           Doctrine Entity manager and DB connection
======================== =============== =============================================================================

