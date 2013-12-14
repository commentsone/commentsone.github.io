---
layout: post
title: "javascript extjs4.1 when using Ext.require new class in Ext.onReady"
description: "javascript"
category: "javascript"
tags: [javascript,extjs4.1]
---
{% include JB/setup %}


    Ext.require('Ext.form.FormPanel');
    Ext.onReady(function(){
            var p = new Ext.form.FormPanel({
                    title:'test',
                    width: 300,
                    height: 150,
                    html:'a floating panel',
                    floating: true,
                    renderTo: Ext.getBody()
            });
            p.setPosition(200,200);
    });

codes above is ok.  

    Ext.require('Ext.form.FormPanel');
    var p = new Ext.form.FormPanel({
            title:'test', 
            width: 300, 
            height: 150, 
            html:'a floating panel', 
            floating: true, 
            renderTo: Ext.getBody()
    });     
    p.setPosition(200,200);
    //Uncaught TypeError: Cannot read property 'FormPanel' of undefined   
