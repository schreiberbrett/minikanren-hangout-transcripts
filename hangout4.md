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

**Claire:**
