Behave tests for Hyperledger Fabric Feature and System Tests
============================================================

.. image:: http://i.imgur.com/ztYl4lG.jpg

Behave is a tool used for Behavior Driven Development (BDD) testing. It uses tests (feature files) written in a natural language called Gherkin. The tests are executed using python as the supporting code.

BDD is an agile software development technique that encourages collaboration between developers, QA and non-technical or business participants in a software project. Feel free to read more about `BDD`_.

.. _BDD: http://pythonhosted.org/behave/philosophy.html


This implementation of system and feature file testing for Hyperledger Fabric

You can install Behave and additional packages from PyPI with

::

    $ sudo easy_install pip
    $ sudo apt-get install python-setuptools python-dev build-essential
    $ sudo pip install behave
    $ sudo pip install google
    $ sudo pip install protobuf
    $ sudo pip install pyyaml

Full documentation and usage examples for Behave can be found in the `online documentation`_.

.. _online documentation: http://pythonhosted.org/behave/


Pre-requisites
--------------
You must have the following installed:
    * python
    * docker
    * docker-compose

Ensure that you have Docker for `Mac`_ or `Windows`_ 1.12 or higher properly installed on your machine.

.. _Mac: https://docs.docker.com/engine/installation/mac/
.. _Windows: https://docs.docker.com/engine/installation/windows/

You should also clone the following repository
    * `hyperledger-fabric`_
    * `hyperledger-fabric-ca`_

.. _hyperledger-fabric: https://gerrit.hyperledger.org/r/fabric
.. _hyperledger-fabric-ca: https://gerrit.hyperledger.org/r/fabric-ca


Getting Started
---------------
Before executing the behave tests, it is assumed that there are docker images and tools that have already been built in the process. 

================
Areas of Testing
================
BDD tests are testing functionality and feature behavior. With this in mind, the following are areas that we plan to be covered in these BDD tests:
   * Basic setup (Happy Path)
   * Orderer Functionality
      * solo
      * kafka
   * Ledgers
   * Endorser and committer peers
   * Fabric-CA (used for SSL connections)
   * Upgrades and Fallbacks
   * Bootstrapping
      * configtxgen
      * cryptogen
   * Growing and shrinking networks
   * Stopping and Starting components
   * … and more (such as different tooling, messages sizes, special scenarios)

The following are not covered in these BDD tests:
   * scalability
   * performance
   * long running tests
   * stress testing
   * timed tests

======================
Building docker images
======================
When executing tests that are using docker-compose fabric-ca images, be sure to have the fabric-ca docker images built. You must perform a `make docker` in the `/path/to/hyperledger/fabric-ca` directory.

The docker images for `peer`, `orderer`, `kafka`, and `zookeeper` are needed. You must perform a `make docker` in the `/path/to/hyperledger/fabric` directory.

=========================
Building tool executables
=========================
The **configtxgen** and **cryptogen** tools are used when bootstrapping the networks in these tests. As a result, you must perform a `make configtxgen && make cryptogen` in the `/path/to/hyperledger/fabric` directory. 


How to Contribute
--------------------------
There are different ways that you can contribute in this area.
 * Writing feature files
 * Writing python test code to execute the feature files
 * Adding docker-compose files for different network configurations

=====================
Writing Feature Files
=====================
The feature files are written by anyone who understands the requirements. This can be a business analyst, quality analyst, manager, developer, customer. The file describes a feature or part of a feature with representative examples of expected outcomes and behaviors. These files are plain-text and do not require any compilation. Each feature step maps to a python step implementation.

The following is an example of a simple feature file:

.. sourcecode:: gherkin

    Feature: Test to ensure I take the correct accessory
      Scenario: Test what happens on a rainy day
        Given it is a new day
        When the day is rainy
        And the day is cold
        Then we should bring an umbrella
      Scenario Outline: Test what to bring
        Given it is a new day
        When the day is <weather>
        Then we should bring <accessory>
      Examples: Accessories
        | weather | accessory |
        |   hot   | swimsuit  |
        |  cold   |  coat     |
        |  cloudy |  nothing  |


Keywords that are used when writing feature files:
 * **Feature**
    * The introduction of the different feature test scenarios
    * You can have multiple scenarios for a single feature
 * **Scenario/Scenario Outline**
    * The title and description of the test
    * You can run the same test with multiple inputs
 * **Given**
    * Indicates a known state before any interaction with the system.
    * **Avoid talking about user interaction.**
 * **When**
    * Key actions are performed on the system.
    * This is the step which may or may not cause some state to change in your system.
 * **Then**
    * The observed and expected outcomes.
 * **And**
    * Can be used when layering any givens, whens, or thens.

========================
Writing python test code
========================
Feature steps used in the feature file scenarios are implemented in python files stored in the “steps” directory. The behave implementation files are named '*<component>_impl.py*' and the utilities are named '*<action>_util.py*' in the steps directory.

Python implementation steps are identified using decorators which match the keyword from the feature file: 'given', 'when', 'then', and 'and' The decorator accepts a string containing the rest of the phrase used in the scenario step it belongs to.


.. sourcecode:: python

    >>> from behave import *
    >>> @given('it is a new day')
    ... def step_impl(context):
    ...     # Do some work
    ...     pass
    >>> @when('the day is {weather}')
    ... def step_impl(context, weather):
    ...     weatherMap = {'rainy': 'an umbrella',
    ...                   'sunny': 'shades',
    ...                   'cold': 'a coat'}
    ...     context.accessory = weatherMap.get(weather, "nothing")
    >>> @then('we should bring {accessory}')
    ... def step_impl(context, accessory):
    ...     assert context.accessory == accessory, "You're taking the wrong accessory!"


====================
Docker-Compose Files
====================
These docker composition files are used when setting up and tearing down networks of different configurations. Different tests can use different docker compose files depending on the test scenario. We are currently using `version 2 docker compose`_ files. 

.. _version 2 docker compose: https://docs.docker.com/compose/compose-file/compose-file-v2/

How to execute Feature tests
----------------------------
There are multiple ways to execute behave tests.
   * Execute all feature tests in the current directory
   * Execute all tests in a specific feature file
   * Execute all tests with a specified tag
   * Execute a specific test


**Executes all tests in directory**
::

    $ behave

**Executes specific feature file**
::

    $ behave mytestfile.feature

**Executes tests labelled with tag**
::

    $ behave -t mytag

**Executes a specific test**
::

    $ behave -n 'my scenario name'


Helpful Tools
-------------
Behave and the BDD ecosystem have a number of tools and extensions to assist in the development of tests. Feel free to explore them including a few described `here`_.

.. _here: http://behave.readthedocs.io/en/latest/behave_ecosystem.html

Helpful Docker Commands
-----------------------
   * View running containers
      * ``$ docker ps``
   * View all containers (active and non-active)
      * ``$ docker ps -a``
   * Stop all Docker containers
      * ``$ docker stop $(docker ps -a -q)``
   * Remove all containers.  Adding the `-f` will issue a "force" kill
      * ``$ docker rm -f $(docker ps -aq)``
   * Remove all images
      * ``$ docker rmi -f $(docker images -q)``
   * Remove all images except for hyperledger/fabric-baseimage
      * ``$ docker rmi $(docker images | grep -v 'hyperledger/fabric-baseimage:latest' | awk {'print $3'})``
   * Start a container
      * ``$ docker start <containerID>``
   * Stop a containerID
      * ``$ docker stop <containerID>``
   * View network settings for a specific container
      * ``$ docker inspect <containerID>``
   * View logs for a specific containerID
      * ``$ docker logs -f <containerID>``
   * View docker images installed locally
      * ``$ docker images``
   * View networks currently running
      * ``$ docker networks ls``
   * Remove a specific residual network
      * ``$ docker networks rm <network_name>``
