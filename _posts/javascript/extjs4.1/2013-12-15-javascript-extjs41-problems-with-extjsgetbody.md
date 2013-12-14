---
layout: post
title: "javascript extjs4.1 Problems with Extjs.getBody"
description: ""
category: ""
tags: [javascript,extjs4.1]
---
{% include JB/setup %}
[FROM](http://stackoverflow.com/questions/11295896/problems-with-extjs-getbody)   
I have this line of code  

    var width_client = Ext.getBody().getWidth(true);

and I get that Ext.getBody() is null. I asume that this check is doen before Ext.getBody() becames not null, but dont' know were or what to change.  
Any idea how to solve this?  
I use extjs 4.0.7  

You can access document.body when it is rendered. From your question I made a conclusion that your code was located inside <head>. When this code is executed there is no <body> rendered yet. You have to wrap your code by a function and pass this function to Ext.onReady (or Ext.onDocumentReady) method:   

    Ext.onReady(function() {
        alert(Ext.getBody().getWidth(true));
    });

