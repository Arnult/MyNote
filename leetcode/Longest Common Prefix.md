# 1. Longest Common Prefix
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".

Example 1:
```
Input: ["flower","flow","flight"]
Output: "fl"
```

Example 2:

```
Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```

## 1.1. my solution

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs.length==0)
            return "";
        StringBuilder result = new StringBuilder();
        int flag=0;
        for(int j =0;j<strs[0].length();j++){
        for(int i=1;i<strs.length;i++){
            if(j+1>strs[i].length()||!strs[0].substring(0,j+1).equals(strs[i].substring(0,j+1))){
                flag=1;
                break;
              }
            }
            if(flag==0)
                result.append(strs[0].charAt(j));
            else if(flag==1)
                break;
       }
        return result.toString();
   }
}
```

## 1.2. Horizontal scanning

```java
 public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++)
        while (strs[i].indexOf(prefix) != 0) {
            prefix = prefix.substring(0, prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    return prefix;
}
```

```puml
start
if (strs.length == 0) then(yes)
:return "";
stop
endif
:String prefix = strs[0] , int i = 1;
while(i<strs.length)
:prefix = prefix.substring(0,prefix.length()-1);
if(prefix.isEmpty()) then(yes)
:return "";
stop
endif
:i++;
endwhile
:return prefix;
stop

```