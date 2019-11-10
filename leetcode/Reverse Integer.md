# 1. Reverse Integer

Given a 32-bit signed integer, reverse digits of an integer.

Example 1:
```
Input: 123
Output: 321
```

Example 2:
```
Input: -123
Output: -321
```

Example 3:
```
nput: 120
Output: 21
```

## 1.1. my solution

```java
class Solution {
    public int reverse(int x) {
        Stack<Integer> stack = new Stack<>();
        String sign = x<0?"-":"";
        int count =0;
        while(x!=0){
            int val =x%10;
            stack.push(Math.abs(val));
            x/=10;
        }
        String result =new String();
              for(Integer var: stack){
                  result=""+result+var;
              }
        result=sign+result;
        int answer=0;
        try{
        answer= Integer.parseInt(result);//耗费资源！！！！！
        }catch(Exception e){
            answer=0;
        }
        finally{
            return answer;
        }
    }
}
```

## 1.2. sao Method

```java
class Solution {
  public int reverse(int x) {
     String reversed = new StringBuilder().append(Math.abs(x)).reverse().toString();
   try {
        return (x < 0) ? Integer.parseInt(reversed) * -1 : Integer.parseInt(reversed);
       } catch (NumberFormatException e) {
    return 0;
    }
 }
}
```

## 1.3. Pop and Push Digits & Check before Overflow

```java
class Solution {
    public int reverse(int x) {
        int rev = 0;
        while (x != 0) {
            int pop = x % 10;
            x /= 10;
            if (rev > Integer.MAX_VALUE/10 || (rev == Integer.MAX_VALUE / 10 && pop > 7)) return 0;
            if (rev < Integer.MIN_VALUE/10 || (rev == Integer.MIN_VALUE / 10 && pop < -8)) return 0;
            rev = rev * 10 + pop;
        }
        return rev;
    }
}
```

```puml
start
:int rev = 0;
while(x != 0)
:int pop = x%10;
:x /= 10;
if(rev > Integer.MAX_VALUE/10 || (rev == Integer.MAX_VALUE / 10 && pop > 7)) then
:return 0;
stop
endif
if(rev < Integer.MIN_VALUE/10 || (rev == Integer.MIN_VALUE / 10 && pop < -8)) then
:return 0;
stop
endif
:rev = rev * 10 + pop;
endwhile
:return rev;
stop
```

2^31 - 1= 2147483647 
-2^31 = -2147483648 
