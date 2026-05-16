In C++11, perhaps the primary use for decltype is declaring function templates where the function’s return type depends on its parameter types

decltype to compute return type:
~~~cpp
template <typename Container, typename Index> 
auto authAndAccess(Container& c, Index i) -> decltype(c[i])  
{ 
	authenticateUser(); 
	return c[i]; 
}
~~~
The use of auto before the function name has nothing to do with type deduction.
Rather, it indicates that C++11’s trailing return type syntax is being used, i.e., that the
function’s return type will be declared following the parameter list (after the “->”)

in C++14:
~~~cpp
template<typename Container, typename Index> 
decltype(auto) authAndAccess(Container& c, Index i)
{
 authenticateUser();
 return c[i];
}
~~~

also towards variables:
~~~cpp
Widget w; 
const Widget& cw = w; 
auto myWidget1 = cw; // auto type deduction:
					 // myWidget1's type is Widget
decltype(auto) myWidget2 = cw; // decltype type deduction:
								// myWidget2's type is
								// const Widget&
~~~

in some cases, decltype might behave strangely; for example, when wrapping in parenthesis:
~~~cpp
decltype(auto) f1()
{
 int x = 0;
 …
 return x; // decltype(x) is int, so f1 returns int
}
decltype(auto) f2()
{
 int x = 0;
 …
 return (x); // decltype((x)) is int&, so f2 returns int&
}
~~~
- decltype almost always yields the type of a variable or expression without any modifications.

- For lvalue expressions of type T other than names, decltype always reports a type of T&. 

- C++14 supports decltype(auto), which, like auto, deduces a type from its initializer, but it performs the type deduction using the decltype rules