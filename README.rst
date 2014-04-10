cd ~
mkdir dev
mkdir dev/gh
cd ~/dev/gh
git clone git@github.com:mdxs/test-ttf-on-gae.git

mkvirtualenv test-ttf-on-gae
cdvirtualenv
echo "export PATH=\$PATH:~/google_appengine:" >> bin/postactivate
echo "cd ~/dev/gh/test-ttf-on-gae" >> bin/postactivate

workon test-ttf-on-gae
dev_appserver.py app.yaml

cd ~
mkdir tst
cd ~/tst
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
