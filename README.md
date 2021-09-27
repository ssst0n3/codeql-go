# codeql-go-vendor-extractor

## WHY?

The [official extractor for golang](https://github.com/github/codeql-go/tree/cd1e14ed09f4b56229b5c4fb7797203193b93897/extractor/cli/go-extractor) only support gomod mode.

It's not graceful for pure vendor mode.

For example, the result of this query below will be empty:

https://lgtm.com/query/8418405387172037343/

```
import go

from CallExpr e
where e.getTarget().getName()="panic"
select e
```

But it should find the function call here:

https://github.com/ssst0n3/go-vendor-test/blob/main/vendor/st0n3/st0n3.go

```
package st0n3

func Crash() {
    panic("crash")
}
```



## How to Use?

TODO