Install some dependencies to make sure that you get ruby with SSL support: 

```
$ sudo apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev libssl0.9.8 libssl-dev
```

The install ruby to get `bundle` support (actually not sure if this is required):
```
$ sudo apt-get install ruby
```

Install testing requirements:

```
sudo apt-get install memcached
```

Install the repository:

```
$ cd ~
$ mkdir -p ~/edx_all/
$ cd ~/edx_all/
$ ln -sd ~/edx-platform edx-platform
$ cd ~
$ git clone git@github.com:Edraak/edx-platform.git
$ cd edx-platform
$ git apply
# Paste the patch in it to make it accept Ubuntu 14.04 LTS (trusty).
$ ./scripts/create-dev-env.sh
```

###Trusty Patch
- hide quoted text -
```
diff --git a/scripts/create-dev-env.sh b/scripts/create-dev-env.sh
index 7f7c9f9..7d83e1e 100755
--- a/scripts/create-dev-env.sh
+++ b/scripts/create-dev-env.sh
@@ -252,7 +252,7 @@ case `uname -s` in
 
         distro=`lsb_release -cs`
         case $distro in
-            wheezy|jessie|maya|olivia|nadia|precise|quantal)
+            wheezy|jessie|maya|olivia|nadia|precise|quantal|trusty)
                 if [[ ! $noninteractive ]]; then
                     warning "
                             Debian support is not fully debugged. Assuming you have standard
@@ -442,6 +442,7 @@ if [[ -n $compile ]]; then
 fi
 
 # building correct version of distribute from source
+true || (
 DISTRIBUTE_VER="0.6.28"
 output "Building Distribute"
 SITE_PACKAGES="$WORKON_HOME/edx-platform/lib/python2.7/site-packages"
@@ -461,7 +462,7 @@ else
   error "Distribute failed to build correctly. This script requires a working version of Distribute 0.6.28 in your virtualenv's python installation"
   exit 1
 fi
-
+)
 case `uname -s` in
     Darwin)
         # on mac os x get the latest distribute and pip
diff --git a/scripts/install-system-req.sh b/scripts/install-system-req.sh
index b4e5bb0..ad6b3e8 100755
--- a/scripts/install-system-req.sh
+++ b/scripts/install-system-req.sh
@@ -31,7 +31,7 @@ case `uname -s` in
         distro=`lsb_release -cs`
         case $distro in
             #Tries to install the same
-            squeeze|wheezy|jessie|maya|lisa|olivia|nadia|natty|oneiric|precise|quantal|raring)
+            squeeze|wheezy|jessie|maya|lisa|olivia|nadia|natty|oneiric|precise|quantal|raring|trusty)
                 output "Installing Debian family requirements"
 
                 # add repositories

```

Now your journey begins... 

After many many trial and errors you should have `pip` installed, among so many dependencies. You'll need to do the following hacks:

```
$ sudo apt-get install libmagickwand-dev
$ pip install -r requirements/edx/paver.txt
$ workon edx-platform
$ pip install distribute==0.6.29
```
Manually (as in manually extract the files) install [TinyUrl-0.1.0-py2.5.egg](https://pypi.python.org/packages/2.5/T/TinyUrl/TinyUrl-0.1.0-py2.5.egg#md5=d9f8873ff61f5a428532bbd949431c76) in 

```
$ cd ~/.virtualenvs/edx-platform/lib/python2.7/site-packages/
$ wget https://pypi.python.org/packages/2.5/T/TinyUrl/TinyUrl-0.1.0-py2.5.egg
$ mv TinyUrl-0.1.0-py2.5.egg TinyUrl-0.1.0-py2.5.zip
$ unzip TinyUrl-0.1.0-py2.5.zip
$ mv EGG-INFO TinyUrl-0.1.0-py2.5.egg-info
```

```
$ cd /usr/include
$ sudo ln -s freetype2 freetype
```

Keep running `$ ./scripts/create-dev-env.sh`
Until `$ workon edx-platform && cd ~/edx-platform && paver run_all_servers --setting_lms=cms.dev` work well.

###Ruby SSL Issue
If ruby complains about SSL you should run this command:
```
$ cd ~/edx-paltform
$ rbenv install
```
This should solve it.

#Configuring and Running the Platform
- hide quoted text -
While the [] guide covers almost all you need. It misses few points. You need to configure your local dev. environment:

```
$ gedit lms/envs/private.py
```

And put the following code in it:

```
from os.path import expanduser
from path import path
home = path(expanduser("~"))

CONTENTSTORE = {
    'ENGINE': 'xmodule.contentstore.mongo.MongoContentStore',
    'DOC_STORE_CONFIG': {
        'host': 'localhost',
        'db': 'xcontent',
    },
    # allow for additional options that can be keyed on a name, e.g. 'trashcan'
    'ADDITIONAL_OPTIONS': {
        'trashcan': {
            'bucket': 'trash_fs'
        }
    }
}

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': home / "db" / "edraak-edx.db",
    }
}
```

And for `cms` `$ gedit ./cms/envs/private.py`:

```
from os.path import expanduser
from path import path
home = path(expanduser("~"))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': home / "db" / "edraak-edx.db",
    }
}
```


You'll have two dummy warnings about missing local configuration files which 
can be easily turned off by putting an empty JSON in each:

```
$ echo '{}' | sudo tee ~/env.json /home.json
```

##Run The Servers
```
$ workon edx-platform
$ cd ~/edx-platform
$ paver run_all_servers --settings_lms=cms.dev
```

##Run the Test *Successfully*
```
$ git checkout benp/fix-js-tests # release and master had failing JS tests so I 
picked up this branch with a passing tests
$ paver test
```

###Non-English Locales
If you're like me and your locale is ar_JO.UTF-8 `$ ls` will display the Arabic
names of the months. This will cause the `lms` tests to fail. Changing the 
locale using `$ unity-control-center language` and set everything to 
English USA. This should solve the issue. 
