# Usefulness of Monad

## Why Monad is Useful

Monad resolves some issues that occur when we use functors in programming.

First, the return type of the function returned from the high-dimensional lift might be inconvenient in some cases.
For example, when we apply the high-dimensional lift such as $\text{Option}.\text{lift}_\text{2d}$ to a function of type $T, U \rightarrow V$, we obtain a function that returns $\text{Option}[\text{Option}[V]]$.

However, we usually don't want this kind of nested generic type, since $\text{Option}[V]$ is enough to represent something that contains no value.
In this case, we can convert the nested $\text{F}[\text{F}[V]]$ to $\text{F}[V]$ using $\text{flat}$ if $F$ is a monad.

Second, more important reason to use a monad is that we can compose functions that return a monad.
Suppose that we're given $f: T \rightarrow M[U]$ and $g: U \rightarrow M[V]$ for a type constructor $M$.
We may have no reason to compose these functions in usual cases, but if $M$ is a monad, we can "naturally" compose them into $h: T \rightarrow M[V]$.

For example, for $f: T \rightarrow \text{Option}[U]$ and $g: U \rightarrow \text{Option}[V]$, we can make a function $h: T \rightarrow \text{Option}[V]$ using the following pseudocode:
```text
h = (t: T) -> Option[V] {
    u = f(t)
    return u == None ? None : g(u)
}
```

The function $h$ returns $\text{Some}(\cdot)$ when both $f$ and $g$ returns $\text{Some}(\cdot)$, and otherwise $\text{None}$.

Technically, it is not a composition of $f$ and $g$, but since we can "naturally" extend $g$ to take $\text{Option}[U]$ here, $h$ can be considered as a kind of "natural" composition of $f$ and $g$.

## Flatlift

As we've seen, we can do the "natural" composition using a monad $M$ by extending $g: U \rightarrow M[V]$ as follows:
```math
\text{flat} \circ (\text{lift}(g)): M[U] \rightarrow M[V]
```
Thus, in the previous example, we could define $h$ without manual implementation.
```math
h = \text{flat} \circ (\text{lift}(g)) \circ f: T \rightarrow \text{Option}[V]
```

We'll see this kind of pattern frequently, so let's define $\text{flatlift}: (T \rightarrow M[U]) \rightarrow (M[T] \rightarrow M[U])$ as follows:
```math
\text{flatlift}(f) = \text{flat} \circ (\text{lift}(f))
```
Though this function is not in the definition of monad, it will be one of the most important functions when we use a monad.

In pseudocode, we can write:
```text
flatlift = (f: T -> M[U]) -> (M[T] -> M[U]) {
    return flat . (lift(f))
}
```
where `.` is a function composition operator (as in Haskell).

## Multi-Dimensional Flatlift

We can also extend multi-variable functions using high-dimensional $\text{flatlift}$.
For example, 2-variable functions $f$ will be extended by
```math
\text{flatlift}_\text{2d}: (T, U \rightarrow M[V]) \rightarrow (M[T], M[U] \rightarrow M[V])
```
<!-- Why do I have to write the hack-ish escape `\_` to make `_` work in the github markdown? -->
Note that the return type of $\text{flatlift}\_\text{2d}(f)$ is $M[V]$, while that of $\text{lift}\_\text{2d}(f)$ is $M[M[V]]$.

The implementation is very similar to what we've done before, so it should be trivial.
```text
flatlift_2d = (f: T, U -> M[V]) -> (M[T], M[U] -> M[V]) {
    return (t_M, u_M) -> M[V] {
        return flatlift((t: T) -> M[V] {
            return flatlift((u: U) -> M[V]) {
                return f(t, u)
            }(u_M)
        })(t_M)
    }
}
```

In this way, we can implement $\text{flatlift}_\text{3d}$ and so on.

This kind of dimensional extension is used a lot in programming, so some PLs support syntax for this.
For example, there is `for` in Scala, and `do` in Haskell.
Actually, this syntax doesn't use $\text{flatlift}$ but something equivalent, but what they do is essentially the same as the dimensional extension of $\text{flatlift}$.
Thus, to appreciate the usefulness of monad, we should understand the dimensional extension of $\text{flatlift}$.

## Examples

What the high-dimensional $\text{flatlift}_\text{2d}$ looks like for each monad?

For $\text{Option}$, suppose that we're given $f: T, U \rightarrow \text{Option}[V]$.
Then, we have
```math
    \text{flatlift}_\text{2d}(f): \text{Option}[T], \text{Option}[U] \rightarrow \text{Option}[V]
```
We can expect it to return $\text{Some}(\cdot)$ when two arguments are $\text{Some}(\cdot)$, and otherwise $\text{None}$, and indeed it does.

For $\text{List}$, we can expect that $\text{flatlift}_\text{2d}$ maps
```math
    [t_1, t_2, t_3], [u_1, u_2] \mapsto [f(t_1, u_1), f(t_1, u_2), f(t_2, u_1), f(t_2, u_2), f(t_3, u_1), f(t_3, u_2)]
```

## Composing Multi-Variable Functions

We can "naturally" compose multi-variable functions, as we've seen.
Suppose that we're given three functions
```math
\begin{align*}
    f_1&: T_1 \rightarrow M[U_1] \\
    f_2&: T_2 \rightarrow M[U_2] \\
    g  &: U_1, U_2 \rightarrow M[V]
\end{align*}
```
Then, we can obtain $h: T_1, T_2 \rightarrow M[V]$ such that
```math
h(t_1, t_2) = \text{flatlift}_\text{2d}(g)(f(t_1), f(t_2))
```

In this way, we can compose functions that take any number of arguments.

## Separating Cross-Cutting Concerns

We can extend not only 1-varaible functions but also multi-variable functions that returns a monad.
Even if it doesn't return a monad, we can make a wrapper function to return a monad using $\text{unit}$.

Thus, we can transform all functions we're using to return a monad, and use only those functions in programming.
This is one way to separate cross-cutting concerns as monad.

For example, we have a program that consists of many functions and they have a common logic for logging.
To separate this, we can
1. define every function to return a monad, and
1. let the $\text{flatlift}$ handle the cross-cutting concern whenever we compose functions.

For the $\text{Option}$ monad, the $\text{flatlift}$ automatically handles the cases when the value is $\text{None}$.
It helps programmers write code as if the code handles only the values that contains something.

## Syntax Support

It may be tedious to call $\text{flatlift}$ manually every time we compose functions.
Thus, it is not hard to come up with the idea that, with some syntax support, functions may be extended automatically.
As we've mentioned before, some PLs support this kind of syntax, such as `for` in Scala, `do` in Haskell, and the syntax looks like this:
```text
do x <- [1, 2, 3]
   y <- [4, 5]
   return x * 10 + y
```
where we can use $M[T]$ as $T$ (for a type $T$ and a monad $M$) without explicit $\text{flatlift}$ calls, which is the advantage of this syntax.

Nothing magical happened, and it can be easily implemented.
`return` is just the $\text{unit}$ function, and `<-` syntax extends the code below using $\text{flatlift}$, and pass the value on right-hand side as its argument.
