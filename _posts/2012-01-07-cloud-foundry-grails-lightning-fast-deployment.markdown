---
date: '2012-01-07 17:45:50'
layout: post
slug: cloud-foundry-grails-lightning-fast-deployment
status: publish
title: Cloud Foundry + Grails = lightning fast deployment
wordpress_id: '46'
categories:
- Grails
tags:
- cloud
- cloud foundry
- deployment
- grails
---

The purpose of this article is to show how quickly and easily deploy [Grails](http://www.grails.org) application to Cloud Foundry platform.





What is Cloud Foundry?





From [cloudfoundry.com](http://www.cloudfoundry.com/about):





> Cloud Foundry is an open platform as a service, providing a choice of clouds, developer frameworks and application services. Initiated by VMware, with broad industry support, Cloud Foundry makes it faster and easier to build, test, deploy and scale applications. It is an open source project and is available through a variety of private cloud distributions and public cloud instances, including CloudFoundry.com.






## Prerequisites



To make a use of this article you'll need to have Grails framework installed.





To check if Grails is properly installed, use 




{% highlight text %}
  grails -version
{% endhighlight %}





You should see something like




{% highlight text %}
  Grails version: 2.0.0
{% endhighlight %}


If you haven't, just follow the [Grails Getting Started Guide](http://grails.org/doc/latest/guide/gettingStarted.html#requirements).





## Let's the fun begin





The whole process is extraordinary simple! Just do the following steps.







  1\. Create Grails app




{% highlight text %}
  grails create-app cloud_foundry_example
{% endhighlight %}



  2\. Change your working directory to a new application directory




{% highlight text %}
  cd cloud_foundry_example
{% endhighlight %}



  3\. Install Cloud Foundry plug-in




{% highlight text %}
  grails install-plugin cloud-foundry
{% endhighlight %}
    



  4\. Configure your Cloud Foundry credentials





You can configure it in both _grails-app/conf/Config.groovy_ and in _~/.grails/settings.groovy_. Since the file will contain sensitive, it's not recommended to put it in source control. That's why the second option is considered the best practice.





So, configure your credentials using




{% highlight text %}
  grails.plugin.cloudfoundry.username = "<your_username>"
  grails.plugin.cloudfoundry.password = "<pass>"
{% endhighlight %}



  5\. Test your config




{% highlight text %}
  grails cf-info
{% endhighlight %}





The output should look like




{% highlight text %}
  VMware's Cloud Application Platform
  For support visit http://support.cloudfoundry.com
  Target:   http://api.cloudfoundry.com (v0.999)
  
  User:     <your_username>
  Usage:    Memory   (0B of 2.0G total)
            Services (0 of 16 total)
            Apps     (0 of 20 total)
{% endhighlight %}



  6\. Deploy your app!




{% highlight text %}
  grails prod cf-push
{% endhighlight %}





You'll be asked about the application URL, and also about some persistence services (in the time of writing: MySQL and PostgreSQL). I advise to accept default address and chose one of these services.





If you want to re-deploy after some changes, just do




{% highlight text %}
  grails prod cf-update
{% endhighlight %}






## That's it!





The app is deployed and ready to work. A proof? [PartyPlanner app](http://partyplanner.cloudfoundry.com/).





In next part I'll describe how to configure different services in Cloud Foundry.



