Do It Yourself (DIY)
====================

- Prepare a Virtual Machine (VM):

  Create a virtual machine (using for example VMware) and
  install Xubuntu 13.10 Desktop i386 into it, for example
  using a downloaded `xubuntu-13.10-desktop-i386.iso` file
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

- Boot it up and update / upgrade to latest packages

  .. code-block:: none

    sudo apt-get update
    sudo apt-get upgrade

- Install some developer tools

  Where ``git`` will be needed to get the project from GitHub,
  the ``apache2-utils`` include `ab` which we'll use to measure;
  and the ``python-dev`` tools help in our Python development.

  .. code-block:: none

    sudo apt-get install git
    sudo apt-get install apache2-utils
    sudo apt-get install python-dev

- Install virtualenvwrapper (including pip and virtualenv)

  .. code-block:: none

    sudo apt-get install virtualenvwrapper
    # note that use on Xubuntu (Debian) is slightly different
    # as compared to the docs... it is even easier for Xubuntu
    less /usr/share/doc/virtualenvwrapper/README.Debian
    # now upgrade in this order to get latest versions
    sudo pip install virtualenvwrapper --upgrade

- Get a local copy of the Google App Engine SDK v1.9.2 for Python

  .. code-block:: none

    cd ~/Downloads
    curl -O https://commondatastorage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.2.zip
    unzip google_appengine_1.9.2.zip
    mv google_appengine ~/

- Prepare development folders

  .. code-block:: none

    cd ~
    mkdir dev
    mkdir dev/gh

- Get the code and prepare the development environment

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

- Run the test project on localhost

  .. code-block:: none

    # switch to the virtualenv (and cd into the project)
    workon test-ttf-on-gae
    dev_appserver.py main
    # keep this console window running...

- Start another console window, and check local delivery of static files

  Note that the files thus obtained equal the same files found
  inside ``main/lib/werkzeug/debug/shared/`` folder of the project.

  .. code-block:: none

    cd ~
    mkdir temp
    cd temp
    wget -S http://localhost:8080/p/FONT_LICENSE
    wget -S http://localhost:8080/p/ubuntu.ttf





Create your test application using the form on https://appengine.google.com/start/createapp

Note in particular the "Application Identifier" (further: App ID) which will need to be unique;
and you may want to use something with a "test" pre- or postfix to avoid spoiling
good identifiers... BEWARE: Once an App ID is reserved, regardless of whether the app is deleted,
it cannot be taken for a new application.

Modify the "application: test-ttf-on-gae" in "main/app.yaml" to use the App ID just created.


appcfg.py --oauth2 update main
# You may need to authenticate and authorize

cd ~
mkdir tst
cd ~/tst
wget -S http://YOUR-APP-ID.appspot.com/p/FONT_LICENSE
wget -S http://YOUR-APP-ID.appspot.com/p/ubuntu.ttf


ab -n 5 \
  -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:26.0) Gecko/20100101 Firefox/26.0" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
  -H "Accept-Language: en-US,en;q=0.5" \
  -H "Accept-Encoding: gzip, deflate" \
  http://YOUR-APP-ID.appspot.com/p/ubuntu.ttf

ab -n 5 \
  -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:26.0) Gecko/20100101 Firefox/26.0" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
  -H "Accept-Language: en-US,en;q=0.5" \
  -H "Accept-Encoding: gzip, deflate" \
  http://test-ttf-on-gae.appspot.com/p/ubuntu.ttf



