# 1. 3Sum

Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note:

The solution set must not contain duplicate triplets.

Example:

```
Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

## 1.1. solution

```java

class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
     List<List<Integer>> solutionSet = new ArrayList<List<Integer>>();        
        Arrays.sort(nums);
        for(int a = 0; a < nums.length - 2; a++) {
            if(a != 0 && nums[a] == nums[a - 1]) continue;
            int b = a + 1;
            int c = nums.length - 1;
            while (b < c) {
                if(nums[a] + nums[b] + nums[c] == 0) {
                    solutionSet.add(Arrays.asList(nums[a], nums[b], nums[c]));
                    b++;
                    while(b < c && nums[b] == nums[b - 1]) b++;
                } else if(nums[a] + nums[b] + nums[c] < 0) {
                    b++;
                } else {
                    c--;
                }
            }
        }
        return solutionSet;
      }
    }
```

```puml
start
:List<List<Integer>> solutionSet = new ArrayList<List<Integer>>();
:Arrays.sort(nums);
while(a<nums.length - 2 )
if(a!=0 && nums[a] == nums[a-1]) then(yes)
:continue;
endif
:int b = a+1;
:int c = nums.length -1;
while(b<c)
if(nums[a]+nums[b]+nums[c] == 0) then(yes)
:solutionSet.add(Arrays.asList(nums[a], nums[b], nums[c]));
:b++;
while(b<c && nums[b] == nums[b-1])
:b++;
endwhile
else if(nums[a]+nums[b]+nums[c] <0) then(yes)
:b++;
else
:c--;
endif
endwhile
:a++;
endwhile
:return solutionSet;
stop
```