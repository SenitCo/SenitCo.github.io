---
title: 数组(Array)和排序
date: 2018-03-25
categories: Data Structure
tags: [code, array, sort]
---
数据结构与算法中数组（Array）问题总结归纳。
<!--more-->

### Merge Intervals
Given a collection of intervals, merge all overlapping intervals. For example, Given [1,3],[2,6],[8,10],[15,18], return [1,6],[8,10],[15,18].
```cpp
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */

//先按元组第一个元素排序，然后判断相邻元组是否存在重叠区间
vector<Interval> merge(vector<Interval>& intervals) 
{
    vector<Interval> result;
    int size = intervals.size();
    sort(intervals.begin(), intervals.end(), [](const Interval& r1, const Interval& r2) {return r1.start < r2.start; });
    for(int i = 0; i < size;)
    {
        int j = i + 1;
        for(; j < size && intervals[i].end >= intervals[j].start; j++)
        {
            if(intervals[j].end > intervals[i].end)
                intervals[i].end = intervals[j].end;
        }
        result.push_back(intervals[i]);
        i = j;
    }
    return result;
}

vector<Interval> merge(vector<Interval>& intervals) 
{
    if(intervals.size() <= 1)
        return intervals;
    vector<Interval> result;
    sort(intervals.begin(), intervals.end(), [](const Interval& r1, const Interval& r2) {return r1.start < r2.start; });
    Interval tmp(intervals[0]);
    for(int i = 1; i < intervals.size(); i++)
    {
        if(tmp.end >= intervals[i].start)
            tmp.end = max(tmp.end, intervals[i].end);
        else
        {
            result.push_back(tmp);
            tmp = intervals[i];
        }
    }
    result.push_back(tmp);  //添加最后一个元组
    return result;
}
```

### Insert Interval
Given a set of non-overlapping intervals, insert a new interval into the intervals (merge if necessary). You may assume that the intervals were initially sorted according to their start times.
Example 1: Given intervals [1,3],[6,9], insert and merge [2,5] in as [1,5],[6,9].
Example 2: Given [1,2],[3,5],[6,7],[8,10],[12,16], insert and merge [4,9] in as [1,2],[3,10],[12,16]. This is because the new interval [4,9] overlaps with [3,5],[6,7],[8,10].
```cpp
/**
先拷贝newInterval前的所有元素，然后比较新元素和数组中每一个元素的大小，start取较小值，end取较大值，
直到newInterval.end < intervals[i].start，然后拷贝剩余元素，很直观的方法
*/
vector<Interval> insert(vector<Interval>& intervals, Interval newInterval) 
{
    vector<Interval> result;
    int i = 0;
    for(; i < intervals.size() && intervals[i].end < newInterval.start; i++)
        result.push_back(intervals[i]);
    for(; i < intervals.size() && intervals[i].start <= newInterval.end; i++)
    {
        newInterval.start = min(newInterval.start, intervals[i].start);
        newInterval.end = max(newInterval.end, intervals[i].end);
    }
    result.push_back(newInterval);
    for(; i < intervals.size(); i++)
        result.push_back(intervals[i]);
    return result;
}
```
