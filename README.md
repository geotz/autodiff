# autodiff
c++14 compile-time symbolic differentiation with zero memory footprint

This is a proof-of-concept. Using modern c++14 techniques we are able to perform symbolic calculus and symbolic differentiation.

```cpp
    using X = Var<0>;
    using Y = Var<1>;
    using Z = Var<2>;
    X x;
    Y y;
    Z z;
    auto r1 = (x*y) / (x+y);
    auto r2 = sin(x)*sin(x) + cos(x)*cos(x);
    auto r3 = (3_K / 12_K) * (x+y)*(x+y);
    cerr << r1.subs( x(2.), y(3.) ) << endl;
    cerr << r1.diff(x,x,x).subs( x(2.), y(3.) ) << endl;
    cerr << r1.diff(x,y).subs( x(2.), y(3.) ) << endl;
    //decltype(r3) = Mul<Mul<Div<Const<1>, Const<4>>, Add<Var<0>, Var<1>>>, Add<Var<0>, Var<1>>> 
```

## Features

* Each symbol is a type.
* Symbolic expressions are types. We push `decltype` to its limits to deduce complex expression trees.
* Notion of *symbols* and *Lvalues*: `operator(T&&)` on a symbol yields a perfectly forwarded object initialization:
  * `Lvalue<std::decay_t<T>,Id>{std::forward<T>(v)}`
* *Nothing* is stored inside the type (empty types). Other approaches store the value *in* the symbol. We store nothing:
  * substitution `subs(const Lvalue<Ts,Is>&...)` will deduce an evaluation sequence at the point of call
  * as a variadic function, it works with *any* type
* Trivial simplifications are supported: `x-x=0`, `x*0=0`, `1*x=x`, `x+x=2*x` etc ...
* `static_assert` can catch errors at compile-time like:
  * trying to substitue a variable twice or not at all
  * division by 0 or undefined quantity 0/0
* Multi-variate and multi-degree differentiation is again a variadic function.
* Fractions are simplified via their GCD at compile time.
* Suffix operator `_K` for constants.

## Dessert

* I have used CRTP to block funny type deductions.
* I had to fiddle a little bit with SFINAE, but I tried to limit its usage (might be unavoidable since I'm exploiting overload resolution...)
* *Of course* everything is `constexpr` :)
