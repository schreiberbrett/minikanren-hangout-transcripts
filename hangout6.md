# miniKanren Hangout #6, 21 April 2013
Youtube link: [https://www.youtube.com/watch?v=MTAhbsWUUxM](https://www.youtube.com/watch?v=MTAhbsWUUxM)

**Will:** Hello everyone. Welcome to the 6th weekly miniKanren hangout. I was
hoping a couple people would talk about a logic puzzle today, but they haven't
arrived. Maybe they'll show up later. Anyway, there's always plenty to talk
about in miniKanren Land. I've got a few topics to talk about, myself. Let's
start out by going around and seeing how everyone's doing and if they have
anything to talk about in terms of miniKanren development or things they're
working on. I know Claire has been working on some stuff, and dan and Jason have
been working on stuff. I'm not sure if they're ready to share what they're
working on. Claire, do you have any updates on the features you've been adding
to miniKanren?

**Claire:** I don't think there's anything to share, beyond what I posted on the
Google Group.

**Will:** Can you give a recap of that?

**Claire:** I can. Let's see. I posted about a number of things. Documentation
now exists for cKanren. I implemented a way to do fair conjunction (and in
general being able to customize your search strategy through the search space).
So now you can define goals to be lazy, and specify if you want to do a
breadth-first or depth-first search through those lazy goals.

Older update is being able to do general unification on any structure that
implements the Unifiable generic. We have attributed variables, a `set`
constraint implementation that is working as far as I can tell. I've started
working on a higher-level language to define ways to rewrite the constraint
store, kind of like CHR. And I implemented the `normalize-store` procedure that
takes two disequality constraints and checks to see if one subsumes the other.
So that's now a one-liner. There's a way to name `conde` branches so that if you
print out information (like print out the packege) you can actually see which
`conde` branch you went through. And there's a `#lang cKanren` now that
incorporates all of this now, which is actually distributable and just works
as-is. You don't have to do anything crazy to make it work. I think that's it.

**Will:** And it took you a whole week to do that? What else were you doing?

**Claire:** Yeah, I actually haven't gotten much work done *this* week for
obvious reasons. But I'm hoping to get back to work after my Systems final.

**Dan:** Am I going to get in trouble because of this, Claire?

**Claire:** I don't think so. Why would you get in trouble?

**Dan:** You're one of my students. \[Your Systems professor\] might be upset
with me.

**Claire:** Oh, he knows!

**Will:** He's my academic brother, I'll take him on. He should be happy
Claire's doing all this stuff.

Excellent. ALright Dan, what have you been up to?

**Dan:** I'll try to characterize it. Will, Jason, and I talked the other day
about writing an interpreter that would run the miniKanren stuff.

**Will:** Something I've only been asking for for 10 years.

**Dan:** I knew it was easy, so I didn't waste my time with it! It turned out to
be not so easy, but not so hard either. We did it in two steps, basically. The
first step was to design and implement a miniKanren-like language that was
devoid of the use of macros. There are basically 5 operators: `run-g`,
`fresh-g`, `==`, `project`, `ifu`, `ifa`, two argument `conde-g`, and `conj-g`
(which took two goals). So that was the little language, and we implemented that
without any macros. It's all functions.

Some of the operators look a little bit like `call/cc` because of the fact that
one of the arguments in a function. For example, `fresh-g` took a function as an
argument whose formal parameter was the new variable, and the body of the
function would be a goal expression. Then when we introduced `conj-g`, which was
conjunction, it just took two goal expressions. And so on.

So we have these functions. And then the only task was to simulate those
functions inside an interpreter, which turned out to be pretty straightforward.
I don't know what else to say about the language. It's a very simple language:
only 2 goals in a conjunction, only 1 goal in `fresh`, 2 goals in `conde`,
`project` only takes one goal, and `ifu` and `ife` take 3 goals (if the first
goal succeeds, then it conjoins it with the second goal; if it fails, it runs
the third goal). Everything has to be manually nested. We realize that macros
will do that for you for the functional version. It's more of a problem when you
rely on macros for the interpreter version, because (thanks to Kent \[Dybvig?\])
not all names are the same. Because if you happen to name some of your
interpreter's special forms with the same name as primitves like `cons`, then it
macro-expands into `%%cons`, which was very unfriendly. We're thinking about
ways of getting around that.

We do have an interpreter that does what we want, and I think it meets the
criteria we discussed Monday night. (I think it was Monday night.)

**Will:** Oh, you mean Monday night ten years ago?

**Dan:** It seems like ten years ago.

**Will:** It *was* ten years ago, because I wanted miniKanren to be an
interpreter, which is not what you did. You did something better than an
interpreter, but now I finally want my interpreter.

Now, of course, part 2 of this exercise is to translate this into miniKanren so
that you have something metacircular.

**Dan:** Which miniKanren do you want to translate it into?

**Will:** What do you mean?

**Jason:** You remember the little language Dan described? He didn't mention
disequality constraints listed as one of those forms, for instance. You didn't
hear `absento`, `presento`, or any of that stuff.

**Will:** Well, start out with the "little book" version. Start out with the
minimal language.

**Dan:** We need the whole language to implement it, because you can't look up a
lexical variable \[without disequality constraints\].

**Will:** You'll have to figure it out. I mean, this is like writing a
metacircular interpreter for a subset of Scheme.

**Dan:** Almost. Not quite. When you write a metacircular interpreter, you have
fixed on both languages.

**Will:** Well, you're going to have to fix those languages.

**Dan:** We have full miniKanren with the 5 special forms.

**Jason:** Writing an interpreter for full miniKanren seems like step two.
Implementing the rest of those operators (to make it self-hosting) seems like
it'll take a while, I think.

**Will:** Okay. Next week then. Sounds good. Austin, how are you doing?

**Austin:** I'm good. I took a week off from logic programming. I think my brain
got full. I'm waiting for some new additions to appear.

But I had an interesting problem come up this week that I can summarize. I'm
working on an abductive planner and I have five classical AI problems (like
[Blocks world](https://en.wikipedia.org/wiki/Blocks_world)) to test it out. For
the last few weeks I could only solve around two of those problems--I can't
solve all of them.

But I made a change this week and suddenly I could solve all of them. I was
happy for just a moment, because I didn't know exactly why my change caused it
to work. But I soon discovered that it was because I switched my implementation
of `findall/3`. The new one is causing solutions to be found because it was
reifying fresh variables (but these were failing branches anyway). My other
solutions were floundering in the whole negation-as-failure issue. So it was not
terminating. It was interesting that this bug helped me see another flaw in my
program. It really helped elucidate the whole floundering and
negation-as-failure issue. Now I've got two problems to solve.

**Will:** Let me see if I understand correctly. You changed your `findall/3` and
it started working, but it was unsound? Is that the problem? Were you succeeding
when you shouldn't have?

**Austin:** Previously, it was not terminating. But when I switched it to a
version which was grounding fresh variables (because those were
negation-as-failure branches), it was able to close \[prune?\] those once it had
a reified value of `'_.0` instead of remaining fresh.

**Will:** When you were diverging, were you trying to do a `run *` (or a
`run 10` when there were only 9 answers)? Or was it actually diverging while
trying to find answers that actually exist?

**Austin:** I haven't been able to investigate it futher, but I believe it was
floundering in negation-as-failure where it's assuming that when it's proving
the negation of a goal it's that whole "closed world" assumption. Since the
variables are fresh and I'm proving the negation, it keeps floundering.

**Will:** Are you paying attention to the G rule when you're doing negation as
failure?

**Austin:** What's the G rule?

**Will:** I assume you're using a `conda` or `condu` somewhere.

**Austin:** Yes.

**Will:** The idea is that `conda` and `condu`, just like Scheme's `cond`, have
*test positions* for expressions. The first goal in each clause is in test
position. The G rule says that in order to use `conda` or `condu` in a way
that's sound, you have to make sure that none of the fresh variables in a goal
in test position end up being ground/instantiated when you run it.

**Austin:** Is this the Second or Third Commandment in the Reasoned Schemer? Or
the "Law of `conda`?

**Will:** Oh! Is there a Law of `conda`? I don't remember! I spend all my time
avoiding `conda`.

**Austin:** Oh, it's the Third Commandment \[the Second Commandment in TRS2e\]:

"If prior to deptermining the question of a `conda` or `condu` line a variable
is fresh, it must remain fresh in that line's question."

That's been a hard one for me to digest.

**Will:** Wow, I just learned something from the Reasoned Schemer. Well, I
didn't know it was in the book. Good for us for putting that in. Shame on us for
putting `conda` and `condu` in.

**Dan:** You didn't internalize it because you knew you'd never need it!

**Will:** I know the rule, I just said it. I didn't know it was in our book! Now
I'm ashamed.

**Austin:** And you call it the G rule?

**Will:** Yeah. It's G for "ground". That's what Oleg called it. Oleg is the one
who taught us about that nastiness. Anyway, are you respecting the G rule,
ensuring things don't become ground? Because what you're talking about sounds
like a G rule related issue.

**Austin:** It definitely is. It's a flaw in how I was posing these problems. I
was leaving it where variables could stay fresh. Basically, I had an axiom where
the head was unifying but the body contains variables that never get grounded. I
wasn't respecting one of the restrictions that the author of the interpreter
stated.

**Will:** Do you think you have a handle on it now, or do you have to revisit
it?

**Austin:** I'll keep plugging away at it. But this problem really helped me see
exactly where the flaw was occuring. Exactly what happens when you try to negate
a fresh variable.

**Will:** Let's see. I've been working with Michael Adams and Andy Keep on some
macro related things in alphaKanren. Hopefully I'll have something to show
people not too far in the future. I think the program we're working on is by far
the most complicated alphaKanren program out there. It's kinda scary trying to
debug this thing.

I think we actually found a bug in the unifier yesterday of one version of
alphaKanren. We're noticing that a certain unification was failing and it didn't
make any sense that it failed. We tried the equivalent in the REPL and it
succeeded. Michael and Andy noticed that if you projected one of the two terms
being unified that unification succeeds instead of failing (`project` is
equivalent to a `walk*`). In other words, if you have some logic variable that
contains a term and some other logic variables, a `walk*` makes sure to replace
all the logic variables with ground values to the greatest extent possible. When
you `walk*` one of the terms, then the unification suddenly succeeded, which
should never ever happen. That was bad. I think Claire thinks the problem might
have to do with suspensions, which seems plausible. We'll have to look at that
clause. We "fixed" it temporarily by making sure the unifier does a `walk*` on
each term, which is extreme, but it's always fine to do a `walk*`--it shouldn't
ever hurt (unless you have circularities, in which case you might diverge).
Anyway, we spent an hour and a half trying to debug that problem before thinking
maybe the unifier is broken. Anyway, that work is pretty cool.

Still doing work on latticeKanren.

I've started to think hard about adding probabilistic programming and machine
learning aspects to miniKanren. That will be a project in the future. If anyone
has thoughts or things they'd like with miniKanren handling probabilities,
machine learning, or natural language things...

**Dan:** If it's possible, I'd like Claire to describe in more detail how she
does parallel conjunction.

**Claire:** Sure. Let me pull up Emacs.

\[Discussion about Google Hangouts muting microphones omitted.\]

**Will:** While Claire's getting set up, if anyone has thoughts about
probabilistic logic programming, machine learning, or handling big data and
their tie-ins to miniKanren, I'm all ears.

**Claire:** Okay. Here is a simple example of how to write a lazy goal.

```scheme
(define-lazy-goal (syms* t out)
  (conde
    [(== t '())
     (== out '())]
    [(symbolo t)
     (== out `(,t))]
    [(fresh (a d a* d*)
       (appendo a* d* out)
       (syms* a a*)
       (syms* d d*)
       (== t `(,a . ,d)))]))

(define-lazy-goal (appendo ls1 ls2 out)
  (conde
    [(== ls1 '())
     (== ls2 out)]
    [(fresh (a d res)
       (appendo d ls2 res)
       (== ls1 `(,a . ,d))
       (== out `(,a . ,res)))]))
```

I defined this new form `define-lazy-goal`. Instead of using Scheme's define,
you'd use this new macro. All `syms*` does is go through a possibly nested tree
and emit all the symbols at the end. If you have a symbol, `out` is a singleton
list of that symbol. If you have a pair, then you find all the symbols in the
`car` and the `cdr` and append them. I have the goals in a different order just
so I can show that it works even when you have the silliest \[recursive\] goal
first. Same with the pair case of `appendo` -- it has the recursive call first.
I have some tests that show this works okay.

So you can see that writing the body of these goals is the same as plain
miniKanren. You just need `define-lazy-goal`.

```scheme
(begin-for-syntax
  (search-strategy 'dfs))
```

I have this, which is basically saying when you do expansions, make sure you use
the search strategy "depth-first search". You can also do a search strategy
that's breadth-first search. I can change it and it can go just fine.

```scheme
(begin-for-syntax
  (search-strategy 'bfs))
```

I changed the symbol `'dfs` to `'bfs`.

On the right side of my screen is a big macro for `define-lazy-goal`. Most of it
is not important--don't try to understand most of what it's doing. It has to do
with this complicated case of whatever search strategy you're using.

The most important thing is that it basically takes a normal miniKanren goal and
writes it as this "enforce" function instead of actually calling the goal.

[26:45](https://youtu.be/MTAhbsWUUxM?t=1605)

If I were to expand this out, `(define-lazy-goal (syms* t out) ...)` would
expand to:

```scheme
(define (syms* t out)
  (goal-construct (lazy-syms* t out)))

(define (lazy-syms* t out)
  ...)

(define (enforce-syms* t out)
  ...)
```

Basically, what happens is the actual `syms*` procedure becomes a kind of lazy
constraint. All it will do is build a call to `enforce-syms*` and then put
itself in the constraint store and not do any more work. As soon as you do a
call such as `(syms* a a^)`, all it will do is put a lazy constraint on `a` and
`a^` in the constraint store. It won't actually do any work; it'll just assert
that sometime in the future, it will enforce that `syms*` call.

At the end, as you can see, I have to build this little closure of calling the
actual goal. I will put that in my constraint store by calling one of these
functions: `update-q-dfs` (update the queue in a DFS way), `update-q-bfs`
(update the queue in a BFS way), or `update-q-hybrid` (update the queue in this
weird hybrid search that I discovered).

I'm going to go over to how it actually adds this closure into the store.

```scheme
(define (update-q-dfs new-enforce)
  (lambdam@ (a : s c q-old t)
    (make-a s c (ext-q q-old new-enforce) t)))
```

All this is going to do is extend my queue by taking my old queue of goals (all
the lazy goals that existed before me) and add this new closure--this new goal
to run--to my queue, and then continue onwards.

Basically, when we get all the way through the program, we end up with all the
recursive programs we *would* have made in the constraint store. Then we have to
perform `enforce-constraints`.

**Jason:** Claire, two questions: One, how can you detect if a call is
recursive? Two, whenever you get done with this, I'd like to hear more about
your new hybrid technology.

**Claire:** Hybrid is thew rong word. It's just that I made a mistake while
trying to implement, and what I got out was a mix of BFS and DFS. It's nothing
fancy.

How do you detect if it's recursive? You don't need to detect if it's recursive,
because, for example, this call to `(syms* a a^)` is just going to be a call to
`(goal-construct (lazy-syms* t out))`. It doesn't know that it's a recursive
call. It just matters that it \[was defined as\] a lazy goal.

At the end, once we have all of our lazy goals in the constraint store, we have
to go through and make sure that they all run when they should.

```scheme
(define (fixpoint-enforce x)
  (lambdag@ (a : s c q t)
    (cond
      [(empty-q? q) a]
      [else
        ((fresh () (q x) (fixpoint-enforce x))
         (make-a s c empty-q t))])))
```

This function pulls out the queue `q` which is where all the lazy goals live, it
runs them by applying `q` (a continuation I've been building up) and then it
recursively calls `(fixpoint-enforce x)` again. Basically, It calls all the lazy
goals until there are no lazy goals left, and then that's the answer.

**Dan:** When you do a recursion, it creates a new lazy goal this time?

**Claire:** Yes. Since `syms*` was defined recursively, calling `enforce-syms*`
actually enqueues two new lazy `syms*` goals.

**Dan:** Is this similar to the same architecture in cKanren, then?

**Claire:** This is close to what is done in cKanren. At one point, I actually
had it implemented as an actual constraint, but it was too slow. So I stuck
another data structure into the package, which is this queue, and I implemented
it with a continuation. I tried doing it as an actual queue, but that was too
slow and it was hard to garbage collect. So instead it builds up a continuation
of things to go.

**Dan:** Is this some sort of CPS in the background, or `call/cc`?

**Claire:** CPS. These places in the code all call `ext-q` which says "wait for
me to get something to apply it to, then apply the first thing in the queue,
then the second thing, etc." I wish I could show this while it was running, but
since I actually implement these as closures, it's completely meaningless--you
can't see the goals. I'm working on a way where you'd be able to look at the
package and seew hat lazy goals are waiting to run.

**Will:** How's the performance?

**Claire:** It's okay. It's not great on applications like quine generation, or
running the evaluator with all fresh arguments. On small things, it is
definitely comparable to doing it strictly. But generating one quine does over
ten thousand calls to that fixpoint trampoline at the end of the program. So
that generates some slowdown. But for smaller, beginner programs like `syms*`,
it would definitely be feasible to use lazy goals.

**Will:** Are you talking on the order of 10x slower? 100x slower?

**Claire:** It's about 10x slower.

**Will:** If I understand this correctly, with this technology you no longer
have to worry about making sure your unifications come before recursive calls.

**Claire:** Yep!

**Will:** However, if you have multiple recursive goals in a clause, do you
still want to worry about their ordering?

**Claire:** Like what?

**Will:** Like an interpreter for `(+ a b)` where you have to evaluate both
sides, `(eval-expo a)` and `(eval-expo b)`. Sometimes the order of complex goals
actually does matter.

**Jason:** Or like summation up to `n` \[where you need both a two-argument
addition and a recursive summation\].

**Claire:** It wouldn't matter if you did a breadth-first search. If you did a
depth-first search, it's going to matter because it will left-recur. If
performance really matters, you'll have to look at what you're doing. This does
not solve all the problems at once! \[*sarcasm*\] I wish...

**Will:** Is this on Github now? Can you point them to where they should go?

**Claire:** I have the information on the Google group. I will post relevant
links into the chat.

**Will:** It's just your Github then?
[www.github.com/calvis/cKanren](www.github.com/calvis/cKanren):
[definition of syms*](https://github.com/calvis/cKanren/blob/8714bdd442ca03dbf5b1d6250904cbc5fd275e68/cKanren/tests/lazy-appendo.rkt#L16).

Cool. Any other questions, Dan?

**Dan:** I think that's pretty cool. I like it. Most of this is all about making
every goal lazy, and then picking it up later on.

**Claire:** Not every goal. You don't want to delay unifications, for example.
You would never want to make a goal like `conso` lazy.

**Dan:** Right.

**Claire:** Basically, anything that makes a recursive call or that could
possibly do mutual recursion with something taht is lazy, you would want to make
lazy.

**Will:** Do you want to say anything about `syntax-parse`? People may not be
familiar with that.

**Claire:** I would say don't worry too much that it's `syntax-parse`. It's
basically `syntax-case` with a little bit more error checking. All it does is
generate some temporary names so that I can get \[gensym?\] the different names
for `syms*`, `lazy-syms*`, and `enforce-syms*`. That's really the only thing
that requires `syntax-parse` over `syntax-rules`.

**Will:** So it makes it easier for you to get friendly error messages in your
macros.

**Claire:** Yes. And I'm not even using those capabilities yet. I just wanted it
in `syntax-parse` since I had the option \[since I'm working in Racket\].

**Will:** Fair enough. Okay, Michael, would you like to talk about your logic
puzzle? I know you and Lindsay were working on it a little bit.

**Michael:** This is something that I tweeted about. I should preface by saying
I'm an absolute beginner to logic programming and relational programming. I have
a very simple puzzle that reflects how far along I am in my learning. I've been
trying to absorb as much as possible. I have the good fortune of living in the
same city as David Nolen, and I was hanging out with him at a bar after the
Clojure meetup. I was saying that it's really interesting when trying to learn
miniKanren that you can spend so much time doing something that can seem so
trivial in another paradigm. The whole new way of learning and new way of
thinking/approaching problem solving is fascinating to me. I have a really good
time trying to figure out how to do some of these basic things. The really basic
puzzle was: Given a list where there could be repeated elements, return a list
with one copy of each duplicate. I'll show you what I mean.

The relation that I defined is called `dupeo`.
```clojure
(l/defne dupeo
  [l q]
  ([[head . tail] _]
    (l/conde
      [(l/membero head tail) (l/== head q)]
      [(dupeo tail q)])))

(defn dupeo-test [ls]
  (l/run* [q]
    (l/fresh [l]
      (l/== l ls)
      (dupeo l q))))

(dupeo-test [1 1 2])
(dupeo-test [1 1 2 2 3 4 4 5 5 3])
(dupeo-test [1 2 2 2 4 5 3 3])
```

I could talk to you about the desired result for these test lists, or we could
look at the relation itself. Which is better?

**Dan:** I have a clarification question.

**Michael:** Sure!

**Dan:** In your first test case `(dupeo-test [1 1 2])` does that just return
`(1)`, or does it return `(1 2)`?

**Michael:** It returns `(1)`. Let me run this and show you what it does now.
Then, we can talk about the defect that exists. Then maybe we can talk about if
there are some interesting ways to solve it.

This is what it returns now:

```
> (dupeo-test [1 1 2])
(1)
```

That's right.

```
> (dupeo-test [1 1 2 2 3 4 4 5 5 3])
(1 2 4 3 5)
```

That's right too. I don't really care about the order of the list that gets
returned. I could, but I don't in this case. It wasn't important for the
application I'm using this for.

```
> (dupeo-test [1 2 2 2 4 5 3 3])
(2 2 2 3)
```

That last one shows the most interesting edge case which is when you have more
than two in a row in the input list. Here it returns three `2`s.

**Dan:** What happens if you have five `2`s?

**Michael:** I'll show you five `2`s.

```
> (dupeo-test [1 2 2 2 2 2 5 3 3])
(2 2 2 2 2 2 2 2 2 2 3)
```

You get a lot of `2`s. Ten `2`s.

**Dan:** Now I don't know the algorithm. I did before, but I don't now.

**Will:** What is `dupeo` supposed to be doing? It's supposed to be giving a
representative element...

**Michael:** A representative element from each of the duplicated members of the
input list where no matter how many times it's duplicated in the input list, it
only appears once in the output list.

**Will:** So you expect to see `(2 3)` back?

**Michael:** I essentiall want the unique elements of what I'm curretnly getting
back. It detects duplicates, but it returns bizarre results when there are
multiple `2`s in a row.

**Will:** This is an overlapping problem. You probably have overlapping clauses.

**Michael:** The suggestions that I've gotten so far were to try to use some
king of `not-membero` in the second `conde` clause. David suggested using
something like `rember*o` to remove all instances of an element within a list.

I was chatting with David and he busted this out in like 13 seconds:

```clojure
(l/defne rember*o [x l o]
  ([_ () ()])
  ([_ (x . xs) _]
    (rember*o x xs o))
  ([_ [y . xs] [y . ys]]
    (l/!= x y)
    (rember*o x xs ys)))

(defn rember*o-test [x l]
  (l/run* [q]
    (rember*o x l q)))
```

```
> (rember*o-test 2 [2 2 1 1 1 2 1])
((1 1 1 1))
```

So that's right, because it removes all the `2`s and preserves the same amount
of `1`s. That's what I would expect that to do.

But it's pretty hard to get that going inside of `dupeo` to get that to do what
you'd expect. I would want to use some test in the second clause before the
recursion to see if the element existed. I don't know. I tried to do that in a
bunch of different ways and couldn't get it to work.

[47:54](https://youtu.be/MTAhbsWUUxM?t=2870)
**Will:** What I ended up showing Lindsey is that you can use the `absento`
contraint, which is relatively new. In fact, the fully generalized `absento` is
quite new. I don't know that it's made its way into `core.logic` yet. `absento`
is based on the "hash" constraint from nominal logic (from alphaKanren, or
core.logic.nominal). The idea is you can add a constraint that some term `t1`
does not appear in some other term `t2`.

It's a lazy constraint: if `t2` is a fresh variable, you don't have to
instantiate `t2` for this to happen. If `t2` eventually becomes more
instantiated (or, ground), this constraint will be checked to see if `t1` occurs
anywhere in there.

So you could use that in the second case. You could use the `absento` version
to say, basically, the head of that list is absent in the tail of the list. You
could say those sorts of things.

**Michael**: Yeah. I would be comfortable doing that. I tried a bunch of
different stuff. I tried to see if another fresh variable there would help to
unify the matching parts of the list, and try to add another `conde` clause that
would only unify that other.

**Will:** The main problem is that those two clauses in your `conde` in `dupeo`
are overlapping, right? If `head` is a member of `tail`, both of those clauses
will succeed instead of just one. You could also just write a `not-membero`
using disequality constraints. Have you tried writing a `not-membero`?

**Michael**: I did. There is a `not-membero`. I think this is a pretty decent
`not-membero`:

```clojure
(l/defne not-membero
  [x l]
  ([_ []])
  ([_ [(y . r)]]
    (l/l= x y)
    (not-membero x r)))
```

But I think I had a problem with one of the terms not being ground or something,
because in certain positions it wouldn't terminate when I was trying to use this
particular `not-membero`. There's another one in `core.logic`. I think that's
what Lindsey said. She asked if there was an existing `not-membero` that I
could use for my second clause.

So that's my simple puzzle.

**Will**: Alright. It seems like there are multiple ways to solve this. Just
with disequality constraints, I think you can do it with `not-membero`.

I'm surprised you saw divergence. It wouldn't surprise me if you left the list
fresh and did something like a `run*`, because then there would be infinitely
many lists of different lengths. But maybe we could try adding `not-membero` to
that `dupeo` call and see if we can get it to diverge. I don't see why it would.

Can you try adding a call to `not-membero` to the second clause of `dupeo`?

**Michael:** Sure. Before the [recursive] call to `dupeo`?

**Will:** Yes.

**Michael**: Let's try that.

```clojure
(l/defne dupeo
  [l q]
  ([(head . tail) _]
    (l/conde
      [(l/membero head tail) (l/== head q)]
      [(not-membero head tail) (dupeo tail q)])))
```

```
> 
```

52:00



















