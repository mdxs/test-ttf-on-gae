Do It Yourself (DIY)
====================

1. Prepare a Virtual Machine (VM)
---------------------------------

Create a virtual machine (using for example VMware_) and
install Xubuntu_ 13.10 Desktop i386 into it, for example
using a downloaded ``xubuntu-13.10-desktop-i386.iso`` file
as installation media and the following parameters:

- Name: ``test``

- With the following virtual hardware:

  - Memory: 1024 MB
  - Virtual Processors: 1
  - Hard Disk (SCSI 0:0): 10.0 GB
  - CD/DVD (IDE 1:0): Auto-detect
  - Ethernet: NAT
  - USB Controller: Present
  - Audio: Auto detect
  - Printer: Present
  - Display: Auto detect

- And an initial user like:

  - Account: ``test``
  - Password: ``********``

Boot it up and update/upgrade to the latest packages:

.. code-block:: none

  sudo apt-get update
  sudo apt-get upgrade

2. Install some developer tools
-------------------------------

Where ``git`` will be needed to get the project from GitHub,
the ``apache2-utils`` include ``ab`` which we'll use to measure
throughput; and ``python-dev`` helps in our Python development.

.. code-block:: none

  sudo apt-get install git
  sudo apt-get install apache2-utils
  sudo apt-get install python-dev

3. Install virtualenvwrapper_
-----------------------------

This includes ``pip`` and ``virtualenv``, which together will
help you protect your configuration, by providing an isolated
development environment for this test project.

.. code-block:: none

  sudo apt-get install virtualenvwrapper
  # note that use on Xubuntu (Debian) is slightly different
  # as compared to the docs... it is even easier for Xubuntu
  less /usr/share/doc/virtualenvwrapper/README.Debian
  # now upgrade in this order to get latest versions
  sudo pip install virtualenvwrapper --upgrade

4. Get the `Google App Engine SDK`_ for Python
----------------------------------------------

Modify the version number as needed to the latest release.

.. code-block:: none

  cd ~/Downloads
  curl -O https://commondatastorage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.2.zip
  unzip google_appengine_1.9.2.zip
  mv google_appengine ~/

5. Prepare development folders
------------------------------

When you opt for a different structure, modify subsequent
instructions accordingly.

.. code-block:: none

  cd ~
  mkdir dev
  mkdir dev/gh

6. Get the test project
-----------------------

Obtain the code and prepare the development environment.

.. code-block:: none

  cd ~/dev/gh
  # change "mdxs" to your GitHub account if you cloned the project
  git clone git@github.com:mdxs/test-ttf-on-gae.git
  # prepare a virtual environment (with an isolated Python)
  mkvirtualenv test-ttf-on-gae
  cdvirtualenv
  # the following will put the GAE SDK on the path in the virtualenv
  echo "export PATH=\$PATH:~/google_appengine:" >> bin/postactivate
  echo "cd ~/dev/gh/test-ttf-on-gae" >> bin/postactivate

7. Run the test project on localhost
------------------------------------

Use one console window to run your app in the development web server:

.. code-block:: none

  # switch to the virtualenv (and cd into the project)
  workon test-ttf-on-gae
  dev_appserver.py main
  # keep this console window running...

Start another console window, and check local delivery of static files:

.. code-block:: none

  cd ~
  mkdir temp
  cd temp
  wget -S http://localhost:8080/p/FONT_LICENSE
  wget -S http://localhost:8080/p/ubuntu.ttf
  du -b ubuntu.ttf
  # probably returns: "70220   ubuntu.ttf"

Note that the files thus obtained equal the same files found
inside ``main/lib/werkzeug/debug/shared/`` folder of the project.

So far, this was to prepare the test project and to check that it
works locally; using the development application server... Which
will *not* attempt to compress any files.

You can confirm this using ``ab``, which should be provided some
parameters to present itself as a browser/client that will accept
compressed content from the server:

.. code-block:: none

  cd ~/temp
  ab -n 5 \
    -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:26.0) Gecko/20100101 Firefox/26.0" \
    -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
    -H "Accept-Language: en-US,en;q=0.5" \
    -H "Accept-Encoding: gzip, deflate" \
    http://localhost:8080/p/ubuntu.ttf

Notice the ``"Document Length: 70220 bytes"`` in the output, which
equals the ``"du -b"`` output seen above... it is *not* compressed locally.
  
8. Modify application to run on GAE servers
-------------------------------------------

First create your new test application using the form
on https://appengine.google.com/start/createapp

Note in particular the *"Application Identifier"* (further: *App ID*)
which will need to be unique; and you may want to use something with
a *"test"* pre- or postfix to avoid spoiling good identifiers...

**BEWARE:** Once an *App ID* is reserved, regardless of whether the app
is deleted later, it cannot be taken for a new application.

Modify the ``application: test-ttf-on-gae`` line in ``main/app.yaml``
to use the *App ID* just created.

9. Upload the appliction to GAE servers
---------------------------------------

Note that you may need to authenticate and authorize (typically in
a browser instance) when executing the following for the first time.

.. code-block:: none

  workon test-ttf-on-gae
  appcfg.py --oauth2 update main
  # Note that you may need to authenticate and authorize

10. Check compression by GAE servers
------------------------------------

Finally we reach the point in which we can prove that static ``.ttf`` files
can be compressed when hosted by the Google App Engine (GAE) servers.

.. code-block:: none

  cd ~/temp
  ab -n 5 \
    -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:26.0) Gecko/20100101 Firefox/26.0" \
    -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
    -H "Accept-Language: en-US,en;q=0.5" \
    -H "Accept-Encoding: gzip, deflate" \
    http://YOUR-APP-ID.appspot.com/p/ubuntu.ttf

Notice the ``"Document Length: 42567 bytes"`` in the output, which is
**almost 40% smaller** (namely 70220 - 42567 = 27653 bytes smaller) than
the actual file; obviously due to compression by the GAE servers.

Also note the ``"Total transferred:"`` bytes for comparison with further
testing, indicating total bytes transferred in the whole process.


.. _google app engine sdk: https://developers.google.com/appengine/downloads
.. _virtualenvwrapper: http://virtualenvwrapper.readthedocs.org/en/latest/
.. _vmware: https://www.vmware.com/products/
.. _xubuntu: http://xubuntu.org/getxubuntu/
