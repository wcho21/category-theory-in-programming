# Monad

We first introduce the formal definition of monad and draw semantic requirements from it to explain what is a monad precisely and concretely without abstract metaphors such as a box or a container, which may be confusing rather than helpful.

## Formal Definition

A monad is a functor $M$ if
```math
\begin{align*}
    \text{unit}[T, U]&: T \rightarrow M[T] \\
    \text{flat}[T, U]&: M[M[T]] \rightarrow M[T]
\end{align*}
```
are given where they satisfy the following conditions:
1. Naturality of $\text{unit}$:
    As we've covered in the previous chapters, for a function $f: T \rightarrow U$, we have $\text{lift}(f): M[T] \rightarrow M[U]$.
    <!-- \begin{tikzcd} T \arrow{r}{\text{unit}} \arrow[swap]{d}{f} & M[T]  \arrow{d}{\text{lift}(f)} \\ U \arrow{r}{\text{unit}} & M[U] \end{tikzcd} -->
    <div align="center"><img src="https://i.upmath.me/svgb/dc1BCoAgEIXhq8wq6jDtbOW4sNSQwsImjIa5ewS5bPvgez-Ofo6JKS735AQUoM15K5yFkfxFfKZIIt-sj2J3w044CDTQa2WgClfFGgNJGzoBRBj-Dl89GECfXK0_"></div>

    When we're given $\text{unit}$, there are two ways to take $T$ to $M[U]$, and the results must be the same, i.e., $\text{lift}(f) \circ \text{unit} = \text{unit} \circ f$.
1. Naturality of $\text{lift}$:
    Similarly, there are two ways to take $M[M[T]]$ to $M[T]$ using $\text{lift}$, and the result must be the same, i.e., $\text{lift}(f) \circ \text{flat} = \text{flat} \circ \text{lift}(\text{lift}(f))$.
    <!-- \begin{tikzcd} M[M[T]] \arrow{r}{\text{flat}} \arrow[swap]{d}{\text{lift}(\text{lift}(f))} & M[T]  \arrow{d}{\text{lift}(f)} \\ M[M[U]] \arrow{r}{\text{flat}} & M[U] \end{tikzcd} -->
    <div align="center"><img src="https://i.upmath.me/svgb/dY7BCoAwDEN_pSdxH-NNT-sO020yFJVZUCz9d1EciOAtJOEl2Po-TkxxODonUOlK18YA2pTmjZMwkt-Jw2hJ5LH1utnFsMvhGANJ-dZBKYECLhZk1qcelADiPdj8D16MxgD6yeWPJw"></div>

There are some other conditions, but we won't cover them in detail here.

Note that, in other materials, it may be defined with $\text{bind}$ or $\text{flatmap}$ instead of $\text{flat}$ where
```math
\begin{align*}
    \text{bind}[T, U]&: M[T] \rightarrow ((T \rightarrow M[U]) \rightarrow M[U]) \\
    \text{flatmap}[T, U]&: (M[T], T \rightarrow M[U]) \rightarrow M[U]
\end{align*}
```
but we won't use this here, since it is the PL's own constraints that led to this kind of modification.

## Semantic Requirements

What kind of functor can be a monad?
Before taking an example, let's discuss what is required semantically to be a monad.

Suppose that $M$ is a monad, and we're given a function $f: T \rightarrow U$ like this:
```math
t_1, t_2, \dots \mapsto u_1, u_2, \dots
```
where $T$ and $U$ are types.

Say $f$ represents some semantic relationship between $t_k$ and $u_k$.
There should be then a similar semantic relationship represented by $\text{lift}(f): M[T] \rightarrow M[U]$, which maps values like this:
```math
\text{unit}(t_1), \text{unit}(t_2), \dots \mapsto \text{unit}(u_1), \text{unit}(u_2), \dots
```
In other words, $\text{unit}$ should preserve some semantic relationship represented by $f: T \rightarrow U$, and should not change the meaning of the values, but only the types $T$ and $U$ to $M[T]$ and $M[U]$, respectively.

Thus, casually speaking, for an arbitrary type $T$, $M[T]$ should be a type that includes a meaning of $T$ or something semantically equivalent to $T$.
For example, if $T$ is `int`, then $M[T]$ can be `double`.

Similarly, $\text{flat}$ should preserve the semantic relationship, at least partially, represented by $\text{lift}(\text{lift}(f)): M[M[T]] \rightarrow M[M[U]]$.
Thus, $M[M[T]]$ is something that represents a concept that can be viewed as $M[T]$ in some regard, even though its meaning isn't directly included in it.

Therefore, a monad is a functor that satisfies the following semantic requirements:
- $M[T]$ semantically includes $T$ with conversion $\text{unit}$.
- $M[M[T]]$ can be seen as $M[T]$ in some regard with conversion $\text{flat}$.

It follows that $\text{unit}$ and $\text{flat}$ are type-level conversions that preserve the meaning of values.

## Examples

### Option

What kind of functor satisfies the requirements?

We've introduced $\text{Option}$ as a type that represents something that may contain no value.
We can observe that:
- $\text{Option}[T]$ means $T$ or nothing, which includes the meaning of $T$.
- $\text{Option}[\text{Option}[T]]$ means $T$ or nothing, or nothing, which is equivalent to $T$ or nothing, so it can be seen as $\text{Option}[T]$.

From this semantic viewpoint, we can define
```math
\begin{align*}
    \text{unit} &= t \mapsto \text{Some}(t) \\
    \text{flat} &= \begin{cases}
        \text{Some}(\cdot) &\mapsto \begin{cases}
            \text{Some}(\text{Some}(t)) &\mapsto t \\
            \text{Some}(\text{None}) &\mapsto \text{None}
        \end{cases} \\
        \text{None} &\mapsto \text{None}
    \end{cases}
\end{align*}
```
i.e., functions that preserve the meaning and only change the types.

Therefore, $\text{Option}$ is a monad.

### List

For $\text{List}$, which represents a sequence of values of type $T$, we can observe that:
- We can view a single value of type $T$ as a sequence with a single element, i.e., $\text{List}[T]$ includes $T$.
- $\text{List}[\text{List}[T]]$, a sequence of sequences of $T$, can be seen somehow as $\text{List}[T]$.

Therefore, $\text{List}$ is a monad.

## Meaning of Monads

If $M$ is a monad, then $M[T]$ is an extension of $T$, where
1. we can extend operations on $T$ to those on $M[T]$, and
1. $M[M[T]]$, the extension on $M[T]$ itself, is semantically equivalent to $M[T]$, or different but can be seen as $M[T]$ in some regard.

Based on this, we can conclude that a monad is an extension on concepts that can be applied meaningfully only once.

## Implementation

Ideally, a monad can be implemented as a class of type generics.
<!-- \begin{tikzcd}[column sep=0mm,arrows=no head] & \text{Functor} \arrow[d] \\ & \text{Monad} \arrow[dl] \arrow[d] \arrow[dr] & \\ \text{Option} & \text{List} & \cdots \end{tikzcd} -->
<div align="center"><img src="https://i.upmath.me/svgb/TY6xDsIwDER_xRMTAz_QlQnEB9QZQmy1EY1dJa5AVPl3pKgUtjvdu9PhnYcoq8XHO1Dtg05LEig8d6eUjj5nfZZOFEb25OAAaPyy9bxIMM0VsBE9OUDc06uKp182uX9sU7mN4da4zRZV6r5wicWaC6RWAFnoe_ED"></div>

For a monad $M$, we'll see that it provides a common interface of $\text{unit}$ and $\text{flat}$, and we'll call each function, for example, for $\text{List}$, $\text{List}.\text{unit}$ and $\text{List}.\text{flat}$.

How can we implement them for $\text{Option}$? $\text{Option}.\text{unit}$ is trivial:
```text
Option[T].unit = (t: T) -> Option[T] {
    return Some(t): Option[T]
}
```
$\text{flat}$ is also not that difficult:
```text
Option[T].flat = (oot: Option[Option[T]]) -> Option[T] {
    return oot = Some(ot) ? ot : None
}
```
