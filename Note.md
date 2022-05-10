# Note-for-C-plus-plus Core-Guidelines



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

###P.2 用ISO的C++标准写代码（Write in ISO Standard C++）

**理由**：这是一套用于编写ISO标准c++（代码）的指南。

**批注（Note）**：有些环境需要扩展，例如访问系统资源。在这种情况下，将必须的扩展局域化，并使用非核心编码指南来控制它们的使用。如果可能的情况话，**封装**扩展的接口，以便在不支持这些扩展的系统上关闭或编译它们。

扩展通常没有严格的语义定义。由于没有严格的标准定义，即使是常见的、由多个编译器实现的扩展也可能有略微不同的行为和边界行为情况。如果充分使用任何此类扩展，预期的可移植性将会受到影响。

**批注（Note）**：**使用有效的ISO c++不能保证可移植性(更不用说正确性了)**。避免依赖于未定义的行为(例如，未定义的求值顺序【后面会提到】)，并清楚构造方法的含义（ be aware of constructs with implementation defined meaning）(例如，sizeof(int))。

**批注（Note）**：**有些环境需要限制使用标准c++语言或库特性**，例如，避免飞机控制软件标准要求的动态内存分配（因为动态内存的分配可能不成功，而这种情况对飞机的安全可能是致命的）。在这种情况下，通过对这些编码指南的扩展来控制它们的(不)使用。

**【**强制执行（Enforcement）**】：
- 使用最新的c++编译器(目前是c++ 20或c++ 17)，（这样的编译器）带有一组不接受扩展的选项。**
