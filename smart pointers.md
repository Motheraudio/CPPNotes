# Why raw pointers suck #
1. It's declaration doesn't really indicate if it matches a single object or an array.
2. It's decalaration says nothing about if you shouldd destroy what it points to when you're done using it/ ownership.
3. If you determine you should destroy what it's pointing to, how? delete, a destructor?
4. Because of 1, if delete needs to be used, how do we know to do "delete" or " delete []"?
5. Hard to ensure destruction happens exactly once in every path of the code.
6. No way to tell if it's a dangling pointer.

# Smart Pointers #

Wrappers for raw pointers; they behave the same way, but avoids its pitfalls. 
Raw pointers in C++11:
1. std::auto_ptr
2. std::unique_ptr
3. std::shared_ptr
4. std::weak_ptr.

std::unique pointer does what auto_pointer does plus more. The only legitimate use case for std::auto_ptr is a need to compile code with c++98 compilers. Nothing else.

# std::unique_pointers: for exclusive ownership #

unique_ptr should be the oone used the most. We can assume they are the same size as raw ptrs, and for most operations, they use the same instructions.

unique_ptr embodies exclusive ownership: a non-null ptr always owns what it points to; moving it transfers ownership and the source ptr gets set to null. 

Copying a unique ptr is not allowed. It's a *move only type*. Upon destruction, a unique ptr destroys its resource by applying delete to the raw ptr inside of it.

unique pointers can be configured to use custom deleters instead of delete:
~~~cpp
auto delInvmt = [](Investment* pInvestment) // custom
 { // deleter
 makeLogEntry(pInvestment); // (a lambda
 delete pInvestment; // expression)
 };
template<typename... Ts> // revised
std::unique_ptr<Investment, decltype(delInvmt)> // return type
makeInvestment(Ts&&... params)
{
 std::unique_ptr<Investment, decltype(delInvmt)> // ptr to be
 pInv(nullptr, delInvmt); // ret
if ( /* a Stock object should be created */ )
 {
 pInv.reset(new Stock(std::forward<Ts>(params)...));
 }
 else if ( /* a Bond object should be created */ )
 {
 pInv.reset(new Bond(std::forward<Ts>(params)...));
 }
 else if ( /* a RealEstate object should be created */ )
 {
 pInv.reset(new RealEstate(std::forward<Ts>(params)...));
 }
 return pInv;
}
~~~
