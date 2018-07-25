# atomic package

atomic packge的实现有点特别，这个涉及到操作系统层面，动用到汇编。

## 源码学习

`doc.go` 文件提供文档，顺便也提供了函数列表，主要函数是：

```go
// Swap*** 函数，原子性的存储新的值到目的地址并返回目标地址的前一个值。
func SwapInt32(addr *int32, new int32) (old int32)

// CompareAndSwap*** 执行经典的 compare-and-swap 操作
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)

// Add*** 原子性的为目标地址增加增量并返回新的值
func AddInt32(addr *int32, delta int32) (new int32)

// Load*** 原子性的装载目标地址的值
func LoadInt32(addr *int32) (val int32)

// Store*** 原子性的存储值到目标地址。
func StoreInt32(addr *int32, val int32)
```

### atomic.Value

`value.go` 中提供一个Value结构体，可以帮助用来存储和装载任意值。

```go
// Value结构体提供原子装载和存储始终如一的有类型的值
type Value struct {
	v interface{}
}

// ifaceWords 是 interface{} 的内部表示
type ifaceWords struct {
	typ  unsafe.Pointer
	data unsafe.Pointer
}

func (v *Value) Load() (x interface{}) {
	vp := (*ifaceWords)(unsafe.Pointer(v))  // 通过unsafe.Pointer做一次指针转换
	typ := LoadPointer(&vp.typ) // 一定要用LoadPointer，以保证原子性，比如读到的不是cpu cache
	if typ == nil || uintptr(typ) == ^uintptr(0) {
		// First store not yet completed.
		return nil
	}
	data := LoadPointer(&vp.data)
	xp := (*ifaceWords)(unsafe.Pointer(&x))
	xp.typ = typ
	xp.data = data
	return
}

func (v *Value) Store(x interface{}) {
	if x == nil {
        // 把panic当NullPointException来用？是不是有点重啊？
		panic("sync/atomic: store of nil value into Value")
	}
	vp := (*ifaceWords)(unsafe.Pointer(v)) // 通过unsafe.Pointer做指针转换
	xp := (*ifaceWords)(unsafe.Pointer(&x))
	for {
		typ := LoadPointer(&vp.typ)
		if typ == nil {
            // 尝试开始第一次存储
            // 关闭抢占（preemption）以便其他goroutine能使用active spin wait来等待完成。
            // 另外这样一来GC也不会意外的看到假类型
			runtime_procPin()
            // 通过acs操作来检查并设置tpy为特殊的标志位
			if !CompareAndSwapPointer(&vp.typ, nil, unsafe.Pointer(^uintptr(0))) {
                // 如果失败，表示其他goroutine抢先
				runtime_procUnpin()
				continue
			}
            // 如果成功表示获得设置存储的权利，执行第一次存储
			StorePointer(&vp.data, xp.data)
			StorePointer(&vp.typ, xp.typ)
			runtime_procUnpin()
			return
		}
		if uintptr(typ) == ^uintptr(0) {
            // 第一次存储进行中，等。
            // 因为我们在第一次存储前后禁用了抢占
            // 我们可以使用active spin来等待
			continue
		}
		// First store completed. Check type and overwrite data.
		if typ != xp.typ {
			panic("sync/atomic: store of inconsistently typed value into Value")
		}
		StorePointer(&vp.data, xp.data)
		return
	}
}
```

TBD: runtime_procPin()和runtime_procUnpin()方法的使用不太理解，也没有找到资料。

这个atomic.Value对于希望通过原子操作来实现不加锁的高并发非常有用。

典型使用场景就是原子读写+copyonwrite。

参考Go官网上给出的例子：

```go
type Map map[string]string
var m Value
m.Store(make(Map))
var mu sync.Mutex // used only by writers
// read function can be used to read the data without further synchronization
read := func(key string) (val string) {
        m1 := m.Load().(Map)
        return m1[key]
}
// insert function can be used to update the data without further synchronization
insert := func(key, val string) {
        mu.Lock() // synchronize with other potential writers
        defer mu.Unlock()
        m1 := m.Load().(Map) // load current value of the data structure
        m2 := make(Map)      // create a new value
        for k, v := range m1 {
                m2[k] = v // copy all data from the current object to the new one
        }
        m2[key] = val // do the update that we need
        m.Store(m2)   // atomically replace the current object with the new one
        // At this point all new readers start working with the new version.
        // The old version will be garbage collected once the existing readers
        // (if any) are done with it.
}
```



