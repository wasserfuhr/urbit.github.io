---
layout: post
category: doc
title: Urbit Made Easy &#58; Chapter I
---

#"What one fool can do, another can"#

A lot of people think that Urbit is complicated. A lot of people are wrong. Urbit was designed to be stupid. 

This guide assumes no prior knowledge of programming, computer science or math.

#Chapter I: What is Nock?#

Let's start with the formal specification of Nock. Don't worry, it's much less
scary than it looks. 

**1. Structures**

	A noun is an atom or a cell.  An atom is any natural number. 
 	A cell is any ordered pair of nouns.

**2. Pseudocode**

	1  ::    nock(a)           *a
	2  ::     a b c]           a [b c]]
	3  ::
	4  ::    ?[a b]            0
	5  ::    ?a                1
	6  ::    +[a b]            +[a b]
	7  ::    +a                1 + a
	8  ::    =[a a]            0
	9  ::    =[a b]            1
	10 ::    =a                =a	
	11 ::
	12 ::    /[1 a]            a
	13 ::    /[2 [a b]]          a
	14 ::    /[3 [a b]]          b
	15 ::    /[(a + a) b]      /[2 /[a b]]
	16 ::    /[(a + a + 1) b]  /[3 /[a b]]
	17 ::    /a                /a
	18 ::
	19 ::    *[a [[b c] d]]      [*[a [b c]] *[a d]]
	20 ::
	21 ::    *[a [0 b]]          /[b a]
	22 ::    *[a [1 b]]          b
	23 ::    *[a [2 [b c]]]      *[*[a b] *[a c]]
	24 ::    *[a [3 b]]          ?*[a b]
	25 ::    *[a [4 b]]          +*[a b]
	26 ::    *[a [5 b]]          =*[a b]
	27 ::
	28 ::    *[a 6 [b [c d]]]    *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
	29 ::    *[a 7 [b c]]        *[a 2 b 1 c]
	30 ::    *[a 8 [b c]]        *[a 7 [[7 [0 1] b] 0 1] c]
	31 ::    *[a 9 [b c]]        *[a 7 c 2 [0 1] 0 b]
	32 ::    *[a 10 [[b c] d]]   *[a 8 c 7 [0 3] d]
	33 ::    *[a 10 [b c]]       *[a c]
	34 ::
	35 ::    *a                *a

As you're going through the first part of this guide, it will be helpful to
have a copy of the specification that you can quickly refer to.

Nock may look intimidating now, but by the end of this guide, you will have
utterly mastered it. 

Nock is what some fancy people call a "computational automaton." All that means
is it's a list of rules about how to turn symbols into other symbols. You put
some symbols in, you follow the rules, and you get some symbols out. 

##Atoms & Cells##

The symbols that go into Nock are called nouns, which are just numbers and
pairs of numbers (and pairs of those pairs, but we'll get to that). The numbers
that go into Nock are called "atoms" and are always positive whole numbers. In
math, these are called "natural numbers," because they're the set of numbers
used for counting. In Nock, we'll write atoms in two different ways. The first
way, when we know what the atom is we'll just write the number: `1`, `2`, `3`, `4`... `424`, `9.904` etc. 

(Note: Nock, and Urbit as a whole, use a period instead of a comma for long numbers.  You'll see why much later on.)

The second way we write atoms is with single letters. We do this when we don't
know what the atom is, so we use letters (`a`, `b`, `c`, `d` etc.) as placeholders or
variables. This isn't actually part of Nock and we only do it in the rules that
make up Nock, so we can describe what those rules are. In real Nock code, there
are no letters. 

This is actually a really important point to keep in mind: The formal
definition of Nock isn't written in Nock, but is actually written in English.
Albeit a really funky kind of English (called "pseudocode")

Pairs of atoms are written using square brackets, and we separate the pairs
with a space. Like so:

	[1 1] 
	[34 45] 
	[123.456.789 987.654.321]

A pair of atoms is called a cell. If we wanted to write pseudocode Nock, we
could write:

	[a 1] 
	[34 b] 
	[c d] 

Where `a`, `b`, `c` and `d` all are standing in for atoms.

Both atoms and cells are called nouns, which are the symbols (the fancy word is
"expressions") that go into Nock. You can think of Nock as a machine that
transforms nouns into other nouns.

I think we're ready to take a look at the first part of the formal Nock
definition:

**1. Structures**

	A noun is an atom or a cell.  An atom is a natural number.  
	A cell is any ordered pair of nouns.

The first two sentences we've covered. The third sentence adds a new concept:
Cells can go inside other cells. 

Remember that since a noun is either an atom or a cell (a number or a pair),
then a pair of nouns can be a pair of atoms, a pair of cells, or the pair of an atom and a cell. Like so:

	[1 1] 
	[1 [2 3]] 
	[[1 1] [2 3]]

Any cell in Nock, you can put inside another cell. This means that the cells
can get arbitrarily long (or nest arbitrarily deep). All of the following are
valid cells:

	[2 3]
	[2 [6 7]]
	[2 [6 [14 15]]]
	[2 [6 [14 [30 31]]]]
	[2 [6 [14 [30 [62 63]]]]]
	[[4 5] 3]
	[[[8 9] 5] 3]
	[2 [[12 13] 7]]
	[2 [[12 13] [14 15]]]

As you can see, the brackets start to pile up, making this somewhat difficult
to read.

##Brackets: Who Needs Them?##

Fortunately, there's a rule in Nock that helps make things more legible: We can
leave out all the brackets that group to the right, because Nock will just add
them back in. That means instead of writing:

	[[2 3]
	[2 [6 7]]
	[2 [6 [14 15]]]
	[2 [6 [14 [30 31]]]]
	[2 [6 [14 [30 [62 63]]]]]
	[[4 5] 3]
	[[[8 9] 5] 3]
	[[2 [[12 13] 7]]
	[2 [[12 13] [14 15]]]

We can just write:

	[2 3]
	[2 6 7]
	[2 6 14 15]
	[2 6 14 30 31]
	[2 6 14 30 62 63]
	[[4 5] 3]
	[[[8 9] 5] 3]
	[2 [12 13] 7]
	[2 [12 13] [14 15]]

The brackets haven't gone away, we've just decided not to write them.

We can do this because of `line 2` of Nock, which puts the brackets back in.

	2  ::    a b c]          a [b c]]

Simply speaking, line 2 says that brackets always group to the right.Before we
go into this, we need to quickly cover two things:

First, we need to extend our understanding of letters being placeholders for
atoms. Letters can also stand in for cells. A letter represents a noun. So
whenever you see a letter, all that that means is "Noun goes here." And that
noun can be, of course, either an atom or a cell.

Second, we need to explain how to read the Nock's rules. Nock has a left-hand
column, and a right-hand column. The left-hand column is for inputs, the
right-hand column, for outputs. You take a noun, match it to the pattern on the
left. And then you transform your noun so that the pattern on the left becomes
the pattern on the right. The whole process is called a "reduction."

We'll walk you through how this works:

Let's say that we have the noun `[2 6 7]`. `Line 2` says:

	2  ::   a b c]           a [b c]]

We can match `[2 6 7]` to `a b c]`, because they have the same structure. `a` is the
placeholder for `2`, `b` for `6` and `c` for 7, and the right bracket in `a b c]` matches
the right bracket in `[2 6 7]`. 

(There are no left brackets in `line 2`. This is not a typo, we'll show you why
when we actually go through the mechanics of applying `line 2`).

Let's line up `[2 6 7]` and `a b c]` so that the right-brackets align.

	[2 6 7] 
	 a b c]

Pretty clearly, `a` is `2`, `b` is `6` and `c` is `7`. So you could rewrite our noun `[2 6 7]` as `[a b c]`.

The right-hand side of `line 2` says:

	a [b c]]

Meaning we need to insert a left-bracket before `b`, and a right bracket after `c`.

Our input noun:

	[2 6 7]

We replace nouns with the matching letters (`a` is `2`, `b` is `6`, `c` is `7`):

	[a b c]

We do the transformation (putting the left-bracket and right-bracket in):

	[a [b c]]

And finally we turn the letters back to nouns

	[2 [6 7]]

And that's our output. We've just done our very first Nock reduction! `[2 6 7]`
goes into Nock, and `[2 [6 7]]` comes out.

Let's try putting the more complicated noun `[2 6 14 15]` into `line 2`:

	2  ::    a b c]           a [b c]]

We start on the inside, because the right-brackets have to align. 

	[2 6 14 15] 
	   a  b  c]

Like we did before, we replace nouns with their matching letters. `a` is `6`, `b` is `14` and `c` is `15`.

	[2 a b c]

We do the transformation and add in the brackets

	[2 a [b c]]

And finally we replace our placeholder letters with their corresponding nouns.

	[2 6 [14 15]]

But we're not done yet! We still have another pair of brackets to add. Unlike
what we did before with the noun `[2 6 7]`, the noun `[2 6 14 15]` has to go
through `line 2` twice before its reduction is complete. We do this by taking 
the output noun from our first time through `line 2` (`[2 6 [14 15]]`) and then putting that noun back in to Nock as input.

We'll go through it a bit faster this time, but the process is still exactly
the same.

Align expressions (remember that letters can be either atoms or cells): 
	[2 6 [14 15]] 
	 a b    c   ]

Turn nouns to letters: 

	[a b c]

Transform noun according to the right-hand side:

	[a [b c]]

Turn letters to nouns (`a` is `2`, `b` is `6`, `c` is `[14 15]`): 

	[2 [6 [14 15]]]

Which is our final output, with all the brackets added back to the noun 
`[2 6 14 15]`. For even more complicated nouns, like `[2 6 14 30 31]` or 
`[2 [12 26 27] 14 15]`, you would have to apply the rule from line 2 even more times.
Every time you transform a noun using a rule in Nock, you have to take the
output of that transformation and run it through Nock again until no more
patterns in the left-hand column match. Nock has further rules that describe
how and when you end the reduction process. We'll get into these rules in a
moment.

Before we do, and completely move on to bigger and better things, we should
briefly cover why `line 2` has mismatched brackets, because that does look a
little odd. The answer to this is simply that if `line `2 had left-brackets,
our pattern-matching with nouns that need multiple transformations wouldn't
work.  We wouldn't have been able to match `[2 6 14 15]` to the pattern `[a b
c]`, because the brackets wouldn't have lined up. To actually have gone through
and done the mechanical reduction this way, we would have had to ignore what
`line 2` actually says, and reconstruct in our minds what it actually means.
Which would have been very confusing for everyone. 

On to Chapter 2, where we'll start learning the notation of Nock reductions!
