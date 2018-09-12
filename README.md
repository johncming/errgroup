# errgroup

参考[golang.org/x/sync/errgroup](https://godoc.org/golang.org/x/sync/errgroup)

改errgroup的问题在于，其中一个goroutine出错后，其它未完成goroutine依旧执行。但是有些情况下，需要立即中止未完成的gorouine操作。

## golang.org/x/sync/errgroup的样例

```golang
func main() {
	g, ctx := errgroup.WithContext(context.Background())

	start := time.Now()

	// short
	g.Go(func() error {
		<-time.After(time.Second * 3)
		return errors.New("3s error")
	})

	// long
	g.Go(func() error {
		<-time.After(time.Second * 30)
		return errors.New("30s error")
	})

	err := g.Wait()
	if err != nil {
		fmt.Printf("%+v\n", err) // output for debug
	}

	select {
	case <-ctx.Done():
		m := time.Now().Sub(start)
		fmt.Printf("%+v\n", m) // output for debug
	}
}
```

output:

```
3s error                                                                            
30.00471109s 
```

实际运行时间为30s
