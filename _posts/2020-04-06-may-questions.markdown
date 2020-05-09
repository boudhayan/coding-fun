---
layout: post
title:  "May 2020 Answers"
date:   2020-05-10 12:32:45 +0100
categories:
---

# 1.What are the differences between forEach and for loop in swift? advantages of forEach over for loop.


forEach can be more useful if the action you want to perform is a single function call on each element in a collection. Passing a function name to forEach instead of passing closure expression can lead to clearer and more concise code. For example, if you are writing a view controller and want to add an array of subviews to `self.view`, you can just use `views.forEach(view.addSubview)` as function type `(UIView) -> Void` is same as the `closure` type. 

There are some differences between `for` loop and `forEach` loop. If a for loop has a return statement in it, rewriting it with forEach can significantly change the code's behavior. Consider the following example, which is written using a for loop with where condition:

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

the return inside closure function does not return to the outer function. It only returns from the closure itself.In this particular case, we’d probably have found the bug, because the compiler generates a warning that the argument to the return statement is unused, but you shouldn’t rely on it finding every such issue.

so in some of the situations, like `addSubview` example `forEach` can be a nicer choice than `for` loop. If you see this type cases then use `for` loop, otherwise use normal `for` loop only.
