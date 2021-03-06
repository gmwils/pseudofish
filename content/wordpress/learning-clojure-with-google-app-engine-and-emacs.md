Title: Learning Clojure with Google App Engine and Emacs
Date: 2010-09-18 23:30
Author: gmwils
Category: programming

This is an overview to my attempts at getting Clojure with Emacs.
Inspired by [this post][], I set out to learn three things
simultaneously: emacs, clojure and Google App Engine.

For comparison, my current web prototyping combination is TextMate,
Rails, and Heroku. Prior to that it was a mix of python, php or perl,
with vim.

There are already number of articles on getting GAE and Clojure to work.
Consider this me making notes for myself because I keep forgetting the
Emacs shortcuts!

### Emacs

I settled on Aquamacs to start out with emacs on the Mac. It seems to be
more forgiving if I use normal Mac keyboard shortcuts while I learn
emacs keys. Also makes it easy to switch between it and TextMate.

The other feature I love is full screen mode: ⌘-Shift-Return

To get command line access, you need to go to Tools-\>Install Command
Line Tools. This gives you an "`aquamacs`" shortcut in the shell. I
aliased it to "`aq`".

For some help on learning emacs, consider:

-   [Emacs Tour][]
-   [Emacs for Vi users][]

You may also want to setup Clojure to run with [Emacs][]. [This
article][Emacs] assumes you are using lein.

### Google App Engine (GAE)

The GAE distribution is available here: [GAE Download][].

I setup my shell to include a $GAE\_BASE directory for where I unpacked
it, and added $GAE\_BASE/bin to my path.

### Creating a Clojure Project

The first post I read assumed that you were starting with a manual
layout of your project. I'm going to use lein, a build manager for
clojure, similar to rake for ruby. Behind the scenes lein uses Java's
maven as a package management system (similar to gems).

To make things a bit more interesting, I plan to write a very basic CMS
to explore the various APIs. I'm keeping everything on the [githubs][].

The steps to getting setup look something like:

    $ lein new gaecms
    $ mkdir -p war/WEB-INF

Then make some changes to your project.clj file:

    :::clojure
    (defproject gae-cms "1.0.0-SNAPSHOT"
      :description "A basic CMS built on Google App Engine (GAE)"
      :dependencies [[org.clojure/clojure "1.2.0"]
                     [org.clojure/clojure-contrib "1.2.0"]
                     [compojure "0.4.1"]
                     [ring/ring-servlet "0.2.1"]
                     [appengine "0.4-SNAPSHOT"]]
    :dev-dependencies [[leiningen/lein-swank "1.2.0-SNAPSHOT"]]
    :compile-path "war/WEB-INF/classes"
    :library-path "war/WEB-INF/lib")

To save some typing, you can clone:
[git://github.com/gmwils/gaecms.git][githubs]

To run the interpreter locally, try the following:

    $ lein deps
    $ lein swank

To test out that the interpreted mode is working, open up
src/gae\_cms/core.clj in AQ.

Then connect to the server:

    M-x slime-connect

From within the editor, you can now run either of the following:

    C-x C-e    % evaluate current line
    C-c C-k    % compile the current file

You also have an interactive clojure shell for typing commands into. Try
adding the following into your source code and then pressing C-x C-e to
evaluate:

    (System/getProperty "java.class.path")

You should see your current CLASSPATH printed out.

### Setup for GAE

In `$HOME/war/WEB-INF/web.xml` map the URLs that you want your custom
servlet to handle. In this case, send everything into the servlet:

    :::xml
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <web-app
      xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
      version="2.5">
      <display-name>CMS on GAE</display-name>
      <servlet>
        <servlet-name>cms</servlet-name>
        <servlet-class>gae-cms.core</servlet-class>
      </servlet>
      <servlet-mapping>
        <servlet-name>cms</servlet-name>
        <url-pattern>/</url-pattern>
      </servlet-mapping>
    </web-app>

In `$HOME/war/WEB-INF/appengine-web.xml` add the following to setup
Google App Engine to refer to your servlet above.

    :::xml
    <appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
      <application>cms-clj</application>  <!-- GAE app id for your app -->
      <version>v0-1</version>  <!-- Arbritrary Version Id -->
    </appengine-web-app>

Next, update src/gaecms/core.clj with:

    :::clojure
    (ns gaecms.core
      (:gen-class :extends javax.servlet.http.HttpServlet)
      (:use
        compojure.core
        [ring.util.servlet :only [defservice]])
      (:require [compojure.route :as route]))

    (defroutes cms-public
      (GET "/" []
        "<html><title>GAE CMS</title><body><h1>Hello World!</h1></body>")  )
    (defroutes cms
      cms-public
      (route/not-found "Page not found")) ; 404 error page

    (defservice cms)

To test this locally, try:

    :::shell
    $ lein deps
    $ lein compile
    $ $GAESDK/bin/dev_appserver.sh war

### Deploying to Google

First go to the [GAE console][] and create an application. The following
should then work:

    :::shell
    $ lein deps
    $ lein compile
    $ $GAESDK/bin/appcfg.sh update war

I'm still playing with getting it working with Google App Engine. The
above is enough to get [hello world][] working.

For more details, I suggest you follow the [Compojure on GAE][] blog.
Specifically, the article on [Deploying to App Engine][]is a good
overview.

Next steps include getting [interactive development][] working with GAE,
and actually handling [dynamic content][].

  [this post]: http://www.hackers-with-attitude.com/2009/08/intertactive-programming-with-clojure.html
  [Emacs Tour]: http://www.gnu.org/software/emacs/tour/
  [Emacs for Vi users]: http://www.elmindreda.org/emacs.html
  [Emacs]: http://riddell.us/ClojureSwankLeiningenWithEmacsOnLinux.html
  [GAE Download]: http://code.google.com/appengine/downloads.html
  [githubs]: http://github.com/gmwils/gaecms
  [GAE console]: https://appengine.google.com/
  [hello world]: http://cms-clj.appspot.com/
  [Compojure on GAE]: http://compojureongae.posterous.com
  [Deploying to App Engine]: http://compojureongae.posterous.com/deploying-to-app-engine
  [interactive development]: http://compojureongae.posterous.com/getting-interactive-development-to-work-again
  [dynamic content]: http://compojureongae.posterous.com/accessing-the-app-engine-datastore
