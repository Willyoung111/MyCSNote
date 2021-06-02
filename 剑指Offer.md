# 写代码时必须考虑的几个问题：

基本功能

边界条件：最大正整数`2^31 - 1`，最小负整数`-1 * 2^31，数组边界···

特殊输入：空指针、空字符串···

错误处理：非期望字符、

## 1、赋值运算函数

- 返回类型为引用，为了连续赋值
- 传输参数为const
- 释放实例自身已有的内存
- 判断是否自身赋值
- 异常安全性(Exception Safety)原则

> Exception Safety:
>
> 1. 不泄露任何资源
> 2. 不允许破坏数据
>
> 异常安全函数的三种等级：
>
> 1. 基本承诺：异常被抛出后，对象内的成员仍保持有效状态，没有数据破坏和泄露，但对象现实状态不可预估。
> 2. 强烈承诺：异常被抛出后，对象的状态保持不变，即调用前的状态。
> 3. 不抛异常保证：函数承诺不会抛出任何异常。一般内置类型的所有操作都有不抛异常的保证。

```c++
class CMyString
{
public:
    CMyString(char* pData = nullptr);
    CMyString(const CMyString &str);
    ~CMyString();
private:
    char* m_pData;
};

CMyString& CMyString::operator=(const CMyString &str)
{
    if(this != &str)
    {
        CMyString strTemp(str);
        
        char *pTemp = strTemp.m_pData;
        strTemp.m_pData = m_pData;
        m_pData = p_Temp;
    }
    
    return *this;
}
```

若先删除原资源，后申请新资源，申请失败抛出异常，则原数据被破坏，无法保证异常安全性。因此在删除原资源之前先申请新资源。

## 2、单例模式

- 私有化构造函数
- 加锁、两次判断保证线程安全

> 在加锁后，考虑如果有多个线程同时到达加锁位置，是否会发生竞争、资源复用、资源释放等冲突

```c++
#include <mutex>

std::mutex mt;

class Singleton
{
public:
    Singleton* GetInstance();
private:
    Singleton();
    static Singleton* m_singleton;
};

Singleton* Singleton::m_singleton = nullptr;

Singleton* Singleton::GetInstance()
{
    if(m_singleton == nullptr)
    {
        mt.lock();
        if(m_singleton == nullptr)
        {
            m_singleton = new Singleton();
        }
        mt.unlock()''
    }
    return m+singleton;
}
```



## 3、数组中的重复数字

题1、长度为n的数组里所有数字都在[0, n-1]范围内。数组中某几个数字重复了，返回数组中任意一个重复的数字，如果没有返回-1

```
input:
	{2,3,1,0,2,5,3}
output:
	2
```

1. 排序后遍历，时间复杂度`O(nlogn)`

2. 哈希表，时间复杂度`O(n)`，空间复杂度`O(n)`
3. 根据下标交换验证，时间复杂度`O(n)`,空间复杂度`O(1)`

```c++
int duplicate(vector<int> &numbers)
{
    if(numbers.empty())
        return -1;
    
    for(auto &num : numbers)
    {
        if(num < 0 || num > numbers.size() - 1)
            return -1;
    }
    
    for(int i = 0; i < numbers.size(); i++)
    {
        while(numbers[i] != i)
        {
            if(numbers[numbers[i]] == numbers[i])
            {
                return numbers[i];
            }
            else
            {
                int temp = numbers[i];
                numbers[i] = numbers[temp];
                numbers[temp] = temp;
            }
        }
    }
    return -1;
}
```

题2、不修改数组找出重复的数字（要求空间复杂度O(1)）

数组的长度为`n+1`，所有的数字都在1~n内，找出重复的数字

思考：1、只有一个重复数字吗？

​			2、多个重复数字的话输出哪一个？

​			3、没有的话输出-1

时间复杂度	`O(nlogn)`，空间复杂度`O(1)`

```c++
int getDuplication(vector<int> &nums)
{
	if(nums.empty()) return -1;
    int n = nums.size();
    int start = 1;
    int end = n-1;
    while(end >= start)
    {
        int middle = (end - start) / 2 + start;
        int count = countRange(nums, start, middle);
        if(end == start)
        {
            if(count > 1)
            {
                return start;
            }
            else
            {
                break;
            }
        }
        
        if(count > (middle - start + 1))
        {
            end = middle;
        }
        else
        {
            start = middle + 1;
        }
    }
    
    return -1;
}

int countRange(vector<int> &nums, int start, int end)
{
    //count number in range [start, end];
    if(nums.empty()) return 0;
    int count = 0;
    for(int i = 0; i < nums.size(); i++)
    {
        if(nums[i] >= start && nums[i] <= end)
        {
            count++;
        }
    }
    return count;
}
```

## 4、二维数组中的查找

> 在一个二维数组中，每一行从左到右递增、每一列从上到下递增，完成函数，对一个输入整数进行查找。

1	2	8	   9

2	4	9	  12

4	7	10	13

6	8	11	15

从右上角开始查找，每次排除一行或者一列，另一种形式的二叉搜索树

```c++
bool Find(vector<vector<int>> &matrix, int number)
{
    int found = false;
    int r = matrix.size();
    if(r <= 0) return false;
    int c = matrix[0].size();
    if(c <= 0) return false;
    
    int i = 0;
    int j = c - 1;
    while(i < r && j >= 0)
    {
        if(matrix[i][j] == number)
        {
            found = true;
        }
        else if(matrix[i][j] < number)
        {
            i++;
        }
        else
        {
            j--;
        }
    }
    
    return found;
}
```

空间复杂度`O(1)`，时间复杂度`O(N^2)`

## 5、替换空格

> 替换字符串中的空格为`%20`

在字符串中间插入字符，需要移动后续所有字符。

遍历一遍，统计空格数量，分配新的空间大小，从后往前插入



用例考虑：

- 正常用例：字符串中间有空格的字符串、字符串开头有空格的字符串、字符串有连续多个空格
- 特殊用例：字符串中没有空格，字符串全是空格
- 异常用例：字符串为空



```c++
void ReplaceBlank(char str[], int length)
{
    //length 为字符数组str的总容量
    if(str==nullptr || length <= 0) 
       	return;
    
    int originalLength = 0;
    int numberOfBlank = 0;
    int i = 0;
    while(str[i] != '\0')
    {
        originalLength++;
        if(str[i] == ' ')
        {
            numberOfBlank++;
        }
        i++;
    }
    
    int newLength = originalLength + numberOfBlank * 2;
    if(numberofBlank == 0) 
       	return;
    if(newLength < length)
    	return;
    
    int indexOfOriginal = originalLength;
    int indexOfNew = newLength;
    
    while(indexOfOriginal >= 0 && indexOfNew >= 0)
    {
        if(str[indexOfOriginal] == ' ')
        {
            str[indexOfNew--] = '0';
            str[indexOfNew--] = '2';
            str[indexOfNew--] = '%';
        }
        else
        {
            str[indexOfNew--] = str[indexOfOriginal];            
        }
        
        indexOfOriginal--;
    }
}
```

类似问题，排序合并数组B到数组A中，数组A中的空间能够容纳B

> 举一反三
>
> 在合并两个数组时，如果从前往后复制每个元素需要重复移动后续数字多次，可以考虑从后往前复制，需要注意的时，无论是从前往后还是从后往前，不移动的前提是不会占用其他未遍历元素的空间。

## 6、从尾到头打印链表

典型的先进后出，可以使用栈或者递归，递归代码简单，但递归函数栈消耗大，且可能导致函数栈调用溢出。

```c++
//链表定义
struct ListNode
{
  	int val;
    ListNode* next;
};

void PrintListReversingly(ListNode *pHead)
{
    std::stack<ListNode*> stNodes;
    
    ListNode* pNode = pHead;
    while(pNode != nullptr)
    {
        stNodes.push(pNode);
       	pNode = pNode->next;
    }
    
    while(!stNodes.empty())
    {
        pNode = stNodes.top();
        cout << pNode->val << endl;
        stNodes.pop();
    }
}
```



## 7、重建二叉树

> 根据二叉树的前序遍历和中序遍历，重建二叉树
>
> 前序遍历：{1，2，4，7，3，5，6，8}
>
> 中序遍历：{4，7，2，1，5，3，8，6}

前序遍历中的结点，在中序遍历中的位置，其左测是其左子树，右侧是其右子树

思考问题：

- 数据是否合法，有可能无法构建二叉树吗？

```c++

int preIndex = 0;

TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    if(preorder.empty() || inorder.empty()) return nullptr;

    return bulidTreeCore(preorder, inorder, 0, inorder.size() - 1);
}

TreeNode* bulidTreeCore(vector<int> &preorder, vector<int>& inorder, int leftIn, int rightIn)
{        
    TreeNode* root = new TreeNode(preorder[preIndex]);

    if(preIndex == preorder.size() - 1 || leftIn == rightIn)
    {
        return root;
    }

    int inIndex = leftIn;
    while(inIndex <= rightIn && inorder[inIndex] != preorder[preIndex])
        inIndex++;
    if(inIndex - leftIn > 0)
    {
        ++preIndex;
        root->left = bulidTreeCore(preorder, inorder, leftIn, inIndex-1);
    }
    if(rightIn - inIndex > 0)
    {
        ++preIndex;
        root->right = bulidTreeCore(preorder, inorder, inIndex+1, rightIn);
    }

    return root;
}
```

