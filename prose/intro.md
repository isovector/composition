## Introduction

Early in my career, I received a prescient piece of advice from an interviewer
--- "stop writing code before you've thought through the problem." At time, I
balked; after all, I *thought* in code. I couldn't separate writing code from
thinking because they were one and the same thing. My approach to solving
algorithmic problems was to start with the innermost for-loop, and work my way
outwards, propagating required information from one place to another as I went.
The process of designing programs in this way was *exploratory;* I'd just write
down the first thing that came to mind, and flail around, madly hoping to make
it work.

As you can imagine, this was not a particularly good strategy for designing
algorithms.

I held it against this person for many years before I was able to shed my hubris
and see the kernel of truth in his advice. I had indeed conflated thinking about
algorithms with writing code, and this was not an asset like I had assumed, but
instead, a detriment.

Although it is no longer taken literally, the Sapir-Whorf Hypothesis is the idea
that the language we speak influences the thoughts we are capable of thinking.
While this might be disputed in the domain of natural language, it certainly
holds some truth when it comes to algorithm design. Programming languages were
not always as they are now. The traditional building blocks that we take for
granted, like `if`, `for`, and `while`, were not standardized in use until the
late 1980s. Before then,it wasn't uncommon to roll your own execution paths via
`goto` and `label`---a laughably barbaric practice in the 21st century. Despite
this, the idea of "structured programming" as it was known had been around since
1966, but took 20 years to find adoption. Clearly, the programmers of the 70s
and 80s thought about programming in a completely different way than
contemporary engineers do. If concepts as basic as `if` and `while` so long to
reach mass conscientiousness, it's worth considering what we might be missing in
the modern age.

Clearly, the programming techniques we understand do influence the programs we
know how to write.

Modern-day programming, when it isn't about gluing together code snippets found
on helpful websites, is instead about knowing a lot of clever algorithms that
other people wrote, and mentally iterating through them to see if any feel like
a good fit for the problem at hand. The more clever algorithms you know, the
more problems you can solve. But the curious among us would ask, where do these
clever algorithms come from in the first place? Each of the Canonical Algorithms
depends on a kernel of brilliant insight that seemingly cannot be replicated by
mere mortals. Is computer programming a discipline like Physics, where we must
wait patiently for Newton and Einstein to come down from on high and present us
with genius applications?

The central thesis of this book is a resounding *no* to that question.
Algorithms can indeed be designed *de novo* in the modern era, and the
difficulty that most of us find when attempting to do so is the result of having
poorly-adapted mental tools for thinking about programming. Thinking in code is
simply *too hard* to accomplish challenging tasks. It is too "in the weeds,"
and, to mix metaphors, misses the forest for the trees.

With very few exceptions, the human cognitive architecture prefers to think in
terms of non-overlapping hierarchies, where we have the ability to freely
separate sub-concerns, solve them individually, and unite the solutions into a
bigger whole. You can see this at work in hotly-debated dichotomies: was it the
chicken or the egg which came first? Is Pluto a planet? Why do we find it so
hard to accept that light is both a wave *and* a particle? The human mind likes
firm, discrete, separable categories, and we don't reason well when that crutch
is taken away from us.

Hierarchies are naturally modeled by trees (the data structure), and trees are
the one domain in which most programmers feel comfortable designing algorithms.
This is not a coincidence. Trees map much better onto our mental architecture
than arrays or graphs do, and thus problems on trees feel more accessible to us
than other problems do.

Fortunately, there are other data structures that map well to our mental
architecture, and better, all programming problems can be solved in terms of
these mentally-available tools. These tools don't map naturally into *software,*
because software as it's currently practiced, is a language for computers, not
for humans. Not all hope is lost however---there are standard procedures for
transforming these better mental tools into software.

Thus, the meta-algorithm for writing software is:

1. Decompose the problem into its constituent mental pieces
2. Solve the problem there
3. Mentally compile the solution into software

These three steps might be labeled *modeling,* *decomposition,* and
*optimization.* Standard software engineering procedure is to attempt to do all
three steps at once, which usually fails. By focusing on these three steps
independently, we can dramatically improve our effectiveness as authors of
algorithms. Better yet, the techniques can be taught individually, in a way that
their composition cannot be.

The situation is analogous to how we think about memory. On paper, RAM is laid
out as one giant array,[^memory] but we prefer to ignore this aspect and think
in terms of discrete objects. If we think about memory at all, we think in terms
of values, with explicit addresses and sizes. We think about if the memory is on
the stack or on the heap. But these are all abstractions that exist in our
mental maps, rather than in the territory of the physical RAM layout. At the end
of the day, it's all just one big array of bytes. The abstractions serve us well
and aid in writing easier-to-understand code than if we treated memory as one
giant array, explicitly indexing things by hand.

<!-- TODO(sandy): good opportunity here to hand compile something into memory -->

[^memory]: This is not true in reality, however. RAM is laid out in matrices of
  matrices, where one byte might be on a completely different chip than the byte
  with the next index.

And yet, this is exactly the same mistake we make when we reach for an array as
a data structure, or reuse a variable for convenience, or twiddle pointers. The
problem is an insufficient differentiation between the things the computer knows
how to run fast and the thoughts we'd prefer to think. Unfortunately, as a
discipline, we have learned to shape our thoughts for the benefit of the
computer, rather than bending the computer to fit our thoughts.


## Forget Everything You Know

The unfortunate side-effect of decomposing our understanding of problem solving
is that a lot of your hard-earned programming knowledge will not be helpful in
the short term. The problem solving techniques introduced in this book will seem
strange and perhaps roundabout, but there is a very good reason for them.

My favorite example of this is to consider the program that finds the biggest
number in an array:

```cpp
const int biggest(const int[] vals, const size_t n) {
  int result = INT_MIN;
  for (size_t i = 0; i < n; i++) {
    if (vals[i] > result) {
      result = vals[i];
    }
  }
  return result;
}
```

This is a very reasonable program. But now compare it to the very similar
problem of finding the *second* biggest number in an array:

```cpp
const int second_biggest(const int[] vals, const size_t n) {
  int bigger, biggest = INT_MIN;
  for (size_t i = 0; i < n; i++) {
    if (vals[i] > biggest) {
      bigger = biggest;
      biggest = vals[i];
    } else if (vals[i] > bigger) {
      bigger = vals[i];
    }
  }
  return bigger;
}
```

Despite computing almost exactly the same thing, the two programs are
dramatically different. Furthermore, it's clear that generalizing this approach
to find the $k$th biggest element in the array will require $O(k)$ work on the
part of the programmer.

We can instead write a program that works for any $k$, in essence, building a
$k$-sized buffer, and sorting the biggest elements in the array as we go. The
implementation given here does something like insertion-sort, but the exact
details aren't important for either the purposes of instruction nor for any
purpose whatsoever.

```cpp
const int kth_biggest(const int[] vals, const size_t n, const size_t k) {
  int bigs[k];
  for (size_t i = 0; i < k; i++) {
    bigs[i] = INT_MIN;
  }

  for (size_t i = 0; i < n; i++) {
    for (size_t j = 0; j < k; k++) {
      if (vals[i] > bigs[j]) {
        for (size_t jj = k - 1; jj >= j; jj--) {
          bigs[jj] = bigs[jj - 1];
        }
        bigs[j] = vals[i];
        break;
      }
    }
  }

  return bigs[k - 1];
}
```

Contrast this to a "naive" solution:

```cpp
const int kth_biggest(const int[] vals, const size_t n, const size_t k) {
  int copy_vals[n];
  mempcy(vals, copy_vals, sizeof(copy_vals));
  sort(copy_vals);
  // TODO(sandy): sort downwards
  return copy_vals[k - 1];
}
```

When put to the test, nobody ever comes up with this solution, despite the fact
that it is significantly less work to write and orders of magnitude easier to
verify the correctness of. Why is that? The sorting solution feels wasteful!
We're doing all of this extra work to sort the lower part of the array, and
that's *wasting precious CPU time.* Maybe that CPU time matters, but probably it
doesn't. If it does, it's easy enough to come back to this function later and
find an optimization that comes at the expense of clarity.

But even this argument depends on unwittingly-internalized ideas of how a
computer must behave. Who says `sort` must sort the whole array? Why can't it
sort just enough of the elements for the returned result to get the answer? This
probably sounds like crazy talk to you, but there exist real, used-in-production
programming languages that only evaluate code when it's required. This is known
as *call-by-name* evaluation, and means you only pay the runtime costs to sort
exactly as much of the list as you need.

As this example shows, we are worried about saving precious CPU cycles even when
it is neither an explicit requirement nor when the "naive" solution is likely to
be fast enough anyway. We generally don't even realize that we're doing it. But
the standard CS education has inculcated in us the idea that we must solve
problems via `for` loops and temporary variables. As if, somehow, that is the
True Nature of programming.

It's not hard to see where this attitude came from. Our discipline of
programming extends back to the first computers, which were only barely powerful
enough to compute with. As a result, the languages and algorithms needed to be
"close to the metal" because there was no extra memory or spare CPU cycles
available. CPU time and memory were, in fact, precious, and thus things to be
horded with extreme parsimony. All popular contemporary programming languages
have descended from these original languages, and the path dependence means that
things are done today as they have always been done---that is, with extreme
reverence for CPU and RAM usage.

Of course, this is not true of how *applications* are programmed, where it's not
uncommon to find programs running on multiple layers of virtual machines, each
one inflating memory and program time by an order of magnitude. This state of
affairs is nothing to aspire to.

We can, in fact, have our cake and eat it too. It's possible to implement clever
new algorithms and have them run fast. We just write the high-level algorithm,
and perform as many program transformations as required to satisfactorily
optimize the solution. Important in this is that the problem gets solved first
and the optimizations come later. Optimizations have a duplicitous tendency to
obscure meaning, and thus their introduction too early into a solution can
render the problem effectively untenable.

