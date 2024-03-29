Testing Networking-odl + neutron
================================

Overview
--------

The unit tests (networking_odl/tests/unit/) are meant to cover as much code as
possible and should be executed without the service running. They are
designed to test the various pieces of the neutron tree to make sure
any new changes don't break existing functionality.

# TODO (Manjeet): Update functional testing doc.

Development process
-------------------

It is expected that any new changes that are proposed for merge
come with tests for that feature or code area. Ideally any bugs
fixes that are submitted also have tests to prove that they stay
fixed!  In addition, before proposing for merge, all of the
current tests should be passing.

Virtual environments
~~~~~~~~~~~~~~~~~~~~

Testing OpenStack projects, including Neutron, is made easier with `DevStack <https://opendev.org/openstack/devstack>`_.

Create a machine (such as a VM or Vagrant box) running a distribution supported
by DevStack and install DevStack there. For example, there is a Vagrant script
for DevStack at https://github.com/bcwaldon/vagrant_devstack.

.. note::

   If you prefer not to use DevStack, you can still check out source code on your local
   machine and develop from there.


Running unit tests
------------------

There are two mechanisms for running tests: tox, and nose. Before submitting
a patch for review you should always ensure all test pass; a tox run is
triggered by the jenkins gate executed on gerrit for each patch pushed for
review.

With these mechanisms you can either run the tests in the standard
environment or create a virtual environment to run them in.

By default after running all of the tests, any pep8 errors
found in the tree will be reported.


With `nose`
~~~~~~~~~~~

You can use `nose`_ to run individual tests, as well as use for debugging
portions of your code::

    . .venv/bin/activate
    pip install nose
    nosetests

There are disadvantages to running Nose - the tests are run sequentially, so
race condition bugs will not be triggered, and the full test suite will
take significantly longer than tox & testr. The upside is that testr has
some rough edges when it comes to diagnosing errors and failures, and there is
no easy way to set a breakpoint in the Neutron code, and enter an
interactive debugging session while using testr.

.. _nose: https://nose.readthedocs.org/en/latest/index.html

With `tox`
~~~~~~~~~~

Networking-odl, like other OpenStack projects, uses `tox`_ for managing the virtual
environments for running test cases. It uses `Testr`_ for managing the running
of the test cases.

Tox handles the creation of a series of `virtualenvs`_ that target specific
versions of Python (2.6, 2.7, 3.3, etc).

Testr handles the parallel execution of series of test cases as well as
the tracking of long-running tests and other things.

Running unit tests is as easy as executing this in the root directory of the
Neutron source code::

    tox

Running tests for syntax and style check for written code::

    tox -e pep8

For more information on the standard Tox-based test infrastructure used by
OpenStack and how to do some common test/debugging procedures with Testr,
see this wiki page:
https://wiki.openstack.org/wiki/Testr

.. _Testr: https://wiki.openstack.org/wiki/Testr
.. _tox: http://tox.readthedocs.org/en/latest/
.. _virtualenvs: https://pypi.org/project/virtualenv/

Tests written can also be debugged by adding pdb break points. Normally if you add
a break point and just run the tests with normal flags they will end up in failing.
There is debug flag you can use to run after adding pdb break points in the tests.

Set break points in your test code and run::

    tox -e debug networking_odl.tests.unit.db.test_db.DbTestCase.test_validate_updates_same_object_uuid

The package oslotest was used to enable debugging in the tests. For more
information see the link:
https://docs.openstack.org/oslotest/latest/user/features.html


Running individual tests
~~~~~~~~~~~~~~~~~~~~~~~~

For running individual test modules or cases, you just need to pass
the dot-separated path to the module you want as an argument to it.

For executing a specific test case, specify the name of the test case
class separating it from the module path with a colon.

For example, the following would run only the TestUtils tests from
networking_odl/tests/unit/common/test_utils.py ::

    $ tox -e py27 networking_odl.tests.unit.common.test_utils.TestUtils

Adding more tests
~~~~~~~~~~~~~~~~~

There might not be full coverage yet. New patches for adding tests
which are not there are always welcome.

To get a grasp of the areas where tests are needed, you can check
current coverage by running::

    $ tox -e cover

Debugging
---------

It's possible to debug tests in a tox environment::

    $ tox -e venv -- python -m testtools.run [test module path]

Tox-created virtual environments (venv's) can also be activated
after a tox run and reused for debugging::

    $ tox -e venv
    $ . .tox/venv/bin/activate
    $ python -m testtools.run [test module path]

Tox packages and installs the neutron source tree in a given venv
on every invocation, but if modifications need to be made between
invocation (e.g. adding more pdb statements), it is recommended
that the source tree be installed in the venv in editable mode::

    # run this only after activating the venv
    $ pip install --editable .

Editable mode ensures that changes made to the source tree are
automatically reflected in the venv, and that such changes are not
overwritten during the next tox run.

Running functional tests
------------------------
Neutron defines different classes of test cases. One of them is functional
test. It requires pre-configured environment. But it's lighter than
running devstack or openstack deployment.
For definitions of functional tests, please refer to:
https://docs.openstack.org/neutron/latest/contributor/index.html

The script is provided to setup the environment.
At first make sure the latest version of pip command::

    # ensure you have the latest version of pip command
    # for example on ubuntu
    $ sudo apt-get install python-pip
    $ sudo pip --upgrade pip

And then run functional test as follows::

    # assuming devstack is setup with networking-odl
    $ cd networking-odl
    $ ./tools/configure_for_func_testing.sh /path/to/devstack
    $ tox -e dsvm-functional


For setting up devstack, please refer to neutron documentation:

* https://wiki.openstack.org/wiki/NeutronDevstack
* https://docs.openstack.org/neutron/latest/contributor/index.html
* https://docs.openstack.org/neutron/latest/contributor/testing/testing.html
