---
title: "逆波兰表达式"
date: 2022-02-02T21:33:41+08:00
draft: false
categories: [八股文,数据结构与算法]
---

```
/* This implementation does not implement composite functions, functions with variable number of arguments, and unary operators. */

while there are tokens to be read:
    read a token
    if the token is:
    - a number:
        put it into the output queue
    - a function:
        push it onto the operator stack 
    - an operator o1:
        while (
            there is an operator o2 other than the left parenthesis at the top
            of the operator stack, and (o2 has greater precedence than o1
            or they have the same precedence and o1 is left-associative)
        ):
            pop o2 from the operator stack into the output queue
        push o1 onto the operator stack
    - a left parenthesis (i.e. "("):
        push it onto the operator stack
    - a right parenthesis (i.e. ")"):
        while the operator at the top of the operator stack is not a left parenthesis:
            {assert the operator stack is not empty}
            /* If the stack runs out without finding a left parenthesis, then there are mismatched parentheses. */
            pop the operator from the operator stack into the output queue
        {assert there is a left parenthesis at the top of the operator stack}
        pop the left parenthesis from the operator stack and discard it
        if there is a function token at the top of the operator stack, then:
            pop the function from the operator stack into the output queue
/* After the while loop, pop the remaining items from the operator stack into the output queue. */
while there are tokens on the operator stack:
    /* If the operator token on the top of the stack is a parenthesis, then there are mismatched parentheses. */
    {assert the operator on top of the stack is not a (left) parenthesis}
    pop the operator from the operator stack onto the output queue
```

C++版本实现：

```cpp
#include <iostream>
#include <string>
#include <list>
#include <stack>
using namespace std;

bool _is_num(char c) {
    return c > '0' && c < '9';
}

int _get_op_priority(char c) {
    switch(c) {
        case '+':
        case '-':
            return 1;
        case '*':
        case '/':
            return 2;
        default:
            return 0;
    }
}

bool _is_op(char c) {
    return _get_op_priority(c) != 0;
}

int calc(const string& bds) {

    stack<char> op_st;
    list<string> suffix_bds;

    for (int i = 0; i < bds.length(); ++i) {
        if (bds[i] == ' ') {
             continue;
        } else if (_is_num(bds[i])) {
            string num;
            num.push_back(bds[i]);
            while (i+1 < bds.length() && _is_num(bds[i+1])) {
                num.push_back(bds[i+1]);
                ++i;
            }
            suffix_bds.push_back(num);
        } else if (_is_op(bds[i])) {
            while (!op_st.empty() && op_st.top() != '(' && _get_op_priority(op_st.top()) >= _get_op_priority(bds[i])) {
                suffix_bds.push_back(string(1, op_st.top()));
                op_st.pop();
            }
            op_st.push( bds[i] );

        } else if (bds[i] == '(') {
            op_st.push( bds[i] );

        } else {
            while (op_st.top() != '(') {
                suffix_bds.push_back(string(1, op_st.top()));
                op_st.pop();
            }
            op_st.pop();
        }
    }

    while (!op_st.empty()) {
        suffix_bds.push_back(string(1, op_st.top()));
        op_st.pop();
    }

    for (const string& sub_bds : suffix_bds) {
        cout << sub_bds << endl;
    }
    return 0;
}

int main() {
    
    calc("1+-2*-3");
    cout << "-----" << endl;
    calc("-1+( 2 -3) *2 /5 + (5333-3 )");
}
```

## 参考：
1. [Reverse Polish notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation)
2. [Shunting-yard algorithm](https://en.wikipedia.org/wiki/Shunting-yard_algorithm)