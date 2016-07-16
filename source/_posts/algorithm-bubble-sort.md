---
title: 算法:冒泡排序
category: 算法
tags:
  - 算法
  - Ruby
date: 2016-07-10 10:40:29
---
冒泡排序算法的原理:
有一组数列A[0..n-1],重复遍历该数列,一次比较两个相邻的元素A[i]和A[i+1],如果A[i]<=A[i+1]则不需要交换位置,如果A[i]>A[i+1],则需要交换位置.
实现:
```ruby
require 'benchmark'

def bubble_sort!(args)
  for i in 0..(args.length-1)
    for j in 0..(args.length-i-2)
      if (args[j+1] <=> args[j]) == -1
        args[j], args[j+1] = args[j+1], args[j]
        # p args
      end
    end
  end
end

Benchmark.bm(7) do |x|
  x.report('冒泡排序:') { bubble_sort! [8,3,1,2,6,5,9,10,4] }
end
```
遍历过程
```ruby
[3, 1, 8, 2, 6, 5, 9, 10, 4]
[3, 1, 2, 8, 6, 5, 9, 10, 4]
[3, 1, 2, 6, 8, 5, 9, 10, 4]
[3, 1, 2, 6, 5, 8, 9, 10, 4]
[3, 1, 2, 6, 5, 8, 9, 4, 10]
[1, 3, 2, 6, 5, 8, 9, 4, 10]
[1, 2, 3, 6, 5, 8, 9, 4, 10]
[1, 2, 3, 5, 6, 8, 9, 4, 10]
[1, 2, 3, 5, 6, 8, 4, 9, 10]
[1, 2, 3, 5, 6, 4, 8, 9, 10]
[1, 2, 3, 5, 4, 6, 8, 9, 10]
[1, 2, 3, 4, 5, 6, 8, 9, 10]
```