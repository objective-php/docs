.. This describes how to create CLI commands with ObjectivePHP
   Added: 2017-03-21
   Author: Gauthier <gauthier@objective-php.org>

============
CLI commands
============


Concept
"""""""

For whatever reason, one could need to run CLI scripts to manipulate some date in their application. Either for administration
purpose or to create workers for instance. In any case, it's often very useful to access the business objects of the application
in the command line commands.

ObjectivePHP provides the developer with a very simple but powerful support for such commands. The idea was to reuse 100% of
the original bootstrap file (typically ``public/index.php``, which is by the way the default path where the bootstrap is looked after).



Quick start
"""""""""""

Create a cli action
^^^^^^^^^^^^^^^^^^^

To implement a CLI command using ``objective-php/cli``, you essentially have to extends one abstract action class:
 and implement both ``__construct()`` (for setting up the command) and ``run()`` (to actually run the command) methods on it:

.. code-block:: php

    namespace My\Project\CliActions;

    use ObjectivePHP\Cli\Action\AbstractCliAction;

    class HelloWorld extends AbstractCliAction
    {
        /**
         * HelloWorld constructor.
         */
        public function __construct()
        {
            // this is the route to the command
            $this->setCommand('hello');
            // this is the description - automatically
            $this->setDescription('Sample command that kindly greets the user');
        }

        /**
         * @param ApplicationInterface $app
         */
        public function run(ApplicationInterface $app)
        {
            $c = new CLImate();
            $c->out('Hello world!);
        }
    }

.. note:: CLImate, from `The league of extraordinary packages <http://thephpleague.com>`_, is bundled by default with ``objective-php/cli`` since it is used internally to produce the ``usage`` command output, but is absolutely not mandatory. That said, you should definitely consider it to handle your output and formatting :)


Setup cli commands routing
^^^^^^^^^^^^^^^^^^^^^^^^^^

Once your command is ready to run, you have to setup the application to make it able to route cli requests. This is done by
registering an instance of ``ObjectivePHP\Cli\Router\CliRouter`` in the MetaRouter middleware (assuming you're using it of course).

This has to be done in the ``init()`` method of your ``Application`` class:

.. code-block:: php

        $cliRouter = new CliRouter();
        $router->register($cliRouter);

Then, on the same instance of the router, register your newly created cli command:

.. code-block:: php

    $cliRouter->registerCommand(HelloWorld::class);

That's it! You can now run your command by executing ``vendor/bin/op hello`` from the root of your project.


Listing available commands
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you're in doubt or are just discovering a new project likely to provide you with CLI commands, you may not know what commands
are available. To list those available commands, just run th ``op`` script without any argument:

::

    starter-kit$ vendor/bin/op
    Objective PHP Command Line Interface wrapper
    No command has been specified. List of available commands:

         - usage                 List available commands and parameters
         - hello                 Sample command that kindly greets the user

.. note:: The ``usage`` command will always be listed since it's automatically added to the set of available commands by ``objective-php/cli`` itself. This is actually the command that produces this very output.

Parameters
""""""""""

It's not unusual that a CLI script requires some parameters, switches and/or arguments. Of course, ``objective-php/cli``
natively supports such a mechanism. There are currently three kinds of parameters: Toggles, Params and Arguments.

All of them implement the ``ObjectivePHP\Cli\Action\Parameter\ParameterInterface`` interface class, which states that a
CLI parameter class should expose the following methods:

.. code-block:: php

    public function getDescription() : string;

    public function getShortName() : string;

    public function getLongName() : string;

    public function hydrate(array $argv) : array;

    public function getValue();

    public function getOptions() : int;

This API is mostly self-explaining: a parameter should always provide a description, a short and/or a long name, a value
and options flag value. On top tf that, any parameter should be able to pick its value from the ``argv`` stack.

Independently from the actual type of parameter you defined for your CLI action, they all are accessible through the ``getParam()``
shortcut method. This method expects a ``$param``name as first parameter, then an optional ``$default`` value and finally,
an optional ``$origin``, which can be ``cli`` (default) or ``env``, to access environment variables.

All parameters are set on a command using ``expects(ParameterInterface $parameter, string $description)`` on the ``AbstractCliAction`` class. Usage examples will be provided with details for each kind of parameter.

At the time being, there are three ``ParameterInterface`` implementations provided by ``objective-php/cli``:

- ``Toggle``
- ``Param``
- ``Argument``

Detailed usage of these classes is presented after the common *naming* and *options* paragraphs.

Common features
^^^^^^^^^^^^^^^

Naming
------

The ParameterInterface class states taht a parameter should/could have both a short and a long name. Actually, this will
depend on the kind of parameter you're setting up. At the time of writing, ``Param`` and ``Toggle`` classes both support
defining a short and/or a long name, while ``Argument`` only expects a long name.

For parameters accepting both names, the ``setName($name)`` will behave as follow:

 - if strlen($name) === 1, $name is considered as a **short** name
 - if strlen($name)  >  1, $name is considered as a **long** name
 - if is_array($name), $name is supposed to contains ['shortName' => 'longName']

.. note:: the ``setName()`` method will also be triggered when passing *$name* to the ``__construct()`` method.
.. note:: in case you pass an array, if 'shortName' length is greater than 1, an Exception will be thrown.

Options
-------

All parameters can receive option flags. There are two reserved options defined in ParameterInterface:

 - ``MANDATORY`` if applied, the command will exit after displaying the *usage* when the parameter is not provided on the command line.
 - ``MULTIPLE`` this option's behavior depends on the parameter it's applied to:
    - ``Toggle`` are *multiple* by default, meaning that all occurrences of the parameter on the command line will increment the toggle value by 1.
    - ``Param`` when *multiple* option is set, multiple occurrences of the parameter on the command line are allowed and the value of the parameter always is an array.


Available parameters
^^^^^^^^^^^^^^^^^^^^

Toggle
------

Toggles are kind of switches: they don't expect any value on the command line. Their value will be the equal to the number
of times they are passed on the command line. Short and long name occurrences are aggregated.

.. code-block:: php

    class Command extends AbstractCliAction
    {
        public function __construct() {
            $this->setCommand('trigger');
            $this->expects(new Toggle(['v' => 'verbose'], 'Verbose output'));
        }

        public function run(ApplicationInterface $app)
        {
            $v = $this->getParam('verbose');

            // with 'op trigger --verbose' ........... $v === 1
            // with 'op trigger --verbose --verbose' . $v === 2
            // with 'op trigger -v --verbose' ........ $v === 2
            // with 'op trigger -vv' ................. $v === 2
            // with 'op trigger -vv --verbose' ....... $v === 3

        }
    }

Param
-----

Params expect a value to be associated to the parameter. Value can be separated from the parameter using a ``space`` or
an ``equal`` sign. When both a short and a long name are set, the latter has priority on the former.

.. code-block:: php

    class Command extends AbstractCliAction
    {
        public function __construct() {
            $this->setCommand('trigger');
            $this->expects(new Param(['o' => 'output'], 'Output directory'));
        }

        public function run(ApplicationInterface $app)
        {
            $o = $this->getParam('output');

            // with 'op trigger --output some/dir' .................. $o === 'some/dir'
            // with 'op trigger --output=some/dir' .................. $o === 'some/dir'
            // with 'op trigger -o=some/dir' ........................ $o === 'some/dir'
            // with 'op trigger --output=other/dir -o some/dir ' .... $o === 'other/dir'

        }
    }

This default behavior, considering long name parameters have precedence over short ones, can be altered by applying the
``MULTIPLE`` option to a parameter. In this case, the parameter value will **always** be an array,


.. code-block:: php

    class Command extends AbstractCliAction
    {
        public function __construct() {
            $this->setCommand('trigger');
            $this->expects(new Param(['o' => 'output'], 'Output directory', Param::MULTIPLE));
        }

        public function run(ApplicationInterface $app)
        {
            $o = $this->getParam('output');

            // with 'op trigger --output=some/dir' ............... $o === ['some/dir']
            // with 'op trigger --output=other/dir -o some/dir ' . $o === ['other/dir', 'some/dir']

        }
    }

Argument
--------

Arguments are a bit special compared to the other two: they are considered as *positional* parameters. As such, they will
always be treated after all other type of parameters. Also, the order they are passerd to ``expects()`` matters. The first
``Argument`` parameter to be registered will match the first argument from the CLI that is not a ``Toggle`` or a ``Param``.

.. code-block:: php

            class Command extends AbstractCliAction
            {
                public function __construct() {
                    $this->setCommand('trigger');
                    $this->expects(new Argument(['o' => 'output'], 'Output directory', Param::MULTIPLE));
                }

                public function run(ApplicationInterface $app)
                {
                    $o = $this->getParam('output');

                    // with 'op trigger other/dir some/dir ' ..... $o === ['other/dir', 'some/dir']

                }
            }


When an ``Argument`` is marked as ``MANDATORY``, it becomes forbidden to stack extra optional (i.e. not flagged as **mandatory**) arguments,
and when marked as ``MULTIPLE``, no other argument can be expected, since all positional arguments after it will be aggregated to its value.

.. code-block:: php

            class Command extends AbstractCliAction
            {
                public function __construct() {
                    $this->setCommand('trigger');
                    $this->expects(new Argument(['i' => 'output'], 'Input directory', Param::MANDATORY));
                    $this->expects(new Argument(['o' => 'output'], 'Output directory'));
                }

                public function run(ApplicationInterface $app)
                {
                    $i = $this->getParam('input');
                    $o = $this->getParam('output');

                    // with 'op trigger other/dir some/dir ' ..... $i === 'other/dir' and $o === 'some/dir']

                }
            }
