# JavaScript Workshop Notes and Examples


    http://nodejs.org/
    http://google.com/chrome
    http://www.sublimetext.com/

    https://tutsplus.com/course/improve-workflow-in-sublime-text-2/
    
## Value & Identity

### Irreducible Elements

#### Primitive values

JavaScript only has five primitive types.

* Numbers
* Booleans
* Strings
* Null
* undefined

The primitive values are represented literally, meaning the symbol and its value are the same. What you type into the console/repl is what you'll get back.

    3.14
    true
    'a sequence of characters'
    null
    undefined


Primitives are immutable. A primitive value is what it is. You can't change it.

    // Each of these causes an error:
    // ReferenceError: Invalid left-hand side in assignment
    null = undefined
    '' = 123
    1 = true
    true = false


#### Objects

Objects are collections of named values that we can refer to as a single entity.
 
    { a: 1, b: 'b', c: true , d: {} }


They have constructors.

  new Object()

Basic JavaScript objects have a literal representation `{}`, which is a shortcut for using the above constructor.

    {}


Everything in JavaScript that's not a primitive value is an object of some kind. Many specialized kinds of objects have their own literal representations.

    new Array()
    []

    // Dates in JavaScript are actually numbers representing
    // the number of milliseconds since January 1, 1970 at 00:00,
    // aka Unix epoch time.
    new Date()

    new RegExp()
    /\s/

    new Function()
    (function () {})


Excepting dates, literals are usually the most convenient way to create these types of objects. You probably shouldn't ever use the `Function` constructor. Here's a [discussion](http://stackoverflow.com/questions/3026089/legitimate-uses-of-the-function-constructor) of when you might want to use it versus the literal.

An example of a function created with the `Function` constructor

    (new Function(['x'], 'return x;'))(123)


Objects are collections of properties, and everything that's not a primitive value is an object, so specialized objects like functions also have properties. 

    function fn (x) {

    }

    fn.name
    fn.length           // number of declared arguments
    fn.constructor


    // objects are mutable
    fn.foo = 'bar'



#### Primitives vs Objects

Primitive values mimic objects in some ways. They have object-like properties which are immutable. 
    'a string'.length

  
    // This looks like it worked, but when you inspect `true.foo` afterwards,
    // you can see it hasn't changed.
    true.foo = 'bar' 

They are made from constructor-like functions, but these functions are not constructors (notice there's no `new` keyword).

    Boolean('false')
    Boolean(1)
    Boolean true

    Number('123.45')

    // If we try to use this as a constructor, we don't get
    // the desired result
    new Number(123.45)



Primitives and objects are stored differently in memory. Primitive values live in memory on a data structure called the stack. They have a constant size, making the stack an efficient structure to use. 

Objects live on a different data structure, called the "heap". The heap is well suited to values that may change in size over time. You can [inspect the heap](https://developers.google.com/chrome-developer-tools/docs/heap-profiling) with Chrome's profiler.

There's an [excellent discussion](http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap) of the stack vs the heap on [stackoverflow.com](http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)


#### Identity

Variables are programmer-defined symbols bound to a memory address on the stack. We use them to refer to values stored on the stack. They're mutable, meaning the values they refer to can be changed over time. They're also dynamically typed, meaning when we change their value, that value can be of a different type.

Primitive values are stored directly on the stack, but remember, objects are not. Objects are assigned to variables by way of a reference to their memory address on the heap (that address is a primitive value). 

All data is passed by value in JavaScript, meaning that it is copied. However, with objects, the value copied is not the object itself, but the location of the object on the heap.  

Some examples help to illustrate:

    // the value is copied from a to b
    var a = 1
      , b = a;

    a === b


    // a primitive value is what it is, whether it's copied or not
    var a = 1
      , b = 1;

    a === b


    // the value copied from `a` to `b` is a reference
    var a = {}
      , b = a;

    a === b


    // These are two separate objects, both empty, each stored separately.
    // on the heap. They are not identical objects, even though they look 
    // the same.

    var a = {}
      , b = {};

    a === b


This distinction is important to remember when passing objects as arguments or copying them. Here is an example of passing an object as an argument. Notice that we're mutating the global variable inside the function.

    // Passing an object as an argument to a function

    var object = { foo: 'bar' };

    function fn(obj) {
      // don't do this,  it's better to return a 
      // new value and mutate the object outside 
      // the function
      obj.foo = 'changed';
    }

    fn(object)

    object.foo

Consider the implications of stack vs heap for copying objects. Suppose we have a nested object `o`:

    var o = {
      a: 1,
      b: {
        foo: 'bar'
      }
    }


We want to copy it and call it `p`:

    var p = {};

    for (key in o) {
      p[key] = o[key];
    }

Now let's mutate the nested object in `p`.

    // apply the function defined in the argument passing example
    fn(p.b)

Notice that this also changed the nested object in `o`. 

    p.b.foo
    o.b.foo

That's because we did a shallow copy. A deep copy would have recursively copied objects all the way through the tree-like structure of the object.

The above examples are for illustration only. If you need to copy objects, and you're using a library in your program like jQuery or Underscore, you'll want to take advantage of the utilities they provide for copying objects. 

    http://api.jquery.com/jQuery.extend/
    http://docs.angularjs.org/api/angular.copy

    // most "frameworks" provide a collection of utilities that solve these
    // kinds of problems for you. Know your lib or framework and take advantage
    // of it. 


If performance doesn't matter and you're not using another library, you can "cheat" by converting your data from JavaScript to JSON and back:

    var q = JSON.parse(JSON.stringify(o))
    q.b === q.p

    // Note that JSON is only available natively in modern environments. If you need
    // to support older browsers, be sure to load the JSON library on your page.
    // https://github.com/douglascrockford/JSON-js


## Evaluating Expressions

All expressions evaluate to a value. Compound expressions are evaluated by a series of substitutions, or reductions, until they reach an irreducible value. For example:

    ((1 + 1) / Math.sqrt(3)) + (4 * Math.pow(6, 3))
    => ((1 + 1) / Math.sqrt(3)) + (4 * 216)
    => ((1 + 1) / Math.sqrt(3)) + 864
    => ((1 + 1) / 1.7320508075688772) + 864
    => (2 / 1.7320508075688772) + 864
    => 1.1547005383792517 + 864
    => 865.1547005383793


[Mozilla Developer Network]() has excellent an excellent [JavaScript reference](), with good [coverage of operators](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Operators/Operator_Precedence). You can control order of evaluation with parentheses. Be aware of operator precedence, associativity, and "short-circuiting."


JavaScript evaluation is "eager", meaning that expressions are evaluated as soon as they occur in a program. By constrast, lazy evaluation would wait to evaluate an expression until its value is needed. 

There is an excellent discussion of this concept in [Structure and Interpretation of Computer Programs, MIT Press](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-10.html#%_sec_1.1.5). In this book, eager and lazy evaluation are referred to as "normal" and "applicative" order.


    function f(x) {
      // do something with x
    }

    // In JS, the Math.sqrt expression would be evaluated 
    // before the function is invoked.
    f(Math.sqrt(9))



## Representing Processes with Functions

> A computational process is indeed much like a sorcerer's idea of a spirit. It cannot be seen or touched. It is not composed of matter at all. However, it is very real. It can perform intellectual work. It can answer questions. It can affect the world by disbursing money at a bank or by controlling a robot arm in a factory. The programs we use to conjure processes are like a sorcerer's spells. They are carefully composed from symbolic expressions in arcane and esoteric programming languages that prescribe the tasks we want our processes to perform.
small Excerpt from <a href='http://mitpress.mit.edu/sicp/full-text/book/book.html' target='_blank'><cite>SICP</cite></a>



### What is a function?
 
Functions are the "verbs" of programming. In the same way that we use variables to assign meaning to values, we use functions to define a vocabulary of <em>processes</em> within a program. Any process we can imagine can be abstracted with a function and then referred to by name. This idea is neither new nor exclusive to computer programming. We already do it in the "real world" every day.

For example, when we add the task "grocery shopping" to our todo list, we're referring to a complex real-world <em>process</em> that we've <em>abstracted by giving it a name</em>. It might mean something like this:

ol 
  li Enter the store. 
  li Obtain a shopping cart. 
  li Shop for produce
  ul
    li Get peppers
    ul
      li Move to peppers
      li Choose peppers
      ul
        li Extend arm
        li Grasp a pepper
        li Retract arm
        li Inspect for blemishes
        li Add to cart
  li ... 

Each of these list items is also a named process that "encapsulates" a series of steps which, in turn, might also name and hide another series of steps... right down to "extend your arm, grasp an object, retract your arm, release the object", and so on. 

Programming with functions works just the same. We can name a calculation or series of instructions, and then treat that named process like a black box. Instead of thinking through every minute action, we say to ourselves "Get peppers", and the details take care of themselves. In other words, functions allow us to "zoom out" and get a birds-eye view of a process. Composing processes with functions enables us to express increasingly complex ideas with fewer explicit parts. 


### Functions in Practice

> A function encloses a set of statements. Functions are the fundamental modular unit of JavaScript. They are used for code reuse, information hiding, and composition. Functions are used to specify the behavior of objects. Generally, the craft of programming is the factoring of a set of requirements into a set of functions and data structures.
  <small><cite>JavaScript, The Good Parts by Douglas Crockford</cite></small>


#### Defining Functions

We have to define functions before we can use them. Here's the definition of a simple JavaScript function that calculates the square of a given number.

    function square (n) {
      return n * n;
    }

The first line of a function is called its <em>signature</em>. The definition starts with the <code>function</code> keyword, followed by the name <code>square</code> and a parenthesized list of <em>arguments</em>, sometimes called <em>parameters</em>. In this example, we have a single argument <code>n</code>. The <em>body</em> of the function is like a block statement, delimited by curly braces, the same as conditionals and loops. 

The body of a function can contain any number of statements. Within the body of <code>square</code> is single special statement called a <em>return</em> statement. A return statement specifies what value the function will evaluate to when we <em>call</em> it. It's customary to use only one return statement per function and place it at the very end of the body. There are some useful exceptions to this rule, but it serves well as guideline.

Note that functions always return value. A JavaScript function without a return statement will return <code>undefined</code> by default.

h5 Calling Functions
p So far, we've only defined the function. Now let's <em>call</em> it (sometimes referred to as <em>invoking</em> or <em>applying</em>).

    square(2)

Try calling <code>square</code> again with some different values and notice the results.

    square(4)
    square(16)
    square(9)
    square(3.14)

We can assign the value returned by function to a variable, like any other value. 

    var squareOfNinety = square(90);

We can also call functions within expressions, where they act as parameterized placeholders for values we don't yet know.

    1000 + square(7)
    var n = false || square(43)


We can even call a function within a function invocation:

    square(square(4))


#### Composing Processes with Functions

Suppose we need to perform the following calculation in a program:

    (4 * 4) + (5 * 5)

This expression shows the literal values <code>4</code> and <code>5</code> along with the explicit procedure for performing the calculation. The expression doesn't tell us what the resulting value means. We can't reuse the <em>structure</em> of the expression with different values without rewriting the whole thing.

Notice that the parenthesized multiplications are actually calculating the square of 4 and 5 respectively. We can improve this expression by using variables instead of values.

    var x = 4, y = 5;
    (x * x) + (y * y)

This clarifies the intent of the code somewhat by showing that we're multiplying a two values by themselves and adding the results together. We can clarify it further by using the <code>square</code> function we created earlier.

    square(4) + square(5)


If summing squares is something we need to do repeatedly in our program, or we want to be very clear about the meaning of the expression, we can create another function.

    function sumOfSquares (x, y) {
      return square(x) + square(y);
    }

Notice that we're calling the function <code>square</code> within the body of another function, <code>sumOfSquares</code>. This shows one way that functions are used to compose a larger process.

Now, instead of explicitly writing the calculation <code>(4 * 4) + (5 * 5)</code>, we can simply call our new function.

    sumOfSquares(4, 5)

The JavaScript interpreter will evaluate this function and arrive at the resulting value through a series of reductions, like so (read <code>=></code> as "evaluates to"):

    sumOfSquares(4, 5)
    => square(4) + square(5)
    => square(4) + (5 * 5)
    => square(4) + 25
    => (4 * 4) + 25
    => 16 + 25
    => 41


#### Functions <em>return</em> values and they <em>are</em> values!

In JavaScript, functions not only <em>return</em> values, <strong>they <em>are</em> values</strong>. Type the following examples into the console/repl and notice the difference between the value <em>of</em> the function versus the value <em>returned</em> by the function.

    square
    square(2)
    sumOfSquares
    sumOfSquares(3)


There are several interesting consequences of functions being "first-class" values. First, we can pass a function as an argument to another function.

    function sumOfWhatevers (a, b, whatever) {
      return whatever(a) + whatever(b);
    }


This lets us change some part of a function at the time it's called. For example, suppose that in addition to <code>square</code>, we also have a function <code>cube</code>. 

    function cube(n) {
      return n * n * n;
    }

We can now call <code>sumOfWhatevers</code> with either function as the last argument.

    sumOfWhatever(6, 7, square)
    sumOfWhatever(6, 7, cube)

Another consequence of first-class functions is that we don't have to define them in order to use them. A function without a name is called an <em>anonymous</em> function. We can assign anonymous functions to variables, use them literally within other expressions, and even pass them literally as arguments.

    sumOfWhatevers(6, 7, function (n) {
      return Math.pow(n, 5);
    })


You'll see anonymous functions used everywhere in JavaScript.

    // in jQuery
    $('.stuff').each(function (item) {
      // 
    });

    // with Arrays
    [...].forEach(function (item) {
      //
    });

    // in Node.js
    app.get('/', function (req, res) {
      res.send(req.params)
    });


.alert.alert-info
  h5
    i.icon-info-sign
    |  Math.pow()
  p JavaScript has some built in objects like <code>Math</code> that provide us with convenient helper functions, like <code>pow()</code>. You can learn more about them in the MDN JavaScript Guide.

A third consequence of functions being first-class values is that they can be return values of other functions. This gives us yet another way to use functions as building blocks. Let's create some functions this way.

    function power(x) {
      return function (n) {
        return Math.pow(n, x);
      }
    }
    
    var toFourth = power(4)
      , toFifth = power(5);
    
    toFourth(2)
    toFifth(2)
    // etc...


Functions as arguments or return values are known as <em>higher-order</em> functions. 


#### Arguments are flexible. 

As long as their ommission doesn't cause an error inside the function, arguments are optional.

    function stringing (q, r, s) {
      return q + r + s;
    }
    
    stringing('q', 'r'); 

JavaScript doesn't provide a way to declare default arguments, but we can take advantage of them being optional to get the same effect. 

    function foo (a, b) {
      var c = b || true;
      return c;
    }

A function can also be called with undeclared arguments. The <code>arguments</code> object, available inside any function, gives us array-like access to whatever is passed in when the function is called.

    function sumOfArguments () {
      var result = 0;
      for (i = 0; i < arguments.length; i++) {
        result += arguments[i];
      }
      return result;
    }
    
    sumOfArguments(1,1,2,3,5,8,13);




### What about `this`?

JavaScript provides every function with a local variable called `this`. It can mean any number of things, depending on how the function is invoked, and is one of the most confusing aspects of the language.

The value of `this` is not bound when the function is defined, but when the function is invoked.

#### Standalone functions

When invoked in the simplest way, as a standalone function, the local variable `this` is a reference to the global object. 

    var q = 'qwerty';

    function tryThis () {
      var q = 'notqwerty';
      return {
        global: this.q,
        local: q
      }
    }


    // `this` is also the global in 
    // nested functions
    function outer() {
      var q = 'notqwerty';

      function inner() {
        return this.q;
      }

      return inner();
    }

    outer();


That is more of a coincidence than an intentional language feature. `this` doesn't really have a good purpose in this context. 

JavaScript has a "[strict mode](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Functions_and_function_scope/Strict_mode)" that changes (improves?) the semantics of the language. In strict mode, `this` inside a standalone function is `undefined` instead of the global. 




#### Method invocation

When a function is invoked as a method on an object, `this` refers to that object.
 

    var dog = {
      greeting: 'woof',
      bark: function () {
        return this.greeting;
      }
    }



    // nested functions get the global (or undefined)
    var dog = {
      greeting: 'woof',
      greet: function () {
        var that = this;

        function growl() {
          console.log(that.greeting);
        }

        function bark() {
          console.log(that.greeting);
        }

        growl()
        bark()
      }
    }

    dog.bark()


    // `this` depends on the context the function is 
    // called in, not the the context it's defined in.
    var bark = dog.bark;
    bark()


#### Constructors

A constructor is a special kind of function that's used like a template for creating objects. Constructors are invoked with the `new` operator, and that operator changes the behavior of the function. Among other things, `new`:

* sets the value of `this` to an empty object, which is the starting point for the object under construction
* causes the constructor to implicitly return `this`

       
    function Dog(greeting) {
      this.greeting = greeting;
    }

    new Dog('woof')


#### Call/apply

Call and apply methods let us explicitly define `this` when invoking a function.


    function foo(a,b,c) {
      this.a = a;
      // ...
      return this;
    }

    foo.apply({}, [1,3,4])
    foo.call({}, 1, 2, 3)



### Function Scopes and Closure

The main context of a program is called <em>global scope</em>. Variables and functions declared in global scope can be referred to from anywhere in the program. Once they're declared, they exist in memory for as long as the program is running. 

When a function is defined in JavaScript, a new program context called a <em>local scope</em> is established for the function. A local scope is temporarily brought to life when we call its function, like a program within a program. Although we can access variables in global scope from inside the function, any arguments or variables we've declared within the function definition are hidden from the rest of the program. When the function returns, all the values existing within its scope are forgotten (released from memory). 

    // This is global scope.
    var message = 'Hello.';
    
    function greeting () {
      // Statements here are evaluated in the local scope.
    }



    // Note: the global scope is actually an object that's made available by
    // your program's runtime. In Node.js it's called `global` and in the browser
    // it's called `window`.

The keyword <code>var</code> has a special relationship to scope. If we declare a variable inside a function without using <code>var</code>, that variable will be created in global scope. Doing this will almost certainly lead to bugs in a sizable program. 

    // don't do this!
    function visitWashington () {
      scandal = 'Everyone knows...';
    }
    
    visitWashington();
    console.log(scandal);   
  
When we call this function, it will initialize or modify a global variable <code>scandal</code> instead of a local one. To keep your local variables local, be sure to use <code>var</code>:

    function visitVegas () {
      var secret = 'What happens in Vegas, stays in Vegas...';
    }        
    
    visitVegas();
    console.log(secret);      // this will cause a Reference Error

This is why we really need to explicitly declare <em>all</em> variables using <code>var</code>. 

A function's arguments are part of its local scope and follow the same rules as local variables. 

We can create a "chain" of scopes, by nesting functions. In the next example, notice that the inner function has access to the outer function's arguments and variables.

    var g = 'I belong to the global scope';
    
    function outer(n) {
    
      function inner(m) {
        console.log(g);
        console.log(n);
        console.log(m);
      }
    
      inner('I belong to the inner function');
    }
    
    outer('I belong to the outer function');



When the <code>console.log(g)</code> statement in the <code>inner</code> function is evaluated, the interpreter looks first in its own scope, then in the "parent" scope (the <code>outer</code> function), then in the global scope. If <code>g</code> didn't exist in the global scope, this statement would cause a reference error.

Note that although we can directly refer to global variables inside a function, this is a bad idea. We should almost always try to write functions that are "self-contained". If we need to provide values to the function from the outside world, it's best to pass them as arguments. This makes it easier to verify the correctness of the function.

If a global and local variable have the same name, the local variable will always be the one evaluated.


<div class="alert">
  <h5>Curly braces do not create scope</h5>
  </p>If statements and loops do not create new scope within their block statements. In JavaScript, only functions can create new scope.</p>
</div>



#### Closure

p Suppose we want to "remember" the values of all the variables and arguments in a function's scope chain, even after it has returned. There's a way to do this called <em>creating a closure</em>. To create a closure in JavaScript, we need to return a function from another function. 

    function power(x) {
    
      // Any values available in this scope will be 
      // available in the returned function, even after 
      // `power` returns.          
    
      return function (n) {
        // This function "closes over" its scope chain.
        // As long as it exists outside `power`, it will
        // keep track of `power`'s variables and arguments
        // from this invocation.
        return Math.pow(n, x);
      }
    }          

Let's see it in action.

    var square = power(2);

Now the variable <code>square</code> is assigned the function returned by <code>power</code>, and it knows that we passed 2 as <code>x</code>. When we call square, it now returns <code>n</code> to the second power.

    square(4);


#### Use of Closure

Simulating private variables.

    // not exactly a secret
    var ceo = {
      secret: 'dirty tricks',
      salary: 'too much',
      tell: function (who) {
        if (who !== 'auditor') {
          console.log(this.secret);
        }
      }
    }

    // this hides `secret`, but you can still access it
    // from methods on the returned object

    var ceo = (function () {
      var secret = 'dirty tricks';

      return {
        salary: 'too much',
        tell: function (who) {
          if (who === 'CFO') {
            return secret;
          }
        }
      }
    })()

Self invoking functions can be used to create create scopes. If you return another function (standalone or as a member of an object) you've got a closure:

    
    (function () {
      // whatever state you create here will be available in any returned functions

      return {
        method: function () {}
      }
    })()


Closures can be used for creating modules in the browser, and for namespacing. 

    // EXAMPLES ON THE WAY.


They can also be used to create "singleton" objects, or objects that only get created once no matter how many times a constructor is called.

    function OneAndOnly() {
      var that = this;
      that.foo = 'bar';

      OneAndOnly = function () {
        return that;
      }

      return that;
    }


    var a = new OneAndOnly()
    var b = new OneAndOnly()

    // check that they are the same
    a === b



#### Recursion

    // RangeError: Maximum call stack size exceeded
    function r() {
      r()
    }


    // conditionally recurse
    function r(list) {
      console.log(list.shift());
      if (list.length > 0) {
        r(list)
      }
    }



    // Here's an example of using recursion to walk a directory tree and 
    // build an array of relative file paths.
    //
    // https://github.com/christiansmith/notch/blob/master/lib/load.js#L59

    function walk (dir) {
      if (fs.statSync(dir).isDirectory()) {
        var listings = fs.readdirSync(dir)
          , files = [];

        listings.forEach(function (listing) {
          var file = path.join(dir, listing)
            , stat = fs.statSync(file);

          if (stat.isFile()) {
            files.push(file);
          } else if (stat.isDirectory()) {
            files = files.concat(walk(file))
          }
        });
      } else {
        var files = [dir];
      }

      return files;
    }



### Managing complexity with functional style

Imperative programming style tends to emphasize machine instruction and be:

* explicit and opaque
* verbose
* error prone
* difficult to test
* unpredictable (not idempotent)

Declarative and functional styles, in contrast, emphasizes _evaluation_, and tends to 

* imply "how" and express meaning ("what")
* be concise
* be referentially transparent and easily verifiable

#### Pure functions vs side effects

"Pure" functions take parameters and return values, but never directly read or mutate program state outside the function. This gives them the property of _idempotence_. Every time you invoke the function with the same arguments you will get the same result. 
    // don't do this
    var thing = {
      foo: 'bar'
    }

    function action() {
      return thing.foo + '!';
    }

    action()
    thing = false;      // now your function won't work as expected




    // instead, take an argument
    function action(msg) {
      return msg + '!';
    }

    action(thing.foo)



    // DEFINITELY DON'T DO THIS
    var thing = {
      foo: 'bar'
    }

    function action() {
      thing.foo = 'wonky';
    }


    // instead, return a value
    function action() {
      return 'cool';
    }

    thing.foo = action();



#### Do one thing/have one goal


#### Avoid writing explicit instructions, try to be "declarative"

    var list = [1,2,3];

    // instead of this
    for (var i = 0; i < list.length; i++) {
      var item = list[i];
      // do something with item
    }

    // do this
    list.forEach(function (item) {
      // do something with item
    })

Another real world examples of this principle is the notion of _data-binding_ vs _DOM manipulation_.



#### Reading on Functional Programming (This list will grow, check back)

* SICP
* Out of the Tar Pit
* [Examples of FP in JS](http://osteele.com/sources/javascript/functional/)




#### Map/reduce and other functional style tools

* http://underscorejs.org/
* http://lodash.com/


Read the annotated source!

http://underscorejs.org/docs/underscore.html


You can use this in the browser or in Node.js.

    $ npm install underscore


Some examples:

    var list = [1,2,3,4,5];

    // explicitly generate another array
    var newList = [];
    for (var i = 0; i < list.length; i++) {
      newList.push(list[i] * 10);
    }


    // use underscore instead...
    _.map(list, function(item) {
      return item * 10; 
    });


    _.reduce(list, function(subtotal, num) {
      return subtotal + num; 
    }, 0);


    _.filter(list, function (n) { 
      return n < 5 && n > 1; 
    });


    _.reject(list, function (n) { 
      return n < 5 && n > 1; 
    });





### Describing Things with Objects

Together with arrays, we can use JavaScript objects to organize the data inside our programs. 

#### A literal object, with no properties

    {}


#### An object and its properties

    var dog = {
      name: 'Maverick',
      breed: 'German Shepherd',
      weight: 90,
      likesSquirrels: true,
      eatsChicken: true
      favoriteToys: ['tennis ball', 'stuffed squirrel']
    };


#### Getting and setting values with dot notation

    dog.name
    dog.likesSquirrels
    dog.favoriteToys[1]
    
    dog.weight += 2
    dog.favoriteToys.push('stick')
    
    dog.hungry = true
    dog.hungry = false

#### Getting and setting values with bracket notation

    dog['name']
    dog['weight'] -= 2

As we'll see in the next section, this syntax is mostly used when we don't know the key ahead of time.

    var key = 'hungry';
    dog[key]
    dog[key] = false


#### Iterating over keys and values

    for (var key in dog) {
      console.log(dog[key]);
    }

#### Enumerating properties

    Object.keys(item)

####  Removing a property from an object.

    delete o.foo





### Lists of Things

Very often when working with JavaScript, we'll find ourselves wanting to create, retrieve and manipulate lists of things. We use arrays and objects together to represent them.

We can create such construct literally like so:

    var stocks = [
      { company: 'Apple Computer', symbol: 'APPL', bid: 500.85, ask: 515.32 },
      { company: 'Google', symbol: 'GOOG', bid: 695.42, ask: 698.23 },
      { company: 'Microsoft', symbol: 'MSFT', bid: 12.42, ask: 13.03 }
    ];


Although we can build up such data structures literally, more often we will build them programmatically or get them as the result of some database query or API call.


Let's add two new stocks to our portfolio.

    stocks.push({
      company: 'Tesla Motors',
      symbol: 'TSLA',
      bid: 32,
      ask: 35
    });


Apple Computer changed their name to just Apple a while back. Let's update our data to reflect that.

    var apple = stocks[0];
    apple.company = 'Apple';

We could have done this too:

    stocks[0].company = 'Apple';


Let's look through our portfolio and decide which stocks to buy and which to sell.

    stocks.forEach(function (stock) {
      if (stock.bid > 500) {
        console.log('Take profit on ' + stock.symbol);
      } else {
        console.log(stock.symbol + ' is cheap! Buy more!')
      }
    });


### Inheritance

There is no formal concept of a "class" in JavaScript. Objects inherit directly from other objects. To fully understand this, we need to explore how objects are created.

#### Constructors
 
A constructor is a special function that creates objects. You can define your own constructors. They act like a specification, or a template for new objects. `this` is the starting point.


    function Car (make, model, year) {
      this.make = make;
      this.model = model;
      this.year = year || (new Date()).getFullYear()
      //this.whatever = 'whatever else you want';
    }


Constructors are invoked with the `new` operator. The `new` operator initializes `this` to a new object and causes the constructor to return it.
  
    var jeep = new Car('Jeep', 'Wrangler')
      , truck = new Car('Ford', 'F150', 2011)
      , van = new Car('Mercedes', 'Sprinter', 2012);


Every JavaScript object is made by a constructor. Even the empty object is created by a constructor:

    new Object()


Literal notation is just a shortcut for `new Object()`.

    {}


Every object has a reference to its constructor. 

    var o = {};
    o.constructor


Here's proof of the shortcut.

    o.constructor === Object


Constructors always create a brand new object.

    {} !== new Object()     




  
#### Prototypes

NOTE: Use the Node REPL to try these examples, some may not work in the browser. Also, the use of `__proto__` here is only for illustration. This is provided by Chrome and Node.js for inspecting/debugging, it is not part of the language, and you can't rely on it working everywhere.



Every object in JavaScript has a prototype (which is another object).

    {}.prototype                      // this is undefined
    {}.__proto__                      // this is a reference to the actual prototype


    // This is called the prototype chain
    {}.__proto__.__proto__            // null
    {}.__proto__.__proto__.__proto__  // cannot read property of null


Every function has a prototype property (it's only used with constructors).

    Object.prototype


The constructor is responsible for linking the objects it creates to a prototype.

    var o = new Object()

    // the constructor links `this` to its prototype property.
    o.__proto__ === Object.prototype
    o.constructor.prototype === Object.prototype 

    // with literals
    {}.__proto__ === Object.prototype
    {}.constructor.prototype === Object.prototype 



You can specify an object's prototype.

    // FOR ILLUSTRATION ONLY, DON'T DO THIS
    o.__proto__ = { foo: 'bar' }
    o.__proto__.foo
    o.__proto__.__proto__ = { boo: 'hoo'}



The whole point of prototypes is that objects inherit their prototype's properties through the prototype chain. 

    o.foo    
    o.boo
    o.blah      // undefined


What are the "rules" of prototypes?

First, mutating an inherited property overrides the prototype, directly on the object.

    o
    o.foo = 'baz'
    o
    o.__proto__


Second, mutating the prototype affects all objects that inherit from it.

    var a = {}, b = {}, p = { color: 'blue' };

    a.__proto__ = p;
    b.__proto__ = p;

    a.color
    b.color

    p.color = 'green'

    a.color
    b.color


As instructive as it is to play with __proto__, don't use it in real code. The constructor is the correct place to set up the prototype relationship.

    function Thing() {}

    // replace the prototype entirely
    Thing.prototype = {
      color: 'orange'
    }

    // mutate the prototype
    Thing.prototype.color = 'orange';



So what do we use prototypes for? The most common use is for holding and object's methods. The reason is efficiency. If we define methods directly on `this`, they will be recreated as new functions that do the same thing each time we create a new object with the constructor. By inheriting methods from the prototype, they will only be declared once, no matter how many objects we create with the constructor.


    function Person (name, dob) {
      this.name = name;
      this.dob = dob;
      // Don't define functions inside the constructor
      //this.age = function () {...}
    }

    Person.prototype.age = function () {
      // doesn't take leap year into account
      var millisecondsPerYear = 1000 * 60 * 60 * 24 * 365
        , elapsedMilliseconds = Date.now() - this.dob; 

      return Math.floor(elapsedMilliseconds / millisecondsPerYear);
    }


    var person = new Person('Miles Davis', new Date('May 26, 1926'));
    person.age()



What about `this`? It's bound at invocation, so it refers to the object the method is called from.


How do we know if a property belongs to an object or one of its prototypes?

    // This will give us everything on
    for (var prop in person) {
      console.log(prop);
    }

    // what if we just want direct properties?
    for (var prop in person) {
      if (person.hasOwnProperty(prop)) {
        console.log(prop)
      }
    }



#### Prototypal Inheritance

What if you wanted to inherit directly from an object, without using a constructor?

    var person = {
      say: function (something) {
        console.log(something);
      }
    }


    // Setup the prototype using a throwaway constructor
    Object.create = function (object) {
      function F() {}
      F.prototype = object;
      return new F();
    }


    // optimized implementation
    Object.create = (function () {
      // F is only created once
      function F() {}
      
      return function (object) {
        F.prototype = object;
        return new F();
      };
    })()




#### Classical Inheritance

The idea of classical inheritance is that objects created by one constructor should inherit properties that come from another constructor...

...

