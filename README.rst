retry2
======

.. image:: https://img.shields.io/pypi/dm/retry.svg?maxAge=2592000
        :target: https://pypi.python.org/pypi/retry2/

.. image:: https://img.shields.io/pypi/v/retry.svg?maxAge=2592000
        :target: https://pypi.python.org/pypi/retry2/

.. image:: https://img.shields.io/pypi/l/retry2.svg?maxAge=2592000
        :target: https://pypi.python.org/pypi/retry2/


Easy to use retry decorator.


[This is a fork of https://github.com/invl/retry which is not maintained anymore]

Features
--------

- No external dependency (stdlib only).
- (Optionally) Preserve function signatures (`pip install decorator`).
- Original traceback, easy to debug.


Installation
------------

.. code-block:: bash

    $ pip install retry2


API
---

retry decorator
^^^^^^^^^^^^^^^

.. code:: python

    def retry(exceptions=Exception, tries=-1, delay=0, max_delay=None, backoff=1, jitter=0, logger=logging_logger,
              on_exception=None):
        """Return a retry decorator.

        :param exceptions: an exception or a tuple of exceptions to catch. default: Exception.
        :param tries: the maximum number of attempts. default: -1 (infinite).
        :param delay: initial delay between attempts. default: 0.
        :param max_delay: the maximum value of delay. default: None (no limit).
        :param backoff: multiplier applied to delay between attempts. default: 1 (no backoff).
        :param jitter: extra seconds added to delay between attempts. default: 0.
                       fixed if a number, random if a range tuple (min, max)
        :param logger: logger.warning(fmt, error, delay) will be called on failed attempts.
                       default: retry.logging_logger. if None, logging is disabled.
        :param on_exception: handler called when exception occurs. will be passed the captured
                             exception as an argument. further retries are stopped when handler
                             returns True. default: None
        """

Various retrying logic can be achieved by combination of arguments.


Examples
""""""""

.. code:: python

    from retry import retry

.. code:: python

    @retry()
    def make_trouble():
        '''Retry until succeed'''

.. code:: python

    @retry(ZeroDivisionError, tries=3, delay=2)
    def make_trouble():
        '''Retry on ZeroDivisionError, raise error after 3 attempts, sleep 2 seconds between attempts.'''

.. code:: python

    @retry((ValueError, TypeError), delay=1, backoff=2)
    def make_trouble():
        '''Retry on ValueError or TypeError, sleep 1, 2, 4, 8, ... seconds between attempts.'''

.. code:: python

    @retry((ValueError, TypeError), delay=1, backoff=2, max_delay=4)
    def make_trouble():
        '''Retry on ValueError or TypeError, sleep 1, 2, 4, 4, ... seconds between attempts.'''

.. code:: python

    @retry(ValueError, delay=1, jitter=1)
    def make_trouble():
        '''Retry on ValueError, sleep 1, 2, 3, 4, ... seconds between attempts.'''

.. code:: python

    # If you enable logging, you can get warnings like 'ValueError, retrying in
    # 1 seconds'
    if __name__ == '__main__':
        import logging
        logging.basicConfig()
        make_trouble()

retry_call
^^^^^^^^^^

.. code:: python

    def retry_call(f, fargs=None, fkwargs=None, exceptions=Exception, tries=-1, delay=0, max_delay=None, backoff=1,
                   jitter=0, logger=logging_logger, on_exception=None):
        """
        Calls a function and re-executes it if it failed.

        :param f: the function to execute.
        :param fargs: the positional arguments of the function to execute.
        :param fkwargs: the named arguments of the function to execute.
        :param exceptions: an exception or a tuple of exceptions to catch. default: Exception.
        :param tries: the maximum number of attempts. default: -1 (infinite).
        :param delay: initial delay between attempts. default: 0.
        :param max_delay: the maximum value of delay. default: None (no limit).
        :param backoff: multiplier applied to delay between attempts. default: 1 (no backoff).
        :param jitter: extra seconds added to delay between attempts. default: 0.
                       fixed if a number, random if a range tuple (min, max)
        :param logger: logger.warning(fmt, error, delay) will be called on failed attempts.
                       default: retry.logging_logger. if None, logging is disabled.
        :param on_exception: handler called when exception occurs. will be passed the captured
                             exception as an argument. further retries are stopped when handler
                             returns True. default: None
        :returns: the result of the f function.
        """

This is very similar to the decorator, except that it takes a function and its arguments as parameters. The use case behind it is to be able to dynamically adjust the retry arguments.

.. code:: python

    import requests

    from retry.api import retry_call


    def make_trouble(service, info=None):
        if not info:
            info = ''
        r = requests.get(service + info)
        return r.text


    def what_is_my_ip(approach=None):
        if approach == "optimistic":
            tries = 1
        elif approach == "conservative":
            tries = 3
        else:
            # skeptical
            tries = -1
        result = retry_call(make_trouble, fargs=["http://ipinfo.io/"], fkwargs={"info": "ip"}, tries=tries)
        print(result)

    what_is_my_ip("conservative")



