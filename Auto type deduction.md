with only one curious exception, auto type deduction is [[Type deduction in templates]].

One way they differ?
~~~cpp
auto x1 = 27;
auto x2(27);
auto x3 = { 27 };
auto x4{ 27 };
~~~

although this initialization is all valid if "int" was the type, with auto:

~~~cpp
auto x1 = 27; // type is int, value is 27 
auto x2(27); // ditto
auto x3 = { 27 }; // type is std::initializer_list, // value is { 27 } 
auto x4{ 27 }; // ditto
~~~
- auto type deduction is usually the same as template type deduction, but auto type deduction assumes that a braced initializer represents a std::initial izer_list, and template type deduction doesn’t. 
- auto in a function return type or a lambda parameter implies template type deduction, not auto type deduction.