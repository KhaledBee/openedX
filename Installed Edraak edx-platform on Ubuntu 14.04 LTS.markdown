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
- show quoted text -
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
- show quoted text -
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
