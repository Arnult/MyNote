# 1. ZigZag Conversion

The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

|     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- |
| P   | #    | A    | #   | H    | #   | N    |
| A    | p    | L    | S   | I    | I   | G   |
| Y    | #   | I   | #   | R   | #    | #   |

Example 1:

```
Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
```

Example 2:

```
Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:

P     I    N
A   L S  I G
Y A   H R
P     I
```

## 1.1. my Solution

```java
class Solution {
    public String convert(String s, int numRows) {
       if(numRows==1)
           return s;
        int n = s.length();
            if(n==1||n==0)
           return s;
        char table[][] = new char[numRows][n];
        int x=0;
        int b=0;
        for(int index=0;index<n;){
            for(int y = numRows-1 ;y>=0;y--){
                if(index<n)
                table[y][x]=s.charAt(index++);
            }
            x++;
            int interval=0;
            while(interval<numRows-2){
                int y=x-b;
                 if(index<n)
                table[y][x]=s.charAt(index++);
                x++;
                interval++;
            }
            b+=numRows-1;
        }
        String str =new String();
        for(int i =numRows-1;i>=0;i--){
            for(int j=0;j<n;j++){
                if(table[i][j]!=0)
                str+=table[i][j];
            }
        }
        return str;
    }
}
```
```puml
start
:int n = s.length();
if(n==1||n==0)
:retrun s;
stop
endif
:char table[][] = new char[numRows][n];
:int x =0,b=0;
:int index = 0;
while (index<n)
:int y =numRows-1;
while(y>=0)
if(index<n)
:table[y][x] = s.charAt(index++);
endif
endwhile
:x++;
:int interval = 0;
while(interval < numRows-2)
:int y = x-b;
if(index<n)
:table[y][x] = s.charAt(index++);
endif
:x++;
:interval++;
endwhile
:b+=numRows-1;
endwhile
stop
```


## 1.2. Sort by Row

```java
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        List<StringBuilder> rows = new ArrayList<>();
        for (int i = 0; i < Math.min(numRows, s.length()); i++)
            rows.add(new StringBuilder());

        int curRow = 0;
        boolean goingDown = false;

        for (char c : s.toCharArray()) {
            rows.get(curRow).append(c);
            if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
            curRow += goingDown ? 1 : -1;
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) ret.append(row);
        return ret.toString();
    }
}
```

```puml
start
if(numRows == 1)
:return s;
stop
endif
:List<StringBuilder> rows = new ArrayList<>();
:int i=0;
while(i<Math.min(numRows,s.length())
:rows.add(new StringBuilder());
:i++;
endwhile
:int curRow = 0;
:boolean goingDown = false;
:i=0;
while(i<s.length())
:rows.get(curRow).append(s.chaAt(i));
if(curRow == 0 || curRow == numRows - 1)
:goingDown = !goingDown;
endif
:curRow += goingDown ? 1 : -1;
:i++;
endwhile
:StringBuilder re = new StringBuilder();
:i=0;
while(i<rows.size())
:ret.append(rows.get(i));
:i++;
endwhile
:return ret.toString();
stop
```