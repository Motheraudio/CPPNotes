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

