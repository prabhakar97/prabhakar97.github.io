---
layout: post
title: "Binary Search like a Staff SWE"
date: 2024-08-13 05:55:00 -0730
comments: true
tags: [leetcode, binary-search, programming, computer-science]
---

### Intro

Every FAANG SWE is familiar with binary search. When it is time to code, most solutions, even by senior candidates appear to be
rote memorized and reproduced to find an integer (or not) in an array. These implementations are one offs and can't be reused for
other problems. This article attempts to build a generic implementation and then use this to solve multiple problems. What is better
than Leetcode problems to test the correctness and efficiency of this implementation?! That's what a FAANG Staff SWE would do! Right?

### Key Idea

I am not going to talk about binary search in great detail since by reaching until this point, you have proven you know its inner workings.
If not, please consult any of the umpteen number of references on this topic available on the interwebs.

Instead I am going to talk about how an abstract binary search implementation can be written that can be applied on multiple problems.

A binary search can be parameterized with the following:

* A comparison function, that can be executed on the middle element to decide whether we have found the target element or whether we should look in
left half of the search space, or the right half
* A function that gives us `i`th element in the list. For an array, it just gives the element at `i`th index. 
* A `startIndex` and an `endIndex` that set the boundaries of the list we are looking into.

With these parameters, let's attempt to write a generic implementation of binary search.

{% highlight java %}
/**
* Given a comparison function, a function to get element at provided index and the start and end bounds of the search space, returns the index
* of element that satisfies the comparison function.
* The `testFunc` should return 0 upon success, a negative integer upon indicating that the element is less than the middle element and positive
* integer indicating that the element is greater than the middle element.
* Returns the index of the element if found, else returns -1.
*/
public static <T> int binSearch(Function<T, Integer> testFunc, IntFunction<T> elementAt, int start, int end) {
    if (end < start) {
        return -1;
    }
    int mid = start + (end - start) / 2;
    int compResult = testFunc.apply(elementAt.apply(mid));
    if (compResult == 0) {
        return mid;
    } else if (compResult < 0) {
        return binSearch(testFunc, elementAt, start, mid - 1);
    }
    return binSearch(testFunc, elementAt, mid + 1, end);
}
{% endhighlight %}

Armed with this implementation, can we solve some common Leetcode problems that require binary search?

Let's try LC 74 [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/description/). 
{% highlight java %}
import java.util.function.Function;

class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        if (matrix.length == 0)
            return false;
        Function<Integer, Integer> testFunc = (elem) -> Integer.compare(target, elem);
        IntFunction<Integer> elementAt = (i) -> matrix[i / matrix[0].length][i % matrix[0].length];
        return binSearch(testFunc, elementAt, 0, matrix.length * matrix[0].length - 1) != -1;
    }

    public static <T> int binSearch(Function<T, Integer> testFunc, IntFunction<T> elementAt, int start, int end) {
        if (end < start) {
            return -1;
        }
        int mid = start + (end - start) / 2;
        int compResult = testFunc.apply(elementAt.apply(mid));
        if (compResult == 0) {
            return mid;
        } else if (compResult < 0) {
            return binSearch(testFunc, elementAt, start, mid - 1);
        }
        return binSearch(testFunc, elementAt, mid + 1, end);
    }
}
{% endhighlight %}

Wow! What was that? Driver function was just three lines (excluding the bad input check) in Java which is considered a very verbose language. That's the magic of higher order functions :-)
[Submission Link](https://leetcode.com/problems/search-a-2d-matrix/submissions/1354960780/)
![Submission](/images/LC74_Accepted.png)

Let's try LC 35 [Search Insert Position](https://leetcode.com/problems/search-insert-position/description/). This one is slightly
trickier to implement with our generic bin search function because in the comparison function we also need to compare with element
before mid position to ascertain the rightful place for the `target`. I modified the `testFunc` parameter to be a `BiFunction` instead of 
a `Function` so that we can make the `mid` index available inside the function which can be used to lookup the element just before the element
at mid index.

{% highlight java %}
import java.util.function.Function;
import java.util.function.BiFunction;

class Solution {
    public int searchInsert(int[] nums, int target) {
        // Get the edge cases out
        if (target > nums[nums.length - 1])
            return nums.length;
        if (target < nums[0])
            return 0;
        
        // For an integer array, elementAt should return element at i
        IntFunction<Integer> elementAt = (i) -> nums[i];
        BiFunction<Integer, Integer, Integer> indexCheck = (elem, mid) -> {
            if (elem == target || (elem > target && nums[mid - 1] < target)) {
                return 0;
            } else if (elem < target) {
                return 1;
            }
            return -1;
        };
        return binSearch(indexCheck, elementAt, 0, nums.length - 1);
    }

    public static <T> int binSearch(BiFunction<T, Integer, Integer> testFunc, IntFunction<T> elementAt, int start, int end) {
        if (end < start) {
            return -1;
        }
        int mid = start + (end - start) / 2;
        int compResult = testFunc.apply(elementAt.apply(mid), mid);
        if (compResult == 0) {
            return mid;
        } else if (compResult < 0) {
            return binSearch(testFunc, elementAt, start, mid - 1);
        }
        return binSearch(testFunc, elementAt, mid + 1, end);
    }
}
{% endhighlight %}

This too runs in 1ms and beats 100% of the solutions in runtime. Here's the [submission link](https://leetcode.com/problems/search-insert-position/submissions/1354993527/)
if you want to verify.
![Submission](/images/LC35_Accepted.png)

The driver function here is slightly longer with the indexCheck method taking up a few lines. Still, the program is very easy to understand!

Let's try LC 153 [Find min in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/submissions/1355057450/)

{% highlight java %}
import java.util.function.Function;
import java.util.function.IntFunction;

class Solution {
    public int findMin(int[] nums) {
        if (nums.length == 0) {
            return -1;
        }

        // Search the pivot index first
        int pivotIndex = -1;
        if (nums[0] < nums[nums.length - 1]) {
            // array is not pivoted
            pivotIndex = 0;
        }
        if (pivotIndex == -1) {
            IntFunction<Integer> elementAt = (i) -> nums[i];
            BiFunction<Integer, Integer, Integer> pivotCheck = (elem, mid) -> {
                if (mid > 0 && nums[mid - 1] > elem)
                    return 0;
                else if (elem < nums[nums.length - 1] && elem < nums[0])
                    return -1;
                else if ((elem > nums[0] && elem > nums[nums.length - 1]) || (mid < nums.length - 1 && nums[mid + 1] < elem))
                    return 1;
                return 0;
            };
            pivotIndex = binSearch(pivotCheck, elementAt, 0, nums.length - 1);
        }
        return nums[pivotIndex];
    }

    public static <T> int binSearch(BiFunction<T, Integer, Integer> testFunc, IntFunction<T> elementAt, int start, int end) {
        if (end < start) {
            return -1;
        }
        int mid = start + (end - start) / 2;
        int compResult = testFunc.apply(elementAt.apply(mid), mid);
        if (compResult == 0) {
            return mid;
        } else if (compResult < 0) {
            return binSearch(testFunc, elementAt, start, mid - 1);
        }
        return binSearch(testFunc, elementAt, mid + 1, end);
    }
}
{% endhighlight %}

[Submission Link](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/submissions/1355057450/)


Let's try LC 33 [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/description/). This one
uses the logic for finding pivot element and then builds on top of that.

{% highlight java %}
import java.util.function.Function;
import java.util.function.IntFunction;

class Solution {
    public int search(int[] nums, int target) {
        if (nums.length == 0) {
            return -1;
        }

        // Search the pivot index first
        int pivotIndex = -1;
        if (nums[0] < nums[nums.length - 1]) {
            // array is not pivoted
            pivotIndex = 0;
        }
        if (pivotIndex == -1) {
            IntFunction<Integer> elementAt = (i) -> nums[i];
            BiFunction<Integer, Integer, Integer> pivotCheck = (elem, mid) -> {
                if (mid > 0 && nums[mid - 1] > elem)
                    return 0;
                else if (elem < nums[nums.length - 1] && elem < nums[0])
                    return -1;
                else if ((elem > nums[0] && elem > nums[nums.length - 1]) || (mid < nums.length - 1 && nums[mid + 1] < elem))
                    return 1;
                return 0;
            };
            pivotIndex = binSearch(pivotCheck, elementAt, 0, nums.length - 1);
        }
        final int pivot = pivotIndex;
        IntFunction<Integer> elementAt = (i) -> nums[(i + pivot) % nums.length];
        BiFunction<Integer, Integer, Integer> numCheck = (elem, mid) -> Integer.compare(target, elem);
        int searchResult = binSearch(numCheck, elementAt, 0, nums.length - 1);
        return searchResult >= 0 ? (searchResult + pivot) % nums.length : searchResult;
    }

    public static <T> int binSearch(BiFunction<T, Integer, Integer> testFunc, IntFunction<T> elementAt, int start, int end) {
        if (end < start) {
            return -1;
        }
        int mid = start + (end - start) / 2;
        int compResult = testFunc.apply(elementAt.apply(mid), mid);
        if (compResult == 0) {
            return mid;
        } else if (compResult < 0) {
            return binSearch(testFunc, elementAt, start, mid - 1);
        }
        return binSearch(testFunc, elementAt, mid + 1, end);
    }
}
{% endhighlight %}
[Submission Link](https://leetcode.com/problems/search-in-rotated-sorted-array/submissions/1355048370/)

![AcceptedImage](/images/LC33_Accepted.png)

With these few examples communicating the idea, I leave it to the reader to solve other binary search problems with this generic template.
