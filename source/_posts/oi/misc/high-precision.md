---
title: 杂项 - 高精度
katex: true
date: 2023-06-13 12:50:00
updated: 2023-06-13 12:50:00
description: 高精度的实现模板，包括存储、运算、以及关于边界处理的分析。
tags:
- Algo
- C++
categories:
- OI
---

## 前言

高精度四则运算模拟的都是人工计算的步骤，其中乘法和除法是一个为高精数字，一个为低精数字。所有的高精数字用 `vector<int>` 逆序存储，比如 `123` 会被存为 `[3, 2, 1]`。

## 高精度加

### 模板

模拟列竖式计算加法的过程。

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> add(vector<int>& a, vector<int>& b) {
    // ensure a.size() >= b.size()
    if (b.size() > a.size()) return add(b, a);

    vector<int> ans;
    int t = 0;
    for (int i = 0; i < a.size(); i++) {
        t += a[i];
        if (i < b.size()) t += b[i];
        ans.push_back(t % 10);
        t /= 10;
    }

    if (t) ans.push_back(t);
    return ans;
}

int main() {
    string a,b;
    cin >> a >> b;
    vector<int> A,B;
    for (int i = a.size() - 1; i >= 0; i--) A.push_back(a[i]-'0');
    for (int i = b.size() - 1; i >= 0; i--) B.push_back(b[i]-'0');

    vector<int> ans = add(A, B);
    for (int i = ans.size() - 1; i >= 0; i--) printf("%d", ans[i]);

    return 0;
}
```

注意储存时逆序排列，以及要加上最后的进位。

## 高精度减

### 模板

减法就要多考虑几种情况了。

```cpp
#include <bits/stdc++.h>
using namespace std;

// A >= B
bool cmp(vector<int>& A, vector<int>& B) {
	if (A.size() != B.size()) return A.size() > B.size();
	
	// Reverse
	for (int i = A.size() - 1; i >= 0; --i) {
		if (A[i] != B[i]) return A[i] > B[i];
	}

	return true;
}

// Requires A>=B
vector<int> sub(vector<int>& A, vector<int>& B) {
	vector<int> ans;
	
	int t = 0;
	for (int i = 0; i < A.size(); i++) {
		t = A[i] - t;
		if (i < B.size()) t -= B[i];
		
		ans.push_back((t + 10) % 10);
		if (t < 0) t = 1;
		else t = 0;
	}
	
	while (ans.size() > 1 && ans.back() == 0) ans.pop_back();

	return ans;
} 

int main() {
    string a,b;
    cin >> a >> b;
    vector<int> A,B;
    for (int i = a.size() - 1; i >= 0; i--) A.push_back(a[i]-'0');
    for (int i = b.size() - 1; i >= 0; i--) B.push_back(b[i]-'0');

	vector<int> ans;
	if (cmp(A, B)) ans = sub(A, B);
	else {
		ans = sub(B, A);
		printf("-");
	}
    for (int i = ans.size() - 1; i >= 0; i--) printf("%d", ans[i]);

    return 0;
}
```

### 分析

1. 比较两数大小时要从高位开始比较，由于我们储存本身就是逆序排列，所以要逆序遍历判断大小。

2. 处理减法的退位时用到变量 `t`，它既充当了退位的变量也作为当前位的运算结果，这里的 `(t + 10) % 10` 可以拆成下面的代码：

   ```cpp
   if (t < 0) ans.push_back(t+10);
   else ans.push_back(t);
   ```

   这两者是等价的。

3. 每位运算完成后判断当前位运算结果是否需要退位，退位 `t=1`，不退 `t=0`。

4. 计算完成后要去掉开头多余的 `0`，但是也要考虑到结果本身是 `0` 的情况，所以加上了 `size > 1` 这条约束。

5. 由于计算的是绝对值，输出时要手动判断是否加负号。

## 高精度乘

### 模板

大数字乘一个小数字。

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> mul(vector<int>& A, int B) {
	int t = 0;
	vector<int> ans;
	for (int i = 0; i < A.size() || t; i++) {
		if (i < A.size()) t = A[i] * B + t;
		ans.push_back(t % 10);
		t /= 10;
	}
	
	while (ans.size() > 1 && ans.back() == 0) ans.pop_back();
	return ans;
}

int main() {
	string a;
	vector<int> A;
	int B;
	cin >> a >> B;
	
	for (int i = a.size() - 1; i >= 0; --i) A.push_back(a[i] - '0');
	
	vector<int> ans = mul(A, B);
	for (int i = ans.size() - 1; i >= 0; --i) printf("%d", ans[i]);

	return 0;
} 
```

### 分析

1. 注意最后去掉多余的 `0`，与高精度减类似。
2. 这里的 `t` 的位数不确定，所以要循环多次取个位数，直接合并到 `for` 循环的条件中了。

## 高精度除

### 模板

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> div(vector<int>& A, int B, int& r) {
	vector<int> ans;
	r = 0;
	for (int i = A.size() - 1; i >= 0; --i) {
		r = r * 10 + A[i];
		ans.push_back(r / B);
		r %= B;
	}
	reverse(ans.begin(), ans.end());
	while (ans.size() > 1 && ans.back() == 0) ans.pop_back();
	return ans;
}

int main() {
	string a;
	vector<int> A;
	int B;
	cin >> a >> B;
	
	for (int i = a.size() - 1; i >= 0; --i) A.push_back(a[i] - '0');
	
	int r;
	vector<int> ans = div(A, B, r);
	for (int i = ans.size() - 1; i >= 0; --i) printf("%d", ans[i]);
	printf("\n%d\n", r);
	return 0;
} 
```

### 分析

1. 除法需要正序计算，每一次把上次余数落下来与当前位拼成一个数字，再去除以 `B` 得到这一位的商，然后取余留到下一位。
2. 结果是正序排列的，要用到 `algorithm` 头文件中的 `reverse` 函数将其调整为逆序。
3. 结果中需要把开头的 `0` 去掉。

