Production Usage
================

Here's a short tutorial on how to use django-gearman on production environment.

Although technically you could start gearman workers inside screen process, we recommend using a specialised tool for monitoring the background processes.

Install supervisor
------------------

[Supervisor]_ Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.

Let's assume your production environment is hosted using virtualenv, under the location of /my/envs.::

  source /my/envs/someenv/bin/activate
  pip install supervisor

This should install supervisor inside your virtualenv. Now, let's start it so that we make sure it works fine.::

  echo_supervisord_conf > /my/envs/supervisord.conf
  supervisord -c /my/envs/supervisord.conf

The above command produces rather large file, yet we're only interested in the very end of it. Look for the 'include' ini section.
It's default value should be simmilar to this:::

  ;[include]
  ;files = relative/directory/*.ini

remove the semicolon so that the section looks similar to this:::

  [include]
  files = /my/project/workers/*.ini /my/other/project/workers/*.ini

The above example assumes, that you will have to define the workers in `*.ini` files inside of your project's 'workers' dir.

Creating worker definitions
---------------------------

Below you can find example worker for the 'default' queue.::

  [program:myproject-default-worker]
  command=/location/to/your/projects/virtualenv/bin/python /location/to/your/project/manage.py gearman_worker -q default

If you don't care about queues (i.e. you want to have just one worker), just ommit the 'queue' parameter, like so:::

  [program:myproject-default-worker]
  command=/location/to/your/projects/virtualenv/bin/python /location/to/your/project/manage.py gearman_worker

We advise you to also create a group for all of the workers from the project. While not mandatory, this is a handy shortcut for restarting all of the
daemons from one project.::

  [group:myproject-all]
  programs=myproject-default-worker,myproject-another-worker,...
  priority=999

After that, restarting all of the workers from your project's group is as easy as typing::
  
  supervisorctl restart myproject-all:*

Also, please note that you *must* restart your workers during the release process, as they won't be able to refresh the code automatically.
The preferred workflow is as following:

* Stop the workers
* Do the release (upload the code, etc)
* Start the workers

.. [Supervisor] http://http://supervisord.org