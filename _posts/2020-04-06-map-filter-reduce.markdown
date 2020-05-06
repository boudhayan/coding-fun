---
layout: post
title:  "Map Filter Reduce FlatMap"
date:   2020-04-06 12:32:45 +0100
categories:
---

<br /><br />c swift:
{% highlight swift %}

//MARK: Custom Implmentation of map, filter, reduce, flatMap
public extension Array {
    //map
    func map2<T>(_ transform: (Element) -> T) -> [T] {
        var mapped = [T]()
        mapped.reserveCapacity(count)
        for element in self {
            mapped.append(transform(element))
        }
        return mapped
    }
    
    //filter
    func filter2(_ isIncluded: (Element) -> Bool) -> [Element] {
        var filtered = [Element]()
        for element in self where isIncluded(element) {
            filtered.append(element)
        }
        return filtered
    }
    
    //reduce
    func reduce2<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) -> Result) -> Result {
        var reduced = initialResult
        for element in self {
            reduced = nextPartialResult(reduced, element)
        }
        return reduced
    }
    
    //flatMap
    func flatMap2<T>(_ transform: (Element) -> [T]) -> [T] {
        var flatMapped = [T]()
        for element in self {
            flatMapped.append(contentsOf: transform(element))
        }
        return flatMapped
    }
}

//MARK: map, filter using reduce
public extension Array {
    func map3<T>(_ transform: (Element) -> T) -> [T] {
        reduce([]) { $0 + [transform($1)] }
    }
    
    func filter3(_ isIncluded: (Element) -> Bool) -> [Element] {
        reduce([]) { isIncluded($1) ? $0 + [$1] : $0 }
    }
}

{% endhighlight %}
