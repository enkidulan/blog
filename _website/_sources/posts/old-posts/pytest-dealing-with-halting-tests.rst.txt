.. post:: 21 Apr, 2021
   :tags: python, pytest, testing
   :category: python
   :author: Maksym Shalenyi
   :excerpt: 2
   :image: 1
   :nocomments:

pytest: dealing with halting tests
==================================



Besides using  --durations option, which does not seem to show the interrupted halted tests, one possible way to deal with halting tests is to use pytest-timeout. However, it does not work with pytest-xdist, which a large issue when you are refactoring a large codebase with lots of tests and not ready to wait 5x times longer.

Fortunately, pytest is an excellent rich framework and provides a concept of hooks. And so to deal with this problem it is possible to use pytest_runtest_call hook to write to a file a time when a test started and when it ended. With this, it should be easy to figure out most of the halting tests in one go.

.. code-block:: python

   @pytest.hookimpl(hookwrapper=True, trylast=False)
   def pytest_runtest_call(item):
      with open("run-stats.log", "a") as f:
         f.write(item.nodeid + " started " + datetime.now().isoformat() + "\n")
      yield
      with open("run-stats.log", "a") as f:
         f.write(item.nodeid + " ended " + datetime.now().isoformat() + "\n")


You can find more information on using hooks in official documentation - pytest.hookspec.pytest_runtest_protocol
