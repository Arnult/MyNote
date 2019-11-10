# 1. Palindrome Number

Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

Example 1:

```
Input: 121
Output: true
```

Example 2:

```
Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
```

Example 3:

```
Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

## 1.1. solution

```java
class Solution {
    public boolean isPalindrome(int x) {
        int temp =x;
        if(x<0||(x%10 == 0 && x!=0))
            return false;
         int revertedNumber = 0;
        while(x !=0) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }
        return temp == revertedNumber;
    }
}
```

```puml
start
:int temp = x;
if(x<0||(x%10==0 && x!=0))
:return false;
endif
:int revertedNumber = 0;
while(x!=0)
:revertedNumber = revertedNumber*10 + x%10;
:x/=10;
endwhile
:return temp== revertedNumber;
stop
```