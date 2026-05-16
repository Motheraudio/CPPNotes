always prefer auto instead of type. why?
 It must be initialized.
~~~cpp
int x1; // potentially uninitialized
auto x2; // error! initializer required
auto x3 = 0; // fine, x's value is well-defined
~~~

# WHEN YOU SHOULDN'T? #
 For example:
~~~cpp
bool highPriority = features(w)[5];
~~~
if we change to 
~~~cpp
auto highPriority = features(w)[5]; 
~~~
Then we have UB. operator '[ ]' for std::vector<bool> doesn't return a reference to an element of a cointainer; it return an object of type std::vector<bool>::reference, which is a class nested inside the vector of bool.

std::vector::reference exists because std::vector is specified to represent its bools in packed form, one bit per bool. That creates a problem for std::vector’s operator[], because operator[] for std::vector is supposed to return a T&, but C++ forbids references to bits.

std::vector::reference is an example of a proxy class: a class that exists for the purpose of emulating and augmenting the behavior of some other type. Proxy classes are employed for a variety of purposes. std::vector::reference exists to offer the illusion that operator[] for std::vector returns a refer‐ ence to a bit, for example, and the Standard Library’s smart pointer types are proxy classes that graft resource management onto raw pointers. The utility of proxy classes is well-established. In fact, the design pattern “Proxy” is one of the most longstanding members of the software design patterns Pantheon.

You therefore want to avoid code of this form: auto someVar = expression of "invisible" proxy class type;

however:
auto highPriority = static_cast<bool>(features(w)[5]); 

is valid and will return well.