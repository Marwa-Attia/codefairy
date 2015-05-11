---
layout: post
title: Simple web app with AngularJS + Play + MongoDB part1
---

I've always hated javascript, so when I came across AngularJS I thought it would be just another javascript library, but it actually turned out to be more interesting than that, so I decided that I should write my experience with AngularJS, and to make things a little more interesting I decided to try out Play framework as well.

So what exactly are we going to do here?

We are going to develop a simple web application using AngularJS, Play Framework and MongoDB.

This will be made in 2 parts:

**Part 1** Create a Play java application using activator templates and integrate it with AngularJS and MongoDB using Morphia

**Part 2** Develop the application itself.

So let's start right away with part 1

#Create a Play Java Project 
###Install Activator
  Play is distributed through a tool called [Typesafe Activator](http://typesafe.com/activator). Typesafe Activator provides the build tool (sbt) that Play is built on, and also provides many templates and tutorials to help get you started with writing new applications.
  Download the latest [Activator distribution](https://www.typesafe.com/get-started) and extract the archive to a location where you have both read and write access. 
###Add the activator script to your PATH
  For convenience, you should add the Activator installation directory to your system _PATH_. On UNIX systems, this means doing something like:
  
    export PATH=$PATH:/relativePath/to/activator

On Windows you’ll need to set it in the global environment variables. This means update the _PATH_ in the environment variables and don’t use a path with spaces.
###Create a new vanilla Play Java application
Use the following activator command to create a vanilla play java application

    $ activator new VanillaJava play-java	
  
Once the application has been created you can use the _activator_ command again to enter the Play console:

    $ cd VanillaJava
    $ activator

To run the current application in development mode, use the run command:

    [VanillaJava] $ run
  
To stop the server, type _Crtl+D_ key, and you will be returned to the Play console prompt.
To compile your application without running the server, just use the compile command:

    [VanillaJava] $ compile
  
You can ask Play to start a JPDA debug port when starting the console. You can then connect using Java debugger. Use the _activator -jvm-debug <port>_ command to do that:

    $ activator -jvm-debug 9999
  
###Configure the application to work on Eclipse IDE
Play provides a command to simplify Eclipse configuration. To transform a Play application into a working Eclipse project, use the eclipse command:

    [VanillaJava] $ eclipse

You then need to import the application into your Workspace with the **File/Import/General/Existing project…** menu (compile your project first).
To debug, start your application with the command:

    activator -jvm-debug 9999 ~run 

Now in Eclipse right-click on the project and select Debug As, Debug Configurations. In the Debug Configurations dialog, right-click on **Remote Java Application and select New**. Change Port to _9999_ and click Apply. 
  
#Set up your application to use AngularJS
  
  Setting up your application to use Angular is as simple as downloading the javascript file from [here](https://angularjs.org/), and including a link to the file in your _main.scala.html_ page as follows:
  
    <script src="@routes.Assets.at("javascripts/angular.js")" type="text/javascript"></script>	
    
  
#Set up your application to Use MongoDB + Morphia
###Download and run MongoDB server
Download MongoDB from [here](https://www.mongodb.org/downloads), now you can either install or unzip your mongodb server depending on which distribution you downloaded.
Now to configure mongo server, navigate to your MongoDB server **[mongo-server-path]** and create your datastore folder: _data/db_ and your log file: _log/ mongo.log_
Now let’s create mongo configuration file _mongo.config_ and add the following configuration to the file:

    ##store data here
    dbpath=[mongo-server-path] \data 
    
    ##all output go here
    logpath=[mongo-server-path] \log\mongo.log
    
    ##log read and write operations
    diaglog=

Now you can start mongo server using this command:
  
    [mongo-server-path]\bin\mongod --config [mongo-server-path] \mongo.config

This command will start mongo server with the configuration we added in the above section.

###Add Morphia dependency  to our vanilla play app
Now let’s add Morphia dependency to the play app we created earlier, to do that just navigate to the root folder of the play app and open _build.sbt_ file and add this line:

    "org.mongodb.morphia" % "morphia" % "0.107"

 So now your build.sbt file should look something like that:
 
    name := """VanillaJava"""
    version := "1.0-SNAPSHOT"
    lazy val root = (project in file(".")).enablePlugins(PlayJava)
    scalaVersion := "2.11.1"
    libraryDependencies ++= Seq(
    javaJdbc,
    javaEbean,
    cache,
    javaWs,
    "org.mongodb.morphia" % "morphia" % "0.107"
      )

Now that’s all the configuration you need to start a Play-AngularJS-MongoDB web application.

To connect to MongoDB from play we have to create global settings class which will run on application start up.
To do that we have also to create a globally accessed utility class:

**MorphiaObject.java**

    package controllers;

    import org.mongodb.morphia.Datastore;
    import org.mongodb.morphia.Morphia;

    import com.mongodb.Mongo;

    public class MorphiaObject {
	    static public Mongo mongo;
	    static public Morphia morphia;
	    static public Datastore datastore;
    }

And **Global.java** which will connect to mongoDB on start up.

    import java.net.UnknownHostException;

    import org.mongodb.morphia.Morphia;

    import play.GlobalSettings;
    import play.Logger;

    import com.mongodb.Mongo;

    import controllers.MorphiaObject;


    public class Global extends GlobalSettings {
    	@Override
    	public void onStart(play.Application arg0) {
		    super.beforeStart(arg0);
		    Logger.debug("** onStart **"); 
		    try {
			    MorphiaObject.mongo = new Mongo("127.0.0.1", 27017);
		    } catch (UnknownHostException e) {
		    	e.printStackTrace();
		    }
	    	MorphiaObject.morphia = new Morphia();
	    	MorphiaObject.datastore = MorphiaObject.morphia.createDatastore(MorphiaObject.mongo, "test");
		    MorphiaObject.datastore.ensureIndexes();   
		    MorphiaObject.datastore.ensureCaps();  

		    Logger.debug("** Morphia datastore: " + MorphiaObject.datastore.getDB());
	    }
    }

If you start your Play server now using the _run_ command you should get something like that on the command line:

![_console]({{ site.baseurl }}/images/part1 console.png)

If you opened  [localhost:9000](localhost:9000) on your browser you should see this page:

![_app]({{ site.baseurl }}/images/part1.png)

That's all you'll need to run an application with AngularJS, Play, MongoDB stack.
In the next post we will start developing a simple example to see how all this work togther.
