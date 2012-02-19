---
date: '2010-10-02 13:10:16'
layout: post
slug: virtualenv-pip
status: publish
title: Virtualenv + pip
wordpress_id: '33'
categories:
- Django
- Programming
- Python
tags:
- django
- english
- programming
- python
---

In the Java world there was always a problem with dependencies. All the jars we had to mange by hand - quite awful. Thankfully there is a Maven project which helps us in requirements management. But what about Python? Is there any way to handle necessary libraries? Of course, there is! It is a tandem of [virtualenv](http://pypi.python.org/pypi/virtualenv) and [pip](http://pip.openplans.org/).

Virtualenv is a tool to create isolated Python environments. I'm sure you can imagine a situation when one project needs a library in version x, while another one needs the same library but in version y. Then, you can't install both libraries to your default site-packages directory. Another example: let's assume that new version of Django has been released and you want to test your application with this version. It'd perfect to create independent environment for the new release, wouldn't it? So, this is a place for virtualenv.

Pip is simply the Python packages installer. It can be called easy_install successor. Here, you can read [pip and easy_install compare](http://pip.openplans.org/#pip-compared-to-easy-install). From our point of view, pip has one extremely useful feature. It can save all requirements information to the text file and use it later to download necessary libraries. The example file is listed below.

    
    Django==1.2.1
    Fabric==0.9.1
    PIL==1.1.7
    South==0.7.2
    ...


OK, but how can we use it together? Here comes how. Let's assume, we've virtualenv installed. Then, just do the following steps.



	
  1. `virtualenv --no-site-packages environment_name` - this command creates new environment with given name. The `no-site-packages` option ensures that this environment won't inherit any libraries previously installed in your system.

	
  2. `cd environment_name` - virtualenv has created a new directory, so we've to go to it

	
  3. Now, we've to install all requirements into a new environment. As a bonus, virtualenv contains pip package and puts it into each environment. So, all you've to do is `bin/pip install package_name`. We need to repeat this command for each library our project depends on.

	
  4. It'd be useful to export all the dependencies into to the file to use it in the future. To do that just run `bin/pip freeze > requirements_file.txt`

	
  5. You can use this file to build your environment later, perhaps on another machine. It's simple: `bin/pip install -r requirements_file.txt`

	
  6. It's almost done. The last thing is to activate our new virtual environment. We need this to run all subsequent commands within its context. Use `source bin/activate` and notice that your command prompt got a prefix - an environment name. To deactivate the environment, simply write `deactivate`.




Of course, the possibilities of virtualenv and pip are a lot more sophisticated. For example, you can create your own bootstrap script with virtualenv to setup a whole web application. On the other hand, you can use pip to download dependencies from version control systems directly or even create your own libraries bundle to use it when you're offline. The use case I've described above is just an introduction. But I hope, it is a useful one.
