# Functor in Programming

## Definition

A type constructor $F$ is a functor if
```math
\text{lift}[T, U]: (T \rightarrow U) \rightarrow (F[T] \rightarrow F[U])
```

is given where the $\text{lift}$ preserves two things: (1) the identity $\text{id}_T$,
    <!-- \begin{tikzcd} T \arrow[loop,swap,distance=5mm,out=120,in=60,swap]{}{\text{id}_T} & F[T] \arrow[loop,swap,distance=5mm,out=120,in=60,swap]{}{\text{lift}(\text{id}_T)=\text{id}_{F[T]}} \end{tikzcd} -->
    <!-- \begin{tikzcd} T \arrow[loop,swap,distance=10mm]{}{\text{id}_T} & F[T] \arrow[loop,swap,distance=10mm]{}{\text{lift}(\text{id}_T)=\text{id}_{F[T]}} \end{tikzcd} -->
    <div align="center"><img src="https://i.upmath.me/svgb/i0lKTc_Mqy7JzK5KTqlVCFGISSwqyi-PzsnPL9ApLk8s0EnJLC5JzEtOtTXNzdXJLy2xNTQy0MnMszUzAMvHVtdWx5SkVpRUZ6bUxofUKqgpuEWHxFJgTk5mWkmtBpKZmrYITjXI8NpahZjUvBSYqwE"></div>

and the composition $g \circ f$.
    <!-- \begin{tikzcd} T \arrow{r}{f} \arrow[swap,out=-30,in=210]{rr}{h = g \circ f} & U \arrow{r}{g} & V & F[T] \arrow{r}{\text{lift}(f)} \arrow[swap,out=-30,in=210]{rr}{\text{lift}(h)=\text{lift}(g) \circ \text{lift}(f)} & F[U] \arrow{r}{\text{lift}(g)} & F[V] \end{tikzcd} -->
    <div align="center"><img src="https://i.upmath.me/svgb/hY_BCsIwEER_ZU_FQIWq51z9graXJoeaJmlQ2rJGKi7771KIEATxOMPMPEZdrA8TxXB9mYGhBtUjzishk-MkuvvaL-X8iHJ_qsowyeOh0oTINIIED8oENOAYCmiyvt-MFgo4d7XOfBXtM9ItuMg7J_5D8vwoZC69SPDvzQ3a_IL6FGg1KDsNn_Nv"></div>

The type constructor takes some type and returns some type.
We can think of the type constructor as a generic type in programming.

## Examples

What kind of type generics satisfies the conditions of a functor?

### Option

Suppose that the $\text{Option}$ type is given by:
```math
\text{Option}[T] = \text{Some}[T] \mid \text{None}
```

which means that it may contain no value.
Many PLs support this under the names such as `Optional` (Java), `Option` (Rust), and `Maybe` (Haskell).

For an arbitrary function $f: T \rightarrow U$, what the lifted function looks like for this type?
We can't think of anything other than the following:
```math
\text{lift}(f) = \begin{cases}
    t: \text{Some[T]} &\mapsto f(t): \text{Some}[U] \\
    t: \text{None}    &\mapsto \text{None}
\end{cases}
```

In pseudocode:
```text
lift[T, U](f) = (ot: Option[T]) -> Option[U] {
    ot == Some(t) ? Some(f(t)) : None
}
```

This lifted function satisfies the conditions.
First, it preserves the identity, since, for $f(t) = t$, we have
```math
\text{lift}(f) = \begin{cases}
    t           &\mapsto f(t) = t \\
    \text{None} &\mapsto \text{None}
\end{cases}
```
i.e., if `f` is the identity, then `lift(f)` is also the identity.
Second, it preserves the composition, since, for $h = g \circ f$, we have
```math
\text{lift}(g) \circ \text{lift}(f) = \begin{cases}
    t           &\mapsto g(f(t)) = h(t) \\
    \text{None} &\mapsto \text{None}
\end{cases}
```
i.e., if `h` is the composition of `g` and `f`, then `lift(h)` is also the composition of `lift(g)` and `lift(f)`.

Therefore, $\text{Option}$ is a functor.

### List

The $\text{List}$ type represents a sequence of values.
```math
\text{List}[T] = [t_1: T, t_2: T, \dots, t_n: T]
```

For this type, we can naturally define $\text{lift}$ to map the sequence as follows:
```math
[t_1, t_2, \dots, t_n] \mapsto [f(t_1), f(t_2), \dots ,f(t_n)]
```

Therefore, $\text{List}$ is also a functor.

Some PLs support a function that behaves exactly like this under the name `map`.
Note that $\text{lift}$ is sometimes called $\text{map}$.

## Implementation

As we've seen above, $\text{Functor}$ is a class of type constructors where $\text{lift}$ is defined.
<!-- \begin{tikzcd}[column sep=0mm,arrows=no head] & \text{Functor} \arrow[dl] \arrow[d] \arrow[dr] & \\ \text{Option} & \text{List} & \cdots \end{tikzcd} -->
<div align="center"><img src="https://i.upmath.me/svgb/i0lKTc_Mqy7JzK5KTqmNTs7PKc3NUyhOLbA1yM3VSSwqyi8vts3LV8hITUyJVVBTiClJrSipdivNSy7JL6pViAGriE7JiYUzEawisIYYqB7_gpLM_LxauBk-mcUlYF5ySn5JsUJMal4KzBkA"></div>

However, very few PLs support this "inheritance" of type generics.
Instead, PLs provide functor types using, for example, inheritance of types.
<!-- \begin{tikzcd}[column sep=0mm,arrows=no head] & \text{Functor[T]} \arrow[dl] \arrow[d] \arrow[dr] & \\ \text{Option[T]} & \text{List[T]} & \cdots \end{tikzcd} -->
<div align="center"><img src="https://i.upmath.me/svgb/i0lKTc_Mqy7JzK5KTqmNTs7PKc3NUyhOLbA1yM3VSSwqyi8vts3LV8hITUyJVVBTiClJrSipdivNSy7JL4oOia1ViAErik7JiYUzEawisJ4YqDb_gpLM_DywLphJPpnFJTCB5JT8kmKFmNS8FJh7AA"></div>
This is why each PL has a different functor implementation.

In this article, we'll use pseudocode that follows concepts properly.
We'll call $\text{lift}$ in $\text{Option}$ simply $\text{Option.lift}$, and implement it as follows:
```text
Option.lift[T, U] = (f: T -> U) -> (Option[T] -> Option[U]) {
    return (ot: Option[T]) -> Option[U] {
        ot == Some(t) ? Some(f(t)) : None
    }
}
```
Note that this is a higher-order function, which takes a function and returns a function.
Also, the return value is a closure that captures `f`.
