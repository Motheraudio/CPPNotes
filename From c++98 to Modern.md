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