# 1. Regular Expression Matching

Given an input string (s) and a pattern (p), implement regular expression matching with support for '.' and '*'.

```
'.' Matches any single character.
'*' Matches zero or more of the preceding element.
```
The matching should cover the entire input string (not partial).

Note:

* s could be empty and contains only lowercase letters a-z.
* p could be empty and contains only lowercase letters a-z, and characters like . or *.

Example 1:

```
Input:
s = "aa"
p = "a"
Output: false
Explanation: "a" does not match the entire string "aa".
```

Example 2:

```
Input:
s = "aa"
p = "a*"
Output: true
Explanation: '*' means zero or more of the preceding element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".
```

Example 3:

```
Input:
s = "ab"
p = ".*"
Output: true
Explanation: ".*" means "zero or more (*) of any character (.)".
```

Example 4:

```
Input:
s = "aab"
p = "c*a*b"
Output: true
Explanation: c can be repeated 0 times, a can be repeated 1 time. Therefore, it matches "aab".
```

Example 5:

```
nput:
s = "mississippi"
p = "mis*is*p*."
Output: false
```

## 1.1. Recursion

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if(p.isEmpty()) return s.isEmpty();
        boolean first_Match =(!s.isEmpty() && (s.charAt(0) == p.charAt(0) || p.charAt(0)=='.'));
        if(p.length()>=2&&p.charAt(1)=='*'){
            return (isMatch(s,p.substring(2))||(first_Match&&isMatch(s.substring(1),p)));
        }
        else{
           return first_Match&&isMatch(s.substring(1),p.substring(1));
        }
    }
}
```

## 1.2. Dynamic Programming

```java
class Solution {
    public boolean isMatch(String text, String pattern) {
        boolean[][] dp = new boolean[text.length() + 1][pattern.length() + 1];
        dp[text.length()][pattern.length()] = true;

        for (int i = text.length(); i >= 0; i--){
            for (int j = pattern.length() - 1; j >= 0; j--){
                boolean first_match = (i < text.length() &&
                                       (pattern.charAt(j) == text.charAt(i) ||
                                        pattern.charAt(j) == '.'));
                if (j + 1 < pattern.length() && pattern.charAt(j+1) == '*'){
                    dp[i][j] = dp[i][j+2] || first_match && dp[i+1][j];
                } else {
                    dp[i][j] = first_match && dp[i+1][j+1];
                }
            }
        }
        return dp[0][0];
    }
}
```

```puml
start
:boolean[][] dp = new boolean[text.length() + 1][pattern.length() + 1];
:dp[text.length()][pattern.length()] = true;
:int i = text.length();
while(i>=0)
:int j = pattern.length() -1;
while(j>=0)
:boolean first_match = (i < text.length() &&(pattern.charAt(j) == text.charAt(i) || pattern.charAt(j) == '.'));
if(j+1 < pattern.length() && pattern.charAt(j+1) == '*') then(yes)
:dp[i][j] = dp[i][j+2] || first_match && dp[i+1][j];
else
:dp[i][j] = first_match && dp[i+1][j+1];
endif
:j--;
endwhile
:i--;
endwhile
:return dp[0][0];
stop
```


|     |  a  |  *  |     |
| --- | --- | --- | --- |
| a   | T    | F   | F   |
| a   | T    | F   | F   |
|     | T    | F    | T    |