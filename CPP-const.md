# 不同位置不同作用

## 指针
```

const int a ，a 是常变量
const int* p /int const * p=&a，不能通过p,修改a,p可以指向其他变量
int* const p p不能指向其他变量，但是可以修改a

```
## 引用

~~略~~

## 变量

~~略~~

## 函数

成员函数
```
class A{
	private:
		int b;
	public :
		void f(int a) const;
}
```
f不能修改b的值，不能修改A对象的成员

一般

void f（const int b);
在f的函数体里面不能修改b

## 其他
就是一些组合了
