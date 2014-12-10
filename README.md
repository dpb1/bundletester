BundleTester
============

A juju-deployer based test runner for bundles.

Installation
============

    pip install bundletester \
      --allow-external   lazr.authentication \
      --allow-unverified lazr.authentication

Introduction
============

This is designed around the fail fast principal. Each bundle is composed of
charms, each of those charms should have tests ranging from unit tests to
integration tests. The bundle itself will have integration tests. Those tests
should run from least expensive in terms of time to most failing as soon as
possible. The theory is that if tests fail in the charms the bundle cannot
function. Bundletester will therefore pull all the charms and look for any
tests it can find and run those prior running its own integration tests.

This test runner uses the following pattern. A bundle has a URL. The runner can
be run directly from within a bundle checkout or will pull the branch pointed
at by the bundle url. From there it will attempt to look for a 'tests'
directory off the project root. Using the rules described below it will find
and execute each test within that directory and produce a report.

This also includes a bundlewatcher script that can be used as a Jenkins script
trigger.

Example Usage
=============

From within the top level of a bundle

    bundletester -r json -o result.json

Using a specific directory of tests

    bundletester -t 'pwd`

Using the -t/--testdir directive you can also run the tests from a charm
directory with all the default config.

    bundletester -l DEBUG -o result --testdir charm/tests

Passing -l/--log-level DEBUG will give additional insight into what steps
bundletester is taking.

Test Directory
==============

The driver file, 'tests/tests.yaml' is used (by default) to control the overall
flow of how tests work. All values in this file and indeed the file itself are
optional. When not provided defaults will be used.

tests.yaml
----------

A sample `tests.yaml` file, with a description of each key:

```
# Allow bootstrap of current env, default: true
bootstrap: false

# Use juju-deployer to reset the env between each test, default: true
reset: false

# optional name of script in test dir to run before each test
setup: script

# optional name of script to run after each test
teardown: script

# glob pattern of files in 'tests/' to treat as tests, only executable
# files will be used.
tests: "[0-9]*"

# create and activate a virutalenv for the running of all tests, default: true
virtualenv: true

# list of package sources to add automatically 
sources:
    - ppas, etc

# list of packages to install automatically with apt
packages:
    - amulet
    - python-requests
```









Finding Tests
-------------

When tests.yaml's test pattern is executed it will yield test that should be run. Each
of these test files can optional have a control file of the same name but with a `.yaml`
extension. If present this file has a similar layout to tests.yaml. 

A sample `01-test.yaml` file::

    bundle: bundle.yaml
    setup: script
    teardown: script

Explanation of keys:

bundle: A bundle file in/relative to the tests directory. If bundle isn't
specified <BUNDLE_ROOT>/bundle.yaml will be used by default.

setup: optional script to be run before this test. This is called after the
tests.yaml setup if present.

teardown: optional script to be run after this test. This is called before the
tests.yaml teardown if present.

Setup/Teardown
--------------

If these scripts fail with a non-zero exit code the test will be recorded as a
failure with that exit code.

Test Execution
--------------

Each test should be executable. If `virtualenv` is true each test will be
executed with that environment activated. 

Each test will have its stdout and stderr captured.

Each tests exit code will be captured. A non-zero return indicates failure.

Reporting Results
-----------------

Any test failures will result in a non-zero exit code. 

Using -r json a JSON string will be outputted on stdout. This string will
contain at minimum the following structure (additional keys are possible)

    [{test result}, ...]

each test result is a dict with at least:

    {'test': test file, result: exit_code, 
      'exit': 'script name which returned exit code',
      'returncode': exitcode of process, 
      'duration': timedelta in seconds}

`exit` will not be included if result is sucess (0)


TODO
====
- Finish Tests, started as TDD and then hulk-smashed the end
- Better runtime streaming
