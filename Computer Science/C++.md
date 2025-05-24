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
- 模版类成员函数实现
	- 使用impl.h, include模版类声明, 在impl.h中实现成员函数
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
- C++ exception(实际throw执行的是不返回的call, 并且中间插入执行landing pad析构对象)
	- 第一步：先调用函数\_\_cxa_allocate_exception申请一片内存，内存上面的数据有两部分组成：第一部分是struct \_\_cxa_refcounted_exception, 第二部分是代码抛出的对象
	- 第二步：再调用__cxa_throw函数, \_\_cxa_throw函数里面主要是调用\_Unwind_RaiseException函数，在正常情况下\_Unwind_RaiseException函数是不返回的。\_Unwind_RaiseException函数主要分为两个阶段（phase）
		- 第一阶段：进行stack unwind，如果在整个unwind的过程中找不到对应的exception handler，\_Unwind_RaiseException将会返回，\_\_cxa_throw会调用std::terminate函数；如果在unwind的过程中找到对应的exception handler了，将会跳转到阶段二。
		- 第二阶段：调用_Unwind_RaiseException_Phase2函数，从头再做一次stack unwind，这次unwind的策略和第一阶段不一样，这次unwind找到第一个landing pad后，就将执行权交给landing pad来执行。landing pad又分两种情况:  
			- 第一种情况：它并不是最终的exception handler，而是中间函数的landing pad, 主要做的事情就是将一些函数内的局部变量进行析构，所有的局部变量处理完后，将会调用_Unwind_Resume函数，\_Unwind_Resume函数会在内部重新调用\_Unwind_RaiseException_Phase2函数，此函数会继续向下做stack unwind，找到下一个landing pad。
			- 第二种情况：它是最终的exception handler，会调用__cxa_begin_catch来处理异常和对异常实例进行销毁/析构。

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

### 工具
- clang-format: 代码格式化工具
- clang-tidy: 静态检查工具

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
- Ref类
	``` cpp
	#pragma once
	
	#include <optional>
	#include <stdexcept>
	
	/*
	 * A Ref<T> represents a "borrowed"-or-"owned" reference to an object of type T.
	 * Whether "borrowed" or "owned", the Ref exposes a constant reference to the inner T.
	 * If "owned", the inner T can also be accessed by non-const reference (and mutated).
	 */
	template<typename T>
	class Ref
	{
	  static_assert( std::is_nothrow_move_constructible_v<T> );
	  static_assert( std::is_nothrow_move_assignable_v<T> );
	
	public:
	  // default constructor -> owned reference (default-constructed)
	  Ref()
	    requires std::default_initializable<T>
	    : obj_( std::in_place )
	  {}
	
	  // construct from rvalue reference -> owned reference (moved from original)
	  Ref( T&& obj ) : obj_( std::move( obj ) ) {} // NOLINT(*-explicit-*)
	
	  // move constructor: move from original (owned or borrowed)
	  Ref( Ref&& other ) noexcept = default;
	
	  // move-assignment: move from original (owned or borrowed)
	  Ref& operator=( Ref&& other ) noexcept = default;
	
	  // borrow from const reference: borrowed reference (points to original)
	  static Ref borrow( const T& obj )
	  {
	    Ref ret { uninitialized };
	    ret.borrowed_obj_ = &obj;
	    return ret;
	  }
	
	  // duplicate Ref by producing borrowed reference to same object
	  Ref borrow() const
	  {
	    Ref ret { uninitialized };
	    ret.borrowed_obj_ = obj_.has_value() ? &obj_.value() : borrowed_obj_;
	    return ret;
	  }
	
	#ifndef DISALLOW_REF_IMPLICIT_COPY
	  // implicit copy via copy constructor -> owned reference (copied from original)
	  Ref( const Ref& other ) : obj_( other.get() ) {}
	
	  // implicit copy via copy-assignment -> owned reference (copied from original)
	  Ref& operator=( const Ref& other )
	  {
	    if ( this != &other ) {
	      obj_ = other.get();
	      borrowed_obj_ = nullptr;
	    }
	  }
	#else
	  // forbid implicit copies
	  Ref( const Ref& other ) = delete;
	  Ref& operator=( const Ref& other ) = delete;
	#endif
	
	  ~Ref() = default;
	
	  bool is_owned() const { return obj_.has_value(); }
	  bool is_borrowed() const { return not is_owned(); }
	
	  // accessors
	
	  // const reference to object (owned or borrowed)
	  const T& get() const { return obj_.has_value() ? *obj_ : *borrowed_obj_; }
	
	  // mutable reference to object (owned only)
	  T& get_mut()
	  {
	    if ( not obj_.has_value() ) {
	      throw std::runtime_error( "attempt to mutate borrowed Ref" );
	    }
	    return *obj_;
	  }
	
	  operator const T&() const { return get(); } // NOLINT(*-explicit-*)
	  operator T&() { return get_mut(); }         // NOLINT(*-explicit-*)
	  const T* operator->() const { return &get(); }
	  T* operator->() { return &get_mut(); }
	
	  explicit operator std::string_view() const
	    requires std::is_convertible_v<T, std::string_view>
	  {
	    return get();
	  }
	
	  T release()
	  {
	    if ( obj_.has_value() ) {
	      return std::move( *obj_ );
	    }
	
	#ifndef DISALLOW_REF_IMPLICIT_COPY
	    return get();
	#else
	    throw std::runtime_error( "Ref::release() called on borrowed reference" );
	#endif
	  }
	
	private:
	  const T* borrowed_obj_ {};
	  std::optional<T> obj_ {};
	
	  struct uninitialized_t
	  {};
	  static constexpr uninitialized_t uninitialized {};
	
	  explicit Ref( uninitialized_t /*unused*/ ) {}
	};
	
	template<typename T>
	static Ref<T> borrow( const T& obj )
	{
	  return Ref<T>::borrow( obj );
	}
	```