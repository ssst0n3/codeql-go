# codeql-go-vendor-extractor

[english version](./README.md)

## WHY?

[官方的go-extractor](https://github.com/github/codeql-go/tree/cd1e14ed09f4b56229b5c4fb7797203193b93897/extractor/cli/go-extractor) 只支持go mod模式。

对于使用纯vendor模式的go项目，是不优雅的。

例如，下面的codeql查询结果是空的。

https://lgtm.com/query/8418405387172037343/

```
import go

from CallExpr e
where e.getTarget().getName()="panic"
select e
```

但是，它本应该可以找出下列会触发panic的函数:

https://github.com/ssst0n3/go-vendor-test/blob/main/vendor/st0n3/st0n3.go

```
package st0n3

func Crash() {
    panic("crash")
}
```

## 如何使用

1. 从 [releases 页面](https://github.com/ssst0n3/codeql-go-vendor/releases) 下载 `go-extractor` (注意版本需要与你的codeql-cli一致)。
2. 复制你刚刚下载的`go-extractor`，覆盖`$HOME/codeql-home/codeql/go/tools/linux64/go-extractor`。
3. 然后就像你往常创建codeql数据库一样。

## Build
```shell
cd extractor/cli/go-extractor/
go build
```

## 原理
codeql官方的go-extractor跳过了vendor目录, 我只是又加了回来。

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