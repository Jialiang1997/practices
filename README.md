This software shows how to use [0MQ](http://zeromq.org/) with broker
processes fetching tasks to do rather than being sent tasks. The
main difference is that tasks are only sent to available workers,
rather than queued in a round-robin fashion amongst all the workers
independently of their status (busy or free).

Also, this broker will give the task to another worker if it has
not been achieved in a given time. The task will be retried at
most a certain number of times, so that if it makes all worker
crash it will not try to run forever.

## Motivations

I wrote this to have some material when answering some
[interesting](http://stackoverflow.com/questions/3692854/how-should-a-zeromq-worker-safely-hang-up/)
[questions](http://stackoverflow.com/questions/4328792/zeromq-xrep-endpoint-disappearing/) asked on 
[StackOverflow](http://stackoverflow.com) about [0MQ](http://zeromq.org).

## How to run the examples

In a terminal, launch the broker with a maximum of 3 attempts to complete each request and a maximum time of 2 seconds by attempt:

    ./broker.py -t 3 -T 2 "tcp://*:4161" "tcp://*:4162" "tcp://*:4163"

In two other terminals, launch two workers by issuing twice:

    ./testworker.py tcp://localhost:4162 tcp://localhost:4163 0.1

Each worker will drop 10% (0.1) of the tasks it receives (and never answer).

In another terminal, launch a client:

    ./testclient tcp://localhost:4161

You should see the tasks arrive on the workers. When a worker decide to drop a task (as would be the case if the task
had caused an exception to be raised), you will see that the task is retried as needed. This is transparent for the
client unless the maximum number of attempts has been reached, in which case it will receive an empty answer indicating
a failure.


