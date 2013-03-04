Task queues
-----------
Queues are a virtual abstraction layer built on top of gearman tasks. An
easy way to describe it is the following example: Imagine you have a task
for fetching e-mails from the server, another task for sending the emails
and one more task for sending SMS via an SMS gateway. A problem you may
encounter is that the email fetching tasks may effectively "block" the worker
(there could be so many of them, it could be so time-consuming, that no other
task would be able to pass through). Of course, one solution would be to add
more workers (via the ``-w`` parameter), but that would only temporarily
solve the problem. This is where queues come in.

The first thing to do is to pass a queue name into the job description, like
this ::

    @gearman_job(queue="my-queue-name")
    def some_job(some_arg):
        pass

You may then proceed to starting the worker that is bound to the specific
queue ::

    python manage.py gearman_worker -w 5 -q my-queue-name

Be aware of the fact that when you don't specify the queue name, the worker
will take care of all tasks.