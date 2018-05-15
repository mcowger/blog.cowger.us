---
layout: post
title: Being Careful With Benchmarks
author: mcowger
categories: [go,ruby,python,benchmarking]
published: true
comments: true
---

Warning: VERY technical post ahead

I read an interesting article last night from Kenny Coleman and Clint Kitson on the [merits of compiled / lower level languages](http://blog.emccode.com/2015/02/26/draw-nodejs-and-go-count-to-1-billlion-with-a-b/#more-106), and they had some fascinating views.

First, they noted that they felt that:

>But in terms of lines of code, if you can remove some of the magic under the covers and expose more of the code in the application then you have more control and thus can ensure better predictability.

I have to say, I don't agree with this at all.  If this were actually true, no one would use standard libraries, because we would all utilize the control we have over the low level parts of the application, and 'ensure' predictability.  The fact is, things that low level languages let you handle (memory management, socket control, etc) are incredibly hard, and that's part of why the various standard libraries exist.  If controlling what happened under the covers were incredibly valuable and *actually* lead to better predictability, we'd see more of it, honestly.  I would argue in the inverse - that building your own very often results in less reliability and predictability.

## The Benchmark

Either way, they decided to run a benchmark, where they would compare Go and Node.JS (why leave out Python, the language you started comparing to?), where their respective languages would run an empty loop in which it counts to a billion.  They got some impressive times:

---

| Go  | Node  |
|---|---|
| 400ms  | 1000ms  |

---

Now, this surprised me, because this, even with a compiled language, should take longer, so I decided to look a bit deeper.

###Go

I started with [Clint's Go example, from his GitHub](https://github.com/clintonskitson/go1billion), and modified it slightly (well, a lot, actually).  I removed all the code required for handling the HTTP connections, and the code for the multiple threads, to end up with the core of his application:

{% highlight go linenos %}
package main
var CountToPer     int
func main() {
  CountToPer = 10000000
  for i := 0; i < CountToPer; i++ {}
}
{% endhighlight %}

Now, anyone familiar with compiler design will recognize where I'm going.  Clint is using a simple for loop with no contents and a constant as a iterator.  Most decent compilers these days will recognize this as a `NOOP` and simply remove it from any compiled code.  I decided to run a quick test, and tested various values of `CountToPer`.  No matter what I used (I tried 0 to 10 billion), I got run times very similar to the 400ms...leading creedence to my theory that the loop thats at the center of this test is a `NOOP`.   So, I pulled out my [favorite decompiler](http://www.hopperapp.com) and took a look at the simplified version of the Go compiled application.  Here's the critical part (removing a bunch of padding, preable, `_text` sections, etc):

{% highlight nasm linenos %}

                     __rt0_amd64_darwin:
0000000000030370         lea        rsi, qword [ss:rsp-0x0+arg_0]               ; XREF=0x17b8
0000000000030375         mov        rdi, qword [ss:rsp-0x0+ret_addr]
0000000000030379         lea        rax, qword [ds:_main]                       ; _main
0000000000030380         jmp        rax                                         ; _main
0000000000030382         db  0x00 ; '.'
0000000000030383         db  0x00 ; '.'
0000000000030384         db  0x00 ; '.'
0000000000030385         db  0x00 ; '.'
0000000000030386         db  0x00 ; '.'
0000000000030387         db  0x00 ; '.'
0000000000030388         db  0x00 ; '.'
0000000000030389         db  0x00 ; '.'
000000000003038a         db  0x00 ; '.'
000000000003038b         db  0x00 ; '.'
000000000003038c         db  0x00 ; '.'
000000000003038d         db  0x00 ; '.'
000000000003038e         db  0x00 ; '.'
000000000003038f         db  0x00 ; '.'
                     _main:
0000000000030390         dq         0xffffffd619058d48                          ; XREF=__rt0_amd64_darwin+9, __rt0_amd64_darwin+16
                        ; endp
{% endhighlight %}

Now, for those of you who don't read assembly, here's whats important.  The method defined as `__rt0_amd64_darwin` is essentially the entry point for the program on a Mach064 platform like the Mac I'm writing this on, and all it does is load the address of the `main()` procedure into memory (`lea        rax, qword [ds:_main]`) and then execute that procedure (`jmp        rax`).

So what really matters is the `_main()` call, and its contents:

```
_main:
0000000000030390         dq         0xffffffd619058d48
```

Thats it - a variable is declared and set and thats it.  **The entirety of that loop has been optimized out**


And there we have the rub - the code being used to test performance is not testing performance of how fast the system can loop, but in fact the effective startup / initialization time.

In other words, this test, doesn't test anything.

### Node.JS

How about Node.js, from Kenny?  Well, debugging on that is a bit tougher, but what do we know about the Node.JS runtime? Well, per the Node.js site:

>Node.js® is a platform built on Chrome's JavaScript runtime

So, its [Chrome V8 Engine then](https://code.google.com/p/v8/).  Chrome is known for performance, and part of how it gets there is described on their pages:

>V8 compiles JavaScript source code directly into machine code when it is first executed

In other words, the V8 engine is very likely doing the same thing that the Go compiler is doing, and simply optimizing out the loop.  Its just harder for me to prove.  I ran the same test as I did before, cutting the app down as much as possible to its core:

{% highlight javascript linenos %}
for(var i = 0; i < 1000000000; i++) {}

{% endhighlight %}

And tried multiple loop values.  No matter what, I always completed in approx 500ms, indicating that the Chrome V8 interpreter had removed the loop altogether.

###Python

So now we've shown that the benchmark used here only effectively measures library initialization time, and nothing else.  So, lets try it in python!  Here's an equivalent:

{% highlight python linenos %}
BILLION = 1000000000
for i in xrange(BILLION):
    pass
{% endhighlight %}

I also used the [`pypy` python interpreter](http://pypy.org), which is a modern version of Python that performs some of the same tricks (code optimization, inlining, JIT) that Go and Node.JS do.  The standard `CPython` interpreter focuses more on correct-ness rather than speed, and doesn't perform these kinds of optimizations.

And the results?  Again, a consistent 800-900ms to execute (this is likely an artifact of the JIT compiler warmup time).

## Deployment Size

Clint's article also makes some claims around deployment size.

>When firing up the applications on run.pivotal.io we were able to see some basic stats.  For the memory footprint, the Go version weighted in at 10MB and the NodeJS version was 55MB.  Big deal? Not only does it represent a bigger memory footprint (5x bigger), but it likely will yield extra layers in getting to the CPU (high level statement).  How about the size of the container? 35MB for Go versus 32MB for NodeJS.  Wait what?  Bigger for Go?  This actually makes sense and brings us to dependencies.

There's a ton of assumptions here that aren't at all valid.  For example, while the memory footprint of Node.JS may have been 5.5x that of Go, its also entirely possible (and likely) that Node.js is loading more into memory (like, for example, and entire VM to just javascript).  But lets think about that - is that really 5x more?  Or is that 55MB of memory for Node.JS, and 10MB of memory for Go, *plus all the system shared libraries that Go has to lead (libc, etc)*, which are not counted in the CF statistics.

Again, looking at only the surface results in missing important parts.

Lastly, the suggestion that higher level languages result in extra layers in getting to the CPU is true only for the most basic and naive of modern languages.  Any decent language/runtime (including `python`, `go`, `node`, `ruby`, `C`) is able to get past these extra layers without issue, especially for CPU-bound applications.  I've clearly demonstrated this above with the code samples I've provided.


##Conclusion

The conclusion is that while Kenny and Clint have some valid points about the good / bad parts of a low level language, they ran into a very basic problem that many people have when running benchmarks (whether thats CPU, language, storage, etc).  Without somewhat intimate knowledge of the system under test, a synthetic naive benchmark is just that; naive.  Just like `dd if=/dev/zero of=/dev/sdb` is a terrible indicator of storage performance, and `specint` is a terrible indicator of overall CPU performance, basic looping benchmarks are a terrible indicator of langauge performance.

In other words: be careful how you benchmark.  And in case its not clear - I have the utmost respect for Kenny and Clint as engineers, and am just using them as an example here *because* I know how good they are and that this is just a silly mistake for them.

As always, interested in comments below!
