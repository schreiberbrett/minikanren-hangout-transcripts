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
       (== s `(,a . ,res))])) ;; <-
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
       (== s `(,a . ,res))]))
```

I meant to do `d` instead of `a`. There we go. Okay:

```
debugging.rkt:33:0: error while running tests
expression: (run* (q) (rembero (quote x) (quote (a b y x)) q))
Expected: ((a b y))
Computed: ((a b y . _.0) (a b y x . _.0))
```

That's not good! 
