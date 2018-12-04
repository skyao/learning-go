# unsafe

golang的unsafe包很强大，基本上很少会去用它。它可以像C一样去操作内存，但由于golang不支持直接进行指针运算，所以用起来稍显麻烦。

## 指针类型背景介绍

Golang中有三种指针：

- `* T`：普通指针，用于传递对象地址，不能进行指针运算。
- unsafe.Pointer：通用指针类型，用于转换不同类型的指针，不能进行指针运算。
- uintptr：用于指针运算，GC 不把 uintptr 当指针，uintptr 无法持有对象。uintptr 类型的目标会被回收。

unsafe.Pointer 的特别之处：

- unsafe.Pointer 可以和 普通指针 进行相互转换。
- unsafe.Pointer 可以和 uintptr 进行相互转换。

也就是说 unsafe.Pointer 是桥梁，可以让任意类型的指针实现相互转换，也可以将任意类型的指针转换为 uintptr 进行指针运算。

- unsafe.Pointer其实就是类似C的`void *`，在golang中是用于各种指针相互转换的桥梁。
- uintptr是golang的内置类型，是能存储指针的整型，uintptr的底层类型是int，它和unsafe.Pointer可相互转换。

uintptr和unsafe.Pointer的区别就是：

- unsafe.Pointer只是单纯的通用指针类型，用于转换不同类型指针，它不可以参与指针运算；
- 而uintptr是用于指针运算的，GC 不把 uintptr 当指针，也就是说 uintptr 无法持有对象，uintptr类型的目标会被回收。









