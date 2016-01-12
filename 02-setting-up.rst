.. The overview file describes the purpose of the specific class
   Added: <date>
   Author: Name <email>

=======================================
Setting up the application
=======================================


Autoloading
"""""""""""

First of all, application classes have to be autoloaded. Objective PHP exclusively rely on ``composer`` autoloader. So, to
make your classes autoloadable, just add the appropriate directive in the composer.json file:

.. code-block:: js

    "autoload": {
        "psr-4": {
            "App\\": "app/src"
        }
    }

Where ``App`` should match your application main namespace. Note that the Objective PHP composer.json file already contains
such a directive, using ``Project`` as namespace. You're invited to update this setting to match your own standards.

.. note:: Any change to the ``autoload`` section of ``composer.json`` needs the command :command:`composer dumpautoload` to be
executed to make changes available to the project.

The Application class
"""""""""""""""""""""

The first class you have to create is the Application class. It should extend ``ObjectivePHP\Application\ApplicationInterface``,
and be placed in your main namespace.

Application instance will the most important object in your application, since it will allow to link all components by
being passed as main argument to all middlewares and more generally all invokables (keep on reading to gert more information
about those concepts).

Barely all logic in the Application class lays in the ``Application::init()`` method, which is automatically triggered when
``Application::run()`` is called.

Defining a workflow
"""""""""""""""""""

It is very important to understand that Objective PHP, as a framework, doesn't provide the application with an hardocded
workflow, but with every components and mechanisms that are needed to create a workflow.

.. note:: While Objective PHP, as a framework, doesn't provide a workflow, the Objective PHP Starter Kit does!

Declaring Steps
===============

The workflow in Objective PHP is defined by first declaring ``Steps``, then plugging ``Middlewares`` to each steps. Since
several middlewares can be plugged into each step, middlewares are stacked, and will be run one after the other, in the
same order they were plugged.

Declaring steps is quite trivial (remember this should take place in ``Application::init()``):

.. code-block:: php

    $this->addSteps('first-step', 'second-step' /* , 'step-n' */ );

Yes, a step is nothing more, from the developer point view, than a label. When running the application, steps also are
run in the same order they were declared using ``addSteps()``.

Plugging Middlewares
====================

Middlewares are basically small chunks of code designed to handle very specific parts of the workflow. Objective PHP
extends this definition to Packages (Objective PHP extensions), that are considered as parts of the application.

.. note:: Most common Middelwares (for handling routing, action excution, rendering, etc.) are provided with the ``objective-php/application`` package and are plugged by default in the starter kit ``Project\Application::init()`` method.


Basic usage
^^^^^^^^^^^

Middlewares are not plugged directly into the application, but into Steps (see previous section). This allow a simple way
to sequence middlewares execution order.

.. code-block:: php

    $this->getStep('bootstrap')->plug(
        function(ApplicationInterface $app) {
            /* anonymous function is an invokable,
               and as such can be used as a Middleware */
        }
    );


Aliasing
^^^^^^^^

When a middleware is plugged into a Step, it is possible to alias it using the ``Step::as()`` method. Aliasing a middleware
is useful for handling substitution: when plugging a middleware with a given alias, if another one was previously plugged with
the same alias, the former will take the latter place in the stack.

.. code-block:: php

    // this plugs the invokable class AnyMiddleware as 'initializer'
    $this->getStep('bootstrap')->plug(AnyMiddleware::class)->as('intializer');

    // this plugs the another invokable class OtherMiddleware also as 'initializer'
    $this->getStep('bootstrap')->plug(OtherMiddleware::class)->as('intializer');

    // at execution time, only OtherMiddleware will actually be run

Objective PHP also offers to use aliasing to plug default middlewares only. By aliasing a middleware using ``Step::asDefault``,
this middleware will be actually registered only if no other was already plugged using the same alias.

This is used for instance in starter kit to plug default operations middlewares, as ``router``: if a package plugs a ``router``
middleware, the default one will simply be ignored:

.. code-block:: php

    // register custom router
    if($whatever = true)
    {
        $this->getStep('route')->plug(CustomerRouter::class)->as('router');
    }


    // this one will be ignored because CustomerRouter was aliased as router prior to SimpleRouter
    $this->getStep('route')->plug(SimpleRouter::class)->asDefault('router');

.. note:: Later on, aliases will also permit to fetch middleware returned values.



Execution filters
^^^^^^^^^^^^^^^^^

Objective PHP allow the developer to filter middlewares actual execution by providing the ``Step::plug()`` method with
extra invokables, expected to return a boolean. In this case, the middleware will be run only if all filter invokables
return ``true``.

.. code-block:: php

    $this->getStep('first-step')->plug(
            Middleware::class,
            function(ApplicationInterface $app) {
                return $app->getEnv() == 'development');
            }
    );

This very simple mechanism allow the developer to setup a very flexible and dynamic workflow, with little efforts. For instance,
it is possible to activate or not a middleware based on current date, user profile, environment variable, and so on. Since the
filters are exepected to return a boolean, they can implement a decision mechanism based on virtually anything.

Objective PHP provides by default several filters, like UrlFilter or ContentTypeFilter, located in ``Application\Workflow\Filter``.
Those default filter ease most typical filtering needs:

.. code-block:: php

    // AnyMiddleware will be run only if URL matches the "/admin/*" pattern
    $this->getStep('action')->plug(AnyMiddleware::class, new UrlFilter('/admin/*'));
