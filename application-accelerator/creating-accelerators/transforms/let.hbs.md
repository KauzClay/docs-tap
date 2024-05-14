# Let transform

This topic tells you about the Application Accelerator `Let` transform in Tanzu Application Platform (commonly known as TAP).

The `Let` transform wraps another transform, creating a new scope
that extends the existing scope.

SpEL expressions inside the `Let` can access variables
from both the existing scope and the new scope.

Variables defined by the `Let` should not shadow existing variables. If they do,
those existing variables won't be accessible.

## <a id="syntax-reference"></a>Syntax reference

```go
let aVar = "a" in {

}
```

## <a id="execution"></a>Execution

The `Let` adds variables to the new scope by computation of
[SpEL expressions](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions).

```go
engine {
  let aVar = true in {
    if(aVar) {
      T1()
    }
  }
}

```

Symbols defined in the `Let` are evaluated in the new scope in the order they are defined.
This means that symbols lower in the list can make use of the variables defined higher in the
list but not the other way around.
