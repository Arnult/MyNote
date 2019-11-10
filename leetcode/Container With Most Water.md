# 1. Container With Most Water

Given n non-negative integers a1, a2, ..., an , where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.


![question_11](_v_images/20191105153433477_820924221.jpg)

The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.

Example:

```
Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```

## 1.1. Brute Force

```java
class Solution {
    public int maxArea(int[] height) {
        int n = height.length;
        int maxArea=0;
        for(int i =0;i<n-1;i++){
            for(int j=i+1;j<n;j++){
                int he = height[i]<height[j]?height[i]:height[j];
                  maxArea= maxArea<(he*(j-i))?(he*(j-i)):maxArea;
            }
        }
        return maxArea;
    }
}
```

```puml
start
:int n = height.length;
:int maxArea = 0;
:int i = 0;
while(i<n-1)
:int j=i+1;
while(j<n)
:int he = height[i]<height[j]?height[i]:height[j];
:maxArea = maxArea<(he*(j-i))?(he*(j-i)):maxArea;
endwhile
endwhile
:return maxArea;
stop
```

## 1.2. Two Pointer Approach

```java
public class Solution {
    public int maxArea(int[] height) {
        int maxarea = 0, l = 0, r = height.length - 1;
        while (l < r) {
            maxarea = Math.max(maxarea, Math.min(height[l], height[r]) * (r - l));
            if (height[l] < height[r])
                l++;
            else
                r--;
        }
        return maxarea;
    }
}
```
```puml
start
:int maxarea = 0, l = 0, r= height.lenght - 1;
while(l<r)
:maxarea = Math.max(maxarea, Math.min(height[l],height[r])*(r-l));
if(height[l] < height[r]) then(yes)
:l++;
else
:r--;
endif
endwhile
:return maxarea;
stop
```