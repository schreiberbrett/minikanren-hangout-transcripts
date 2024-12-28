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

Just this variable `q`. That's not very helpful at all. What we really want to know is exactly what `q` is when it gets into the goal. And, if we had a `fail` here, Scheme would actually still print out that `q`:

```scheme
(run* (q) fail (my-goal q))
```

```
#_(q)
'()
```

Scheme will keep going until you hit the goal. It'll see that this is a `fail` goal and that `my-goal` is a function, it will print, and it will stop when it hits this `conde`. 









