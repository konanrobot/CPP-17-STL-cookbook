# 实现分割算法

很多情况下，STL已存在的算法并不够我们使用。不过，我们可以自己去实现。在解决具体问题之前，我们需要坚定的确定一点，就是所有很多问题都有通解。当我们自己遇到一些问题时，我们可以实现一些辅助函数帮助我们，这些辅助函数逐渐的就可以形成库。这里关键是要明白什么样的代码是足够通用的，否则我们就需要创造一套通用语言了。

本节我们将实现一个算法，叫做**分割**(split)。该算法可以通过给定的值对任何范围的元素进行分割，分割完成后，将分割后的结果块拷贝到输出区域中。

## How to do it...

本节我们将实现类似于STL的算法，叫做分割，并且用这个算法对字符串进行分割：

1. 首先包含必要的头文件，并声明相应的命名空间。

   ```c++
   #include <iostream>
   #include <string>
   #include <algorithm>
   #include <iterator>
   #include <list>
   
   using namespace std; 
   ```

2. 本节的所有算法都围绕分割来进行。其接受一对begin/end迭代器，和一个输出迭代器，其用法和std::copy或std::transform类似。其他参数为split_val和bin_func。split_val参数是要在输入范围内要查找的值，其表示要当碰到这个值时，要对范围进行分割。bin_func参数是一个函数，其会表示分割的子序列的开始和结尾。我们可以使用std::find对输入范围进行迭代查找，这样我们就能直接跳转到split_val所在的位置。当将一个长字符串分割成多个单词，我们可以通过分割空格字符达到目的。对于每一个分割值，我们我们都会做相应的分割，并将对应的分割块拷贝到输出范围内：

   ```c++
   template <typename InIt, typename OutIt, typename T, typename F>
   InIt split(InIt it, InIt end_it, OutIt out_it, T split_val,
   		  F bin_func)
   {
       while (it != end_it) {
           auto slice_end (find(it, end_it, split_val));
           *out_it++ = bin_func(it, slice_end);
           
           if (slice_end == end_it) { return end_it; }
           it = next(slice_end);
       }
       return it;
   }
   ```

3. 现在我们尝试一下我们的新算法。我们构建一个需要进行分割的字符串。其中的字符使用'-'进行连接：

   ```c++
   int main()
   {
   	const string s {"a-b-c-d-e-f-g"};
   ```

4. 创建一个bin_func对象，其能接受一组迭代器，我们需要通过该函数创建一个新的字符串：

   ```c++
   	auto binfunc ([](auto it_a, auto it_b) {
       	return string(it_a, it_b);
       });
   ```

5. 输出的子序列将保存在std::list中。我们现在可以调用split算法：

   ```c++
       list<string> l;
       split(begin(s), end(s), back_inserter(l), '-', binfunc);
   ```

6. 为了看一下结果，我们将对子字符串进行打印：

   ```c++
   	copy(begin(l), end(l), ostream_iterator<string>{cout, "\n"});
   } 
   ```

7. 编译并运行程序，就可以看到如下输出。其子序列将不会包含破折号，只有单个单词(在我们的例子中，为单个字母)：

   ```c++
   $ ./split
   a
   b
   c
   d
   e
   f
   g
   ```



## How it works...

split算法与std::transform的工作原理很类似，因为其能接受一对begin/end迭代器和一个输出迭代器。其也会将最终的算法结果拷贝到输出迭代器所在的容器。除此之外，其接受一个 split_val值和一个二元函数。让我们再来看一起其整体实现：

```c++
template <typename InIt, typename OutIt, typename T, typename F>
InIt split(InIt it, InIt end_it, OutIt out_it, T split_val, F bin_func)
{
    while (it != end_it) {
        auto slice_end (find(it, end_it, split_val));
        *out_it++ = bin_func(it, slice_end);
        
        if (slice_end == end_it) { return end_it; }
        it = next(slice_end);
    }
    return it;
}
```

实现中的循环会一直进行到输入范围结束。每次迭代中都会调用std::find用来在输入范围内查找下一个与split_val匹配的元素。在我们的例子中，分割字符就是'-'。每次的下一个减号字符的位置会存在slice_end。每次循环迭代之后，it迭代器将会更新到下一个分割字符所在的位置。这样，循环起始范围将从一个减号跳到下一个减号，而非每一个独立的元素。

在这一系列的操作中，迭代器it指向的是最后子字符串的起始位置，slice_end指向的是子字符串的末尾位置。通过这两个迭代器，就能表示分割后的子字符串。对于字符串"foo-bar-baz"来说，循环中就有三个迭代器。对于用户而言，迭代器什么的并不重要，他们想要的是子字符串，所以这里就是bin_func来完成这个任务。当我们调用split时，我们可以给定其一个如下的二元函数：

```c++
[](auto it_a, auto it_b) {
	return string(it_a, it_b);
}
```

split函数会将迭代器传递给bin_func，并通过迭代器将结果放入输出迭代器中。这样我们就能通过bin_func获得相应的单词，这里的结果是"foo"，"bar"和"baz"。

## There's more...

我们也可以实现相应的迭代器来完成这个算法的实现。我们现在不会去实现这样一个迭代器，但是我们可以简单的看一下。

迭代的每次增长，都会跳转到下一个限定符。

当我们对迭代器进行解引用时，其会通过迭代器指向的当前位置，创建一个字符串对象，就如同我们之前用到的binfunc函数那样。

我们的迭代器类可以称为split_iterator，用来替代算法split，用户的代码可以写成如下的样式：

```c++
string s {"a-b-c-d-e-f-g"};
list<string> l;

auto binfunc ([](auto it_a, auto it_b) {
	return string(it_a, it_b);
});

copy(split_iterator{begin(s), end(s), "-", binfunc},{}, back_inserter(l));
```

虽然在使用中很方便，但是在实现时，迭代器的方式要比算法的形式复杂许多。并且，迭代器实现中很多边缘值会触发代码的bug，并且迭代器实现需要经过非常庞杂的测试。不过，其与其他STL算法能够很好的兼容。