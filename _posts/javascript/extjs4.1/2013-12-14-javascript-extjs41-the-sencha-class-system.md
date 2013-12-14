---
layout: post
title: "javascript extjs4.1 The Sencha Class System"
description: ""
category: ""
tags: []
---
{% include JB/setup %}
[FROM](http://www.sencha.com/learn/sencha-class-system)  
The Sencha Class System was first introduced in Ext JS 4.0 and was a major step forward in making it easy to build object oriented JavaScript code. As a core part of the Sencha JavaScript platform, it’s now a shared component between Ext JS and Sencha Touch 2.  

Ext JS and Sencha Touch 2 use the class system internally to manage dependencies, make code more reusable as well as provide a rich set of features that are commonly found in other class-based programming languages. Developers writing code using Ext JS 4 and Touch 2 can leverage the class system to use the built-in APIs in the frameworks, as well as in their own code to create well structured object oriented JavaScript.

Creating a Class
At the lowest level, creating a new class is as simple as instantiating Ext.Class:

var Person = new Ext.Class({});
The first argument for new Ext.Class is an object with key-value pairs representing the class members. A Person class with a name property and a walk method may look like this:

var Person = new Ext.Class({
    name: 'Mr. Unknown',
    walk: function(steps) {
        alert(this.name + ' is walking ' + steps + ' steps');
    }
});
Let’s write a small piece of code to test the above Person class:

var person = new Person();
person.walk(10); // alerts "Mr. Unknown is walking 10 steps"
Class Constructor
A class constructor is the class method that gets invoked immediately when a new instance of that class is created. Adding a constructor is as trivial as adding it as a class member at definition time.

var Person = new Ext.Class({
    name: 'Mr. Unknown',
 
    constructor: function(name) {
        this.name = name;
 
        return this;
    },
 
    walk: function(steps) {
        alert(this.name + ' is walking ' + steps + ' steps');
    }
});
 
var jacky = new Person('Jacky');
jacky.walk(10); // alerts "Jacky is walking 10 steps"
Class Namespacing
Namespacing keeps your code well-organized, which translates directly into maintainability and reuseablity. Think of namespacing as categorizing or grouping your classes. Multiple grouping criteria can be used. The suggested namespacing convention is as follows:

<Company / Product ID / Organization>.<Functionality Group(s)>.<Class Name>

Some examples of good namespaces:

Ext.chart.Label
Ext.data.writer.Xml
MyCompany.util.VideoConverter
From this point forward, we will namespace all our sample classes in this article under My.sample. Let’s start with the same old Person class:

Ext.ns('My.sample');
My.sample.Person = new Ext.Class({ /*…*/ });
Notice the use of Ext.ns. It’s a shorthand to make it easier to work with nested object hierarchies. This method ensures the namespace objects exist before value assignment, a verbose version of which would look something like this:

if (!Ext.isObject(My)) {
    My = {};
}
 
if (!Ext.isObject(My.sample)) {
    My.sample = {};
}
 
My.sample.Person = new Ext.Class({ /*…*/ });
Class Files Organization
Keeping your class files organized also helps with reusability and makes it easier for other developers to understand the structure of your project. As a best practice, we recommend:

There should be only one class per JavaScript file
The file name should directly map to the class name. For example: My.cool.Class is located inside My/cool/Class.js
Creating a Class — Revisited
Let’s go back to the way we have been defining our class so far:

Ext.ns('My.sample');
My.sample.Person = new Ext.Class({ /*…*/ });
It’s time to dig a little deeper to see what is technically going on here. We are creating an anonymous class with new Ext.Class, then assigning it to the Person property of the My.sample object. It works well but has limitations:

We have to manually ensure the namespace object exists before value assignment, for every single class being created.
What if our class depends on some other classes existing first? A typical example would be a parent-child relationship whereby the child class needs the parent class to exist beforehand. In such case, the parent class may need to be retrieved asynchronously, and the child class is not yet ready at definition time.
What if you need to run two or more different versions of the same class (all of which have the same name) on the same application without having them overwrite each other?
What if you want to retrieve the current name of the class from within its own methods for better debugging capability. For example, error messages could have the class name from where they originate. Methods can have their full name in the stack-trace via Function.displayName instead of just ‘anonymous’
The answer to all the questions above is Ext.define

Introducing Ext.define
Ext.define significantly simplifies class definition code. This method takes 3 arguments:

Ext.define((String) className, (Object) classMembers, (Optional Function) onClassCreatedCallback);
The parameters mean:

className is the full class name in dot-namespaced format
classMembers is an object represents a collection of class members in key-value pairs
onClassCreatedCallback is an optional function callback to be invoked when all dependencies of this class are ready, and the class itself is fully created. Due to the new asynchronous nature of class creation, this callback can be useful to wrap logic that needs to be executed right after the class is defined. It is generally the place for backward-compatibility overrides.
Following this, our Person class becomes:

Ext.define('My.sample.Person', {
    name: 'Mr. Unknown',
 
    constructor: function(name) { /* … */ },
 
    walk: function(steps) { /* … */ }
});
With a tiny change and even less code, Ext.define immediately provides solutions to the challenges raised above:

There’s no longer a need to care about namespace object existence. Ext.define handles that automatically.
Simply by defining the class using the string class name, we’re no longer assigning the class reference directly to a variable. That way, the whole class creation process can be asynchronous, and the actual reference to the newly created class can be assigned to the provided namespace when it’s ready, at a later time.
Running multiple versions of the same class (also known as sandboxing) becomes trivial. The string class name can be changed to something else prior to value assignment. For example: the later-defined Ext.Button can be automatically rewritten to Ext4.Button so we can use an Ext JS 4.x button alongside an Ext JS 3.x button, without having to change a single line of code.
All classes can now be aware of their own names. As a result, you can retrieve the class name from within any class methods. All class methods can also have their full displayName signature. This enables much better debugging.
In short, unless you explicitly want to create anonymous classes, use Ext.define instead of new Ext.Class.

Class Processors
The Sencha Class System is built on top of processors, which are divided into two groups: pre-processors and post-processors. Although the term “pre-processors” may sound complex, class pre-processors are much simpler than you may think. They are simply a set of “hooks” that run before that class is actually created and ready to be used. Let’s walk through what’s happening behind Ext.define at a high-level:

Iterate through the provided classMembers object and set them to the class prototype.
During iteration, if there’s any “special” property such as: “extend”, “mixins”, “requires”, “config”, etc., process them through their corresponding pre-processors.
That said, when you define the class, in the classMembers object, if you provide:

an “extend” property, the “extend” pre-processor will handle inheritance for that class.
a “mixins” property, the “mixins” pre-processor will mix those other classes into the current class
a “config” property, the “config” pre-processor will automatically generate setters and getters methods for each config item
a “requires” property, the registered Ext.Loader’s pre-processor will automatically load these dependencies prior to creating the class.
a “statics” property, the “statics” pre-processor will make all properties of that object static members of the class
Similarly, class “post-processors” are hooks that execute as soon as the class is ready. Some useful “post-processors” introduced in Ext JS 4 are:

“singleton”, which creates a single instance of that class right after it’s created.
“alternateClassName”, which aliases that class to other names, either for convenience or backwards compatibility
“alias”, which generates xtypes and mapping for convenient shorthands.
With this design, the new class system is not only rich in features but also highly extensible. You can create your own “hooks” with ease and tap into any point of the class creation process.

Pre-processors
Class Statics
Static members of a class are declared inside the special statics object property. The scope (this) of static methods defaults to a reference to the class itself. For example:

// My/sample/Point.js
Ext.define('My.sample.Point', {
    statics: {
        fromEvent: function(e) {
            return new this(e.pageX, e.pageY);
        }
    },
 
    x: 0,
 
    y: 0,
 
    constructor: function(x, y) {
        this.x = x;
        this.y = y;
    }
});
 
// app.js
/* … */
var point = My.sample.Point.fromEvent(e);
Class Inheritance
To make class A inherit from class B, add an extend property with value 'B' when defining class A.

// My/sample/Developer.js
Ext.define('My.sample.Developer', {
    extend: 'My.sample.Person', // Will automatically load My.sample.Person
                                // from file: My/sample/Person.js
                                // if it has not been loaded before
 
    code: function(language) {
        alert(this.name + ' is coding in ' + language);
    }
});
 
// app.js
var tommy = new My.sample.Developer('Tommy');
 
tommy.walk(5);              // alerts "Tommy is walking 5 steps"
tommy.code('JavaScript');   // alerts "Tommy is coding in JavaScript"
As you may immediately notice, My.sample.Person is wrapped inside a string. The same concept applies from here on. By always using string class names, you never have to worry about handling dependencies since Ext.Loader will always dynamically load them if they don’t exist. That’s a huge benefit from just 2 extra characters (quotes).

Overridden methods from the parent class can be invoked using this.callParent() inside the overridding methods. On the above example, let’s override the walk() method of class My.sample.Person with a little fun logic:

// My/sample/Developer.js
Ext.define('My.sample.Developer', {
    extend: 'My.sample.Person'
 
    code: function(language) { /* … */ },
 
    walk: function(steps) {
        if (steps > 100) {
            alert("Are you serious? That's too far! I'm lazy…");
        }
        else {
            return this.callParent(arguments);
        }
    }
});
 
// app.js
var tommy = new My.sample.Developer('Tommy');
 
tommy.walk(50);    // alerts "Tommy is walking 50 steps"
tommy.walk(101);   // alerts "Are you serious? That's too far! I'm lazy…"
Notice that we are passing whatever arguments provided in the walk() method to the parent class walk() method. If you need to modify these arguments or pass different arguments, simply replace it with an array, for example: this.callParent([50])

Class Mixins
Mixins are common and reusable groups of features that can be shared among other classes. They are best described as “abilities”. In Ext JS and Sencha Touch, mixins are simply classes. Any class can be used as a mixin. Some good examples of mixins:

Identifiable mixin which gives any target class a getId() method to retrieve a unique ID.
Observable mixin which enable events on any target class.
Traversable mixin which bring tree-like API to any target class.
A class can have more than one mixin. To make class A mix-in class B and C, add a mixins property when defining class A with a value of an object describing B and C.

To illustrate the benefits of mixins, let’s create 3 classes representing different abilities a Person can do, namely:

CanSing
CanPlayGuitar
CanComposeSongs
Afterwards, we will create a CoolGuy class, and a Musician class. A CoolGuy both CanSing and CanPlayGuitar. A Musician @CanSing@, CanPlayGuitar and additionally, CanComposeSongs as well.

// My/sample/CanSing.js
Ext.define('My.sample.CanSing', {
    sing: function(songName) {
        alert("I'm singing " + songName);
    }
});
 
// My/sample/CanPlayGuitar.js
Ext.define('My.sample.CanPlayGuitar', {
    playGuitar: function() {
        alert("I'm playing guitar");
    }
});
 
// My/sample/CanComposeSongs.js
Ext.define('My.sample.CanComposeSongs', {
    composeSongs: function() {
        alert("I'm composing songs");
 
        return this;
    }
});
 
// My/sample/CoolGuy.js
Ext.define('My.sample.CoolGuy', {
    extend: 'My.sample.Person',
    mixins: {
        canSing: 'My.sample.CanSing',
        canPlayGuitar: 'My.sample.CanPlayGuitar'
    }
});
 
// My/sample/Musician.js
Ext.define('My.sample.Musician', {
    extend: 'My.sample.Person',
    mixins: {
        canSing: 'My.sample.CanSing',
        canPlayGuitar: 'My.sample.CanPlayGuitar',
        canComposeSongs: 'My.sample.CanComposeSongs'
    }
});
 
// app.js
var nicolas = new My.sample.CoolGuy("Nicolas");
nicolas.sing("November Rain"); // alerts "I'm singing November Rain"
Notice that our mixins are specified in key-value pairs. The key is used to reference back overridden mixin methods. For example:

// My/sample/CoolGuy.js
Ext.define('My.sample.CoolGuy', {
    extend: 'My.sample.Person',
    mixins: {
        canSing: 'My.sample.CanSing',
        canPlayGuitar: 'My.sample.CanPlayGuitar'
    },
 
    sing: function() {
        alert("Attention!");
 
        // this.mixins is a special object holding references to all mixins' prototypes
        return this.mixins.canSing.sing.apply(this, arguments);
    }
});
 
// app.js
var nicolas = new My.sample.CoolGuy("Nicolas");
nicolas.sing("November Rain"); // alerts "Attention!"
                               // alerts "I'm singing November Rain"
Again we used string class names for mixins, so the classes will automatically be loaded when necessary.

Class Configuration
Note: this is a new feature of the upcoming Ext JS 4.1+ and Sencha Touch 2.0+. Ext JS 4.0 does not make use of this feature.

Declared via the "config" object, properties of this object describe the public API of a class. Each config item will have its own setter and getter method automatically generated inside the class prototype during class creation time, if the class does not have those methods explicitly defined.

As an example, let’s convert the name property of our Person class to be a config item, then add extra age and gender items.

Ext.define('My.sample.Person', {
    config: {
        name: 'Mr. Unknown',
        age: 0,
        gender: 'Male'
    },
 
    constructor: function(config) {
        this.initConfig(config);
 
        return this;
    }
 
    /* … */
});
Within the class, this.name still has the default value of “Mr. Unknown”. However, it’s now publicly accessible without sacrificing encapsulation, via setter and getter methods.

var jacky = new Person({
    name: "Jacky",
    age: 35
});
 
alert(jacky.getAge());      // alerts 35
alert(jacky.getGender());   // alerts "Male"
 
jacky.walk(10);             // alerts "Jacky is walking 10 steps"
 
jacky.setName("Mr. Nguyen");
alert(jacky.getName());     // alerts "Mr. Nguyen"
 
jacky.walk(10);             // alerts "Mr. Nguyen is walking 10 steps"
Notice that we changed the class constructor to invoke this.initConfig() and pass in the provided config object. Two key things happened:

The provided config object when the class is instantiated is recursively merged with the default config object.
All corresponding setter methods are called with the merged values.
Beside storing the given values, thoughout the frameworks, setters generally have two key responsibilities:

Filtering / validation / transformation of the given value before it’s actually stored within the instance.
Notification (such as firing events) / post-processing after the value has been set, or changed from a previous value.
By standardize this common pattern, the default generated setters provide two extra template methods that you can put your own custom logics into, i.e: a “applyFoo” and “updateFoo” method for a “foo” config item, which are executed before and after the value is actually set, respectively. Back to the example class, let’s validate that age must be a valid positive number, and fire an 'agechange' if the value is modified.

Ext.define('My.sample.Person', {
    config: { /* … */ },
 
    constructor: { /* … */ }
 
    applyAge: function(age) {
        if (typeof age != 'number' || age &lt; 0) {
            console.warn("Invalid age, must be a positive number");
            return;
        }
 
        return age;
    },
 
    updateAge: function(newAge, oldAge) {
        // age has changed from "oldAge" to "newAge"
        this.fireEvent('agechange', this, newAge, oldAge);
    }
 
    /* … */
});
 
var jacky = new Person({
    name: "Jacky",
    age: 'invalid'
});
 
alert(jacky.getAge());      // alerts 0
 
alert(jacky.setAge(-100));  // alerts 0
alert(jacky.getAge());      // alerts 0
 
alert(jacky.setAge(35));    // alerts 0
alert(jacky.getAge());      // alerts 35
In other words, when leveraging the config feature, you mostly never need to define setter and getter methods explicitly. Instead,“apply*” and “update*” methods should be implemented where neccessary. Your code will be consistent throughout and only contain the minimal logic that you actually care about.

When it comes to inheritance, the default config of the parent class is automatically, recursively merged with the child’s default config. The same applies for mixins.

Class Dependencies
It is common that a class makes use of other classes inside its member methods. To let Ext.Loader know the list of extra dependencies, simply put them inside the requires array property. For instance:

Ext.define('My.sample.Person', {
    requires: ['My.sample.Validator', 'My.sample.Formatter'],
 
    /* … */
 
    applyAge: function(age) {
        if (!My.sample.Validator.validateAge(age)) {
            console.warn("Invalid age, must be a positive number");
            return;
        }
 
        return age;
    },
 
    applyName: function(name) {
        return My.sample.Formatter.capitalize(name);
    }
 
    /* … */
});
Post-processors
Singleton
In Ext JS, a singleton is a single global instance of a class. To create that single instance right after the class is created and assign the namespace to that object, set the singleton property to true, for example:

Ext.define('My.sample.Validator', {
    singleton: true,
 
    validateNumber: function(number) {
        return typeof number == 'number';
    }
 
    validateAge: function(number) {
        return this.validateNumber(number) && number >= 0;
    }
 
    /* … */
});
 
alert(My.sample.Validator.validateNumber('invalid')); // alerts false
Note that the value of My.sample.Validator is now an instance object, not a class. To get back the reference to its class, use Ext.getClass, for example:

var ValidatorClass = Ext.getClass(My.sample.Validator);
Alternative Class Names
Sometimes you need to alias your class to some other alternative names, either for convenience or backwards compatibility. The alternateClassName property is used just for that. The value can either be a string for a single name, or a string array for multiple names. For example:

Ext.define('My.sample.form.Field', {
    alternateClassName: ['My.FormField', 'My.sample.FormField']
 
    /* … */
});
 
// My.sample.form.Field, My.FormField and My.sample.FormField now reference the same class
Wrap up
The Sencha Class System is a sophisticated way to use objected oriented techniques in the Sencha JavaScript frameworks. It’s a stable platform today to build your own applications and for you to use when working with the Sencha frameworks. We’re continuing to evolve the class system with more features to automate repetitive tasks during your day to day coding mission, and we’re excited to evolve it to make it easier than ever to develop with. As a bonus point, the class system is designed to work with popular server-side JavaScript shells out-of-the-box, such as with Node.js and JSDB.

