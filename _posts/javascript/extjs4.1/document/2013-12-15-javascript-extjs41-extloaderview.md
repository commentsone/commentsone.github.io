---
layout: post
title: "javascript extjs4.1 Ext.LoaderView"
description: ""
category: ""
tags: [javascript,extjs4.1]
---
{% include JB/setup %}
Ext.Loader is the heart of the new dynamic dependency loading capability in Ext JS 4+. It is most commonly used via the Ext.require shorthand. Ext.Loader supports both asynchronous and synchronous loading approaches, and leverage their advantages for the best development flow. We'll discuss about the pros and cons of each approach:  

##Asynchronous Loading  
###Advantages:

- Cross-domain  
- No web server needed: you can run the application via the file system protocol (i.e: file://path/to/your/index .html)  
- Best possible debugging experience: error messages come with the exact file name and line number  

###Disadvantages:  

- Dependencies need to be specified before-hand  

###Method 1: Explicitly include what you need:  

    // Syntax
    Ext.require({String/Array} expressions);

    // Example: Single alias
    Ext.require('widget.window');

    // Example: Single class name
    Ext.require('Ext.window.Window');

    // Example: Multiple aliases / class names mix
    Ext.require(['widget.window', 'layout.border', 'Ext.data.Connection']);

    // Wildcards
    Ext.require(['widget.*', 'layout.*', 'Ext.data.*']);

###Method 2: Explicitly exclude what you don't need:
   
     // Syntax: Note that it must be in this chaining format.
     Ext.exclude({String/Array} expressions)
        .require({String/Array} expressions);

    // Include everything except Ext.data.*
    Ext.exclude('Ext.data.*').require('*');

    // Include all widgets except widget.checkbox*,
    // which will match widget.checkbox, widget.checkboxfield, widget.checkboxgroup, etc.
    Ext.exclude('widget.checkbox*').require('widget.*');

##Synchronous Loading on Demand

###Advantages:

There's no need to specify dependencies before-hand, which is always the convenience of including ext-all.js before  

###Disadvantages:

Not as good debugging experience since file name won't be shown (except in Firebug at the moment)  
Must be from the same domain due to XHR restriction  
Need a web server, same reason as above  

There's one simple rule to follow: Instantiate everything with Ext.create instead of the new keyword  

    Ext.create('widget.window', { ... }); // Instead of new Ext.window.Window({...});

    Ext.create('Ext.window.Window', {}); // Same as above, using full class name instead of alias

    Ext.widget('window', {}); // Same as above, all you need is the traditional `xtype`

Behind the scene, Ext.ClassManager will automatically check whether the given class name / alias has already existed on the page. If it's not, Ext.Loader will immediately switch itself to synchronous mode and automatic load the given class and all its dependencies.  

##Hybrid Loading - The Best of Both Worlds

It has all the advantages combined from asynchronous and synchronous loading. The development flow is simple:  

###Step 1: Start writing your application using synchronous approach.  
Ext.Loader will automatically fetch all dependencies on demand as they're needed during run-time. For example:  

    Ext.onReady(function(){
        var window = Ext.createWidget('window', {
            width: 500,
            height: 300,
            layout: {
                type: 'border',
                padding: 5
            },
            title: 'Hello Dialog',
            items: [{
                title: 'Navigation',
                collapsible: true,
                region: 'west',
                width: 200,
                html: 'Hello',
                split: true
            }, {
                title: 'TabPanel',
                region: 'center'
            }]
        });

        window.show();
    })

###Step 2: Along the way, when you need better debugging ability, watch the console for warnings like these:  

    [Ext.Loader] Synchronously loading 'Ext.window.Window'; consider adding Ext.require('Ext.window.Window') before your application's code  
    ClassManager.js:432
    [Ext.Loader] Synchronously loading 'Ext.layout.container.Border'; consider adding Ext.require('Ext.layout.container.Border') before your application's code

Simply copy and paste the suggested code above Ext.onReady, i.e:  

    Ext.require('Ext.window.Window');
    Ext.require('Ext.layout.container.Border');

    Ext.onReady(...);

Everything should now load via asynchronous mode.

##Deployment

It's important to note that dynamic loading should only be used during development on your local machines. During production, all dependencies should be combined into one single JavaScript file. Ext.Loader makes the whole process of transitioning from / to between development / maintenance and production as easy as possible. Internally Ext.Loader.history maintains the list of all dependencies your application needs in the exact loading sequence. It's as simple as concatenating all files in this array into one, then include it on top of your application.  
This process will be automated with Sencha Command, to be released and documented towards Ext JS 4 Final.  
##Config options

disableCaching : Boolean
Appends current timestamp to script files to prevent caching. ...  
     
disableCachingParam : String  
The get parameter name for the cache buster's timestamp. ...  
     
    view sourceenabled : Boolean
    Whether or not to enable the dynamic dependency loading feature. ...
     
    Ext.Loader
    view sourcegarbageCollect : Boolean
    True to prepare an asynchronous script tag for garbage collection (effective only if preserveScripts is false) ...
     
    Ext.Loader
    view sourcepaths : Object
    The mapping from namespaces to file paths { 'Ext': '.', // This is set by default, Ext.layout.container.Containe...
     
    Ext.Loader
    view sourcepreserveScripts : Boolean
    False to remove and optionally garbage-collect asynchronously loaded scripts, True to retain script element for brows...
     
    Ext.Loader
    view sourcescriptChainDelay : Boolean
    millisecond delay between asynchronous script injection (prevents stack overflow on some user agents) 'false' disable...
     
    Ext.Loader
    view sourcescriptCharset : String
    Optional charset to specify encoding of dynamic script content. ...
    Defined By
    Properties
     
    Ext.Loader
    view sourcehistory : Array
    An array of class names to keep track of the dependency loading order. ...
    Defined By
    Methods
     
    Ext.Loader
    view sourceexclude( Array excludes ) : Object
    Explicitly exclude files from being loaded. ...
     
    Ext.Loader
    view sourcegetConfig( String name ) : Object
    Get the config value corresponding to the specified name. ...
     
    Ext.Loader
    view sourcegetPath( String className ) : String
    Translates a className to a file path by adding the the proper prefix and converting the .'s to /'s. ...
     
    Ext.Loader
    view sourceloadScript( Object/String options )
    Loads the specified script URL and calls the supplied callbacks. ...
     
    Ext.Loader
    view sourceonReady( Function fn, Object scope, Boolean withDomReady )
    Add a new listener to be executed when all required scripts are fully loaded ...
     
    Ext.Loader
    view sourcerequire( String/Array expressions, [Function fn], [Object scope], [String/Array excludes] )
    Loads all classes by the given names and all their direct dependencies; optionally executes the given callback functi...
     
    Ext.Loader
    view sourcesetConfig( Object config ) : Ext.Loader
    Set the configuration for the loader. ...
     
    Ext.Loader
    view sourcesetPath( String/Object name, String path ) : Ext.Loader
    Sets the path of a namespace. ...
     
    Ext.Loader
    view sourcesyncRequire( String/Array expressions, [Function fn], [Object scope], [String/Array excludes] )
    Synchronously loads all classes by the given names and all their direct dependencies; optionally executes the given c...
