#  once

Once确保动作只执行一次。

## 源码解析

Once是一个结构体：

```go
type Once struct {
	m    Mutex   // once的锁
	done uint32  // 标志动作是否已经执行过
}
```

Do方法：

```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 { //  检查标志位，如果已经设置为1，则表示已经执行过，直接返回
		return
	}
	// Slow-path.
	o.m.Lock()		// 加锁
	defer o.m.Unlock()  // 通过defer来解锁，因此即使panic了也能解锁
	if o.done == 0 {	// 检查标志位，一定要是0才能继续
        defer atomic.StoreUint32(&o.done, 1)	// 通过defer来设置标志位为1，因此即使f()函数panic，也能保证设置done为1
        // 这意味着，f()函数panic也被视为已经执行过一次
		f()
	}
}
```



