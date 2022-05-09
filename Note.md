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
