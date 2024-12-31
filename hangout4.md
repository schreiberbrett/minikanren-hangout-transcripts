# miniKanren Hangout #4, 7 April 2013 
Youtube link: [https://www.youtube.com/watch?v=WMjvyizzE78](https://www.youtube.com/watch?v=WMjvyizzE78)

**Will:** Welcome back. Claire Alvis is going to talk about debugging in miniKanren. And we will talk about constraint logic programming using sets. In between, we will have general discussion. Without further ado, since we're running behind schedule, I'll give the floor to Claire.

**Claire:** Okay. Right now I'm using a vanilla version of cKanren with just tree unification and disequality constraints (for later). I'd like to mainly talk about debugging multiplatform [miniKanren] programs--nothing specific to the Scheme, Racket, or Clojure version. I'll focus mostly on what you can do in any of them.

One thing to start off with are the changes I've recently made to cKanren to eliminate silly syntax errors that are still present in the Scheme version. I'm not going to talk about these, I just want to point out that, e.g., having an extra set of parentheses around a goal is no longer a problem. When you run it, it actually tells you that you're using a goal incorrectly. I won't talk about those kinds of errors, or misusing goals like this, where I want to run this goal, but really it's a fake goal.

```scheme
> (define fake-goal (lambda (x) "hi"))
> (run* (q) (fake-goal 5))
expression evaluated to non-goal where a goal was expected
  value: "hi"
```
THese aren't the kinds of debugging problems I'm going to be talking about, because there are other ways to solve those.

THe main thing that I want to talk about are these four ways to debug a miniKanren program.

1. Testing.
2. Printing.
3. Tracing.
4. Failing.

Testing is the most important thing. You'll want to write a lot of tests. Make them simple so that you can pinpoint exactly what's going wrong in your program. You'll want to print out a lot of information. You'll want to trace your programs. If worse comes to worst, you can start failing. I'll explain what each of these mean in due time.

I'll focus on an example that I'm hoping everyone is familiar with: `rembero` such that if I run it on `'x`, I get back a list with no `'x`s.

I know how to write this program in Scheme. I know [in miniKanren] I have some element `x`, some list `l` to remove from, and an out variable `s`. If I just do the normal recursion, I know I will have a case where `l` is empty, I have a case with this `(== l \`(,x . ,d))` situation, and I have a case with this other `(== l \`(,a . ,d))` situation.

```scheme
(define (rembero x l s)
  (conde
    [(== l '())]
    [(fresh (a)
       (== l `(,x . ,d)))]
    [(fresh (a d)
       (== l `(,a . ,d)))]))
```

I will start off by writing a test taken straight from my example.

```scheme
(test "rembero"
      (run* (q) (rembero 'x '(a b y x) q))
      '((a b y)))
```

That looks good. Now I have to fill in what's going on in my cases. If the thing I am looking for is at the front of my list, then I just want to continue into the `cdr`, and that will be my answer. Otherwise, I want to go into the `cdr` with this different result variable `res`, and then my out variable will be the `car` consed onto `res`.

```scheme
(define (rembero x l s)
  (conde
    [(== l '())]
    [(fresh (a)
       (== l `(,x . ,d))
       (rembero x d s))] ;; <-
    [(fresh (a d)
       (== l `(,a . ,d)) ;; <-
       (rembero x d res)
       (== s `(,a . ,res)))])) ;; <-
```

I think I've done everything fine, and then I run the program:

```
d: unbound identifier in module in: d
```

Unbound identifier. What did I do?

**Jason:** Is this pedagogical, or did you forget the fresh `d`?

**Claire:** No, I actually did forget the fresh `d`!


```scheme
(define (rembero x l s)
  (conde
    [(== l '())]
    [(fresh (d) ;; <-
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (== l `(,a . ,d))
       (rembero x d res)
       (== s `(,a . ,res)))]))
```

I meant to do `d` instead of `a`. There we go. Okay:

```
debugging.rkt:33:0: error while running tests
expression: (run* (q) (rembero (quote x) (quote (a b y x)) q))
Expected: ((a b y))
Computed: ((a b y . _.0) (a b y x . _.0))
```

That's not good! That's not the answer I expected. I'm getting `a`, `b`, and `y`, but I'm ending up with this extra answer, and I'm also ending up with a fresh tail `_.0` instead of the empty list at the end. This is something that happens to me a lot. We will see how testing can get you to see what's going wrong with this program.

I notice immediately that this example is too complicated. It's what I want `rembero` to do if it's working perfectly, but I really need some unit tests. What if I just did a `run 1`, taking `'x` out of the empty list?

```scheme
(test "test 1"
      (run 1 (q) (rembero 'x '()))
      '(()))
```

This is a simple sanity check.

```
debugging.rkt:33:0: error while running tests
expression: (run 1 (q) (rembero (quote x) (quote ()) q))
Expected: (())
Computed: (_.0)
```

I immediately know which line this in, since I know exactly what `l` is [in this case]. Of course, I see the problem that I have forgotten [to unify `s` to the empty list] when I know that `l` is the empty list.

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())] ;; <-
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (== l `(,a . ,d))
       (rembero x d res)
       (== s `(,a . ,res)))]))
```

So that is how making a simple unit test for the base case will help you find the problem in this program.

```
Testing test 1
Testing rembero
debugging.rkt:37:0: error while running tests
expression: (run* (q) (rembero (quote x) (quote (a b y x)) q))
Expected: ((a b y))
Computed: ((a b y) (a b y x))
```

But that still doesn't work. It looks a little better. Let's see what happens if I do another test. I expect to get the empty list again.

```scheme
(test "test 2"
      (run 1 (q) (rembero 'x '(x) q))
      '(()))
```

That works! What about a `run 2`?

```scheme
(test "test 3"
      (run 2 (q) (rembero 'x '(x) q))
      '(()))
```

Each time, I'm adding only a little bit more information to what I'm working with. Even though I'm doing a `run 2`, I still expect to get the same answer out, and I don't.

```
expression: (run 2 (q) (rembero (quote x) (quote (x)) q))
Expected: (())
Computed: (() (x))
```

There's an extra answer here, and that's not good. We've narrowed down exactly the problem, and that's pretty much as far as testing can get us. I'm going to start again by doing printing instead.

We need to figure out which of these `conde` clauses is causing the problem. What you can do is print a message to see which `conde` clause is getting through.

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s)
       (prtm "Line 2\n"))] ;; <-
    [(fresh (a d)
       (== l `(,a . ,d))
       (rembero x d res)
       (== s `(,a . ,res))
       (prtm "Line 3\n"))])) ;; <-
```

So if it gets through the second `conde` line, it'll print out "Line 2", and if it gets through the third `conde` line, it'll print out "Line 3".

```
Testing test 3
Line 2
Line 3
expression: (run 2 (q) (rembero (quote x) (quote (x)) q))
Expected: (())
Computed: (() (x))
```

I see that when I'm testing "test 3" I get an answer from Line 2 *and* and answer from Line 3. That's not good. I've now figured out that I get my two different answers [requested by `run 2`] from two different lines. To figure out which one [the false positive] is, I'll use failing. I will "fail out" line 3.

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s)
       (prtm "Line 2\n"))]
    [fail ;; <-
     (fresh (a d)
       (== l `(,a . ,d))
       (rembero x d res)
       (== s `(,a . ,res))
       (prtm "Line 3\n"))]))
```

I don't want to delete any of my code, since I'm pretty sure it's close to being correct. I'll just put a `fail` first so that it won't actually go down this clause. When I do that:

```
Testing test 3
```

I get the answer I was expecting for test 3. Now we know that case 3 is what's causing the problem. How do I narrow down exactly what's going wrong here? I will print.

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (== l `(,a . ,d))
       (rembero x d res)
       prt ;; <-
       (== s `(,a . ,res)))]))
```

When I run test 3, what does my substitution look like right after the recursive call?

```
Testing test 3
(((#_(res) . ())
  (#_(a) . x)
  (#_(d) . ())) . ())
```

I see that `res` is the empty list (which is should be). But I also see that `a` is bound to `'x`! I don't want this to be my "`a` is `'x`" line. We all know that you need the disequality here. I can look at my substitution and see right away that I wasn't expecting `a` to be `'x`. So:

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (== l `(,a . ,d))
       (=/= a x) ;; <-
       (rembero x d res)
       (== s `(,a . ,res)))]))
```

```
Testing test 1
Testing test 2
Testing test 3
Testing rembero
```

[No test failures]. Hooray! That's all well and good. But what if I had done something silly like this?

```scheme
(define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (rembero x d res) ;; <-
       (== l `(,a . ,d))
       (=/= a x)
       (== s `(,a . ,res)))]))
```

Now if you're a seasoned miniKanren user, you know to put your recursive calls last. But maybe I didn't know that; maybe I diverged like I am now:

```
Testing test 1
Testing test 2
Testing test 3
... user break
```

How do I know what's generating all these answers? I guess I can try tracing. In cKanren, there's a macro that will trace what your variables are when you come into a function.

An aside: here's why the normal [`printf`] doesn't work. If you do something like this:

```scheme
(define (my-goal q)
  (printf "~s\n" q)
  (conde
    [(== q 5)]
    [(== q 6)]))

(run* (q) (my-goal q))
```

This will print out:

```
#_(q)
'(5 6)
```

Just this variable `q`. That's not very helpful at all. What we really want to know is exactly what `q` is when it gets into the goal. Even if we had a `fail` here, Scheme would actually still print out that `q`:

```scheme
(run* (q) fail (my-goal q))
```

```
#_(q)
'()
```

Scheme will keep going until you hit the goal. It'll see that this is a `fail` goal and that `my-goal` is a function, it will print, and it will stop when it hits this `conde`. Even with the `fail` [appearing first], it looks like you're getting into `my-goal` because `#_(q)` is printed. That is very deceiving.

The way to get around this is by *tracing* as soon as you get into the goal:

```scheme
(trace-define (rembero x l s) ;; <-
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (rembero x d res)
       (== l `(,a . ,d))
       (=/= a x)
       (== s `(,a . ,res)))]))
```

If I do this, I'll probably get a lot of spam...

```
Testing test 3
(rembero x (x) #_(q))
(rembero x () #_(q))
(rembero x #_(d) #_(res))
(rembero x #_(d) #_(res))
(rembero x #_(d) #_(res))
(rembero x #_(d) #_(res))
...
```

Yeah. Okay. So this is printing out the current value of all my variables in the substitution when `rembero` is entered--not when `(rembero x l s)` happens, but rather it's right about to evaluate this `conde`. It's the same as wrapping a `fresh` around the goal and doing a `project`.

```scheme
(define (rembero x l s)
  (fresh ()                        ;; <-
    (project (x l s)               ;; <-
      (printf "~a ~a ~a\n" x l s)) ;; <-
    (conde
      [(== l '()) (== s '())]
      [(fresh (d)
         (== l `(,x . ,d))
         (rembero x d s))]
      [(fresh (a d)
         (rembero x d res)
         (== l `(,a . ,d))
         (=/= a x)
         (== s `(,a . ,res)))])))
```

Let's look at my trace. In "test 3", the repeated traces of `(rembero x #_(d) #_(res))` must be coming from the third `conde` case `(rembero x d res)`. I'm guessing this is what's going wrong, because it shouldn't be printing`#_(res)` anymore, and not `#_(d)` anymore. I know, since I'm running this in a ground list [in test 3], that these should have values. That really, these should have values first.

```scheme
(trace-define (rembero x l s)
  (conde
    [(== l '()) (== s '())]
    [(fresh (d)
       (== l `(,x . ,d))
       (rembero x d s))]
    [(fresh (a d)
       (== l `(,a . ,d))
       (=/= a x)
       (== s `(,a . ,res))
       (rembero x d res))])) ;; <-
```

That's the conclusion that any seasoned miniKanrener would do immediately.

```
Testing test 1
(rembero x () #_(q))
Testing test 2
(rembero x (x) #_(q))
(rembero x () #_(q))
Testing test 3
(rembero x (x) #_(q))
(rembero x () #_(q))
Testing rembero
(rembero x (a b y x) #_(q))
(rembero x (b y x) #_(res))
(rembero x (y x) #_(res))
(rembero x (x) #_(res))
(rembero x () #_(res))
```

Now I've gotten to what I was expecting.

That is all I was planning on talking about. Does anyone have any questions?

**Will:** Don't be shy. Any questions? Or are there other miniKanren debugging techniques that they would like to share?

**Unknown:** I have a question. Would it be possible in `trace-define` to have it introspect which [parent] `conde` clause corresponds to which binding? You can sorta infer.

**Claire:** Yes. With a little more machinery, that would definitely be possible. This version, no. The Scheme-y solution would be to assign a unique [gensym?] variable for *this* branch, and a unique variable for *that* branch, etc. When you go to print out one of these results, you can print out all the `conde` branch variables that you know about. Definitely possible. It's on the horizon. I haven't gotten there yet, though.

**Ambrose:** How do the printing goals correspond with the interleaving search? Do they interact in weird ways?

**Claire:** Yes! You're going to see answers printed out in the same order that the search goes. So you might be doing one thing and then be in a completely different place in the next printout. It would definitely be helpful to remember the path it was coming from so that you could print them out in some kind of more concise way.

**Will:** Austin, I know that you 've been interested in learning more about miniKanren debugging. Is this helpful?

**Austin:** Yeah, I've been doing something similar, especially with tracing the output. Unfortunately in the program I'm working with, there's so much going on. It's really easy to get lost in the sea of output, especially with that interleaving issue. One thing I've found to be really useful is that I set it up to print --similar to what Claire has--but it only prints when the depth has exceeded a certain bound, like 1000 iterations. Then I get back that single branch's path. That's helpful to route through all the hard-to-follow noise. But I feel like I need both: one with complete output and one with just the branch, because they tell different stories.

In doing this, one thing I've been wondering about is if there's a higher level way of looking at this (instead of just tracing branches). In the same way that we *program* these logic programs on a higher level [than the execution sequence]. Is there a higher level way of debugging, thinking about goals, and thinking about what's happening in the system than just following the thread of execution?

**Will:** I don't know if you have any thought about higher level debugging techniques, Claire. Some of the things Nada has been doing with metainterpreters have that feel to me. Also, there are these Prolog techniques that have been developed over time, like the Byrd Box model (which I think is named after Don [Lawrence?] Byrd). It would be cool to explore it in the miniKanren context.

Claire and Nada, you both have given though to debugging (and Nada's been doing metainterpreter stuff). Do you have any suggestions for higher level abstractions for debugging?

**Claire:** I'd definitely like to be able to trace the path of a single variable through the program. That would be incredibly helpful to me. Really, I wish there was some way to interact with the program as its running to step through the execution and be able to stop it, tweak variables, and continue in order to see what's going wrong. I think either of those would be very helpful.

**Austin:** Claire, have you looked at the SWI Prolog debugger?

**Claire:** Yes. It's out there. I wish there was one for miniKanren.

**Austin:** I just got into it last week. It's really interesting to be able to do things interactively. The fact that you can trigger a break at any time, see the current stack of goals trying to be solved, and start stepping through at that point. I thought it was really, really useful. I'd love to have something like that.

**Will:** Yes, that would be really cool. Nada, do you have any thoughts? The metainterpreter stuff is like debugging at a super high level, in a way.

**Nada:** I'm still thinking about how the metainterpreter stuff could be used in a more general setting than just type systems and programming languages, which was really the focus (the way I shaped it to be). I'm curious to ask Claire: Do you think it would be possible to integrate with Racket's debugging? How extensible is it? Have you looked into how to use the debugging facilities in Racket to use with new languages.

**Claire:** I haven't explored it very much, but I know they have a way to step through programs with the debugger. I just don't know how useful that would be given the structure of how miniKanren programs run. The debugging information I get when trying to debug cKanren is mostly unhelpful. There's a lot that would have to be done to get it to be transparent enough to work with Racket facilities.

**Will:** Dan and Jason, you guys have lots of experience debugging. (Maybe more experience debugging the implementation than debugging user-level programs). But I'm curious what techniques you have. Do you have debugging tidbits, thoughts you'd like to share, or tools you'd like to see?

**Jason:** A lot of it isn't terribly sophisticated, like print statements. I'm not above having a massive `trace-define`, watch my test suite run, find out where it breaks, and then walk back up. I don't like having to go to that, but it works.

Sometimes inserting a fake goal and printing out the state every so often. Really, by printing out the state every so often, I can check if it's a problem before I hit the reifier or after I hit the reifier. Figuring out which end of the system its on is really helpful and already halfway to the answer.

**Dan:** That sounds about right to me. Debugging is tricky in miniKanren, in terms of implementation. It's subtle.

**Jason:** I've found it to be useful to have a really nice, well-orchestrated test suite. Even make sure `(run 1 (q) fake-goal)` runs through the system where `fake-goal` does not use `==`, `=/=`, or other constraints. Even doing nothing, does the system work? Then add a couple `==`s and see how that goes. Start mixing in the features one at a time. We have the test suite orchestrated so that if it breaks on test 43, I know what I added between test 42 and test 43.

**Dan:** And it always breaks on 43. That's life.

**Will:** Those are good points when debugging the implementation, which is different than debugging a user program. But you're right about trying to find out if the problem is in reification or before reification. Because reification is complicated (and getting *more* complicated with all the constraints). A lot of times, you'll actually be producing the right answers but then reification chokes on it.

**Dan:** One of the things we're planning on doing (that we just haven't done yet) is: just as we have a `reify` goal (so to speak--it looks like a goal) at the end, we're also going to pull out the fixpoint as its own, fake, pre-reify goal. Every `fresh` used inside `run` will end up having two fake goals instead of one, which will probably give us a good opportunity to see where things happen. I don't like that they're all meshed together now. That's going to be a fairly simple change; I can't imagine it taking more than 10 minutes. We just need to get around to doing it.

**Jason:** That fixpoint itself is something to keep an eye on. We found that if you weren't careful, those fixpoint functions could interact and cause an infinite loop. If you're not aware of that (or not planning for that) when you're building the system, it's part of what makes adding constraints an *expert's* task.

**Will:** Very good. Thank you, Claire, for sharing your debugging approach. I hope that it will become more sophisticated over time, that our tools become better, that we can do more stuff like SWI Prolog, come up with higher level debugging techniques, and understand the metainterpreter stuff. I think we'll get better. I think because miniKanren still has a research-y feeling, these sorts of tools and debugging techniques have not gotten the attention they deserve. I think we can start addressing that a little more seriously. Hopefully core.logic will also get some of those things.

**Dan:** Claire, in your implementation of `trace-define`, did you just redefine `fresh` or something to use `project` in appropriate places? How did you do that?

**Claire:** All that `trace-define` does is it immediately inserts a `(fresh () ...)`, projects the arguments, prints them, and exits the fresh.

**Dan:** Can you do something special when exiting the [relation definition] so that you can see what happened to those variables?

**Claire:** Yes, I do remember doing that. It was not as helpful as I thought it would be. I forget why.

**Dan:** You saw a lot of unbound variables, probably.

**Claire:** Yes, and it didn't happen where I thought it would. If there was a recursive call, I wanted to see the printout right before the recursive call, but I got it [after?]. It was still confused about the mixed [interleaved] evaluation. It was not as helpful as I thought it would be.

**Dan:** It would really be nice if, somehow, we always knew when we were making a recursive call, but we don't. There's no way in the current model to know when you're making a recursive call. It would be really handy if we had some sort of technology like that.

**Jason:** That seems undecidable [Rice's theorem?]. You'd have to detect cycles [mutual recursion?].

**Will:** Time for a miniKanren compiler/rewrite system! A source-to-source compiler, or something like that. You could probably do that with a rewriting system. Let's open the floor to general discussion, questions, or things people are working on or interested in. After that, we can talk about constraint logic programming using sets that Claire and Nada have been working on. That stuff is pretty cool, but also complicated.

Are there any general miniKanren topics people would like to bring up?

**Austin:** I have one. A lot of this is new to me. I'm finding out what the boundary is between miniKanren and logic programming in general. A lot of the reference material is in Prolog. I'm curious what's the distinction and if there are big philosophical differences between Prolog and miniKanren. I know that computing forwards *and* backwards is a defining aspect in miniKanren. Is there anything else to be aware of where the nature of those systems is different?

**Dan:** I have a response to that. When Prolog first came out, the thing that made it really difficult for me to deal with Prolog was that absolutely everything was global. In miniKanren, the design was that everything should be lexical: if you want to define a relation, you have the option of setting it up inside a `letrec` or in a `global` define, however you want to do it. But you can make absolutely everything ready to be garbage collected at a moment's notice just by definiing all of your relations directly in a `letrec` so that when the `letrec` goes out of scope, you are done with it. That's a huge, huge difference. You don't think in terms of a global space that you can add to arbitrarily whenever I want (which you can do with Prolog) and then deal with on whatever level. That takes away power, but it's a conscious decision to take away that power.

Does that make sense Austin?

**Austin:** Yes it does. Currently I'm looking at a piece of Prolog code that I'm trying to port to core.logic. It might just be the syntax and semantics of how to put things together in miniKanren, but is there a big distinction between the goals in miniKanren and the clauses in Prolog?

**Dan:** Let me address that a little bit. Prolog's rules have a certain fixed syntactic format. A lot of Prolog code will process the expressions: taking apart pieces, putting them back together, and then throwing them into the global database. Prolog has this extra flexibility. If you don't know you'er involved in it, then it's very complicated to figure out what's going on.

Plus, they have this ability to cut `!/0` everywhere, which can cause a lot of pain. That's yet another variation.

What I decided, working on Kanren, was that if I really wanted to take apart and pull pieces together of a particular goal expression, I could do that by making a thunk out of it using `lambda` and parameterizing the things I want to be playing with. There was no loss for me, abstractly, between having a fixed grammar versus allowing myself to just take an existing rule and put it to sleep on certain variables that I could fill in later on with a piece of code. Once I convinced myself that I could do all the things that [Prolog programmers] wanted to do, I concluded I knew Scheme well enough that I wouldn't have to [transform first order goals], so I don't have to play around with that notion anymore.

You'll probably find a fair number of sophisticated Prolog programs using all this, which is called "meta-level reasoning". I don't know if that's helpful. How many lines of code is the piece of Prolog you're looking at?

**Austin:** Including whitespace and comments, it's about 500 lines.

**Dan:** That could take a while.

**Austin:** I have a working version in core.logic that I feel decent about. I'm still trying to understand al the semantics of it. But now I think I'm hitting problems that I think involve the logical structure of the program rather than the implementation.

**Dan:** Are there cuts everywhere?

**Austin:** Yes.

**Dan:** Okay. One of the fundamental things that the Prolog people insist on is being able to use a lot of cuts. That seems to be their growing interest. There's a really good, really elegant book [Clause and Effect](https://link.springer.com/book/10.1007/978-3-642-58274-5).

**Will:** That Clause and Effect book is by Clocksin and Mellish. \[Editor's note: Clocksin is sole author on Clause and Effect, Clocksin and Mellish co-wrote [Programming in Prolog](https://link.springer.com/book/10.1007/978-3-642-55481-0)\]. 

It's probably easier to translate from core miniKanren to Prolog in a sense. Although now we're really into the constraint stuff, which is not so trivial to translate. Like CLP(set) stuff that we're talking about: if your implementation of Prolog doesn't have support for that, then it's not so easy to implement at a user level.

The more cuts there are, the more uses of `copy_term/2`, `(is)/2`, and metalogical operators, the harder it will be to translate to pure miniKanren. If the code is pure Prolog, then it shouldn't be that difficult. Like you said, it just depends on the logical structure.

Philosophically, the emphasis we've had in miniKanren is towards purity, lexical scope, and all those sorts of things. For example, we don't have a way to add or retract a fact in a global database. `assert/1` and `retract/1` in Prolog are impure operators that Prolog programmers like to play with.

The way Prolog tends to be used in practice is that: yes, declarative programming is good; we'll start out as a declarative specification. But then (because it's easier to implement, or more efficient), we'll start adding all these non-declarative aspects to the program and try to reason about those. The miniKanren philosophy is more like: Let's really commit to being declarative. That has some tradeoffs, but we'll see how far we can push it.

The analogy I'd make is between a language like Scheme, which has `set!` in in and other "affecting operators", and a language like Haskell. If you get in a jam in Scheme where your purely functional program isn't efficient enough, then you can just do `set!`. But in Haskell, you don't really have that option (maybe you could go into a monad and do things under the hood), but it forces you to be more pure. I think that purity has lead to a lot of interesting results in the world of Haskell. I think that's also true in miniKanren: we're having to learn new techniques and come up with new constraints so that we can write things without using these extralogical operators. But it's definitely harder.

Any other questions? WHat have you been working on Ambrose?

**Ambrose:** I'm thinking of a question dealing with local type inference and solving polymorphic type variables in the presence of subtyping, but I don't know how to word the question, so I'll ask it after the CLP(set) stuff. I'm working on implementing a new local type inference implementation for Typed Clojure. I'm trying to think how to shoehorn core.logic in there somewhere, just to keep up-to-date with the constraint stuff. Subtyping and constraints sound pretty similar. That's pretty much what was missing when I was playing around with it a couple years ago, unification in the presence of subtyping. I have no idea how subtyping would work. Maybe it'll be better. I'll think of a better way to ask that question in an hour or so.

**Will:** My understanding is that one of the reasons Nada wants the CLP(set) stuff is to do subtyping. Is that right, Nada?

**Nada:** That's right. To even just do record subtyping, it makes sense to have CLP(set). In my wimpy implementation of CLP(set), I started doing this record subtyping and semantically it seems to be a good fit, at least for the simple structural subtyping of records. I just had to do a few tricks to avoid the usual nontermination issues. Like if I want to do type inference in addition to typechecking with subtypes and get the empty list instead of divergent behavior. I've been playing with this a little bit.

One thing I've been running into in my implementation of CLP(set) is I get a lot of redundant answers. I think in Claire's port of CLP(set) to cKanren she's *really* using constraints, and that makes it better in retrospect. I think soon we'll have a good story for simple record subtyping.

**Ambrose:** I guess I've got an extra complication. I've got normal polymorphic type variables. I guess they'd correspond to miniKanren logic variables. And I've got these dotted type variables, higher-kinded type variables, and these sorts of things. It's not really clear how to represent them. Maybe do the dotted type variables first and then other type variables. That's something I have no ideas for yet, just an interesting problem.

**Nada:** I'm not sure in your particular case. I also have a more complex type system that I want to use core.logic for. In my case, I also think using type constraints might be useful. Either different type constraints for the different kinds of things, or just even tagged lists, but I'm not really sure how to go about it in general.

If your type system is such that typechecking things that are annotated is decidable, but type inference in general is undecidable, then it might just be impossible [to avoid nontermination] when you runs things backwards.

**Ambrose:** The classic case I always think of is trying to enumerate descendants of all the subtypes of a class in Java; there's just no way to do it. You can get all the ancestors, but going the other way is not a good idea.

**Will:** What about Peter, Amir, or Nathan? Any questions?

**Unknown:** I'm interested in hearing about constraint programming. I've got some things I want to try with it. I'm eager to hear more about that.

**Will:** The set constraints will be super useful. We already have a number of uses in mind. Once we have it, I think that we'll realize there are lots of problems in the past that we didn't know how to solve which we now will have a chance at solving. Even things like going through Pierce's Types and Programming Languages book, I got stuck because of the sets. A lot of the things in that book are concerned with set manipulation.

If noone else has any questions ore comments right now, we can change gears and talk about CLP(set) a little bit.

\[Editor's note: A discussion about scheduling future miniKanren hangouts is omitted.\]

Claire and Nada, would either of you like to explain what CLP(Set) is, and what the core operators are in that constraint system?

**Nada:** I have the paper from [Dovier, Piazza, Pontelli, and Rossi called Sets and Constraint Logic Programming](https://dl.acm.org/doi/10.1145/365151.365169). This was my starting point to develop the wimpy miniKanren version that I have.

In this paper, they support a set operation that takes a set and and element and constructs a set that adds the element. It's a term constructor [like `cons`], so you can unify with that. It also has membership, disequality, unification, union (a three place relation taking two sets and their union), a "disjoint" operation (which says that two sets have no elements in common), and a type constraint to say that some term is a set. Depending on how you use these other operations, it infers type constraints on them. For example, if you use the union operation, it infers that each term involved has to be a set. That's basically what you have in the language.

All of these operations can be negated, so you have "union" and also "not union", "disjoint" and "not disjoint", etc.

Only some of these operations have to be lazy constraints; the others can be computed directly. One example is membership, which can be expressed entirely in terms of these other operations.

One catch in general is: in the way the rules are defined in this paper, many of the rewritings have `conda`s, essentially. That implies you can rewrite something into many options, and if any one of them succeed, then that's fine. That partly where you get the redundant answers. In this paper, they just show the rewrite rules and how you can get a solved form. I looked at their implementation in Prolog, `{log}`, and they use Prolog's backtracking to do this "any of" disjunction and rewriting. So in my miniKanren implementation, I did the same thing. I didn't use cKanren; I just started with what Dan Friedman presented last time. I have `conda`s all over the place--even in unification! Surprisingly, it works, but it's just a hack of course.

So now I'm going to pass on to Claire to describe how she made things better in the cKanren setting.

**Claire:** Like Nada said, things like unification (and especially `uniono`) can cause the number of answers to increase. As soon as you unify two sets (and maybe there are four ways to unify them) you will start getting more answers. In cKanren, `union`, `==`, etc. are all lazy constraints. It goes through the program, gets as much information as possible, and queues most of the unification and union work in the constraitn store. Right before reification (for those familiar with cKanren, in the constraint enforcement phase), it'll go through, pick out all the constraints, and [reify those it can't get any more information out from?].

**Will:** Could you show some of the test cases and explain what they mean? For the CLP(set) stuff, if you see a couple examples, you get an idea of what it's trying to do.

**Jason:** Does it support infinite sets? [No.]

**Claire:** Here's a test case.

```scheme
> (run* (q)
    (fresh (x y r s)
      (== q `(,x ,y ,r ,s))
      (== (set `(,x) r) (set `(,y) s))))
'((_.0 s.1 _.0 s.1)
  ((_.0 (set (_.1 : s.2)) _.1 (set (_.0 : s.2))) : (!in (_.0 s.2) (_.1 s.2)))
  ((_.0 s.1 _.0 (set (_.0 : s.1))) : (!in (_.0 s.1)))
  ((_.0 (set (_.0 : s.1)) _.0 s.1) : (!in (_.0 s.1))))
```

 We have some fresh variables, we unify them in a list, and then we do a final unification. What this syntax says is that `(set \`(,x) r)` is a set containing element `x` and the rest of the elemnts are in the set `r`. `(set \`(,y) s)` is some set containing `y` and the rest is some other set `s`. We have four different ways to unify this, which is why we get four answers.

* `(_.0 s.1 _.0 s.1)`: It's possible that `x` and `y` are the same and that `r` and `s` are the same.
* `(_.0 (set (_.1 : s.2)) _.1 (set (_.0 : s.2))) : (!in (_.0 s.2) (_.1 s.2))`: `x` and `y` are different and `r` and `s` are different, but they both extend the same fresh set `s.2`. Since these have to be proper subsets, `_.0` is not in `s.2`, and `_.1` is not in `s.2`.
* `((_.0 s.1 _.0 (set (_.0 : s.1))) : (!in (_.0 s.1)))` and `((_.0 (set (_.0 : s.1)) _.0 s.1) : (!in (_.0 s.1))))`: These last two answers are similar to each other. [In the former], `x` and `y` are the same, `r` is `s.1`, `s` is `s.1` with `_.0` [already included] so that when you insert `_.0` to both, you get the same result. [And vice versa for the latter].

These answers are really hard to read.

**Will:** Claire, did you point out what `s.1` means? It's a set. You're changing how reification works. If you see "s" instead of an underscore, it's a set.

**Claire:** Oh yeah. If you do:

```scheme
> (run* (q) (seto q))
`(s.0)
```

If you say that `q` is a set, you'll get out `s.0` instead of `_.0`. `_.0` would be a fresh variable without any type, and `s.0` would be a set. If you look at Nada's version, she has constraints on all of her set variables that say something is a set. This is just convenience. Instead of a bunch of side constraints, if you have `s.0`, it must be a set.

Here's a little more intuition on what `set` means.

```scheme
> (set '(x y z) (empty-set))
{ x y z | ∅ }
```

This is a set with some terms `x`, `y`, and `z`, all extended the empty set.

I don't have everything working yet. Nada has everything working.

**Will:** I have to say, I like those braces and the empty set notation. It's pretty classy.

**Claire:** Yeah. `set` is actually Racket's equivalent of records. I redefined how sets print out. I got it to print out with these cute braces so that it's easier to read.

**Will:** Can you show an example using set union?

**Claire:** Let's see.

```
> (run* (q)
    (uniono (set '(x y) (empty-set))
            (set '(y z) (empty-set))
            q))
'((set (x y z : ∅)))
```

If you union the set `{x y}` and the set `{y z}` then you get out the set `{x y z}` (built off the empty set). I'm glad that worked!

**Jason:** Claire, is there a notion of sets in a permuted order? How does the representation deal with that? If I add to the empty set `'(a b c d)` and another set `'(d c b a)`, how does unification handle whether those two are the same set? In general, how is that problem dealt with? When I was hacking together a naive implmentation, it about just made me throw up my hands.

**Claire:** This case?

```scheme
> (run* (q)
    (== (set '(x y) (empty-set))
        (set '(y x) (empty-set))))
'(_.0)
```

**Jason:** Something like that, yes.

**Claire:** Well, that's pretty much handled by the rules at the end. There are a bunch of cases about when you can actually do the unification. It's all covered by the `conde`s at the end. In this case, when everything is ground, you just do some `memq`s, and if everything works out, then just throw it away. But in a case with variables:

```scheme
> (run* (q)
    (fresh (x y)
      (== (set `(,x ,y) (empty-set))
          (set `(,y ,x) (empty-set)))
      (== q `(,x ,y))))
'((_.0 _.0) (_.0 _.1) (_.0 _.0) (_.0 _.0) (_.0 _.0) (_.0 _.0) (_.0 _.0) (_.0 _.0))
```

I guess there are 8 different ways. to unify those together. If they're the same they unify just fine, and if they're different they unify just fine. That's a lot of answers.

**Jason:** Dan wants to ask if there's a change to `unify` that you had to do.

**Claire:** Yes! `unify` is totally different.

**Dan:** Can you show us the code?

**Claire:** Yes, it's available on a branch [https://github.com/calvis/cKanren/tree/sets](https://github.com/calvis/cKanren/tree/sets). I haven't merged it in yet because I literally had to rewrite how cKanren works to make this go.

**Dan:** What were the philosophical changes to cKanren?

**Claire:** It's basically facilitated by Racket generics. I had to define what's called a "generic interface" in Racket that my set structure can implement. I said sets are "unifiable", and use a certain procedure to unify them. So `tree-unify` sort of punts on everything now. If it's a pair, or some symbols or numbers, I know what to do, great. Otherwise, see if this thing is unifiable and if so, punt to whatever procedure implements it.

**Dan:** Like ad-hoc polymorphism, basically?

**Claire:** Yes. This link specifically is what `unify` turned into:

[https://github.com/calvis/cKanren/blob/sets/tree-unify.rkt#L21](https://github.com/calvis/cKanren/blob/sets/tree-unify.rkt#L21)
```scheme
;; unifies two things, u and v
(define (unify-two u v)
  (lambdam@ (a : s c)
    (let ([u (walk u s)]
          [v (walk v s)])
      (cond
       [(eq? u v) a]
       [(and (unifiable? u)
             (unifiable? v)) 
        (bindm a 
         (composem
          (compatible u v)
          (compatible v u)
          (gen-unify u v)))]
       [else #f]))))
```

It asks if the things are actually `eq?`. If they aren't, it checks to see if they're both unifiable where it tries a generic unify method which is defined rgiht below. I needed Racket's generic interfaces to make that work.

**Dan:** In [Typed Logical Variables in Haskell by Koen Claessen](https://www.sciencedirect.com/science/article/pii/S1571066105805444), he basically has that happen automatically with Haskell's typeclasses.

**Claire:** That makes sense. I had to find some defaults for things like pairs and vectors that are builtin to Racket; I couldn't have them atually implement the interface. For everything else, you have to define it.
