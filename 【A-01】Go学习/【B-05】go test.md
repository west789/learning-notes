# go test

> 在某一路径下执行go test -v index_test.go 命令报方法找不到, 其中index.go中有 loadSourceToMap方法?

index.go

```go
func loadSourceToMap() {
	fmt.Println("111")
}
```

​	index_test.go

```go
func TestLoadSourceToMap(t *testing.T) {
	_, fileName, _, _ := runtime.Caller(1)
	fmt.Println(path.Dir(fileName))
	loadSourceToMap()
}
```

执行***go test -v index_test.go***报错:

```go
# command-line-arguments [command-line-arguments.test]
.\index_test.go:13:2: undefined: loadSourceToMap
FAIL    command-line-arguments [build failed]
FAIL

```

正确用法:

***go test -v index_test.go index.go***

