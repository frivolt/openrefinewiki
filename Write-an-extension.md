(This document is for the version 2.0 and later only. This is still in flux.)

Google Refine makes use of the [Butterfly framework](http://code.google.com/p/simile-butterfly/) to provide an extension architecture. Google Refine extensions are Butterfly modules. You don't really need to know about Butterfly itself, but you might encounter "butterfly" here and there in the code base.

Note that Butterfly was chosen over OSGi because

* Butterfly modules can contain both Java code (server-side) as well as webapp code (client-side); and
* Unlike OSGi plugins, Butterfly modules don't have to be jarred and copied into one specific directory--both of which tasks make development very painful.

Extensions that come with the code base are located under [the extensions subdirectory](../tree/master/extensions), but when you develop your own extension, you can put its code anywhere as long as you point Butterfly to it. That is done by any one of the following methods

* refer to your extension's directory in [the butterfly.properites file](../tree/master/main/webapp/WEB-INF/butterfly.properties) through a butterfly.modules.path setting.
* specify the butterfly.modules.path property on the command line when you run Google Refine. This overrides the values in the property file, so you need to include the default values first e.g. -Dbutterfly.modules.path=modules,../../extensions,/path/to/your/extension

## Directory Layout

A Google Refine extension (implemented as a Butterfly module) sits in a file directory that contains the following files and sub-directories:

    build.xml
      src/
         com/foo/bar/... *.java source files
      module/
        *.html, *.vt files
        scripts/... *.js files
        styles/... *.css and *.less files
        images/... image files
        MOD-INF/
            lib/*.jar files
            classes/... java class files
            module.properties
            controller.js
The file named module.properties (see [example](extensions/sample-extension/module/MOD-INF/module.properties)) contains the extension's metadata. Of importance is the name field, which gives the extension a name that's used in many other places to refer to it. This can be different from the extension's directory name.

      name = my-extension-name
Your extension's client-side resources (.html, .js, .css files) stored in the module/ subdirectory will be accessible from [http://127.0.0.1:3333/extension/my-extension-name/](http://127.0.0.1:3333/extension/my-extension-name/), when OpenRefine is running. Note: the last slash is important, without it you'll get HTTP Error 404.

Also of importance is the dependency

     requires = core
which makes sure that the core module of Google Refine is loaded before the extension attempts to hook into it.

The file named controller.js is responsible for registering the extension's hooks into Google Refine. Look at the sample-extension extension's [controller.js](../tree/master/extensions/sample/module/MOD-INF/controller.js) file for an example. It should have a function called init() that does the hook registrations.

The build.xml file is an [Apache Ant](http://ant.apache.org/) build file. You can make a copy of the sample extension's build.xml file to get started. The important point here is that the Java classes should be built into the module/MOD-INF/classes sub-directory.

Note that your extension's Java code would need to reference some libraries used in Google Refine and Refine's Java classes themselves. The libraries are in [master/main/webapp/WEB-INF/lib/](../tree/master/main/webapp/WEB-INF/lib/) and Refine's classes are in [master/main/webapp/modules/core/MOD-INF/classes/](../tree/master/main/webapp/modules/core/MOD-INF/classes/).


*Check out* [Sample extension](Sample-extension).