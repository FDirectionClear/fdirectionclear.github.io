---
layout: post
title: 种花问题
date: 2019-05-03 
tag: 日常算法练习
---

### 算法结果：
*   题目：假设你有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花卉不能种植在相邻的地块上，它们会争夺水源，两者都会死去。
给定一个花坛（表示为一个数组包含0和1，其中0表示没种植花，1表示种植了花），和一个数 n 。能否在不打破种植规则的情况下种入 n 朵花？能则返回True，不能则返回False。
* 难度： 中等
* 结果： 通过所有测试用例，执行用时击败95.93%的用户

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/can-place-flowers

### 算法分析
* 知识点：动态规划，也可不使用动态规划，可用简单的for循环代替。需要注意的是对边界条件的判断。
* 思路：递归的判断当前位置是否能够插花，一次性只考虑一个位置，只要找到插花的条件即可。
* 技巧点：当前如果已经插花，就不用从下一个位置进行递归，直接越过一个坑位即可，可以减少递归次数提高性能。
* 坑点：不要忘记如果当前位置为空位，但是上一个位置是1，那么依旧不能插花，且这时候不能越位检测。

### 算法源码
```js
let canPlaceFlowers = function(flowerbed/* 花坛数组 */,n /* 还有多少花没插 */,start = 0 /*每一轮递归的开始位置*/){
    if(n == 0){
        // 如果n为0则说明什么情况的花坛都符合条件
        return true
    }
    if(flowerbed.length == 1 && flowerbed[0] != 0){
        return false
    } 
    let result 
    // 获取第一个0的位置
    let index = flowerbed.indexOf(0,start) 
    if(index == -1 && n > 0){
        // 如果一个0都没有，并且还有花要插的时候n>0
        return false
    }
    if(
        (flowerbed[index-1] == 0 || !flowerbed[index-1]) 
        && 
        (flowerbed[index + 1] == 0 || !flowerbed[index + 1])
    ){
        // 如果当前0的前后都是0，那么符合插花条件
        // 如果当前未知前后有undefined，那么说明这个位置是数组的边界，只需要考虑一侧即可
        flowerbed[index] = 1
        result = canPlaceFlowers(flowerbed,--n,index + 2)
    } else {
        result = canPlaceFlowers(flowerbed,n,index + 1)
    }
    return result
} 
```

### 改进的算法
改进理由：上一个算法额外扩展了更多的空间（result,index...）。我们不许要这么多额外的空间。原地运算即可。并且上一个算法改变了主函数的参数数量。
```js
var canPlaceFlowers = function(flowerbed, n) {
    // 如果需要插花0朵，那肯定可以
    if(n === 0) return true
    // 如果插花不为0，花坛缺没有坑就不可以
    if(flowerbed.length === 0) return false
    const canArranged = (start) => {
        if(start > flowerbed.length - 1 && n > 0) {
            // 如果此时插花位置已经脱离花坛，但是还有花没插，就拒绝
            return false
        } else if(n > 0){
            if(
                flowerbed[start] === 0 && 
                flowerbed[start + 1] !== 1 
            ) {
                // 如果当前坑没花，并且下一个位置也没花
                if(flowerbed[start - 1] === 1 )
                    // 如果当前位置的前一位有花，那么就直接检测下一位
                    return canArranged(start + 1)
                // 如果当前位置前后都没花，就插上
                n --
            }
            // 如果当前位置有花，那么就越坑检测
            return canArranged(start + 2)
        } else {
            // 如果此时插花位置已经脱离花坛，但是没花插了，就成功
            return true
        }
    }
    return canArranged(0)
};
```
可以看到，加上了更多行的注释依旧能保持行数更少。

### 其他网友的更高明的答案
```js
export default (arr, n) => {
  // 计数器
  let max = 0
  // 右边界补充[0,0,0],最后一块地能不能种只取决于前面的是不是1，所以默认最后一块地的右侧是0（无须考虑右侧边界有阻碍）
  arr.push(0)
  for (let i = 0, len = arr.length - 1; i < len; i++) {
    if (arr[i] === 0) {
      if (i === 0 && arr[1] === 0) {
        max++
        i++
      } else if (arr[i - 1] === 0 && arr[i + 1] === 0) {
        max++
        i++
      }
    }
  }
  return max >= n
}

```
代码量比我改进后的还少3行，而且没用到递归，明显更简洁

