# Chain transform

This topic tells you about the Application Accelerator `Chain` transform in Tanzu Application Platform (commonly known as TAP).

The `Chain` transform uses function composition to produce its final output.

![Diagram of a chain transform.](images/chain.svg)

## <a id="syntax-reference"></a>Syntax reference

```go
engine {
  T1()
  T2()
  T3()
}
```

## <a id="behavior"></a>Behavior of chain

A chain of **T1** then **T2** then **T3** first applies transform **T1**.
It then applies **T2** to the output of **T1**, and finally applies **T3** to
the output of that. In other words, **T3** to **T2** to **T1**.

## <a id="dot-notation"></a>Dot notation

```go
engine {
  T1().T2().T3()
}
```

## <a id="behavior-dot-notation"></a>Behavior of dot notation

A chain of **T1** then **T2** then **T3** first applies transform **T1**.
It then applies **T2** to the output of **T1**, and finally applies **T3** to
the output of that. In other words, **T3** to **T2** to **T1**.

The dot notation is included to add the concept of operator priority.
A good use for this concept is when a [Merge](merge.md) is included in the chain of transforms.

For example:

```go
engine {
  T1()
  + T2()
  + T3().T4()
}
```

In the example, the `T3().T4()` is resolved first before processing the
declared merges as the transforms are using the dot notation.
