cpp 中的 string_view 这个类型 ，一般用在
```
void f(std::string_view s){
	//对s只读
}
std::string a;
std::string_view b;
f(a),f(b); //编译没有问题
// string_view 类型本身 就能接收string类型的数据
```
反正你就把他当作一个java中的插槽+cpp中 const & 类型就行了