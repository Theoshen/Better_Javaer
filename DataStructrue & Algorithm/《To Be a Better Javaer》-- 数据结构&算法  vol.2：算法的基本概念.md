# 《To Be a Better Javaer》-- 数据结构&算法  vol.2：算法的基本概念

## 算法的基本概念

​	算法（ Algorithm ）是对特定问题求解步骤的一种描述，它是指令的有限序列， 其中的每条指令表示一个或多个操作。

### 算法的特性

1. **有穷性**：算法必须总在执行有穷步之后结束，且每一步都可以在有穷时间内完成。
2. **确定性**：算法中每条指令必须有明确的含义，对于相同的输入只能得出相同的输出。
3. **可行性**：算法中描述的操作都可以通过已经实现的基本运算执行有限次来实现。
4. **输入**：一个算法有零个或多个输入，这些输入取自于某个特定的对象的集合。
5. **输出**：一个算法有一个或多个输出，这些输出是与输入有着某种特定关系的量。

### 算法的目标

1. **正确性**：算法应该能够正确地解决求解问题。
2. **可读性**：算法应该具有良好的可读性。
3. **健壮性**：输入非法数据时，算法能适当地做出反应或进行处理，而不会产生莫名其妙的输出结果。
4. **效率与低存储量需求**：效率是指算法执行的时间，存储量需求是指算法执行过程中所需耍的最大存储空间，这两者都与问题的规模有关。

## 算法效率的度量

​	算法效率的度量时通过**时间复杂度**和**空间复杂度**来描述的。

- 时间维度：是指执行当前算法所消耗的时间，我们通常用「**时间复杂度**」来描述。
- 空间维度：是指执行当前算法需要占用多少内存空间，我们通常用「**空间复杂度**」来描述。

### 时间复杂度

​	时间复杂度实际上是一个函数，代表基本操作重复执行的次数，进而分析函数虽变量的变化来确定数量级，数量级用O表示，所以算法的时间复杂度为：

```
T(n)= O(f(n))；
```

​	称函数T(n)以f(n)为界或者称T(n)受限于f(n)。

- 取 f(n) 中随 n 增长最快的项，将其系数置为 1 作为时间复杂度的度量。例如， f(n) ＝ an^3+ bn^2 + cn 的时间复杂度为 O(n^3)。

​	如果一个问题的规模是n，解这一问题的某一算法所需要的时间为T(n)。T(n)称为这一算法的“时间复杂度”。算法的时间复杂度不仅依赖于问题的规模 n，也取决于待输入数据的性质（如输入数据元素的初始状态）。 例如，在数组 A[0...n-1 ］中，查找给定值 k 的算法大致如下：

```c
 i = n - 1;
 while( i >= 0 && (A[i] != k))
 i--;
 return i;
```

该算法中的频度不仅与问题规模 n 有关，而且与输入实例中 A 的各元素的取值及 k 的取值有关

1. 若 A 中没有与 k 相等的元素，则 `i--` 的频度 f(n) = n 。
2. 若 A 的最后一个元素等于 k，则 `i--`  的频度 f(n)是常数 0 。

> 最坏时间复杂度是指在最坏情况下，算法的时间复杂度。
> 平均时间复杂度是指所有可能输入实例在等概率出现的情况下，算法的期望运行时间。
> 最好时间复杂度是指在最好情况下，算法的时间复杂度。

###### 一般总是考虑在最坏情况下的时间复杂度， 以保证算法的运行时间不会比它更长。

​	分析时间复杂性，有两条规则：

- 加法规则：T(n)  =  T1(n)＋T2(n) = O(f(n)) + O(g(n)) = O(max(f(n),g(n)))

- 乘法规则：T(n)  =  T1(n) × T2(n) = O(f(n)) × O(g(n)) = O(f(n)×g(n)) 

  常见的渐近时间复杂度

  ```
  O(1) < O(log2 n) < O(n) < O(nlog2 n) < O(n^2) < O(n^3 ) < 0(2'') < O(n!) < O(n^n) 
  ```

### 空间复杂度

> 是对一个算法在运行过程中**临时**占用存储空间的度量，一个算法在计算机存储器上所占用的存储空间包括存储算法本身所占用的空间，算数和输入输出所占用的存储空间以及临时占用存储空间三个部分，算法的输入输出数据所占用的存储空间是由待解决的问题来决定的，通过参数表由调用函数而来，它随本算法的不同而改变，存储算法本身所占用的存储空间有算法的书写长短成正比。算法在运行过程中占用的临时空间由不同的算法决定。

​	算法的空间复杂度 S(n） 定义为该算法所耗费的存储空间，它是问题规模 n 的函数。记为

```
S(n) = O(g(n)) 
```



