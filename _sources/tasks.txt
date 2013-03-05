Tasks
=====

Registering jobs
----------------

Create a file `gearman_jobs.py` in any of your django apps, and define as many
jobs as functions as you like. The jobs must return the result of the operation, 
if applicable.

Mark each of these functions as gearman jobs by decorating them with
`django_gearman.decorators.gearman_job`.

For an example, look at the `gearman_example` app's `gearman_jobs.py` file.

Job naming
~~~~~~~~~~
The tasks are given a default name of their import path, with the phrase
'gearman_jobs' stripped out of them, for readability reasons. You can override
the task name by specifying `name` parameter of the decorator. Here's how::

    @gearman_job(name='my-task-name')
    def my_task_function(foo):
        pass

Gearman-internal job naming: ``GEARMAN_JOB_NAME``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The setting ``GEARMAN_JOB_NAME`` is a function which takes the original task
name as an argument and returns the gearman-internal version of that task
name. This allows you to map easily usable names in your application to more
complex, unique ones inside gearman.

The default behavior of this method is as follows::

    new_task_name = '%s.%s' % (crc32(getcwd()), task_name)

This way several instances of the same application can be run on the same
server. You may want to change it if you have several, independent instances
of the same application run against a shared gearman server.

If you would like to change this behavior, simply define the
``GEARMAN_JOB_NAME`` function in the ``settings.py``::

    GEARMAN_JOB_NAME = lambda name: name

which would leave the internal task name unchanged.

Task parameters
---------------
The gearman docs specify that the job function can accept only one parameter
(usually refered to as the ``data`` parameter). Additionally, that parameter
may only be a string. Sometimes that may not be enough. What if you would like
to pass an array or a dict? You would need to serialize and deserialize them.
Fortunately, django-gearman can take care of this, so that you can spend
all of your time on coding the actual task. ::

    @gearman_job(name='my-task-name')
    def my_task_function(foo):
        pass

    client.submit_job('my-task-name', {'foo': 'becomes', 'this': 'dict'})
    client.submit_job('my-task-name', Decimal(1.0))

Tasks with more than one parameter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can pass as many arguments as you want, of whatever (serializable) type
you like. Here's an example job definition ::

    @gearman_job(name='my-task-name')
    def my_task_function(one, two, three):
        pass

You can execute this function in two different ways ::

    client.submit_job('my-task-name', one=1, two=2, three=3)
    client.submit_job('my-task-name', args=[1, 2, 3])

Unfortunately, executing it like this ::

    client.submit_job('my-task-name', 1, 2, 3)

would produce the error, because `submit_job` from Gearman's Python bindings
contains **a lot** of arguments and it's much easier to specify them via
keyword names or a special ``args`` keyword than to type something like seven
``None``s instead ::

    client.submit_job('my-task-name', None, None, None, None, None, None, None, 1, 2, 3)

The only limitation that you have are gearman reserved keyword parameters. As of
Gearman 2.0.2 these are:

    * data
    * unique
    * priority
    * background
    * wait_until_complete
    * max_retries
    * poll_timeout

So, if you want your job definition to have, for example, ``unique`` or
``background`` keyword parameters, you need to execute the job in a special,
more verbose way. Here's an example of such a job and its execution. ::

    @gearman_job(name='my-task-name')
    def my_task_function(background, unique):
        pass

    client.submit_job('my-task-name', kwargs={"background": True, "unique": False})
    client.submit_job('my-task-name', args=[True, False])

Finally ::

    client.submit_job('my-task-name', background=True, unique=True, kwargs={"background": False, "unique": False})

Don't panic, your task is safe! That's because you're using ``kwargs``
directly. Therefore, Gearman's bindings would receive ``True`` for
``submit_job`` function, while your task would receive ``False``.

Always remember to double-check your parameter names with the reserved words
list.

Starting worker(s)
------------------
To start a worker, run `python manage.py gearman_worker`. It will start
serving all registered jobs.

To spawn more than one worker (if, e.g., most of your jobs are I/O bound),
use the `-w` option ::

    python manage.py gearman_worker -w 5

will start five workers.

Since the process will keep running while waiting for and executing jobs,
you probably want to run this in a `screen` session or similar.