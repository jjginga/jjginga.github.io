---
layout: distill
title: chalenge - binary multiple of 3
description: the challenge was to create a regular expression capable of recognizing multiples of 3. 
tags: regex dfa binary
giscus_comments: true
date: 2024-02-28
featured: true



toc:
  - name: Challenge
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Journey
  - name: Solution & Reflextion
  - name: Further information
  
# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## Challenge

After I run my marathon I took some days to rest by the sea. Little did I know that instead of a conventional boat I would embark on a computational odyssey where on a sea of zeros and ones. It happened when I crossed paths with [this CodeWars kata](https://www.codewars.com/kata/54de279df565808f8b00126a). It seemed simple, since I only had to create a regex expression. And it had a hight reward - 4 kata. So, I decided to take a try. After all, I just had to find a pattern amidst the cascade of 0s and 1s of the binary numbers multiple of 3.

```
In this kata, your task is to create a regular expression capable of evaluating binary 
strings (strings with only 1s and 0s) and determining whether the given string 
represents a number divisible by 3.

Take into account that:

An empty string might be evaluated to true (it's not going to be tested, so you don't 
need to worry about it - unless you want)
The input should consist only of binary digits - no spaces, other digits, alphanumeric
 characters, etc.
There might be leading 0s.
```

## Jorney

I started by writting a bunch of binary representations of multiples of 3, in the hope of finding a pattern visually.

| Decimal       | Binary        |
| ------------- | :-----------: |
| 3             |   11          |
| 6             |   110         |
| 9             |   1001        |
| 12            |   1100        |
| 15            |   1111        |
| 18            |   10010       |
| 21            |   10101       |
| 24            |   11000       |
| 27            |   11011       |
| 30            |   11110       |

There is one, but I wasn't able to recognize it, but a qick search on the internet showed me that if I sum all the odd-positioned bits and subtract to it the sum of all of the even-position bits and get a result that is diviseble by 3, then the original binary number is also divisible by 3. 

```
27 in binary is 11011
odd bits: 1+0+1 = 2
even bitts: 1+1 = 2
differente: 2-2 = 0, ence 27 is diviseble by 3.
```

Converting this logic to a regular expression is not straightforward, they work well for patern matching and not for mathematical operations. So, it hit me. I could use an Automata.
The Automata would have states representing the remainder when divided by 3 (the possible remainders are 0, 1 and 2), and transactions based on the next binary digit. And we could then determine the divisibility by 3 by observing the state after processing all the bits.

With the available info it is easy to design the following DFA

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/DFA.png" class="img-fluid rounded z-depth-1" zoomable=true %}
</div>

We could directly write a regular expression from this DFA, bur we can do better than that. We start by defining it mathematically 

$$

Given the DFA ( M = (Q, \Sigma, \delta, q_0, F) ), where:
   ( Q = { q_0, q_1, q_2 } )
   ( \Sigma =  { 0, 1 } )
   ( \delta ) is defined by the transitions:
  \[
  \begin{aligned}
    \delta(q_0, 0) &= q_0 \\
    \delta(q_0, 1) &= q_1 \\
    \delta(q_1, 0) &= q_2 \\
    \delta(q_1, 1) &= q_0 \\
    \delta(q_2, 0) &= q_1 \\
    \delta(q_2, 1) &= q_2
  \end{aligned}
  \]
  ( q_0 ) is the start state
  ( F = { q_0 } ) is the set of accept states

$$

Then we can start by writting the equations for all the states:

$$
q_0 = \epsilon + q_0 0 + q_1 1
q_1 = q_0 1 + q_2 0
q_2 = q_1 0 + q_2 1
$$

To simplify $$ q_2 $$ and $$ q_1 $$ we can use [Arden's theorem](https://www.geeksforgeeks.org/ardens-theorem-in-theory-of-computation/). We first note that the theorem states that if a regular expression $$ R $$ satisfies the equation $$ R = Q + RP $$, where $$ Q $$ and $$ P $$ are regular expressions and $$ P $$ does not contain the empty string $$ \epsilon $$ then $$ R = QP* $$ is a solution for the equation.

Aplying it to $$ q_2 $$ we get $$ q_2 = ( q_1 0 ) ( 1 * )  $$ - this represents the language accepted by $$ q_1 $$ followed by a 0 and then any number of 1s.

Now substituting this on the equation for $$ q_1 $$ we get $$ q_1 = q_0 1 + ( q_1 0 ) (\ 1 * )  0 $$.

We can now apply Arden's Theorem again: $$ q_1 =  ( q_0 1 ) ( 0 ( 1 * )  0 ) * $$ - and you get the language accepted by $$ q_0 $$ followed by a 1 and then any number of repetitions starting with 0, followed by zero or more 1s and ending with 0.

Now we can write $$ q_0 = \epsilon + q_0 0 + q_0 1 ( 0 ( 1 * )  0 ) * 1 = \epsilon + q_0 ( 0 + 1 ( 0 ( 1 * )  0 ) * 1 ) $$.

Applying Arden's Theorem again we get:

$$ q_0 = ( \epsilon ) ( 0 + 1 ( 0 ( 1 * )  0 ) * 1 ) * $$ 

That can be further simplified to

$$ q_0 = ( \epsilon ) ( 0 + 1 ( 0 ( 1 * )  0 ) * 1 ) * $$ 

This is the final expression for $$ q_0 $$ that describe all the strings accepted by the automata that start and end in this state. 


## Solution and reflexion

{% highlight c++ %}
const std::string multiple_of_3_regex = "^(0|1(01*0)*1)*$";
{% endhighlight %}

It's fascinating how theoretical computer science principles blend into practical programming challenges. It's a vivid reminder that beneath every line of code lies a rich tapestry of logic and mathematics, waiting to be explored and appreciated. Every challenge is not just a test of skill, but an invitation to an intellectual adventure, revealing the interconnectedness of concepts we sometimes take for granted.

## Further information

[Arden's theorem](https://www.geeksforgeeks.org/ardens-theorem-in-theory-of-computation/)
[Khan Academy - DFA to Regular Expression](https://www.youtube.com/watch?v=SmT1DXLl3f4)