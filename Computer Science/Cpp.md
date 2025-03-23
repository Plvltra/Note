## C++小技巧
``` cpp
// GET_MEMBER_NAME_CHECKED用于编译是判断MemberName是否在ClassName存在
#define GET_MEMBER_NAME_CHECKED(ClassName, MemberName) \  
    ((void)sizeof(UEAsserts_Private::GetMemberNameCheckedJunk(((ClassName*)0)->MemberName)), FName(TEXT(#MemberName)))

// A junk function to allow us to use sizeof on a member variable which is potentially a bitfield  
template <typename T>  
bool GetMemberNameCheckedJunk(const T&);  
template <typename T>  
bool GetMemberNameCheckedJunk(const volatile T&);  
template <typename R, typename ...Args>  
bool GetMemberNameCheckedJunk(R(*)(Args...));

PropertyName == GET_MEMBER_NAME_CHECKED(UPanguSplineCenterPoint, Location)

// 1. 用sizeof包裹，使得其中的内容不会在运行时执行产生开销
// 2. ((UPanguSplineCenterPoint*)0)->Location可以判断名称对应的成员是否存在
// 3. (a, b)的返回值是b, 表达式用于运行时判返回值
```
- 对于已有的裸指针，可以建立GUID与指针的映射获取指针。当GUID查不到视为指针失效
- union使得结构体大小变为指定大小
	``` c
	// thread需要8k保存运行栈，还需要一点空间保存其他变量
	typedef union thread {
    struct {
        const char    *name;
        void          (*entry)(void *);
        Context       context;
        union thread  *next;
        char          end[0];
    };
    uint8_t stack[8192];
} Thread;
	```
- COW(Copy-on-Write)
	``` cpp
	template <typename T>
	class Cow
	{
	public:
		Cow() : data(make_shared<T>()) {}
		template<typename... Args>
		explicit Cow(Args&&... args)
			: data(make_shared<T>(std::forward<Args>(args)...)) {}
		Cow(const Cow& other)
			: data(other.data) {}
		Cow& operator=(const Cow& other) = default;
		Cow(Cow&& other) noexcept = delete;
		Cow& operator=(Cow&& other) noexcept = delete;
	
		const T& read() const {
			return *data;
		}
		T& write() {
			if (data.use_count() > 1)
				data = make_shared<T>(*data);
			return *data;
		}
	
	protected:
		shared_ptr<T> data;
	};
	```

## C++语言语法
- enable_shared_from_this: https://www.zhihu.com/question/30957800/answer/3079826725 用于内部创建智能指针而不重复析构(1.weak_ptr需要在创建shared_ptr的时候初始化 2.主要意义在于从类内部创建**异步lambda** shared_ptr的时候, 不重复析构)
- C++ lambda实现原理:  相当于实现了一个内部类，重载operator()实现仿函数
``` cpp
int main() {
  int x = 5;
  auto fun = [x]() { printf("%d\n", x); };
  fun();
  return 0;
}
```
``` cpp
int main()
{
  int x = 5;
    
  class __lambda_8_14
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      printf("%d\n", x);
    }
    
    private: 
    int x;
    
    public:
    __lambda_8_14(int & _x)
    : x{_x}
    {}
    
  };
  
  __lambda_8_14 fun = __lambda_8_14{x};
  fun.operator()();
  return 0;
}
```

- Trivial Type & Standard Layout & POD
	- Trivial Type(最大的特征是可以有多个访问说明符, 用户可添加额外的构造函数)
		- 没有虚函数或虚基类。
		- 由编译器生成默认的特殊成员函数，包括默认构造函数、拷贝构造函数、移动构造函数、赋值运算符、移动赋值运算符和析构函数。
		- 数据成员同样需要满足条件 1 和 2。
	- Standard Layout(用以与C语言对象内存结构兼容)
		- 没有虚函数或虚基类。
		- 所有非静态数据成员都具有相同的访问说明符（public / protected / private）。
		- 在继承体系中最多只有一个类中有非静态数据成员。
		- 子类中的第一个非静态成员的类型与其基类不同。
	- POD = Trivial Type&&Standard Layout
	
	https://andreasfertig.blog/2021/01/cpp20-aggregate-pod-trivial-type-standard-layout-class-what-is-what/
	https://zhuanlan.zhihu.com/p/479755982

- explicit: 防止隐式的类型转换如Cow的例子中
	```cpp
	template <typename T>
	class Cow
	{
	public:
		Cow() : data(make_shared<T>()) {}
		template<typename... Args>
		explicit Cow(Args&&... args) // explicit
			: data(make_shared<T>(std::forward<Args>(args)...)) {}
		Cow(const Cow& other)
			: data(other.data) {}
		Cow& operator=(const Cow& other) = default;
		Cow(Cow&& other) noexcept = delete;
		Cow& operator=(Cow&& other) noexcept = delete;
	
		const T& read() const {
			return *data;
		}
		T& write() {
			if (data.use_count() > 1)
				data = make_shared<T>(*data);
			return *data;
		}
	
	protected:
		shared_ptr<T> data;
	};
	
	Cow<string> A{ "aaa" }; // 构造函数
	Cow<string> B = A;  // 如果构造函数没有explicit, 这里会调用构造函数而不是拷贝构造, 
						// 使用Cow<string>构造string引发编译报错
	```

- atomic
	**compare_exchange_strong**：传入[期望原值]和[设定值]，当前值与期望值相等时，修改当前值为设定值，返回true；当前值与期望值不等时，将期望值修改为当前值，返回false。
	**compare_exchange_weak**：同上，但允许偶然出乎意料的返回（比如在字段值和期待值一样的时候却返回了false），比strong性能更好些，但需要循环来保证逻辑的正确性。
```cpp
template <typename T>
class LockFreeLinkedList {
public:
    struct Node {
        T value;
        std::atomic<Node*> next;

        Node(T val) : value(val), next(nullptr) {}
    };

    LockFreeLinkedList() : head(new Node(T())) {}
    ~LockFreeLinkedList() {
        // 清理链表的内存
        Node* node = head.load();
        while (node) {
            Node* next = node->next.load();
            delete node;
            node = next;
        }
    }

    void insert(T value) {
        Node* newNode = new Node(value);
        Node* oldHead;
        do {
            oldHead = head.load();
            newNode->next.store(oldHead);
        } while (!head.compare_exchange_weak(oldHead, newNode));
    }

    bool remove(T value) {
        Node* prev = nullptr;
        Node* curr = head.load();

        while (curr) {
            if (curr->value == value) {
                Node* next = curr->next.load();
                if (prev) {
                    if (prev->next.compare_exchange_strong(curr, next)) {
                        delete curr;
                        return true;
                    }
                } else {
                    if (head.compare_exchange_strong(curr, next)) {
                        delete curr;
                        return true;
                    }
                }
            } else {
                prev = curr;
            }
            curr = curr->next.load();
        }
        return false;
    }

private:
    std::atomic<Node*> head;
};
```
### 调试技巧
- log: 使用RAII打印进出函数
	```cpp
    #ifdef TRACE_F
        #define TRACE_ENTRY printf("[trace] %s:entry\n", __func__)
        #define TRACE_EXIT printf("[trace] %s:exit\n", __func__)
    #else
        #define TRACE_ENTRY ((void)0)
        #define TRACE_EXIT ((void)0)
    #endif

    void f() {
        TRACE_ENTRY;
        printf("This is f.\n");
        TRACE_EXIT;	
	```

## C语言
- \_\_sync_lock_release: 让C Compiler不会将load和store指令穿过这一行

#### 代码示例
- 使用({ }) GCC编译器扩展特性,  实现**语句表达式**。允许在一个表达式内嵌多条语句，以最后一条语句的值作为整个表达式的值。
```cpp
#define io_read(reg) \
  ({ reg##_T __io_param; \
    ioe_read(reg, &__io_param); \
    __io_param; })
```
- 使用宏进行批量初始化操作
	``` cpp
	#define DEVICES(_) \
		_(0, input_t, "input",    1, &input_ops) \
		_(1, fb_t,    "fb",       1, &fb_ops) \
		_(2, tty_t,   "tty1",     1, &tty_ops) \
		_(3, tty_t,   "tty2",     2, &tty_ops) \
		_(4, sd_t,    "sda",      1, &sd_ops) \
	
	#define DEV_CNT(...) + 1
	device_t *devices[0 DEVICES(DEV_CNT)]; // 巧妙地获得了硬编码的数量
	
	#define INIT(id, device_type, dev_name, dev_id, dev_ops) \
		devices[id] = dev_create(sizeof(device_type), dev_name, dev_id, dev_ops); \
		devices[id]->ops->init(devices[id]);
	DEVICES(INIT); // 巧妙地初始化
	```