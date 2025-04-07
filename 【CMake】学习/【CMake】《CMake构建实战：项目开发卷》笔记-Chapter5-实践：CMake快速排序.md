# 第5章　实践：CMake快速排序

到这里，终于将CMake中比较常用的命令都讲解完了。本章是本书中第一个实践章节。实践章节不会介绍任何的新内容，而是通过一个具体的项目来实战演练前面学习到的知识。鉴于之前我们一直把CMake当作通用脚本语言来学习，本章也不会含糊，直接实现一个经典算法：快速排序！

快速排序的基本原理很简单：找到数列的一个基准值，将数列分成比该值小和比该值大的两部分子数列；对这两部分子数列分别进行快速排序，也就是递归调用自身；将二者与基准值合并在一起作为快速排序后的结果。其具体代码如下所示。

```
# 数列划分
# 
# arr: 数列
# pivot: 基准值
# left: 划分后的子数列变量名（比基准值小的部分）
# right: 划分后的子数列变量名（比基准值大的部分）
function(partition arr pivot left right)
    # 遍历数列
    foreach(x ${arr})
        # 根据当前值与基准值的比较结果，分别将当前值追加到不同的子数列中
        if(${x} LESS ${pivot})
            list(APPEND _left ${x})
        else()
            list(APPEND _right ${x})
        endif()
    endforeach()

    # 将两个子数列定义到上层作用域的变量中
    set(${left} ${_left} PARENT_SCOPE)
    set(${right} ${_right} PARENT_SCOPE)
endfunction()

# 快速排序
# 
# input: 输入数列
# res: 存放排序结果的数列变量名
function(quick_sort input res)
    # 取数列长度
    list(LENGTH input input_len)

    # 若数列长度小于等于1，则无须排序，直接设置结果
    if(${input_len} LESS_EQUAL 1)
        set(${res} "${input}" PARENT_SCOPE)
        return()
    endif()

    # 取数列第一个元素作为基准值
    list(GET input 0 pivot)
    # 将基准值从数列中删掉，即从第2个元素开始取子数列
    list(SUBLIST input 1 -1 input)

    # 划分出两部分子数列
    partition("${input}" ${pivot} left right)
    # 递归调用自身，对两个子数列进行快速排序
    quick_sort("${left}" left)
    quick_sort("${right}" right)

    # 将比基准值小的子数列、基准值、比基准值大的子数列连接起来
    list(APPEND _res ${left} ${pivot} ${right})
    # 设置到上层作用域的结果变量中
    set(${res} "${_res}" PARENT_SCOPE)
endfunction()

# 接受命令行输入的参数
# 从第4个参数开始，因为要忽略以下前4个参数：
# "cmake" "-P" "快速排序.cmake" "--"
foreach(i RANGE 4 ${CMAKE_ARGC})
    list(APPEND input ${CMAKE_ARGV${i}})
endforeach()

message("排序前：${input}")
quick_sort("${input}" res)
message("排序后：${res}")
```

其中用到了`CMAKE_ARGC`和`CMAKE_ARGV<N>`变量，分别代表CMake脚本模式下命令行参数的个数及第N个参数的值（包括cmake命令行名称本身）。如果希望传递自定义参数到CMake脚本程序中，可以在调用`cmake -P`命令行时，在`--`后传递自定义参数。例如：

```
> cd CMake-Book/src/ch005
> cmake -P 快速排序.cmake -- 8 9 1 3 10 4 6 5 7 2
排序前：8;9;1;3;10;4;6;5;7;2
排序后：1;2;3;4;5;6;7;8;9;10
```
