# C++ Style Guide

Consistency is the most important aspect of style. The second most important aspect is following a style that the average C++ programmer is used to reading.

C++ allows for arbitrary-length identifier names, so there's no reason to be terse when naming things. Use descriptive names, and be consistent in the style.

We also provide a clang-format file to automatically ident the code using QtCreator built-in shortcut `Ctrl + i`. To do so you need to import our file under:
 * `Edit` > `Preferences...` > `C++` > `Code Style` > `Import...`

## Platform

 * Language: C++ 17
 * Compiler: GCC
 * Debugger: GDB
 * Build system generator: CMake
 * Build system: Ninja


## Common C++ Naming Conventions

 * Types start with upper case: `MyClass`.
 * Functions and variables start with lower case: `myMethod`.
 * Constants are all upper case: `const double PI=3.14159265358979323;`.
 * Macro names use upper case with underscores: `INT_MAX`.
 * Template parameter names use Pascal case: `InputIterator`.
 * All other names use snake case: `unordered_map`.

## Distinguish Private Object Data

We use the Hungarian notation to include the scope of a variable, separated by an underscore:

 * `g_nWheels`: member of a global namespace, integer
 * `m_nWheels`: member of a structure/class, integer
 * `m_wheels`: member of a structure/class
 * `s_wheels`: static member of a class
 * `c_wheels`: static member of a function

## Distinguish Function Parameters

The most important thing is consistency within your codebase; this is one possibility to help with consistency.

Name function parameters with an `t_` prefix. `t_` can be thought of as "the", but the meaning is arbitrary. The point is to distinguish function parameters from other variables in scope while giving us a consistent naming strategy.


```cpp
struct Size
{
  int width;
  int height;

  Size(int t_width, int t_height) : width(t_width), height(t_height) {}
};

// This version might make sense for thread safety or something,
// but more to the point, sometimes we need to hide data, sometimes we don't.
class PrivateSize
{
  public:
    int width() const { return m_width; }
    int height() const { return m_height; }
    PrivateSize(int t_width, int t_height) : m_width(t_width), m_height(t_height) {}

  private:
    int m_width;
    int m_height;
};
```


## Don't Name Anything Starting With `_`

If you do, you risk colliding with names reserved for compiler and standard library implementation use:

http://stackoverflow.com/questions/228783/what-are-the-rules-about-using-an-underscore-in-a-c-identifier


## Well-Formed Example

```cpp
class MyClass
{
public:
  MyClass(int t_data)
    : m_data(t_data)
  {
  }

  int getData() const
  {
    return m_data;
  }

private:
  int m_data;
};
```


## Use `nullptr`

C++11 introduces `nullptr` which is a special value denoting a null pointer. This should be used instead of `0` or `NULL` to indicate a null pointer.

## Comments

Comment blocks should use `//`, not `/* */`. Using `//` makes it much easier to comment out a block of code while debugging.

```cpp
// this function does something
int myFunc()
{
}
```

To comment out this function block during debugging we might do:

```cpp
/*
// this function does something
int myFunc()
{
}
*/
```

which would be impossible if the function comment header used `/* */`.

## Never Use `using namespace` in a Header File

This causes the namespace you are `using` to be pulled into the namespace of all files that include the header file.
It pollutes the namespace and it may lead to name collisions in the future.
Writing `using namespace` in an implementation file is fine though.


## Include Guards

Header files must contain guard. To be fair Header Guard are probably the most reliable to all possible compilers because they're part of the standard. But since we're using GCC 12 we should be able to use `#pragma once` without any problems making the code shorter and cleaner.

```cpp
// Good Idea
// But we don't use it
#ifndef MYPROJECT_MYCLASS_HPP
#define MYPROJECT_MYCLASS_HPP

namespace MyProject {
  class MyClass {
  };
}

#endif

// Good Idea
// We're gonna use it
#pragma once

namespace MyProject {
  class MyClass {
  };
}

```

You may also consider using the `#pragma once` directive instead which is quasi-standard across many compilers.
It's short and makes the intent clear.


## {} Are Required for Blocks.
Leaving them off can lead to semantic errors in the code.

```cpp
// Bad Idea
// This compiles and does what you want, but can lead to confusing
// errors if modification are made in the future and close attention
// is not paid.
for (int i = 0; i < 15; ++i)
  std::cout << i << std::endl;

// Bad Idea
// The cout is not part of the loop in this case even though it appears to be.
int sum = 0;
for (int i = 0; i < 15; ++i)
  ++sum;
  std::cout << i << std::endl;


// Good Idea
// It's clear which statements are part of the loop (or if block, or whatever).
int sum = 0;
for (int i = 0; i < 15; ++i) {
  ++sum;
  std::cout << i << std::endl;
}
```

## Keep Lines a Reasonable Length

```cpp
// Bad Idea
// hard to follow
if (x && y && myFunctionThatReturnsBool() && caseNumber3 && (15 > 12 || 2 < 3)) {
}

// Good Idea
// Logical grouping, easier to read
if (x && y && myFunctionThatReturnsBool()
    && caseNumber3
    && (15 > 12 || 2 < 3)) {
}
```

Many projects and coding standards have a soft guideline that one should try to use less than about 80 or 100 characters per line.
Such code is generally easier to read.
It also makes it possible to have two separate files next to each other on one screen without having a tiny font.


## Use "" for Including Local Files
... `<>` is [reserved for system includes](http://blog2.emptycrate.com/content/when-use-include-verses-include).

```cpp
// Bad Idea. Requires extra -I directives to the compiler
// and goes against standards.
#include <string>
#include <includes/MyHeader.hpp>

// Worse Idea
// Requires potentially even more specific -I directives and
// makes code more difficult to package and distribute.
#include <string>
#include <MyHeader.hpp>


// Good Idea
// Requires no extra params and notifies the user that the file
// is a local file.
#include <string>
#include "MyHeader.hpp"
```

## Initialize Member Variables
...with the member initializer list.

For POD types, the performance of an initializer list is the same as manual initialization, but for other types there is a clear performance gain, see below.

```cpp
// Bad Idea
class MyClass
{
public:
  MyClass(int t_value)
  {
    m_value = t_value;
  }

private:
  int m_value;
};

// Bad Idea
// This leads to an additional constructor call for m_myOtherClass
// before the assignment.
class MyClass
{
public:
  MyClass(MyOtherClass t_myOtherClass)
  {
    m_myOtherClass = t_myOtherClass;
  }

private:
  MyOtherClass m_myOtherClass;
};

// Good Idea
// There is no performance gain here but the code is cleaner.
class MyClass
{
public:
  MyClass(int t_value)
    : m_value(t_value)
  {
  }

private:
  int m_value;
};

// Good Idea
// The default constructor for m_myOtherClass is never called here, so
// there is a performance gain if MyOtherClass is not is_trivially_default_constructible.
class MyClass
{
public:
  MyClass(MyOtherClass t_myOtherClass)
    : m_myOtherClass(t_myOtherClass)
  {
  }

private:
  MyOtherClass m_myOtherClass;
};
```

In C++11 you can assign default values to each member (using `=` or using `{}`).

### Assigning default values with =

```cpp
// ... //
private:
  int m_value = 0; // allowed
  unsigned m_value_2 = -1; // narrowing from signed to unsigned allowed
// ... //
```
This ensures that no constructor ever "forgets" to initialize a member object.

### Assigning default values with brace initialization

Using brace initialization does not allow narrowing at compile-time.

```cpp
// Best Idea

// ... //
private:
  int m_value{ 0 }; // allowed
  unsigned m_value_2 { -1 }; // narrowing from signed to unsigned not allowed, leads to a compile time error
// ... //
```

Prefer `{}` initialization over `=` unless you have a strong reason not to.

Forgetting to initialize a member is a source of undefined behavior bugs which are often extremely hard to find.

If the member variable is not expected to change after the initialization, then mark it `const`.

```cpp
class MyClass
{
public:
  MyClass(int t_value)
    : m_value{t_value}
  {
  }

private:
  const int m_value{0};
};
```

Since a const member variable cannot be assigned a new value, such a class may not have a meaningful copy assignment operator.

## Always Use Namespaces

There is almost never a reason to declare an identifier in the global namespace. Instead, functions and classes should exist in an appropriately named namespace or in a class inside of a namespace. Identifiers which are placed in the global namespace risk conflicting with identifiers from other libraries (mostly C, which doesn't have namespaces).


## Use .hpp and .cpp for Your File Extensions

Ultimately this is a matter of preference, but .hpp and .cpp are widely recognized by various editors and tools. So the choice is pragmatic.


## Never Put Code with Side Effects Inside an assert()

```cpp
assert(registerSomeThing()); // make sure that registerSomeThing() returns true
```

The above code succeeds when making a debug build, but gets removed by the compiler when making a release build, giving you different behavior between debug and release builds.
This is because `assert()` is a macro which expands to nothing in release mode.


## Class and Struct Differences

Use class if the class has an invariant; use struct if the data members can vary independently The use of class alerts the programmer to the need for an invariant. This is a useful convention.
`N.B.`: An invariant is a logical condition for the members of an object that a constructor must establish for the public member functions to assume. After the invariant is established (typically by a constructor) every member function can be called for the object. An invariant can be stated informally (e.g., in a comment) or more formally using Expects.

Example
```cpp
// Members can vary independently
struct Pair
{
    string name;
    int volume;
};

// Validate in the constructor that {yy, mm, dd} is a valid date
class Date
{
public:
    Date(int yy, Month mm, char dd);
    
private:
    int year;
    Month month;
    char day;
};
```

If a class has any private data, a user cannot completely initialize an object without the use of a constructor. Hence, the class definer will provide a constructor and must specify its meaning. This effectively means the definer need to define an invariant.

## Don't Be Afraid of Templates

They can help you stick to [DRY principles](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself).
They should be preferred to macros, because macros do not honor namespaces, etc.

## Use Operator Overloads Judiciously

Operator overloading was invented to enable expressive syntax. Expressive in the sense that adding two big integers looks like `a + b` and not `a.add(b)`. Another common example is `std::string`, where it is very common to concatenate two strings with `string1 + string2`.

However, you can easily create unreadable expressions using too much or wrong operator overloading. When overloading operators, there are three basic rules to follow as described [on stackoverflow](http://stackoverflow.com/questions/4421706/operator-overloading/4421708#4421708).

Specifically, you should keep these things in mind:

* For all operators, only overload them when they are used in a context that is commonly connected to these operators. Typical scenarios are concatenating things with +, negating expressions that can be considered "true" or "false", etc.
* Always be aware of the [operator precedence](http://en.cppreference.com/w/cpp/language/operator_precedence) and try to circumvent unintuitive constructs.
* Do not overload exotic operators such as ~ or % unless implementing a numeric type or following a well recognized syntax in specific domain.
* [Never](http://stackoverflow.com/questions/5602112/when-to-overload-the-comma-operator?answertab=votes#tab-top) overload `operator,()` (the comma operator).
* Use non-member functions `operator>>()` and `operator<<()` when dealing with streams. For example, you can overload `operator<<(std::ostream &, MyClass const &)` to enable "writing" your class into a stream, such as `std::cout` or an `std::fstream` or `std::stringstream`. The latter is often used to create a string representation of a value.

More tips regarding the implementation details of your custom operators can be found [here](http://courses.cms.caltech.edu/cs11/material/cpp/donnie/cpp-ops.html).

## Avoid Implicit Conversions

### Single Parameter Constructors

Single parameter constructors can be applied at compile time to automatically convert between types. This is handy for things like `std::string(const char *)` but should be avoided in general because they can add to accidental runtime overhead.

Instead mark single parameter constructors as `explicit`, which requires them to be explicitly called.

### Conversion Operators

Similarly to single parameter constructors, conversion operators can be called by the compiler and introduce unexpected overhead. They should also be marked as `explicit`.

```cpp
//bad idea
struct S {
  operator int() {
    return 2;
  }
};
```

```cpp
//good idea
struct S {
  explicit operator int() {
    return 2;
  }
};
```


# Considering Safety

## Const as Much as Possible
`const` tells the compiler that a variable or method is immutable. This helps the compiler optimize the code and helps the developer know if a function has a side effect. Also, using `const &` prevents the compiler from copying data unnecessarily. The  [comments on `const` from John Carmack](https://web.archive.org/web/20131211065348/https://kotaku.com/454293019) are also a good read.

```cpp
// Bad Idea
class MyClass
{
public:
  void do_something(int i);
  void do_something(std::string str);
};


// Good Idea
class MyClass
{
public:
  void do_something(const int i);
  void do_something(const std::string &str);
};

```

### Carefully Consider Your Return Types

 * Getters
   * Returning by `&` or `const &` can have significant performance savings when the normal use of the returned value is for observation
   * Returning by value is better for thread safety and if the normal use of the returned value is to make a copy anyhow, there's no performance lost
 * Temporaries and local values
   * Always return by value.


### Do not pass and return simple types by const ref

```cpp
// Very Bad Idea
class MyClass
{
public:
  explicit MyClass(const int& t_int_value)
    : m_int_value(t_int_value)
  {
  }

  const int& get_int_value() const
  {
    return m_int_value;
  }

private:
  int m_int_value;
}
```

Instead, pass and return simple types by value. If you plan not to change passed value, declare them as `const`, but not `const` refs:

```cpp
// Good Idea
class MyClass
{
public:
  explicit MyClass(const int t_int_value)
    : m_int_value(t_int_value)
  {
  }

  int get_int_value() const
  {
    return m_int_value;
  }

private:
  int m_int_value;
}
```

Why? Because passing and returning by reference leads to pointer operations, instead of much faster passing of values in processor registers.

## Avoid Raw Memory Access

Raw memory access, allocation and deallocation, are difficult to get correct in C++ without [risking memory errors and leaks](http://blog2.emptycrate.com/content/nobody-understands-c-part-6-are-you-still-using-pointers). C++11 provides tools to avoid these problems.

```cpp
// Bad Idea
MyClass *myobj = new MyClass;

// ...
delete myobj;


// Good Idea
auto myobj = std::make_unique<MyClass>(constructor_param1, constructor_param2); // C++14
auto myobj = std::unique_ptr<MyClass>(new MyClass(constructor_param1, constructor_param2)); // C++11
auto mybuffer = std::make_unique<char[]>(length); // C++14
auto mybuffer = std::unique_ptr<char[]>(new char[length]); // C++11

// or for reference counted objects
auto myobj = std::make_shared<MyClass>();

// ...
// myobj is automatically freed for you whenever it is no longer used.
```

## Use `std::array` or `std::vector` Instead of C-style Arrays

Both of these guarantee contiguous memory layout of objects and can (and should) completely replace your usage of C-style arrays for many of the reasons listed for not using bare pointers.

Also, [avoid](http://stackoverflow.com/questions/3266443/can-you-use-a-shared-ptr-for-raii-of-c-style-arrays) using `std::shared_ptr` to hold an array.

## Use Exceptions

Use exceptions. An exception, can be caught and handled potentially all the way up the highest level of the application with a log and automatic restart of the application. For instance look at this code:

```cpp
File_handle::File_handle(const string& name, const string& mode)
    : f{fopen(name.c_str(), mode.c_str())}
{
    if (!f)
        throw runtime_error{"File_handle: could not open " + name + " as " + mode};
}
```

It is often said that exceptions are meant to signal exceptional events and failures. However, that’s a bit circular because “what is exceptional?” Examples:
 * A precondition that cannot be met
 * A constructor that cannot construct an object (failure to establish its class’s invariant)
 * An out-of-range error (e.g., v[v.size()] = 7)
 * Inability to acquire a resource (e.g., the network is down)

Don't use exception not for error handling:

```cpp
int find_index(vector<string>& vec, const string& x)
{
    try {
        for (gsl::index i = 0; i < vec.size(); ++i)
            if (vec[i] == x) throw i;  // found x
    }
    catch (int i) {
        return i;
    }
    return -1;   // not found
}
```

This is more complicated and most likely runs much slower than the obvious alternative. There is nothing exceptional about finding a value in a vector.

## Use C++-style cast instead of C-style cast
Use the C++-style cast (static\_cast<>, dynamic\_cast<> ...) instead of the C-style cast. The C++-style cast allows more compiler checks and is considerably safer.

```cpp
// Bad Idea
double x = getX();
int i = (int) x;

// Not a Bad Idea
int i = static_cast<int>(x);
```
Additionally the C++ cast style is more visible and has the possibility to search for.

But consider refactoring of program logic (for example, additional checking on overflow and underflow) if you need to cast `double` to `int`. Measure three times and cut 0.9999999999981 times.

# Considering Maintainability


## Avoid Compiler Macros

Compiler definitions and macros are replaced by the preprocessor before the compiler is ever run. This can make debugging very difficult because the debugger doesn't know where the source came from.

```cpp
// Bad Idea
#define PI 3.14159;

// Good Idea
namespace my_project {
  class Constants {
  public:
    // if the above macro would be expanded, then the following line would be:
    //   static const double 3.14159 = 3.14159;
    // which leads to a compile-time error. Sometimes such errors are hard to understand.
    static constexpr double PI = 3.14159;
  };
}
```

## Consider Avoiding Boolean Parameters

They do not provide any additional meaning while reading the code. You can either create a separate function that has a more meaningful name, or pass an enumeration that makes the meaning more clear.

See http://mortoray.com/2015/06/15/get-rid-of-those-boolean-function-parameters/ for more information.



## Properly Utilize 'override' and 'final'

These keywords make it clear to other developers how virtual functions are being utilized, can catch potential errors if the signature of a virtual function changes, and can possibly [hint to the compiler](http://stackoverflow.com/questions/7538820/how-does-the-compiler-benefit-from-cs-new-final-keyword) of optimizations that can be performed.

# Considering Threadability

## Avoid Global Data

Global data leads to unintended side effects between functions and can make code difficult or impossible to parallelize. Even if the code is not intended today for parallelization, there is no reason to make it impossible for the future.

### Statics

Besides being global data, statics are not always constructed and deconstructed as you would expect. This is particularly true in cross-platform environments. See for example, [this g++ bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=66830) regarding the order of destruction of shared static data loaded from dynamic modules.

### Shared Pointers

`std::shared_ptr` is "as good as a global" (http://stackoverflow.com/a/18803611/29975) because it allows multiple pieces of code to interact with the same data.

### Singletons

A singleton is often implemented with a static and/or `shared_ptr`.

## Avoid Heap Operations

Much slower in threaded environments. In many or maybe even most cases, copying data is faster. Plus with move operations and such and things.

## Mutex and mutable go together (M&M rule, C++11)
For member variables it is good practice to use mutex and mutable together. This applies in both ways:
* A mutable member variable is presumed to be a shared variable so it should be synchronized with a mutex (or made atomic)
* If a member variable is itself a mutex, it should be mutable. This is required to use it inside a const member function.

For more information see the following article from Herb Sutter: http://herbsutter.com/2013/05/24/gotw-6a-const-correctness-part-1-3/

See also [related safety discussion](04-Considering_Safety.md#consider-return-by-value-for-mutable-data-const--for-immutable) about `const &` return values

# Considering Performance

## Build Time


### Forward Declare When Possible

This:

```cpp
// some header file
class MyClass;

void doSomething(const MyClass &);
```

instead of:

```cpp
// some header file
#include "MyClass.hpp"

void doSomething(const MyClass &);
```


This applies to templates as well:

```cpp
template<typename T> class MyTemplatedType;
```

This is a proactive approach to reduce compilation time and rebuilding dependencies.

*Note: forward declaration does prevent more inlining and optimizations. It's recommended to use Link Time Optimization or Link Time Code Generation for release builds.*

### Avoid Unnecessary Template Instantiations

Templates are not free to instantiate. Instantiating many templates, or templates with more code than necessary increases compiled code size and build time.

For more examples see [this article](http://blog2.emptycrate.com/content/template-code-bloat-revisited-smaller-makeshared).


## Runtime


### Simplify the Code

The cleaner, simpler, and easier to read the code is, the better chance the compiler has at implementing it well.

### Use Initializer Lists

```cpp
// This
std::vector<ModelObject> mos{mo1, mo2};

// -or-
auto mos = std::vector<ModelObject>{mo1, mo2};
```

```cpp
// Don't do this
std::vector<ModelObject> mos;
mos.push_back(mo1);
mos.push_back(mo2);
```

Initializer lists are significantly more efficient; reducing object copies and resizing of containers.

### Reduce Temporary Objects

```cpp
// Instead of
auto mo1 = getSomeModelObject();
auto mo2 = getAnotherModelObject();

doSomething(mo1, mo2);
```

```cpp
// consider:

doSomething(getSomeModelObject(), getAnotherModelObject());
```

This sort of code prevents the compiler from performing a move operation...

### Enable move operations

Move operations are one of the most touted features of C++11. They allow the compiler to avoid extra copies by moving temporary objects instead of copying them in certain cases.

Certain coding choices we make (such as declaring our own destructor or assignment operator or copy constructor) prevents the compiler from generating a move constructor.

For most code, a simple

```cpp
ModelObject(ModelObject &&) = default;
```

would suffice. However, MSVC2013 doesn't seem to like this code yet.

### Kill `shared_ptr` Copies

`shared_ptr` objects are much more expensive to copy than you'd think they would be. This is because the reference count must be atomic and thread-safe. So this comment just re-enforces the note above: avoid temporaries and too many copies of objects. Just because we are using a pImpl it does not mean our copies are free.

### Reduce Copies and Reassignments as Much as Possible

For more simple cases, the ternary operator can be used:

```cpp
// Bad Idea
std::string somevalue;

if (caseA) {
  somevalue = "Value A";
} else {
  somevalue = "Value B";
}
```

```cpp
// Better Idea
const std::string somevalue = caseA ? "Value A" : "Value B";
```

More complex cases can be facilitated with an [immediately-invoked lambda](http://blog2.emptycrate.com/content/complex-object-initialization-optimization-iife-c11).

```cpp
// Bad Idea
std::string somevalue;

if (caseA) {
  somevalue = "Value A";
} else if(caseB) {
  somevalue = "Value B";
} else {
  somevalue = "Value C";
}
```

```cpp
// Better Idea
const std::string somevalue = [caseA](){
    if (caseA) {
      return "Value A";
    } else if (caseB) {
      return "Value B";
    } else {
      return "Value C";
    }
  }();
```


### Get rid of “new”

We already know that we should not be using raw memory access, so we are using `unique_ptr` and `shared_ptr` instead, right?
Heap allocations are much more expensive than stack allocations, but sometimes we have to use them. To make matters worse, creating a `shared_ptr` actually requires 2 heap allocations.

However, the `make_shared` function reduces this down to just one.

```cpp
std::shared_ptr<ModelObject_Impl>(new ModelObject_Impl());

// should become
std::make_shared<ModelObject_Impl>(); // (it's also more readable and concise)
```

### Prefer `unique_ptr` to `shared_ptr`

If possible use `unique_ptr` instead of `shared_ptr`. The `unique_ptr` does not need to keep track of its copies because it is not copyable. Because of this it is more efficient than the `shared_ptr`. Equivalent to `shared_ptr` and `make_shared` you should use `make_unique` (C++14 or greater) to create the `unique_ptr`:

```cpp
std::make_unique<ModelObject_Impl>();
```

Current best practices suggest returning a `unique_ptr` from factory functions as well, then converting the  `unique_ptr` to a `shared_ptr` if necessary.

```cpp
std::unique_ptr<ModelObject_Impl> factory();

auto shared = std::shared_ptr<ModelObject_Impl>(factory());
```

### Get rid of std::endl

`std::endl` implies a flush operation. It's equivalent to `"\n" << std::flush`.


### Limit Variable Scope

Variables should be declared as late as possible, and ideally only when it's possible to initialize the object. Reduced variable scope results in less memory being used, more efficient code in general, and helps the compiler optimize the code further.

```cpp
// Good Idea
for (int i = 0; i < 15; ++i)
{
  MyObject obj(i);
  // do something with obj
}

// Bad Idea
MyObject obj; // meaningless object initialization
for (int i = 0; i < 15; ++i)
{
  obj = MyObject(i); // unnecessary assignment operation
  // do something with obj
}
// obj is still taking up memory for no reason
```

For C++17 and onwards, consider using init-statement in the `if` and `switch` statements:

```cpp
if (MyObject obj(index); obj.good()) {
    // do something if obj is good
} else {
    // do something if obj is not good
}
```


### Prefer `++i` to `i++`
... when it is semantically correct. Pre-increment is [faster](http://blog2.emptycrate.com/content/why-i-faster-i-c) than post-increment because it does not require a copy of the object to be made.

```cpp
// Bad Idea
for (int i = 0; i < 15; i++)
{
  std::cout << i << '\n';
}

// Good Idea
for (int i = 0; i < 15; ++i)
{
  std::cout << i << '\n';
}
```

Even if many modern compilers will optimize these two loops to the same assembly code, it is still good practice to prefer `++i`.


### Char is a char, string is a string

```cpp
// Bad Idea
std::cout << someThing() << "\n";

// Good Idea
std::cout << someThing() << '\n';
```

This is very minor, but a `"\n"` has to be parsed by the compiler as a `const char *` which has to do a range check for `\0` when writing it to the stream (or appending to a string).



### Know The Standard Library

Properly use the already highly optimized components of the vendor provided standard library.
