10. Balanced binary reduction
==============================

## Alice in wonderland 

Let us attack the problem of finding not just the smallest
of `n` elements, but the smallest and second smallest.
The problem has a very distinguished pedigree it was first addressed by a well-known British mathematician 
[Charles Dodgson][carroll] (Lewis Carroll).
If you haven't heard of him, you should.
There is a very important book which he wrote, not the [mathematical book][carroll-logic]
, but the book called ["Alice's Adventures in Wonderland"][alice].
If you haven't read it, do.
No person should be hired ever unless you read "Alice in Wonderland".
in any case he was also a mathematician.
He also dabbled in all kind of games
apparently he [invented Scrabble][scrabble] and bunch of other games.

At some point he decided that there is a clear problem with tennis tournaments.
He observed that with a very high probability,
if you have say a tournament with 64
players, the guy who gets the second prize is actually not the second strongest.
He observed that the strongest and
second strongest could be paired in the first round.
Therefore the second strongest guy gets eliminated and doesn't get the second prize,
in spite of his prowess.
This is is why now they use a technique known is [seeding][seed] to assure
that people of similar ability are spread out to different parts of the tree.
But, he wanted to come up with an algorithm
which assures that the second guy is truly the second guy.
He published it in 1883.
The algorithm wasn't quite an algorithm and it was clearly not optimal.
It took 15 more years before the problem was stated correctly.
People realized that you could talk about minimum
number of comparisons but it took another thirty years, until 1964 when a
Russian mathematician [Sergei S. Kislitsyn][sergei] published a paper which 
proved there is an optimal algorithm and described it.

By the way, all of this information is available[^aoc-ref] in 
a
book called ["The Art of Computer Programming"][aoc] by [Donald Knuth][knuth] .
You really should buy it.
You make a certain commitment.
You spend $150 saying that you really care about programming.
This is not a book which is "useful" meaning it's not "programming in Python for idiots",
or "information retrieval for 21st century" or something like that.
This is one of the fundamental books which you buy and then spend your
lifetime getting the information out of it.
You get beautiful things which you then could use for programming in Python or information retrieval or
other things.
I'm going to be mentioning Knuth throughout the course.
It's not a perfect book, it is just the greatest book we've got.
Some people think it's a good reference book.
No, it's not, because you have to
basically do linear search to find what you're interested in.
Another important thing, do not spend too much time solving problems.
Read the solutions.
They are  right at the end.
Lots of very important algorithms are described in the solutions to his problems.
Reading Knuth has to become a lifelong activity.

[^aoc-ref]: See chapter 5.3.3 in Volume 3 of "The Art of Computer Programming".

[alice]: https://www.gutenberg.org/ebooks/11
[carroll]: https://en.wikipedia.org/wiki/Lewis_Carroll
[scrabble]: http://www.bananagrammer.com/2009/10/games-of-lewis-carroll.html
[seed]: https://en.wikipedia.org/wiki/Seed_(sports)
[knuth]: https://en.wikipedia.org/wiki/Donald_Knuth
[aoc]: https://en.wikipedia.org/wiki/The_Art_of_Computer_Programming
[sergei]: http://www.mathnet.ru/eng/person27000
[carroll-logic]: https://www.gutenberg.org/ebooks/28696


## Smallest and second smallest element


The problem of finding the smallest and second smallest element,
is in "The Art of Computer Programming", but somehow Knuth does not implement it.
He describes it fully but doesn't implement it as an algorithm.
As we shall see its actually a little tricky.
I pick this algorithm not because it is of paramount importance for your future work.
I pick this algorithm because it allows us to learn how to do decomposition
and learn components along the way (like lists).

How many comparisons do you need to solve this problem?
Same as min-max from last time?
No. 
You can do it in fewer.
Let us try to use some logic .
How many comparisons do we need to find the winner of the
tournament? `n - 1`.
It is necessary to find the winner, in order to find the second place guy.
We could sketch a proof.
Let us assume there are two potential guys greater than him.
If there is none, he isn't second place.
If there are two, he isn't second place either.

What do we know about second place,
specifically the games he lost.
*He only lost one game, and it was to the winner*.
If the winner remembers all the games he won,
how could we determine second place?
How many people did he beat?

For example, [Wimbledon][wimbledon] has 64 players who are admitted.
The winner doesn't play 63 games.
The tournament is structured as a binary tree.
This tree is how deep? 
If you have `n` elements? It's `ceil(log_2(n))`.
We could somehow arrange our tournament so that the 
winner will defeat this many people.
Now that we have a list of the `ceil(log_2(n))` people who played the winner,
we just find the best out of them which is 
`ceil(log_2(n)) - 1` comparisons.

To review, we 
have `n - 1` comparisons to get the winner.
`ceil(log_2(n)) - 1` to get the second best, 
from the list he played.
So the upper bound for the algorithm is:

    n + ceil(log_2(n)) - 2

Actually implementing this algorithm effectively
will require us to create several components.
We will build these up over the next few lessons.

[wimbledon]: https://en.wikipedia.org/wiki/The_Championships,_Wimbledon

### Unoptimal divider and conquer approach

It might appear you could use divide and conquer.
First split it in two, 
find min and second min of the first half,
and the second half, and then merge them together doing two comparisons.
It sounds very elegant because it's all recursive.
But, let us think about how many comparisons it's going to do
with simple mathematics.

1. We start with `n` we need to pair and compare them.
   So the first round is `n/2` comparisons.

2. In the second round we pair up the results,
   so we need `n/4` "games".
   But, each game requires two comparisons (to find min and max).
   So we have another `n/2` comparisons.

3. And so on...

So that would be:
    
    n/2 + (n/2 + n/4 + n/8 + ...)

    = n/2 + n-1

total comparisons.
That's not what we're trying to accomplish,
so divide and conquer doesn't always do what we think.

### Tournament tree shapes

First, we need rearrange the tournament we play.
Right now `min_element` plays a tree structure that looks like this:

    Unbalanced Tree
    /\
      /\
        /\
          ... 
           /\

It has `n - 1` internal nodes.
But, we don't want the winner to play `n - 1` matches.
We need to transform that into the way they play Tennis.
We need to balance the tree.

    Balanced Tree
        / \
       /\ /\
        ...
      /\ /\ /\

How do we do it?
One way is to just pair up elements and build up.
But then we need lots of memory to save the intermediate results.
What do we mean when we say lots of memory?
`O(n)` is bad, `O(sqrt(n))` is pretty bad.

Note that once a bottom-level round has been played,
they are ready to move up.
Our goal is to basically to become eager.
Whenever guys are ready to be paired we want to pair them.
So if we only store only the winner at each level,
we never need to store `log(n)` things.
We can define the **power** of each element
to be the number of games they have played.

Realize that suddenly we see something which has nothing to do with our problem.
*The foundation of our algorithm is the ability to take a tree like
the linear (unbalanced) tree and transform it into a balanced tree*.
What mathematical property allows us to do such a transformation?
Specifically why can we convert one kind of computation to the other.
Associativity[^associativity].
As long as our operation is associative, 
What property don't we need? Commutativity.
We keep them in the same order,
we just rebalanced parenthesis.

If you think about it,
our `min` is not quite commutative.
In mathematics `min` is commutative.
But, because we want to preserve stability it is not.
We distinguish between the left and right argument.


[^associativity]: [Associativity][associative] is a fundamental property studied in abstract
    algebra.
    Informally, it is that you can apply an operation to arguments
    in any order you want.
    Formally:

        f(f(a, b), c)) = f(a, f(b, c))

    For example, multiplication:

        (a * b) * c = a * (b * c)

    You can see why associativity is often referred to as being
    able to "re-paranthesize".

[associative]: https://mathworld.wolfram.com/Associative.html


## Binary counting and reduction

Here we come to the amazing idea of how to do this transformation.
This is one of the most beautiful ideas which they
kept secret from you.
They should have taught it in high school.
But, they want to publish papers
themselves and not tell you the general mechanism.

We can create a counter, and in every bit of this counter we're
going to keep a singleton. 
In each bit we keep the person who had `n` victories.
We will never combine things unless they have the same weight/parity.

Initially the counter has `zero` in every entry:

    1 2  ...   32 (bits/singletons)
    0 0  ...   0

Take a new guy `x` who has never played any games,
and you look at the guy in the first slot of the counter.
The existing guy is either zero or not.
If it's zero (`_`), put him in the counter.

    1 2  ...   32         1 2  ...    32
    0 0  ...   0    >>>   x 0  ...    0


If it's not zero, he plays a game with the existing guy.
If he wins, he replaces the loser in the counter.

    1 2  ...   32           1 2  ...   32
    y 0  ...   0     >>>    x 0  ...   0


Otherwise, the existing guy has now won a game.
So he needs to be promoted to the next level,
he follows the same rules with the guy in that slot.
It's a carry propagation.

    1 2  ...   32           1 2  ...   32
    y 0  ...   0    >>>     0 y  ...   0

If we end up with a guy in slot 32, it's an overflow,
exactly like integer arithmetic.
What do we do?
Whenever we don't know to proceed,
do something sensible and let whomever uses it figure out what is a sensible thing
to do.
Return the carry[^carry].


Let us be lazy. 
The great success in
programming comes because there are lazy people who say, "I don't want to know now,
I'll find out later."
Right now we are solving this problem.
We have an associative binary operation of some kind
 and what we discovered that if we have associativity,
we can make this counter and it will work for us.

If you are familiar with [numerical analysis][numerics],
whenever you sum up large number you don't really want to
add small quantities to big quantitative.
Bad things happen to the errors[^errors].
So, you could use the same device for balancing your addition.
If you want to implement [merge sort][merge-sort] you can use exactly the same device, since
merge is associative[^ryan].
The idea with merge sort, is only want to merge lists if they are roughly the same length
and this helps you do it.

When we become grownups we learn about advanced data structures,
such as [binomial forest][binomial].
They use the same idea.
The counter helps us combine things only when they have the same weight.
nless they are of the same weight it's a general algorithmic technique


[^carry]: The terms **carry** and **carry propagation**
    are usually associated with the algorithm of binary addition
    and especially its implementation as
    an electronic/logical circuit called an [adder][adder].

    An adder has two inputs `a` and `b` for
    the two digits to add together.
    It outputs a digit `s` which is the digit
    to display for this place value.
    There is supplementary output called a carry
    which is the amount that needs to move to the 
    higher place value.
    
    The circuit behavior is determined by the following input/output table:

        a  b   c  s
        ----------
        1  0 | 0  1
        0  1 | 0  1
        1  1 | 1  0

    This adds a single digit, but we want to add entire numbers.
    To, do so we add an additional input `l` for carried digits
    from lower place values.

        a  b  l    c  s
        ----------------
        1  0  0  | 0  1
        0  1  0  | 0  1
        1  1  0  | 1  0
        1  0  1  | 1  0
        0  1  1  | 1  0
        1  1  1  | 1  1

    Now we can chain `n` of them togeter to be able to add `n` digits

            s_1 c_1-----|      s_2 c_2----  ...
            |   |       |      |   |
         -----------    |   ------------
         |    |    |    |   |    |    |
        l_1  a_1  b_1   |_ l_2   a_2  b_2

    An **overflow** is when the last adder has a non-zero carry.

    
[^ryan]: When I first learned about this algorithm, I showed a friend
    and he immediately had the idea to use it 
    for merge sort, without any hint this was possible!

[^errors]: Numerical floating point calculation
    is a subtle subject, but the basic issue Alex is 
    referring to is straightforward.
    Floating point stores large numbers as decimals to some power.
    If you add a small decimal to that, it may not change it all.
    To try for yourself, compare the output of:

        (152500.0 * 5000.0)

    and

        (152500.0 * 5000.0) + 0.01

[adder]: https://en.wikipedia.org/wiki/Adder_(electronics)
[numerics]: https://en.wikipedia.org/wiki/Numerical_analysis
[merge-sort]: https://en.wikipedia.org/wiki/Merge_sort
[binomial]: https://en.wikipedia.org/wiki/Binomial_heap


### Implementation

The first function will add an element to the counter
using the process we just described.

    template <typename T, typename I, typename Op>
    // requires Op is BinaryOperation(T)
    // and Op is associative 
    // and I is ForwardIterator and ValueType(I) == T
    T add_to_counter(I first, I last, Op op, const T& zero, T carry) {
        // precondition: carry != zero
        while (first != last) {
            if (*first == zero) {
                *first = carry;
                return zero;
            }
            carry = op(*first, carry);
            *first = zero;
            ++first;
        }
        return carry;
    }

`op` is not commutative, so we need to be careful about the order
it is evaluated in.
Elements in the counter got there before, so they were to the left of the
element we are inserting.

Notice that zero is `const` reference because we don't plan to modify it,
but we do modify carry,
so it should be passed by value.

The second function applies the operation
to all the elements left sitting in the counter.

    template <typename T, typename I, typename Op>
    // requires Op is BinaryOperation(T)
    // and Op is associative 
    // and I is ForwardIterator and ValueType(I) == T
    T reduce_counter(I first, I last, Op op, const T& zero) {
        while (first != last && *first == zero) {
            ++first;
        }
        if (first == last) return zero;

        T result = *first;
        while (++first != last) {
            if (*first != zero) {
                result = op(*first, result);
            }
        }
        return result;
    }

We have to be careful with zero.
Sometimes it will work with the operation so apply `op(x, zero)`
gives you `x`, but sometimes that won't happen.
So we can't really initialize to zero.

**Exercise:** Use these functions to sum up an array of `double`s. 

**Exercise:** Rewrite `min_element` using these functions (just `min_element`, don't worry about second best).

## Binary counter object

### Start with algorithms

We want to take
these two algorithms and combine them into an object.
We have two beautiful algorithms but they're stateless.
Everything is outside, and this is what we should always do,
no matter what we has been taught in software engineering classes by very wise
object-oriented professors.
You start with algorithms.
You don't start with objects.
Figure out what you're going to do first.
But you don't have to stop there. 
Because you can then put things together into an object.

It's very easy when you write an algorithm to
have a minimal iterator interface.
They externalize the counter.
We say, "we don't want to know about him,
we're just algorithms people".
We assume the principle that  we will have no state for about
five minutes, and stay very functional, but then
turn around and deal with state.
We look at the whole thing.

### Counter storage

So what do we think is the state of this counter?
How should we store the counter?
Don't we have millions of elements to reduce?
The counter is size `log(n)` which will
never be greater than 64.
So, it's actually a small fixed size.
We will store it in a `std::vector`.

    template <typename Op, typename T = typename Op::argument_type>
    class binary_counter
    {
    private:
      std::vector<T> counter;
      Op op;
      T zero;

    public:
      binary_counter(const Op& op, const T& zero) :
        op(op), zero(zero) {}
      
      void reserve(size_t n) { counter.reserve(n); }

      void add(T x) {
        x = add_to_counter(counter.begin(), counter.end(), op, zero, x);
        if (x != zero) counter.push_back(x);
      }

      // returns: value of the counter
      T reduce() {
        return reduce_counter(counter.begin(), counter.end(), op, zero);
      }
    };

Counter is private, we don't
want people to muck up our counter.
Same with our operation

Using initializer lists instead of assignment in constructor is important.
If you initialize in the body, it will first call default constructors
for members, then you overwrite all the work with an assignment.

I think it is very beautiful.
We could compete with Steve Jobs for elegance of our design[^alex-joke].

[^alex-joke]: Alex: Maybe we should make it in China.
    That's a necessary prerequisite for beautiful design.
    [Designed in Cupertino][designed-by-apple] assembled in China. 
    So let's try to assemble our machine in Palo Alto.

[designed-by-apple]: https://signalvnoise.com/posts/2710-designed-by-apple-in-california

### What is in-place memory usage?

How significant is the storage of our counter?
We use the term ["in-place"][in-place] to indicate the memory
usage of an algorithm is not significant.
A long time ago people thought 
algorithms were in-place if they didn't require any  extra memory.
Then they decided  constant memory was enough.
But, then they said it doesn't quite work because our favorite algorithm
doesn't work. Of course the most important algorithm is [quicksort][quicksort].
It's not in-place because it's recursive.
Recursive algorithms tend to require `O(log(n))`.
Quicksort splits roughly in the middle let's assume on average.

So people say `log(n)` is good so that will count as in-place.
Then they said , "what if we have nested things.
So is `log(n)^2` ok"? 
In our universe `log(n) <= 64` so this is 4096.

Then theoreticians said it's really alright if we have "poly-logarithmic" storage
Basically as long as the memory requirement is `O(p(log(n)))`
where `p` is a polynomial, it's alright.
Polynomials get a lot bigger than square,
but let's go with "poly-logarithmic" being "in-place".

[quicksort]: https://en.wikipedia.org/wiki/Quicksort
[in-place]: https://en.wikipedia.org/wiki/In-place_algorithm

## Code

- [binary_counter.h](code/binary_counter.h)


