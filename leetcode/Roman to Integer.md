# 1. Roman to Integer


For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9. 
X can be placed before L (50) and C (100) to make 40 and 90. 
C can be placed before D (500) and M (1000) to make 400 and 900.
Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

Example 1:
```
Input: "III"
Output: 3
```

Example 2:
```
Input: "IV"
Output: 4
```

Example 3:
```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

## 1.1. my solution
```java
class Solution {
    public int romanToInt(String s) {
        int result =0;
        Map<String,Integer> map = new  HashMap<String,Integer>(){
            {
                put("I",1);
                put("V",5);
                put("X",10);
                put("L",50);
                put("C",100);
                put("D",500);
                put("M",1000);
                put("IV",4);
                put("IX",9);
                put("XL",40);
                put("XC",90);
                put("CD",400);
                put("CM",900);
            }
        };
        int n = s.length();
        int i=0;
        while(i<n){
            if(i+1!=n&&(s.charAt(i)!=s.charAt(i+1))&& map.containsKey(s.substring(i,i+2))){
            result +=map.get(s.substring(i,i+2));
                i+=2;
            }else{
                result +=map.get(s.substring(i,i+1));
                i++;
          }
        }
        return result;
    }
}

```