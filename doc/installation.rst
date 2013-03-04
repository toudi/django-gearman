Installation
------------
It's the same for both the client and worker instances of your django project:

    pip install -e git://github.com/fwenzel/django-gearman.git#egg=django-gearman

Add ``django_gearman`` to the `INSTALLED_APPS` section of `settings.py`.

Specify the following setting in your local settings.py file:::

    # One or more gearman servers
    GEARMAN_SERVERS = ['127.0.0.1']