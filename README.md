# codeql-go-vendor-extractor

[中文文档](./README-ZH.md)

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

1. download release (same version with your codeql-cli) from [releases page](https://github.com/ssst0n3/codeql-go-vendor/releases).
2. copy the extractor tool `go-extractor` you downloaded into `$HOME/codeql-home/codeql/go/tools/linux64/go-extractor`.
3. just create database like before.

## Build
```shell
cd extractor/cli/go-extractor/
go build
```

## How it works?
The official extractor for golang pass the vendor directory, what I did is just removing the vendor in the regexp pattern.

https://github.com/github/codeql-go/blob/codeql-cli/v2.6.2/extractor/extractor.go#L171-L182
```go
noExtractRe := regexp.MustCompile(`.*(^|` + sep + `)(\.\.|vendor)($|` + sep + `).*`)

// extract AST information for all packages
packages.Visit(pkgs, func(pkg *packages.Package) bool {
    return true
}, func(pkg *packages.Package) {
    for root, _ := range wantedRoots {
    relDir, err := filepath.Rel(root, pkgDirs[pkg.PkgPath])
    if err != nil || noExtractRe.MatchString(relDir) {
        // if the path can't be made relative or matches the noExtract regexp skip it
        continue
    }
    ...
```