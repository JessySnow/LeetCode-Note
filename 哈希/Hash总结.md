# Hash 总结

## Hash 一般可以进行时间复杂度上的优化，比如可以使用 Hash 将一个时间复杂度为 O(n)，的查找操作通过 HashMap 或者 HashSet 进行缓存，最终变成一个平均时间复杂度为 O(1) 的操作，这样就完成了一次降低时间复杂度的操作

## 运用 Hash 降低时间复杂度的方式有很多，通过 HashMap、HashSet 比较常见，两个数据结构内部通过一个 Hash函数 将元素映射到了一个 Object数组上，这两个数据结构能比较好的平衡时间复杂度和空间复杂度，但是在性能上有更好的选择，数组就是一个天然的 HashMap，通过索引和元素之间额映射数组可以很快的解决一些问题，但是因为大部分的题目不能准确计算出数组需要的长度或者数组的长度会很大，这两种情况就不太适合使用数组来模拟 HashMap 了

---

### 对于使用 Hash 进行优化时间复杂度的题目最棘手的一般是去重的操作，题目可能会要求最后的结果中不允许有重复的元素，去重一般有两种思路，一种对加入 HashMap 或者 HashSet 的元素记录下来，这样就又需要一个 HashSet；第二种思路则比较耗费时间，即排序，通过排序我们也能够保证不会出现重复的情况，见下解析

```plain text
    Hash 优化循环的内循环往往是一个查询 HashMap 或者 HashSet 的过程，在这一步我们无法对返回的结果进行去重，那么只能将去重的过程放在循环代码中。
    从最内层循环开始考虑这个去重的过程：假如最内存循环在某一次循环中取到了和上次一样的数据，那么我们能够肯定在查询 HashMap 或者 HashSet 时，一定会取出相同的元素，这就导致重复的出现，所以需要对该层循环进行去重，怎么去重？这里就要借助数组已经被排序这个特点了，由于数组是已经排序过的，我们只要保证这次取出的数据和上次不一样，就能够保证今后不会取出和之前一样的元素了。
    讲完了最内层的循环，其实向往层循环推广，这个解释也是可行的，对于每一层循环的代码，我们只要进行去重的操作就能够保证最终的结果不会出现重复
```

---

[LeetCode15:三数之和](https://leetcode.com/problems/3sum/)

#### 三数之和是一道乍一看能用 Hash 写出来，实际上用 Hash 进行去重十分复杂的题目，因为题目要求的结果中三元组不能出现重复的，所以不能如果要用 Hash 进行去重其比较三元组的每个元素的过程就是一个很大的开销，这题考虑使用**排序去重**结合**双指针**的思路，代码如下

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new LinkedList<List<Integer>>();
        Arrays.sort(nums);
        
        int len = nums.length;
        for(int i = 0; i < len; ++ i){
            // 去重
            if(i != 0 && nums[i] == nums[i - 1]) continue;
            int left = i + 1;
            int right = len - 1;
            int target = -nums[i];
            
            // 双指针循环进行时间复杂度上的优化
            while(left < right){
                // 去重
                if(left != i + 1 && nums[left] == nums[left - 1]) {
                    ++ left;
                    continue;
                }
                
                // 双指针走位判断
                if(nums[left] + nums[right] > target){
                    -- right;
                }else if(nums[left] + nums[right] < target){
                    ++ left;
                }else{
                    res.add(Arrays.asList(new Integer[]{nums[i], nums[left], nums[right]}));
                    ++ left;
                    -- right;
                }
            }
        }
        
        return res;
    }
}
```

---

[LeetCode18:四数之和](https://leetcode.com/problems/4sum/)

#### 四数之和的思路和三数之和一样，也是排序和双指针的思路，只不过因为是四数之和，所以需要三层循环才能够解决

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        List<List<Integer>> res = new LinkedList<>();   
        int len = nums.length;
        Arrays.sort(nums);
        
        for(int i = 0; i < len; ++ i){
            if(i != 0 && nums[i] == nums[i - 1]) continue;
            for(int j = i + 1; j < len; ++ j){
                if(j != i + 1 && nums[j] == nums[j - 1]) continue;
                
                int left = j + 1, right = len - 1;
                int innerTarget = target - (nums[i] + nums[j]);
                while(left < right){
                    if(left != j + 1 && nums[left] == nums[left - 1]){
                        ++ left;
                        continue;
                    }
                    
                    if(nums[left] + nums[right] > innerTarget) -- right;
                    else if(nums[left] + nums[right] < innerTarget) ++ left;
                    else{
                        res.add(new ArrayList<>(Arrays.asList(nums[i], nums[j], nums[left], nums[right])));
                        ++ left;
                        -- right;
                    }
                }
            }
        }
        
        return res;
    }
}
```

---

[LeetCode454:四数之和Ⅱ](https://leetcode.com/problems/4sum-ii/)

#### 四数之和Ⅱ和四数之和虽然都是求四个数的和，让其相加为某一个数，但是四数之和Ⅱ和四数之和又是完全不同的一类题目，四数之和Ⅱ更中的四个数字分别来自四个数组，且没有要求需要去重，即最后的三元组可以是重复的，使用 HashMap 将原本的四重循环拆分为 两个 二重循环，就是一个很直接而且高效的思路

```java
class Solution {
    public int fourSumCount(int[] nums1, int[] nums2, int[] nums3, int[] nums4) {
        HashMap<Integer, Integer> map = new HashMap<>();
        
        for(int num1 : nums1)
            for(int num2 : nums2)
                map.put(num1 + num2, map.getOrDefault(num1 + num2, 0) + 1);
        
        int res = 0;
        for(int num1 : nums3)
            for(int num2 : nums4)
                res += map.getOrDefault(-(num1 + num2), 0);
        
        return res;
    }
}
```

---

[LeetCode383:赎金信件](https://leetcode.com/problems/ransom-note/)

#### 这题就是一道可以使用数组进行 Hash 的题目，一般这类题目都有一个很明显的特征就是给定的元素的种类是有限的，比如本题中信件中的字母只有 26 个小写字母，那么使用一个大小为 26 的数组装下所有的字母和字母个数的映射即可

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        int[] count = new int[26];
        
        for(char ch : ransomNote.toCharArray())
            count[ch - 'a'] ++;
        
        for(char ch : magazine.toCharArray())
            count[ch - 'a'] --;
    
        for(int i = 0; i < count.length; ++ i)
            if(count[i] > 0)
                return false;
        
        return true;
    }
}
```
