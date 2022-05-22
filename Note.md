# Note-for-C-plus-plus Core-Guidelines

 @[TOC]



## 理念/哲学（Philosophy）

### P.1 **用代码直接表达思想（Express ideas directly in code）**

**理由**：编译器不会阅读注释(或设计文档)，许多程序员(一般)也不会。相比之下，（原则上）如果你在代码中明确地表达了/定义了相关的内容，那么可以由编译器和其他工具进行检查。

**例子**

    class Date {
    public:
        Month month() const;  // 这样做。因为这个语句明确表达了返回值类型，且表明该函数不修改参数值
        int month();  // 不要这样做
    // ...
    };


**作为一名c++程序员，应该了解标准库的基础知识，并在适当的地方使用它。任何程序员都应该了解正在进行的项目中（涉及的）基础库的基础知识，并适当地使用它们。**

**错误案例：**

    void f(vector<string>& v)
    {
    	string val;
   		cin >> val;
        // ...
        int index = -1;// 不好，应当使用 gsl::index
        for (int i = 0; i < v.size(); ++i) {
        if (v[i] == val) {
        index = i;
        break;
            }
        }
        // ...
    }

**对以上错误案例的修改（更清晰的表达）：**

	void f(vector<string>& v)
	{
   		 string val;
   		 cin >> val;
   		 // ...
   		 auto p = find(begin(v), end(v), val);  //更好
   		 // ...
	}


**案例：**
	

	change_speed(double s);   // 不好，无法知道s代表什么
	change_speed(Speed s);    // 更好: s的意思明确了，但是无法说明s是变化后速度的增量，还仅仅是变化后的速度
	// ...
	change_speed(2.3);        // error: 无单位
	change_speed(23_m / 10s);  // 米每秒

        在这里，我们可以接受一个普通的(无单位的)double作为速度的增量，但是那样很容易出错。如果我们既想要绝对速度又想要增量，我们可以定义一个Delta**类型**作为速度的增量。


**【强制执行（Enforcement）->非常严格】:**

   - **坚持使用const关键字（检查成员函数是否修改了它们的对象;检查函数是否通过指针或引用传递的方式修改了参数）；**
   - **多用（明确的）说明代替类型（flag uses of casts (casts neuter the type system)）**
   - **检查既有代码中是否有类似标准库的代码，进行替换-->严格执行（detect code that mimics the standard library (hard)）**

### P.2 用ISO的C++标准写代码（Write in ISO Standard C++）

**理由**：这是一套用于编写ISO标准c++（代码）的指南。

**注意（Note）**：有些环境需要扩展，例如访问系统资源。在这种情况下，将必须的扩展局域化，并使用非核心编码指南来控制它们的使用。如果可能的情况话，**封装**扩展的接口，以便在不支持这些扩展的系统上关闭或编译它们。

扩展通常没有严格的语义定义。由于没有严格的标准定义，即使是常见的、由多个编译器实现的扩展也可能有略微不同的行为和边界行为情况。如果充分使用任何此类扩展，预期的可移植性将会受到影响。

**注意（Note）**：**使用有效的ISO c++不能保证可移植性(更不用说正确性了)**。避免依赖于未定义的行为(例如，未定义的求值顺序【后面会提到】)，并清楚构造方法的含义（ be aware of constructs with implementation defined meaning）(例如，sizeof(int))。

**注意（Note）**：**有些环境需要限制使用标准c++语言或库特性**，例如，避免飞机控制软件标准要求的动态内存分配（因为动态内存的分配可能不成功，而这种情况对飞机的安全可能是致命的）。在这种情况下，通过对这些编码指南的扩展来控制它们的(不)使用。

**【强制执行（Enforcement）】**：
- **使用最新的c++编译器(目前是c++ 20或c++ 17)，（这样的编译器）带有一组不接受扩展的选项。**

### P.3 （代码应明确地）表达意图/目的(Express intent)

**理由**：除非某些代码的意图/目的被声明(例如，在名称或注释中)，否则不可能知道代码是否做了它应该做的事情。

**e.g:**
	
	//不好的例子，因为我们无法变量v的意图（需要对其修改，还是仅仅取值，还是...）
	gsl::index i = 0;
	while (i < v.size()) {
	// ... do something with v[i] ...
	}


**e.g.:**

    //更好的例子1，说明了v的值不能修改
    for (const auto& x : v) { /* do something with the value of x */ }
    for (auto& x : v) { /* modify x */ }//需要修改则使用不带const的变量

有时候，用命名的（named）算法会更好。例如for_each，它直接表达了使用的意图/目的：

    for_each(v, [](int x) { /* do something with the value of x */ });
    //表明，我们对v的处理顺序并不关心
    for_each(par, v, [](int x) { /* do something with the value of x */ });

**一个程序员，应当对以下内容熟悉：**

  - 该指南所支持的库
  - ISO C++标准库
  - 当前项目所使用的任何基础库
 
**注意**：思维切换（Alternative formulation）：说我们应该做什么，而不是某件事应该如何做。

**注意**：相对于其他语言结构，有些语言结构更能够表达（函数/方法）的意图。

**e.g.**：如果两个int类型被用来表示坐标，那么可以这样表示：

    draw_line(int, int, int, int); // 很模糊，你无法明白这四个int是干嘛的
    draw_line(Point, Point); // 很清晰，我知道有两个点（Point）对象


**【强制执行（Enforcement）】**：**代码的优化和半自动化空间依然很大**：用一些更好的方法去替代常见的一些模式
  
  - 简单的**for**循环 vs. **range-for**循环
  - f(T*,int)接口 vs. f(span<T>)接口
  - 在太大的范围内循环变量
  - 裸露的new和delete（个人推荐开始学习用智能指针，非作者建议）
  - 具有许多内置类型的函数

### P.4 理想情况下，程序应该是静态类型安全的（Ideally, a program should be statically type safe）
**【理由】**：理想情况下，程序应该是完全静态(编译时)类型安全的。不幸的是，这是不可能的。

问题涉及的区域:

- 联合类型（unions）
- 类型转换（casts）
- 数组的退化（array decay，数组的类型和维度的丢失称为数组的退化。当通过值或指针将数组传递给函数时，通常会发生这种情况。）
- 越界（range errors）
- 收缩转换（narrowing conversions,可以理解为截断，即浮点型转化为整型，就会产生截断）

**【注意】**：

这些区域是严重问题的来源(例如，崩溃和安全违规)，我们尝试提供替代技术。

**【强制执行（Enforcement）】**：
我们可以根据每个程序的需要和可行性，分别禁止、限制或检测个别问题类别，（并）总是建议/给出另一种可替代的选择。例如：

- 联合类型 - 使用**variant**(在c++ 17中)
- 类型转换 - 减少它们的使用，**模板**（templates）可以提供帮助
- 数组退化 - 使用**span**（from the GSL）
- 越界 - 使用**span**
- 收缩转换/截断 - 减少截断/收缩转换的使用，必要时使用**narrow**或**narrow_cast**(来自GSL)

### P.5 优先选择编译时检查而不是运行时检查（Prefer compile-time checking to run-time checking）

**【理由】**：
（提高）代码的清晰性和性能。你不需要为编译时捕获的错误编写错误处理程序。

**e.g**：

    // Int是integers的一个别名
    int bits = 0; // 不要这样做：这是可以避免的代码（avoidable code）
    for (Int i = 1; i; i <<= 1)
    ++bits;
    if (bits < 32)
    cerr << "Int too small\n";

这个示例未能实现它试图实现的目标(因为是溢出是未定义的)，应该用一个简单的static_assert来替换：

    // Int is an alias used for integers
    static_assert(sizeof(Int) >= 4); // 这样做: 编译时检查

或者更好的方法是使用类型系统，将Int替换为int32_t。

**e.g**：

    void read(int* p, int n); // 最大读n个整数到*p
    int a[100];
    read(a, 1000); // 不好，到达了末端（个人理解：这里的1000是一个纯数字，会造成两个问题：1.代码不容易理解；2.如果1000写错了会造成代码运行异常或者结果输出错误）
修改后（better）：

    void read(span<int> r); // 读入整数r的范围
    int a[100];
    read(a); // 更好: 让编译器弄清楚元素的个数

**可选方案：不要把在编译时可以完成的工作推迟到运行时。**

**【强制执行】**：

- 寻找指针参数；
- 查找运行时检查的越界行为（range violations）。
