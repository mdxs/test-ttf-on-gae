tl;dr
=====

Use ``mime_type: font/ttf`` for static `TrueType Font (.ttf)`_ files in the
``app.yaml`` to benefit from lower bandwidth usage and faster load times.
This project provides an example_ to use and a detailed procedure_ to test it.

.. _example: https://github.com/mdxs/test-ttf-on-gae/blob/master/main/app.yaml


Description
===========

Project to test the current `Google App Engine (GAE)`_ server implementation
when handling static `TrueType Font (.ttf)`_ files. The default approach of
the SDK, to declare such files as ``application/octet-stream`` on upload,
may not provide the best results for app owners and users.

It appears that an explicit declaration of the `mime type`_ for .ttf files
as ``font/ttf`` can reduce bandwidth usage (for the app owner) and decrease
load times (for users). With this mime type, and only this mime type it
seems, the GAE *will* use gzip compression on .ttf files.

Doing so also addresses *"Could not guess mimetype for .ttf files"* warnings
while uploading your application using the Python SDK based ``appcfg.py``
tool (at least on some platforms, as reported in `Issue 6183`_).

While other mime types for .ttf are suggested on the internet, testing
shows that **only** ``mime_type: font/ttf`` in the ``app.yaml`` will
produce the gains in bandwidth usage and load times.

Interestingly enough, mime types starting with ``font/`` are not even
technically valid, while ``application/x-*`` is the valid format for
non-standard mime types (as fonts have no standard). [1]_

.. [1] `Comment #3 to Issue 6183`_ by ``jchu...@gmail.com``


Disclaimer
==========

**BEWARE**: These findings are subject to change; as this appears to be
an implementation detail of the GAE servers. Your milage may vary, which
is why this project provides a detailed procedure; so that you can check
this yourself.


Credits
=======

See `comment #3 to Issue 6183`_ for the very clear hint that helped develop
this test project.

So, thank you ``jchu...@gmail.com`` !


.. _procedure:

Procedure
=========

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
  ab -n 1 \
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
  ab -n 1 \
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


Experiment
==========

Change the ``main/app.yaml`` file and repeat steps 9 and 10 above to see
the effect. The following changes are provided as examples:

- Comment out the special case handling for ``.ttf`` files:

  .. code-block:: none
  
    ...
    handlers:
    ## Special case for .ttf files needing specific mime_type
    ## to enjoy gzip encoding/compression from GAE hosting.
    ## Order is important: this must precede "/p/" static_dir
    # - url: /p/(.*\.ttf)
    #   static_files: static/\1
    #   upload: static/(.*\.ttf)
    #   mime_type: font/ttf
    #   expiration: 1000d

    - url: /p/
      static_dir: static/
      expiration: 1000d

    - url: /.*
      script: main.app
    ...

  You probably notice some *"Could not guess mimetype warnings for .ttf files"*
  warnings/notifications while uploading. Though perhaps some Operating Systems
  detect and provide a mime type to the ``appcfg.py`` process; as some Mac OS X
  users reported they didn't see these messages.

  I have seen for example the following:
  
  .. code-block:: none
  
    ...
    04:27 PM Scanning files on local disk.
    Could not guess mimetype for static/FONT_LICENSE.  Using application/octet-stream.
    Could not guess mimetype for static/ubuntu.ttf.  Using application/octet-stream.
    Could not guess mimetype for static/FONT_LICENSE.  Using application/octet-stream.
    Could not guess mimetype for static/ubuntu.ttf.  Using application/octet-stream.
    04:27 PM Cloning 2 static files.
    ...

  Which doesn't seem to hinder the actual deployment.
  
  It does affect the result of step 10 above though, dropping any compression by
  the GAE servers: with ``ab`` showing ``"Document Length: 70220 bytes"`` and a
  much higher ``"Total transferred:"`` bytes count for the ``ubuntu.ttf`` file.

- Use another mime type for ``.ttf`` files:

  .. code-block:: none
  
    ...
    handlers:
    # Special case for .ttf files needing specific mime_type
    # to enjoy gzip encoding/compression from GAE hosting.
    # Order is important: this must precede "/p/" static_dir
    - url: /p/(.*\.ttf)
      static_files: static/\1
      upload: static/(.*\.ttf)
      mime_type: font/x-font-ttf
      expiration: 1000d

    - url: /p/
      static_dir: static/
      expiration: 1000d

    - url: /.*
      script: main.app
    ...

  Which will use ``font/x-font-ttf`` for the  ``ubuntu.ttf`` file, suppressing
  the related warnings in the upload. But also (silently) dropping the compression
  by GAE servers (as you can see in the ``ab`` output when repeating step 10).

  .. code-block:: none
  
    wget -S http://YOUR-APP-ID.appspot.com/p/ubuntu.ttf
    
  Will show you that it is using ``Content-Type: font/x-font-ttf`` and that
  there are more differences compared to a ``wget`` when using ``font/ttf``
  is being used (most notably the transfer rate and "Transfer-Encoding").

- In step 10, you can also try modifying the ``ab`` command to ``ab -n 100 ...``
  and ``ab -n 100 -c 10 ...`` (for concurrency) to perform more request; and
  thus get better averages.


.. _comment #3 to issue 6183: https://code.google.com/p/googleappengine/issues/detail?id=6183#c3
.. _google app engine (gae): https://developers.google.com/appengine/
.. _google app engine sdk: https://developers.google.com/appengine/downloads
.. _issue 6183: https://code.google.com/p/googleappengine/issues/detail?id=6183
.. _mime type: http://en.wikipedia.org/wiki/Mime_type
.. _truetype font (.ttf): http://en.wikipedia.org/wiki/TrueType
.. _virtualenvwrapper: http://virtualenvwrapper.readthedocs.org/en/latest/
.. _vmware: https://www.vmware.com/products/
.. _xubuntu: http://xubuntu.org/getxubuntu/
