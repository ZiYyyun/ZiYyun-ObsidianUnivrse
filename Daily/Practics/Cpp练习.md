#实操/CS/Cpp

#### 寻找最长单词
**（基础）** 编写程序，接受用户[[C++输入输出#输入]]的一串单词，[[C++输入输出#输出]]最长的单词及其长度。  
>    **示例输入**：`apple banana cherry dragonfruit`  
>    **示例输出**：`Longest word: dragonfruit (11 letters)`

```cpp
#include <iostream>
#include <string>
#include <sstream>

int main()
{
	std::string input;
	std::cout >> "Type some words here,splice with space:";
	std::getline(std::cin,input);

	std::string word;
	std::string longestword;
	int maxLength = 0;

	while(iss >> word) {
		if (word.length() > maxLength) {
			maxLength = word.length();
			longestword = word;
		}
	}
	std::cout << "Longest word: " << word << std::endl;
	
	return 0;
}
```


**（综合）** 统计输入中每个单词出现的次数，用`vector<string>`存储单词，`vector<int>`存储次数。  
    **挑战**：能否优化代码，避免重复单词多次遍历`vector`？

```cpp

```

#### 老王开枪
```cpp

```