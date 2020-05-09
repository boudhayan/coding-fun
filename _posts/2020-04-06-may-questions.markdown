---
layout: post
title:  "May 2020 Answers"
date:   2020-05-10 12:32:45 +0100
categories:
---

# 1. What are the differences between forEach and for loop in swift? advantages of forEach over for loop.


forEach can be more useful if the action you want to perform is a single function call on each element in a collection. Passing a function name to forEach instead of passing closure expression can lead to clearer and more concise code. For example, if you are writing a view controller and want to add an array of subviews to `self.view`, you can just use `views.forEach(view.addSubview)` as function type `(UIView) -> Void` is same as the `closure` type. 

There are some differences between `for` loop and `forEach` loop. If a for loop has a return statement in it, rewriting it with forEach can significantly change the code's behaviour. Consider the following example, which is written using a for loop with where condition:

{% highlight swift %}
public extension Array where Element: Equatable {
    func firstIndex(of element: Element) -> Int? {
        for idx in self.indices where self[idx] == element {
            return idx
        }
        return nil
    }
}
{% endhighlight %}

We can not directly replicate the same scenario using forEach, but we can rewrite(incorrectly) this like below-

{% highlight swift %}
public extension Array where Element: Equatable {
    func firstIndex(of element: Element) -> Int? {
        self.indices.filter { self[$0] == element }.forEach { return $0 }
        return nil
    }
}
{% endhighlight %}

the return inside closure function does not return to the outer function. It only returns from the closure itself. In this particular case, we’d probably have found the bug, because the compiler generates a warning that the argument to the return statement is unused, but you shouldn’t rely on it finding every such issue.

so in some of the situations, like `addSubview` example `forEach` can be a nicer choice than `for` loop. If you see this type cases then use `for` loop, otherwise use normal `for` loop only.

# 2. Implement map, filter, reduce, flatMap function. Implement map, filter using reduce function.

Below are the high-level implementations of map, filter, reduce and flatMap-

<br /><br /> swift:
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

# 3. How array subscripting restricts unauthorized memory access?

Modern operating systems run all applications in protected memory regions using a virtual memory manager. So it is not super easy to simply read or write to a location that exists in REAL space outside the region that has allocated/assigned to your process.

Read operation will rarely damage another process, however it can indirectly damage a process if you happen to read a KEY value used to encrypt, decrypt or validate a program. So reading out of bounds can have somewhat unexpected effects on your code if you are making decisions based on the data you are reading. 

The only way you could really DAMAGE something by writing to a location accessible by a memory address is if that memory address that you are writing to is actually a hardware register (a location that is not for data storage but for controlling some piece of hardware) not a RAM location. In all fact, you still won't normally damage something unless you are writing some one-time programmable location that is not re-writable (or something of that nature).

Now coming to swift, subscripting by index has been implemented like below-

{% highlight swift %}

@inlinable
  public subscript(index: Int) -> Element {
    get {
      let wasNativeTypeChecked = _hoistableIsNativeTypeChecked()
      let token = _checkSubscript(
        index, wasNativeTypeChecked: wasNativeTypeChecked)
      return _getElement(
        index, wasNativeTypeChecked: wasNativeTypeChecked,
        matchingSubscriptCheck: token)
    }
    _modify {
      _makeMutableAndUnique() // makes the array native, too
      _checkSubscript_native(index)
      let address = _buffer.subscriptBaseAddress + index
      yield &address.pointee
    }
  }
{% endhighlight %}

You can see subscript calls `_checkSubscript` method to make sure the index is in range and wasNativeTypeChecked is valid. Now if we see `_checkSubscript` method implementation, basically for read, write operation in the array this method checks the given index is valid for subscripting, e.g: `0 <= index < count` otherwise it throws `precondition` failure message `index out of range` and the application terminates.

{% highlight swift %}
/// Check that the given `index` is valid for subscripting, i.e.
  /// `0 ≤ index < count`.
  @inlinable
  @inline(__always)
  internal func _checkSubscript_native(_ index: Int) {
    _ = _checkSubscript(index, wasNativeTypeChecked: true)
  }
  /// Check that the given `index` is valid for subscripting, i.e.
  /// `0 ≤ index < count`.
  @inlinable
  @_semantics("array.check_subscript")
  public // @testable
  func _checkSubscript(
    _ index: Int, wasNativeTypeChecked: Bool
  ) -> _DependenceToken {
#if _runtime(_ObjC)
    _buffer._checkInoutAndNativeTypeCheckedBounds(
      index, wasNativeTypeChecked: wasNativeTypeChecked)
#else
    _buffer._checkValidSubscript(index)
#endif
    return _DependenceToken()
  }
  /// Check that the specified `index` is valid, i.e. `0 ≤ index ≤ count`.
  @inlinable
  @_semantics("array.check_index")
  internal func _checkIndex(_ index: Int) {
    _precondition(index <= endIndex, "Array index is out of range")
    _precondition(index >= startIndex, "Negative Array index is out of range")
  }
{% endhighlight %}

# 4. What are the differences between error and exception?


Exceptions cause applications to crash if you do not handle it properly. It generally occurs when you try to operate on an object incorrectly, such as trying to access an out of bounds index from an array. 
Unhandled exceptions in your live apps must be avoided at all cost. Customers will not be happy if your app crashes and business can suffer from it. They are warnings to developers that a serious coding issue has occurred and needs to be fixed. They are expected to occur in the development phase of your app and provide information that will help you to solve the issue before you ship the app.

Errors are used in a quite different way from exceptions. They don’t get thrown, and they don’t cause the application to crash. Instead, they are created to hold information about failure, and then bubbled up through calling methods where it may be ‘handled’ in some way such as displaying a message to the user. Failures that result in errors being created can be considered common or even expected issues. A lot of the built-in cocoa errors are related to file system issues – such as files not being found, or running out of memory while writing.

