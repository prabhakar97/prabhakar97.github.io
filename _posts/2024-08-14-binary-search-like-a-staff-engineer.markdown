---
layout: post
title: "Binary Search like a Staff SWE"
date: 2024-08-14 05:55:00 -0730
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
{% endhighlight %}

The second overload uses a `BiFunction` instead of a `Function` and passes the current index of `mid` as the second parameter
so that in certain problems `mid` is made available to the `testFunc`. In this pattern of problems `testFunc` needs to check for a
condition with any of the elements adjacent to mid.

Here are the equivalent C# implementations.

{% highlight csharp %}
public static int BinSearch<T>(Func<T, int> testFunc, Func<int, T> elementAt, int start, int end)
{
    if (end < start) {
        return -1;
    }
    int mid = start + (end - start) / 2;
    int compResult = testFunc(elementAt(mid));
    if (compResult == 0)
    {
        return mid;
    }
    else if (compResult < 0)
    {
        return BinSearch(testFunc, elementAt, start, mid - 1);
    }
    return BinSearch(testFunc, elementAt, mid + 1, end);
}

public static int BinSearch<T>(Func<T, int, int> testFunc, Func<int, T> elementAt, int start, int end)
{
    if (end < start) {
        return -1;
    }
    int mid = start + (end - start) / 2;
    int compResult = testFunc(elementAt(mid), mid);
    if (compResult == 0)
    {
        return mid;
    }
    else if (compResult < 0)
    {
        return BinSearch(testFunc, elementAt, start, mid - 1);
    }
    return BinSearch(testFunc, elementAt, mid + 1, end);
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
}
{% endhighlight %}

Wow! What was that? Driver function was just three lines (excluding the bad input check) in Java which is considered a very verbose language. That's the magic of higher order functions :-)
[Submission Link](https://leetcode.com/problems/search-a-2d-matrix/submissions/1354960780/)
![Submission](/images/LC74_Accepted.png)

Let's try another one. LC 69 [Sqrt(x)](https://leetcode.com/problems/sqrtx/).

{% highlight java %}
import java.util.function.Function;
import java.util.function.IntFunction;
class Solution {

    public int mySqrt(int x) {
        if (x == 0 || x == 1)
            return x;
        IntFunction<Integer> elementAt = (i) -> i;
        Function<Integer, Integer> sqrtCheck = i -> {
            if (x / i == i || (x / (i + 1)  == i))
                return 0;
            else if (x / i > i)
                return 1;
            return -1;
        };
        return binSearch(sqrtCheck, elementAt, 0, x / 2 + 1);
    }
}
{% endhighlight %}

Let's try LC 162 [Find Peak Element](https://leetcode.com/problems/find-peak-element/).

{% highlight java %}
class Solution {
    public int findPeakElement(int[] nums) {
        IntFunction<Integer> elementAt = (i) -> nums[i];
        BiFunction<Integer, Integer, Integer> peakCheck = (elem, i) -> {
            if ((i == 0 || elem > elementAt.apply(i - 1)) &&
                (i == nums.length - 1 || elem > elementAt.apply(i + 1)))
                return 0;
            if (elem < elementAt.apply(i + 1))
                return 1;
            return -1;
        };
        return binSearch(peakCheck, elementAt, 0, nums.length - 1);
    }
}
{% endhighlight %}
[Submission Link](https://leetcode.com/problems/find-peak-element/submissions/1355459214/)

![Submission Stats](/images/LC162_Accepted.png)

Let's try LC 35 [Search Insert Position](https://leetcode.com/problems/search-insert-position/description/).

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

}
{% endhighlight %}

This too runs in 1ms and beats 100% of the solutions in runtime. Here's the [submission link](https://leetcode.com/problems/search-insert-position/submissions/1354993527/)
if you want to verify.
![Submission](/images/LC35_Accepted.png)

The driver function here is slightly longer with the indexCheck method taking up a few lines. Still, the program is very easy to understand!

Let's try LC 153 [Find min in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/). This
involves finding the index at which array is pivoted i.e. rotated and then returning the element at that index.

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
}
{% endhighlight %}

[Submission Link](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/submissions/1355057450/)


Let's try LC 33 [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/description/). This one
uses the logic for finding pivot element from the previous problem and then builds on top of that.

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
}
{% endhighlight %}
[Submission Link](https://leetcode.com/problems/search-in-rotated-sorted-array/submissions/1355048370/)

![AcceptedImage](/images/LC33_Accepted.png)

How about LC 981 [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/description/)
{% highlight csharp %}
public class TimeMap
{
    private struct Pair
    {
        public int Timestamp;
        public string Value;
    }
    private Dictionary<string, List<Pair>> data = new();

    public void Set(string key, string value, int timestamp)
    {
        List<Pair> list;
         try
         {
             list = data[key];
         }   
         catch (KeyNotFoundException)
         {
             list = new();
             data[key] = list;
         }
        list.Add(new Pair{Timestamp = timestamp, Value = value});
    }
    
    public string Get(string key, int timestamp)
    {
        try
        {
            var dataList = data[key];
            if (timestamp < dataList[0].Timestamp)
                return "";
            int foundIndex = BinSearch((elem, mid) =>
                {
                    if (elem.Timestamp == timestamp) 
                        return 0;
                    if  ((elem.Timestamp < timestamp) &&
                            ((mid == dataList.Count - 1) || dataList[mid + 1].Timestamp > timestamp))
                        return 0;
                    if (elem.Timestamp > timestamp)
                        return -1;
                    return 1;
                },
                (i) => dataList[i],
                0,
                dataList.Count - 1);
            if (foundIndex == -1)
                return "";
            return dataList[foundIndex].Value;
        }
        catch (KeyNotFoundException)
        {
            return "";
        }
    }
}
{% endhighlight %}

[Submission Link](https://leetcode.com/problems/time-based-key-value-store/submissions/1360430234/)

How about LC 875 [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)?

{% highlight csharp %}
public class Solution
{
    public int MinEatingSpeed(int[] piles, int h)
    {
        int max = piles.Max() + 1;
        Func<int, int> elementAt = (i) => i; 
        Func<int, int> testFunc = (mid) =>
        {
            if (!CanEatWithinHours(piles, h, mid))
                return 1;
            if (CanEatWithinHours(piles, h, mid))
            {
                if (mid == 1 || !CanEatWithinHours(piles, h, mid - 1))
                    return 0;
            }
            return -1;
        };
        return BinSearch(testFunc, elementAt, 1, max);
    }

    private static bool CanEatWithinHours(int[] piles, int permittedHours, int eatingSpeed)
    {
        int hoursToEat = 0;
        for (int i = 0; i < piles.Count(); i++)
        {
            bool finishEvenly = piles[i] % eatingSpeed == 0;
            hoursToEat += (piles[i] / eatingSpeed) + (finishEvenly ? 0 : 1);
        }
        return hoursToEat <= permittedHours;
    }
}
{% endhighlight %}
[Submission Link](https://leetcode.com/problems/koko-eating-bananas/submissions/1363612732/)

With these few examples communicating the idea, I leave it to the reader to solve other binary search problems with this generic template.
