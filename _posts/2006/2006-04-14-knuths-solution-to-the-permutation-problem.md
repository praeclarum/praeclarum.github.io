---
layout: post
title: Knuth’s Solution to the Permutation Problem
tags: article
---

<a href="http://www-cs-faculty.stanford.edu/~knuth/">Dr. Donald Knuth</a> considered the problem of generating all permutations of an <i>n</i>-tuple whose integral elements were less than some number <i>m</i> in <a href="http://www.amazon.com/gp/product/0201853930/qid=1145044392/sr=1-4/ref=sr_1_4/002-0766178-3037666?s=books&amp;v=glance&amp;n=283155">Section 7.2.1.1 of The Art of Computer Programming</a>. Let’s look at his Algorithm M and consider how it can be used to solve permutation problems. To demonstrate the power of the solution, we’ll use C# and investigate the most idiomatic solution in that language.


## Generate all <i>n</i>-tuples (<i>a</i><sub>1</sub>,…,<i>a</i><sub><i>n</i></sub>) where 0 ≤ <i>a</i><sub><i>j</i></sub> &lt; 2 and 1 ≤ <i>j</i> ≤ <i>n</i>

In this article we are going to switch our notation to that of Knuth’s. In this notation the elements are subscripted versions of <i>a</i> instead of <i>n</i> with subscripts that begin at 1 (purely to simplify the algorithm). Also, the number of elements in the tuple is now <i>n</i> instead of <i>k</i>.

Knuth’s algorithm is the natural extension of a powerful change in view: consider the case where each element of the tuple can be either 0 or 1. In this case, we can consider the tuple to be a binary string. Programming languages give us a lot of tools for manipulating binary strings. One of these tools, the use of binary strings to represent unsigned integers, enables us to write a very streamlined permutation solution.

Thanks to C#’s funny type system, we know that binary strings are easily represented as unsigned integers. We can therefore regard an 8-tuple as a byte, a 16-tuple as a ushort, a 32-tuple as a uint, and a 64-tuple as a ulong. Why should we do this? Well, to generate all permutations of a binary string represented as an integer, we merely have to initialize the integer to 0 and keep adding 1. At each step, we have a new permutation of the bit string!

In code, it would look something like:

```csharp
class BinaryString8Permuter
{
    protected virtual void Visit(byte a)
    {
        for (int i = 7; i &gt;= 0; i--) {
            System.Console.Write((a &gt;&gt; i) &amp; 1);
        }
        System.Console.WriteLine();
    }

    public void VisitAll()
    {
        byte a = 0;
        do {
            Visit(a);
        } while ((++a) != 0);
    }
}
```

In this class, the public VisitAll function generates all the permutations. The virtual Visit function receives one of those permutations and does something with it. In the default case, I have written it to simply display the tuple’s value on the console.

In order to generate the permutations, VisitAll simply has to initialize the integer to 0, continuously increment it by 1 to generate the permutations, and detect when all the permutations have been generated. As C# does not provide the overflow result of arithmetic operations, we have to rely on the integer overflowing to 0 in order to detect that we have visited all permutations.

This code works well and displays the expected 256 permutations.


## Generate all <i>n</i>-tuples (<i>a</i><sub>1</sub>,…,<i>a</i><sub><i>n</i></sub>) where 0 ≤ <i>a</i><sub><i>j</i></sub> &lt; <i>m</i> and 1 ≤ <i>j</i> ≤ <i>n</i>

With the knowledge of that first example, let’s move on to the more interesting problem of generating tuples of any length (<i>n</i>) with any integer limit (<i>m</i>).

Knuth builds upon the last idea by recognizing that this problem can be solved by considering the tuple to be an <i>m</i>-ary number of <i>n</i> digits that is continuously incremented by unity (the number 1 represented as an <i>m</i>-ary number). In the last example, we considered a binary number of 8 digits that was incremented by the value 1. Now we will look at generalizing the algorithm.

Knuth’s Algorithm M actually allows for each element of the tuple to have a different termination value (called <i>m</i><sub><i>j</i></sub>). To keep things in line with last week’s work, let us focus only on the single terminator <i>m</i>. With this change, the modified form of his algorithm is:

1. [Initialize.] Set <i>a</i><sub><i>j</i></sub>←0 for 0≤<i>j</i>≤<i>n</i>.

2. [Visit.] Visit the <i>n</i>-tuple (<i>a</i><sub>1</sub>,…,<i>a</i><sub><i>n</i></sub>). (The program that wants to examine all <i>n</i>-tuples now does its thing.

3. [Prepare to add one.] Set <i>j</i>←<i>n</i>.

4. [Carry if necessary.] If <i>a</i><sub><i>j</i></sub>=<i>m</i>-1, set <i>a</i><sub><i>j</i></sub>←0, <i>j</i>←<i>j</i>-1, and repeat this step.

5. [Increase, unless done.] If <i>j</i>=0, terminate the algorithm. Otherwise set <i>a</i><sub><i>j</i></sub>←<i>a</i><sub><i>j</i></sub>+1 and go back to step M2.

What does this look like in code? Well here’s my version of it in C#:

```csharp
class IntNTuplePermuter
{
    protected virtual void Visit(int[] a)
    {
        string head = "";
        for (int j = 1; j &lt; a.Length; j++) {
            System.Console.Write("{0}{1}", head, a[j]);
            head = ", ";
        }
        System.Console.WriteLine();
    }

    public void VisitAll(int m, int n)
    {
        // Initialize. Because C# automatically initializes
        // integers to 0, we need not explictely do so.
        // We allocate n + 1 integers to make room for a0.
        int   j;
        int[] a = new int[n + 1];

        for (;;) {
            Visit(a);

            // Prepare to add one.
            j = n;

            // Carry if necessary.
            while (a[j] == (m - 1)) {
                a[j] = 0;
                j -= 1;
            }

            // Increase unless done.
            if (j == 0) {
                break; // Terminate the algorithm
            }
            else {
                a[j] = a[j] + 1;
            }
        }
    }
}
```

The code after the call to Visit is very reminiscent of last week’s solution. All you have to do is realize that “fastest” in incr and “j” in Algorithm M play the same role. So there we are, instead of devising a solution ourselves, we could have just opened Knuth’s tome and received revelation.


## Digressing with IEnumerable

Last week’s permutation article ended with a generic implementation of the incr function. That was nice, but in C# we can do much better! Using the clarity of Knuth’s solution and the venerable IEnumerable interface, we can create a permuter that is able to visit tuples whose elements are of any type that implements the IEnumerable interface. Let’s see how.

IEnumerable provides the GetEnumerator function that gives an object that implements the IEnumerator interface. This interface provides two methods and one property: Reset and MoveNext, and Current. Current simply returns the current value of the object pointed to by the enumerator. Reset sets the enumerator to point to the first element in the IEnumerable collection, and MoveNext advances the enumerator to the next element in the collection.

With these interfaces we can write a new permuter that loosely follows Knuth’s algorithm. The most important revelation is the use of two collections: <i>a</i> which holds the enumerators (and subsequently the values), and <i>m</i> which holds the IEnumerables. When we need to increment a value, we simply call MoveNext. When we need to reset a value, we simply call Reset followed by MoveNext. The call to MoveNext after Reset (and GetEnumerator) is required because reset enumerators actually point to the element before the first element in the collection.

With all that said, here is our best permuter to date:

```csharp
class EnumerablePermuter
{
    protected virtual void Visit(System.Collections.IEnumerator[] a)
    {
        string head = "";
        for (int j = 1; j &lt; a.Length; j++) {
            System.Console.Write("{0}{1}", head, a[j].Current);
            head = " ";
        }
        System.Console.WriteLine();
    }

    public void VisitAll(params System.Collections.IEnumerable[] m)
    {
        // Initialize.
        int n = m.Length;
        int j;
        System.Collections.IEnumerator[] a =
            new System.Collections.IEnumerator[n + 1];

        for (j = 1; j &lt;= n; j++) {
            a[j] = m[j - 1].GetEnumerator();
            a[j].MoveNext();
        }
        a[0] = m[0].GetEnumerator();
        a[0].MoveNext();

        for (; ; ) {
            Visit(a);

            // Prepare to add one.
            j = n;

            // Carry if necessary.
            while (!a[j].MoveNext()) {
                a[j].Reset();
                a[j].MoveNext();
                j -= 1;
            }

            // Increase unless done.
            if (j == 0) {
                break; // Terminate the algorithm
            }
        }
    }
}
```

You can see that the most work is done in the initialization section when we have to create the enumerators (and the one dummy enumerator). The rest is just Knuth’s algorithm.

I find it very pleasing that we were able to move from using integers to permute binary strings to an algorithm for permuting tuples of any length comprised of any type of object in just a few steps.


## Examples of the IEnumerable Permuter in Action

Let’s look at the above EnumerablePermuter in action by constructing a few simple tests. First, let’s generate some simple sentences:

```csharp
string[] nouns = new string[] { "cat", "dog" };
string[] verbs = new string[] { "sniffs", "eats" };
(new EnumerablePermuter()).VisitAll(nouns, verbs, nouns);
```

This code generates the silly output:

```text
cat sniffs cat
cat sniffs dog
cat eats cat
cat eats dog
dog sniffs cat
dog sniffs dog
dog eats cat
dog eats dog
```

The code works! We now know that some cat sniffed a dog, and that some other dog ate a dog!

What else can we permute? How about the cards in a deck?

```csharp
object[] values = new object[] {
    "Ace", 2, 3, 4, 5, 6, 7, 8, 9, "Jack", "Queen", "King" };
string[] suits = new string[] {
    "Spades", "Hearts", "Clubs", "Diamonds" };
string[] of = new string[] { "of" };
(new EnumerablePermuter()).VisitAll(values, of, suits);
```

This code generates the predictable output:

```text
Ace of Spades
Ace of Hearts
Ace of Clubs
Ace of Diamonds
…
King of Diamonds
```

Note that I used a dirty trick in order to inject the word “of” into the output. In a real application, I would override the Visit function of EnumerablePermuter. But this isn’t a real application, so we’re allowed to play dirty tricks!


## Conclusion

I know Knuth’s The Art of Computer Programming is quite large and is difficult to understand at times, but there are a lot (what an understatement!) of gems to be found in it. Give it a try. Read it in small doses. Whenever he presents an algorithm, try coding it in your favorite language. I think you will find that there is a lot to be gained from this exercise; I certainly did.