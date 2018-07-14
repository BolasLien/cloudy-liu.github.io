---
title: leetcode20-括号匹配
date: 2018-07-01 13:21:55
tags: [leetcode]
---

括号匹配习题 <!-- more --> 

### [题目](https://leetcode.com/problems/valid-parentheses/description/)

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

```
Input: "()"
Output: true

Input: "([)]"
Output: false
```

### 思路

题目说仅有各种左括号和右括号的组合。可以使用栈的数据结构，因为匹配的括号的话，一定是成对出现的，若将左括号一次压入栈，遇到匹配的右括号在弹出栈，则若是匹配的话，栈最后一定为空。

遍历字符串，当遇到左括号时候，将其压入栈中，当是右括号时，看是否栈为空，若是空则直接返回(")"，否则在看是否和栈顶元素匹配，若匹配，则出栈，若不匹配则直接返回false。遍历完成后，若栈为空，则返回true，否则返回false。

### Solution

基于以上的解法

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (int i = 0; i < s.length(); i++) {
        char currentChar = s.charAt(i);
        if ((currentChar == '(' || currentChar == '[' || currentChar == '{')) {
            stack.push(currentChar);
        } else if (stack.isEmpty()) {
            return false;
        } else {
            Character top = stack.peek();
            if ((top == '[' && currentChar == ']') || (top == '{' && currentChar == '}')
                    || (top == '(' && currentChar == ')')) {
                stack.pop();
            } else {
                return false;
            }
        }
    }
    return stack.isEmpty();
}
```

此题虽然AC了，但是看了 leetcode 讨论区，发现一个非常简洁的写法，它将其翻转过来，上面我们压入栈的是左括号，而他压入栈的是右括号，在遇到右括号时，弹出栈，判断是否弹出的栈顶元素就是当前扫描的右括号，若不是，则不是匹配的字串，返回false，最后遍历完成后，判断栈是否为空。代码简洁好多。

```java
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == '(') {
                stack.push(')');
            } else if (c == '[') {
                stack.push(']');
            } else if (c == '{') {
                stack.push('}');
            } else if (stack.isEmpty() || stack.pop() != c) {// consider opposite side, very smart
                return false;
            }
        }
        return stack.isEmpty();
    }
```

[完整source code](https://github.com/cloudy-liu/BDSA/blob/master/leetcode/20-Valid%20Parentheses/ValidParentheses.java)





