# Uniform Initialization: #

~~~cpp
int x(0); // initializer is in parentheses
int y = 0; // initializer follows "="
int z{ 0 }; // initializer is in braces
~~~
Braced initialization lets you express the formerly inexpressible. Using braces, speci‐ fying the initial contents of a container is easy:
~~~cpp
std::vector v{ 1, 3, 5 }; // v's initial content is 1, 3, 5
~~~

Braces can also be used to specify default initialization values for non-static data members. This capability—new to C++11—is shared with the “=” initialization syn‐ tax, but not with parentheses:

~~~cpp
class Widget {
 …
private:
 int x{ 0 }; // fine, x's default value is 0
 int y = 0; // also fine
 int z(0); // error!
};
~~~
initializing with {} checks for narrowing conversions.

CAREFUL. If a constructor exists with an initializer list, using {} will always lead to calling the initializer_list constructor.

# nullptr vs 0 vs NULL #

0 is int NULL is probably int or long; can be interpreted as such specially on calls to overloaded constructors or templates.

nullptr has no integral type, it convers to all raw pointer types. 

# Prefer alias declarations to typedefs #

In Cpp98
~~~cpp
typedef
 std::unique_ptr<std::unordered_map<std::string, std::string>>
 UPtrMapSS;
~~~
in Cpp11
~~~cpp
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
~~~

Why? alias declarations may be templatized ("Alias templates"). Typedefs can't.

# Prefer scoped enums to unscoped enums #

Because scoped enums are declared via “enum class”, they’re sometimes referred to as enum classes.
~~~cpp
enum class Color { black, white, red }; // black, white, red
										 // are scoped to Color
auto white = false; // fine, no other
					 // "white" in scope
Color c = white; // error! no enumerator named
				 // "white" is in this scope
Color c = Color::white; // fine
auto c = Color::white; // also fine (and in accord
					 // with Item 5's advice)
~~~
# Prefer  deleted functions to private undefined ones #

usually, preventing the use of calling a particular function is done by not declaring it, but c++ may declare functions for you, specially with the *special member functions*, i.e., member functions that c++ generates when needed (copy constructor, copy asignment, etc.). In C++ 98, you usually declare private + not define it.

In C++ 11, there's a better way: "= delete". Declare them publicly:
~~~cpp
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
 …
 basic_ios(const basic_ios& ) = delete;
 basic_ios& operator=(const basic_ios&) = delete;
 …
};
~~~
deleted functions must not be used in any way; this is better than making the private and not defining them because it dissallows member and friend functions from compiling.

We can use this to prevent calls:

~~~cpp
bool isLucky(int number); // original function
bool isLucky(char) = delete; // reject chars
bool isLucky(bool) = delete; // reject bools
bool isLucky(double) = delete; // reject doubles and floats
~~~

Another trick: prevent use of template instatiations that should be disabled:

~~~cpp
template<typename T>
void processPointer(T* ptr);
template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
template<>
void processPointer<const void>(const void*) = delete;
template<>
void processPointer<const char>(const char*) = delete;
~~~
This code disables calls with void*, char* and their const varieties.

# Declare overriding functions *override* #
virtual function implementations override implementations of their base class; 

For override to happen, Since C++98 there must be:
-> Base class function must be virtual
->Base and derived function names must be identical except for destructors.
-> paramenter types must be identitcal.
-> constness of the base and derived class must be identitcal.
-> return types and exception specification must be compatible.

And in C++11 we add:
-> The function's reference qualifiers must be identitcal. they make it possible to limit use of a member function to lvalues or rvalues only:

~~~cpp
class Widget {
public:
 …
 void doWork() &; // this version of doWork applies
 // only when *this is an lvalue
 void doWork() &&; // this version of doWork applies
}; // only when *this is an rvalue
…
Widget makeWidget(); // factory function (returns rvalue)
Widget w; // normal object (an lvalue)
…
w.doWork(); // calls Widget::doWork for lvalues
 // (i.e., Widget::doWork &)
makeWidget().doWork(); // calls Widget::doWork for rvalu
~~~

Since declaring derived clas override is hard, c++ gives us a way to make it explicit:

~~~cpp
class Derived: public Base {
public:
 virtual void mf1() override;
 virtual void mf2(unsigned int x) override;
 virtual void mf3() && override;
 virtual void mf4() const override;
};
~~~
This won't compile if they are not override! (good!)

# Prefer const_iterators to iterators #
const_iterators are the STL equivalent of pointers-to-const.

Whenever possible use const iterators; in c++ 98, const iterators had a halfhearted support. It wasn't easy to create them, and once done, the used was limited:

~~~cpp
typedef std::vector<int>::iterator IterT; // typetypedef std::vector<int>::const_iterator ConstIterT; // defs
std::vector<int> values;
…
ConstIterT ci =
 std::find(static_cast<ConstIterT>(values.begin()), // cast
 static_cast<ConstIterT>(values.end()), // cast
 1983);
values.insert(static_castIter<T>(ci), 1998); // may not
 // compile;
~~~
the cast in this code in the call to std::find are needed because the values container is non-const. In c++ 98 there was no simple way to get a const_iterator from non-const container. But even then, locations for insertion and erasures could only be done with iterators, not with const_iterators. That's why there's a cast to Iterator.

In C++11 though, the member functions cbegin and cend produce ocnst iterators, even for non-const containers, and STL member functions that use iterators to identify positions like insert or erase actually use const iterators.  Same code in C++11:
~~~cpp
std::vector<int> values; // as before
…
auto it = // use cbegin
std::find(values.cbegin(),values.cend(), 1983); // and cend
values.insert(it, 1998);
~~~

# Declare functions as noexcept if they won't emit exceptions #

In C++11, the only truly meaningful information about exception emitting behavior of a function was simply if it emitted any. noexcept is for functions that guarantee they won't emit exceptions.

# Use constexpr whenever possible #

constexpr is a beefed up form of const in objects. In the case of functions it has a different meaning.

Constexpr indicates a value that's not only constant, but also that it's known during compile time. YOU CAN'T ASSUME THAT THE RESULT OF A CONSTEXPR FUNCTION IS CONST.

In the case of constexpr objects:
-> They are const and they have values known during compile time.
->These values are privileged. they can be placed on read-only memory.
-> integral values that are constant and known during compile time can be used in contexts where c++ requires an integral constant expression, for example, specificilation of array sizes, integral template arguments(length of std::array objects included), enumerator values, etc:

~~~cpp
int sz; // non-constexpr variable
…
constexpr auto arraySize1 = sz; // error! sz's value not
 // known at compilation
std::array<int, sz> data1; // error! same problem
constexpr auto arraySize2 = 10; // fine, 10 is a
 // compile-time constant
std::array<int, arraySize2> data2; // fine, arraySize2
//is constexpr
~~~
all constexpr objects are const, but not all const objects are constexpr. *If you want to make sure compilers guarantee that the variable has a value that can be used in contexts requiring compile-time constants, you use constexpr*

constxpr functions produce compile time constants when called with compile time constants. If they're called with values not know until runtime, they produce  runtime values:
1. constexpr functions can be used in contexts that demand compile-time con‐
stants. If the values of the arguments you pass to a constexpr function in such a
context are known during compilation, the result will be computed during
compilation. If any of the arguments’ values is not known during compilation,
your code will be rejected.

3. When a constexpr function is called with one or more values that are not
known during compilation, it acts like a normal function, computing its result at
runtime. This means you don’t need two functions to perform the same opera‐
tion, one for compile-time constants and one for all other values. The constexpr
function does it all.

Something like 
~~~cpp
constexpr
Point midpoint(const Point& p1, const Point& p2) noexcept
{
 return { (p1.xValue() + p2.xValue()) / 2, // call constexpr
 (p1.yValue() + p2.yValue()) / 2 }; // member funcs
}
constexpr auto mid = midpoint(p1, p2); // init constexpr
 // object w/result of
 // constexpr function
~~~

is viable! tat means that mid can be constructed at compile time! USE CONST EXPR WHEN POSSIBE!

# Make const member functions thread safe #

Use mutexes/atomics safely; they can't be copied, only moved.

For a single variable or memory location requiring synchroni‐
zation, use of a std::atomic is adequate, but once you get to two or more variables
or memory locations that require manipulation as a unit, you should reach for a
mutex.
1. Make const member functions thread safe unless you’re certain they’ll never
be used in a concurrent context.

2. Use of std::atomic variables may offer better performance than a mutex, but
they’re suited for manipulation of only a single variable or memory location.

# Understand special member function generation #

C++ special functions are the ones C++ is willing to generate on its own (C++98: def. constructor, destructor, copy constructor and copy asignment operator). They're generated only if needed; The default constructor for example only gets generated if the class declares no constructor at all.

Generated functions are implicitly public and inline, as well as nonvirtual unless the generated function is a destructor in a derived class inheriting from a base class with virtual destructor.

In C++11, there is two more special member functions: *move constructor* and *move asignment operator*.
~~~cpp
class Widget {
public:
 …
 Widget(Widget&& rhs); // move constructor
 Widget& operator=(Widget&& rhs); // move assignment operator
 …
};
~~~
Move constructors perform "memberwise moves" on each of the non-static data members of the class; it steals the whole thing; also happens with base class parts, and the move assignment operator does the same.

However, when talking about "move constructing" or move assigning a data member or a base class, there is no guarantee that the move will take place, it's more like a move request, because some types arent move enabled; in those ccases, they'll be "moved" via copy operations.

If you declare a move operation (assignment or constructor), the compiler will not generate the other so you must declare both.

If member-wise copying of the class's non static data members is what you want, you can just go "=default" in c++11:
~~~cpp
class Widget {
public:
 …
 ~Widget(); // user-declared dtor
 … // default copy ctor
 Widget(const Widget&) = default; // behavior is OK
 Widget& // default copy assign
 operator=(const Widget&) = default; // behavior is OK
 …
};
~~~

Rules of special member functions in C++:
1. Default constructor: Same rules as C++98. Generated only if the class contains
no user-declared constructors.
2. Destructor: Essentially same rules as C++98; sole difference is that destructors
are noexcept by default. As in C++98, virtual only if a base class
destructor is virtual.
3. Copy constructor: Same runtime behavior as C++98: memberwise copy con‐
struction of non-static data members. Generated only if the class lacks a userdeclared copy constructor. Deleted if the class declares a move operation.
Generation of this function in a class with a user-declared copy assignment oper‐
ator or destructor is deprecated.
4. Copy assignment operator: Same runtime behavior as C++98: memberwise
copy assignment of non-static data members. Generated only if the class lacks a
user-declared copy assignment operator. Deleted if the class declares a move
operation. Generation of this function in a class with a user-declared copy con‐
structor or destructor is deprecated.
5. Move constructor and move assignment operator: Each performs memberwise
moving of non-static data members. Generated only if the class contains no userdeclared copy operations, move operations, or destructor.

Remember:
1. The special member functions are those compilers may generate on their own:
default constructor, destructor, copy operations, and move operations.
2. Move operations are generated only for classes lacking explicitly declared
move operations, copy operations, and a destructor.
3. The copy constructor is generated only for classes lacking an explicitly
declared copy constructor, and it’s deleted if a move operation is declared.
The copy assignment operator is generated only for classes lacking an explic‐
itly declared copy assignment operator, and it’s deleted if a move operation is
declared. Generation of the copy operations in classes with an explicitly
declared destructor is deprecated.
4. Member function templates never suppress generation of special member
functions.
