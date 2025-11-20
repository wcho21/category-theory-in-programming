# High-Demensional Lift

## 2-Dimensional Lift

$\text{lift}$ transforms 1-variable functions.
However, functions can be multi-variable as well.
Can we apply $\text{lift}$ to multi-variable functions?

In short, we can make a multi-dimensional version of $\text{lift}$.
For example, the 2-dimensional version is given by
```math
F.\text{lift}_{\text{2d}}: (T, U \rightarrow V) \rightarrow (F[T], F[U] \rightarrow F[F[V]])
```
for a type constructor $F$.

## Implementation

To implement this, we first transform a function of type $T, U \rightarrow V$ type into a function of $T, F[U] \rightarrow F[V]$.
Then, we transform it again into a function of $F[T], F[U] \rightarrow F[F[V]]$.

How can we create it using $\text{lift}$ that transforms only 1-variable functions?
In the first step, we view a function $f: (t: T, u: U) \rightarrow V$ as a function of $U \rightarrow V$ with $t$ fixed, so that we can apply $\text{lift}$ to it.
Similarly, we view the result as a function of type $T \rightarrow F[V]$ and apply $\text{lift}$ to it.

In pseudocode:
```text
life_2d = (f: T, U -> V) -> (F[T], F[U] -> F[F[V]]) {
    return (t_F: F[T], u_F: F[U]) -> F[F[V]] {
        return lift((t: T) -> F[V] {
            return lift((u: U) -> V {
                return f(t, u)
            })(u_F)
        })(t_F)
    }
}
```

In this implementation, the nested `lift` views `f(t, u)` as a function of `u`.
```text
            return lift((u: U) -> V {
                return f(t, u)
            })(u_F)
```
The result is a function of type `F[U] -> F[V]`.

The outer `lift` views this result as a function of `t`.
```text
        return lift((t: T) -> F[V] {
            // return a function of F[U] -> F[V], with a fixed t
        })(t_F)
```

So far, we've implicitly used `u_F` and `t_F` as constants.
Lastly, `life_2d` returns a function that takes those values.

In this way, $\text{lift}_\text{3d}$ can be easily implemented.
```text
life_3d = (f: T, U, V -> W) -> (F[T], F[U], F[V] -> F[F[F[W]]]) {
    return (t_F: F[T], u_F: F[U], v_F: F[V]) -> F[F[F[W]]] {
        return lift((t: T) -> F[F[W]] {
            return lift((u: U) -> F[W] {
                return lift((v: V) -> W {
                    return f(t, u, v)
                })(v_F)
            })(u_F)
        })(t_F)
    }
}
```

We will encounter a function of nested lambdas like this again when we cover monads.

## Dimensional Extension

We extended $\text{lift}$ to multiple dimensions.
Let's call it the dimensional extension.
This kind of extension is what we usually do when we need to extend a 1-variable operation to multi-variable operations.

For example, for a multi-variable integration
```math
\int_{y_1}^{y_2}\int_{x_1}^{x_2} f(x, y)\,dxdy
```
we integrate over $x$ by viewing $f$ as a 1-variable function of $x$, and then integrate it over $y$.

## Example: List

We can apply $\text{lift}_\text{2d}$ to $\text{list}$.
For an arbitrary function $f: T, U \rightarrow V$, we can expect that it maps as follows:
```math
    [t_1, t_2, t_3], [u_1, u_2] \mapsto [[f(t_1, u_1), f(t_1, u_2)], [f(t_2, u_1), f(t_2, u_2)], [f(t_3, u_1), f(t_3, u_2)]]
```

For this type, the result of $\text{lift}_\text{2d}(f)$ is
```text
List[T].lift_2d(f) = (t_list: List[T], u_list: List[U]) -> List[List[V]] {
    return lift((t: T) -> List[V] {
        return lift((u: U) -> V {
            return f(t, u)
        })(u_list)
    })(t_list)
}
```

The innermost `lift` maps each element of `List[U]` and returns `List[V]`.
```math
    [u_1, u_2] \mapsto [f(t, u_1), f(t, u_2)]
```
The outer `lift` then view the result as a function of $T$, which maps each element of `List[T]` to the list above.
```math
    [t_1, t_2, t_3] \mapsto [[f(t_1, u_1), f(t_1, u_2)], [f(t_2, u_1), f(t_2, u_2)], [f(t_3, u_1), f(t_3, u_2)]]
```
