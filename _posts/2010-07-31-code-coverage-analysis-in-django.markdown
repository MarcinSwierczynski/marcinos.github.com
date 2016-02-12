---
date: 2010-07-31 17:09:22 +0100
layout: post
slug: code-coverage-analysis-in-django
status: publish
title: Code coverage analysis in Django
wordpress_id: '18'
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

Code coverage analysis could be very valuable if you want to do reliable testing. 100% coverage doesn't give you a guarantee that everything will be fine, but still - it's extremely useful tool. How can we use it in Django-based application?




First of all, we'll need an awesome module called [coverage.py](http://nedbatchelder.com/code/coverage/) and written by [Ned Batchelder](http://nedbatchelder.com/). I recommend to install it on a development machine instead of just attaching it to the project, because it can give us a huge performance boost -- almost 600% during my tests.




Our next step could be a [test runner](http://docs.djangoproject.com/en/1.2/topics/testing/#defining-a-test-runner) concept. It is a quite good approach, because we haven't to worry about running code coverage analysis -- we can just start our tests, as we usually do. I've found a great [code snippet](http://djangosnippets.org/snippets/705/) with such a runner. But what if we don't want to run analysis every time we test our application? Code coverage can be quite time-consuming while the first rule of TDD says -- tests should be as fast as possible. With this thought in mind, I've decided to write my own Django custom command. It is based on a snippet I've mentioned and its aim is to run tests with code coverage analysis using a separate command. It is also able to generate pretty HTML reports (see [example](http://nedbatchelder.com/code/coverage/sample_html)).




The main part of this command is a Command class.




```python
#imports

class Command(test.Command):
    args = '[app_name ...]'
    help = 'Generates code coverage report'

    option_list = test.Command.option_list + (
            make_option('--format', '-f', dest='format', default='txt', help='Change report output format (html or txt, default: txt)'),
            make_option('--directory', '-d', dest='directory', default='.',
                        help='Change html report output directory. Default: current directory'),
        )

    def __init__(self):
        self.cov = coverage.coverage()
        self.coverage_modules = []

        self.cov.use_cache(0)

        try:
            from south.management.commands import patch_for_test_db_setup
            patch_for_test_db_setup()
        except ImportError:
            pass

    def handle(self, *test_labels, **options):
        self.__run_tests_with_coverage_analyse(test_labels, options)

        if test_labels:
            self.__add_selected_applications_to_report(test_labels)
        else:
            self.__add_all_aplications_to_report()

        if self.coverage_modules:
            self.cov.report(self.coverage_modules, show_missing=1)
            if options['format'] == 'html':
                dest_path = os.path.join(options['directory'], 'coverage_report')
                self.__generate_html_report(dest_path)


        self.cov.erase()

    def __run_tests_with_coverage_analyse(self, test_labels, options):
        self.cov.start()
        super(Command, self).handle(*test_labels, **options)
        self.cov.stop()

    def __add_selected_applications_to_report(self, test_labels):
        for label in test_labels:
            # Don't report coverage if you're only running a single
            # test case.
            if '.' not in label:
                app = get_app(label)
                self.coverage_modules.extend(self.__get_coverage_modules(app))

    def __add_all_aplications_to_report(self):
        for app in get_apps():
            self.coverage_modules.extend(self.__get_coverage_modules(app))

    def __generate_html_report(self, dest_dir):
        print "Generating HTML report in %s..." % dest_dir
        self.__delete_directory_content(dest_dir)
        self.cov.html_report(self.coverage_modules, directory=dest_dir)

    def __get_coverage_modules(self, app_module):
        """
        Returns a list of modules to report coverage info for, given an
        application module.
        """
        app_path = app_module.__name__.split('.')[:-1]
        coverage_module = __import__('.'.join(app_path), {}, {}, app_path[-1])

        #ignore external modules/applications
        module_path = coverage_module.__path__[0]
        if 'webapp/apps' not in module_path:
            return []

        return [attr for name, attr in
            getmembers(coverage_module) if ismodule(attr) and name != 'tests']

#some other stuffs
```


As we can see in lines 7-11, it has two additional options:

  * `-f` or `--format` which can take a `html` value to generate an HTML report


  * `-d` or `--directory` which can take a path to destination directory where an HTML report will be saved to




Either we decide to generate an HTML report or not, we'll always get a text report in the console. Moreover, the default localization of this report is current directory, so we haven't to declare a -d parameter at all.




Line 21 is very important too. You've to call this function, if you use South migrations in your application. Otherwise your tests will fail.




Obviously, this command works just as classic test command. You can call all tests with `./manage.py code_coverage`. You can also run tests for selected modules, ex. `./manage.py code_coverage app1 app2`.




I hope it'll be helpful. I'm also open to any ideas.
