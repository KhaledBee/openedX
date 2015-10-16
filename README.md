# openedX
----------------------------------------------              1            ---------------------------------------------------
Environment: Devstack vs Fullstack
The devops team at edX maintains two different Vagrant configuration for the Open edX project: devstack and fullstack. Devstack is designed to be used for development purposes: exploring the codebase, debugging problems, writing new features, etc. Fullstack is designed to more closely mimic an environment you might find in production: debugging is disabled, security defaults are tighter, more Open edX features are enabled, etc. You'll need to decide which of these two configurations you want to use. Generally speaking, you should choose devstack unless there is a specific reason that you want to use fullstack.
----------------------------------------------              2            ---------------------------------------------------
Logs
When something goes wrong with Open edX, the first thing to do is to check the application logs – they will give you some good information on how to debug the issue. On your VM, there is a directory at /edx/var/log, which contains subdirectories for the various different parts of Open edX. Each different part will write its logs to one of these subdirectories.
----------------------------------------------------------------------------------------------------------------------------
