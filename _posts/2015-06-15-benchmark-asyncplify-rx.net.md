---
layout: post
title: "asyncplify vs rx.net"
description: "Performance benchmark between asyncplify in nodejs and rx in c#.net"
category: performance
tags: [asyncplify,rx.net,rxjs]
imagefeature: cover10.jpg
modified: 2015-06-15
---

<section id="table-of-contents" class="toc">
  <header>
    <h3 >Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

## Introduction
Last year, I've worked on a projet requiring lots of calculations to calculate financial indicators. The amount of data to process was huge.
I managed to get it worked using Rx.Net. Coming from a .Net/C# background, it was a natural fit. 

After a year of working on nodejs and angularjs, I felt in love with javascript. I decided test rxjs but the performance was not convincing me.
I decided to implement my own framework based on rxjs but with performance in mind.

The asyncplify framework was born.

## Machine

- Microsoft Windows 8.1 Pro 64Bit
- Intel i7 2.20GHz
- 12 GB Memory
- .Net Framework 4.5.1
- nodejs 0.12.4

## Results
![results]({{ site.url }}/images/asyncplify-rxnet-rxjs.jpg)

## Tables

<div class="row">
    <div class="large-12 columns">
        <table>
          <thead>
            <tr>
              <th></th>
              <th>rx.net</th>
              <th>rxjs</th>
              <th>asyncplify</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>sum</td>
              <td>1497ms</td>
              <td>930ms</td>
              <td>19ms</td>
            </tr>
            <tr>
              <td>count</td>
              <td>1360ms</td>
              <td>1024ms</td>
              <td>5ms</td>
            </tr>
            <tr>
              <td>groupBy/flatMap</td>
              <td>1491ms</td>
              <td>1538ms</td>
              <td>37ms</td>
            </tr>
            <tr>
              <td>multiple sum/subscribe</td>
              <td>21ms</td>
              <td>961ms</td>
              <td>3ms</td>
            </tr>
          </tbody>
        </table>
    </div>
</div>

## Code used for the test

### rx.net code
{% highlight c# %}
using System;
using System.Diagnostics;
using System.Reactive.Linq;

namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            var w = Stopwatch.StartNew();

            Observable
                .Range(0, 1000000)
                .Sum(v => (long)v)
                .Subscribe(x => Console.WriteLine("sum = {0} in {1}ms", x, w.ElapsedMilliseconds));

            w.Restart();

            Observable
                .Range(0, 1000000)
                .Count()
                .Subscribe(x => Console.WriteLine("count = {0} in {1}ms", x, w.ElapsedMilliseconds));

            w.Restart();

            Observable
                .Range(0, 1000000)
                .GroupBy(x => x % 10)
                .SelectMany(g => g)
                .Subscribe(_ => { }, () => Console.WriteLine("groupBy/flatMap in {0}ms", w.ElapsedMilliseconds));
                
            w.Restart();

            for (var i = 0; i < 1000; i++)
            {
                Observable
                    .Range(0, 10)
                    .Sum()
                    .Subscribe();
            }

            Console.WriteLine("multiple sum {0}ms", w.ElapsedMilliseconds);

            Console.ReadLine();
        }
    }
}
{% endhighlight c# %}

Used Rx.net version 2.2.5. I have compile this code using visual studio .net 2013 in release mode 
and start the program without a debugger attached.

### rxjs code
{% highlight js %}
var rx = require('rx').Observable;
var d = new Date();
	
rx
	.range(0, 1000000)
	.sum()
	.subscribe(function (x) { console.log('sum = %d in %dms', x, new Date() - d); });
	
d = new Date();
	
rx
	.range(0, 1000000)
	.count()
	.subscribe(function (x) { console.log('count = %d in %dms', x, new Date() - d); });
	
d = new Date();
	
rx
	.range(0, 1000000)
	.groupBy(function (x) { return x % 10; })
	.flatMap(function (g) { return g; })
	.subscribe(function () {}, function () {}, function () { console.log('groupBy/flatMap in %dms', new Date() - d); });
    
d = new Date();
	
for (var i = 0; i < 1000; i++) {
	rx
		.range(0, 1000)
		.sum()
		.subscribe();
}

console.log("multiple sum in %dms", new Date() - d);
{% endhighlight js %}

used rx version 2.5.3

### asyncplify code

{% highlight js %}
var asyncplify = require('asyncplify');
var d = new Date();

asyncplify
	.range(1000000)
	.sum()
	.subscribe(function (x) { console.log('sum = %d in %dms', x, new Date() - d); });
	
d = new Date();
	
asyncplify
	.range(1000000)
	.count()
	.subscribe(function (x) { console.log('count = %d in %dms', x, new Date() - d); });
	
d = new Date();
	
asyncplify
	.range(1000000)
	.groupBy(function (x) { return x % 10; })
	.flatMap(function (g) { return g; })
	.subscribe({ end: function () { console.log('groupBy/flatMap in %dms', new Date() - d); } });
    
d = new Date();

for (var i = 0; i < 1000; i++) {
	asyncplify
		.range(10)
		.sum()
		.subscribe();
}

console.log("multiple sum in %dms", new Date() - d);
{% endhighlight js %}

used asyncplify version 0.4.2 

## Conclusion
We can see that asyncplify is fast, faster than the 2 other rx framework.
rxjs is in most case faster than rx.net but when using many rxjs subscription, rx.net is faster than rxjs.
After seeing such positive result, I will continue to develop asyncplify.

God bless you...
Dany
