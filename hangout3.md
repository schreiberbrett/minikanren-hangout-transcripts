# Transcript of miniKanren Hangout #3, 31 March 2013

Youtube link: [https://www.youtube.com/watch?v=su3os5qMLr0](https://www.youtube.com/watch?v=su3os5qMLr0)

**Will:** Welcome to the third Google hangout for miniKanren. Today we have Dan Friedman, who will be explaining the implementation of core miniKanren: miniKanren with unification (`==`), `conde`, `fresh`, and the `run` interface to Scheme. The way we will structure this time is Dan will have the floor for has long as he needs. If we have time left over, we can get into questions and discussion. Hopefully it won't go the whole time. But if it does, that's fine. If it needs to be a 2 part thing, then maybe we can stop for questions and carry on next time. Without further ado, this is Dan Friedman.

**Dan:** Hi everyone. I don't want to spend too much time [with my face] on the screen, because what I really want to talk about is the code. What I'm going to talk about in the code is not only what the code is doing, but also what transpired to make certain things happen. I hope [the history] will give it a little humor. I'm going to do a screen share right now. Then I'll start talking about what the code is all about. Here we go.

I had a few concepts in the back of my mind when I wanted to write yet another implementation of a logic programming language. This time, my main goal was to make it efficient. I had written several others that did the things I wanted, but they were anything but efficient. One of the metaphors that I used was to use macros whenever I could verify something was finite. For example, the number of goals in an expression was finite, so I could use a macro to keep track of the number of goals. Another use of macros was to use `let` to hold the logic variables, because at any point in time there's only a finite number of them. It was always a dream of mine to somehow "trick" lexical scope into managing all the of the behavior of the logic variables, but after a while of thinking hard about it, I realized that wasn't going to happen. My next dream (which has yet to happen) is to have some percentage of the logical variables be managed by `let` and only `let`, but that has also not taken place yet.

You'll notice right away that almost everything on the page you're looking at  starts out with the `define` syntax. The code here is based on what I would have liked at the time of the first publishing of The Reasoned Schemer to have been. So this is something that is idealized (metaphorically) for a second edition that didn't have any of the material that we're going to be adding to the second edition. So now let's skip right past all of this stuff. This is all very simple stuff. You all know we have `vector`s. You don't know *why* we have vectors instead of cons cells for representing (or holding, so to speak) the logic variables. Although the initial version didn't use them, it became so much simpler to manage. So we went with vectors rather than lists or cons cells.

Now, `walk` was actually a fairly large discovery. In fact, Will was in my office at the time when I was drawing this picture. I was showing him that this was neat and to look at how this works. It isn't rocket science, but `walk` has the neat property that it (metaphorically) does not require any heap allocation. It's something you can do without paying for any memory. Of course if you really understand how the arrow `=>` works in `cond`, that `lambda` is actually built. That `lambda` does cause a cons cell allocation, so to speak. But for readability, we leave that arrow in. In actually running this system, we would use `let` [instead of `=>`], and that way there would be no allocation of memory whatsoever. That's how carefully we worked trying to manage the minimum amount of memory possible.

`ext-s` requires no comment.

`unify` actually does require a comment. For almost all [other] implementations of `unify`, when you get to the third line which says what if the other one's a variable, they just do a recursion. But we couldn't afford to do a recursion because we had already spent the energy `walk`ing the second argument. Having walked both of them, we're in a position where we have to do `ext-s-check` [in both cases]. You'll rarely see this in the literature on unification. The use of `and` in `unify` was actually a clever observation by Oleg. I know it doesn't look like any big deal, but we were [previously] doing it in a slightly more complicated way using `cond` with `=>`. This `and` turns out to be much more elegant. Shoutout to Oleg on that one.

The `ext-s-check` is, of course, as trivial as it could possibly be. I would like to mention that we're going to use the concept of "checking" in future code to mean: if it returns true, you have failed. So if you see some code in the near future with `-check` at the end, you`ll know always that returning true [there] means failure. I know that's sounds a little weird, but that's how it's going to be.

There's nothing to be said about 'occurs-check`.

`walk*` is only used by `reify` in this code. `walk*` has the property that it *will* be using heap allocation. But at the time that you are doing that heap allocation, you are done. You get to `reify`, you've done all your analysis, and now you do `walk*`. A little historical response on `walk*`: it's the only one that does a tree walk on the whole [term?]. Anyway, `walk*` is the only one that does heap allocation, and it does a simple Lisp-style tree walk.

One of the other features of miniKanren is we wanted that every possible program (that could be conceivably legal to run) would run. So it's meshing with all of Scheme (or in David's case, all of Clojure). It's guaranteed that all legal expressions in Scheme would also be legal in miniKanren. Of course it may not be the smartest thing to unify two goals, for example, because goals are just functions and functions don't unify very well. But if you wrote it and [the goals] happened to be `eq`, then you got whatever behavior you asked for. This was trump. We made a really large effort to get a seamless integration.

We're looking at `reify-s` now: approximately 3 lines of code [along] with a `walk`. It was 37 lines at one time. It was the second half of the chapter on unification. Will, Oleg, and I had a section in the book on unification. We also had a section in the chapter that was twice as long as unification. Unification should have been the most important part. Reification has some interesting aspects, but it wasn't the most important part. But it was taking up a lot of code, and a lot of code means a lot of explanation. We didn't want to have a lot of explanation; we wanted a small amount of explanation. We postponed submitting the final draft of the book to MIT press because this was disturbing Will and me a lot. We couldn't bear the thought of putting this monster amount of code in.

Just a little historical note here: Oleg was in California and Will and I were in Indiana. I called Oleg at midnight (our time) and I said, "Oleg, please let me just implement `reify-s` with CPS. Sure it'll be a little longer than I want it to be, but at least it'll all fit." And I got this very, very lovely response from Oleg that said, "No, we can only have one `walk*`." (Meaning there's only going to be one program for building the tree, searching the tree, etc.) I had to go along with that because I trust Oleg's judgement. I went to sleep that night, having looked at the code a lot over the last couple of days. This is what came out: this `reify-s`. It actually uses a metaphor (in the way it does the recursion) that was in the 37 lines. So `reify-s` was quite an event. Once we got `reify-s` to work, it was okay to send the book off and we can publish it. It was easy to throw out a lot of English lines and examples.

If you look closely at this, you'll see the first thing that happens with the recursion in the `pair` case is that it does the `car`. That means it's guaranteed to do the `car` side first and then it does the `cdr`. In each case, it returns the appropriate substitution.

`reify-name` is a trivial thing that makes one of our underscore-dot-number things [like `_.0`].

`size-s` is just `length` for the substitution.

What I haven't said yet is that the underscore-dot [notation] was also not exactly an accident. We have this concept of an `==` operator, the one that we all use to do unification. But at the time, when you would write a program, you would get different answers depending on the order of arguments to `==`. The different answers were alpha-equivalent: they were basically the same answers, but they were different! (Because they were equal up to alpha-equivalence, so to speak). We wanted to somehow come up with a way of guaranteeing that the answer would already take care of the alpha equivalence on its own. We came up with this idea of picking a name after the fact (using the reifier) that would, basically, normalize the information. The alpha equivalence issue that exists in many Prologs is that you get these random gensym-like objects coming out. And you think obviously that's an unbound variable. But there's no cohesion to it. By doing it like this [instead], we were able to say unequivocally that no matter what the arguments to `==` were, we were *guaranteed* that they would produce the same answers. That was really important, because Will started getting into this thing where he wanted all things to be commutative. It doesn't matter what the order of the lines are in `conde`. And it doesn't matter what goes on in `fresh`--those have to be commutative as well. We began to get into this very commutative state where everything has to behave in sorta this natural fashion.

This is the essence of `reify`, which basically says that we start with the value `v` coming in (whatever's bound to `q`, our query variable). Then we do this `walk*` over it. Basically, the only thing that survives after you do a `walk*` will be fresh variables and constants throughout the expression. So now we have this thing with only fresh variables and constants--it's as ground as it can be. This is consistent with the idempotent model of substitution, which corresponds to the way Prolog sees the world. The idempotent substitution (which I've played with a fair amount) does a huge amount of heap allocation, which means it's doing a lot of heap allocation prior to failing. I didn't think that was a good thing to do. The other thing, of course, is that I wanted to have this whole thing be failure-driven. What were the numbers we discovered, Jason?

**Jason:** I think it was 6-to-1.

**Dan:** It fails 6 times more than it succeeds. What was the example?

**Jason:** This was our full test suite.

**Dan:** Yes, our full test suite includes multiple calls to quines and other things like that.

Anyway, we get back this [`walk*`-ed] value `v`. We then form something called a "named substitution", which takes all the fresh variables and associates them with one of these underscore-things. Then we take the same `v` which has these fresh variables in them and do a `walk*` over [the named substitution], and we get exactly what we want: the variables' names are now these underscore entities.

Let's take a look at these next three macros:

`mzero` is a polite way of saying "fail".

`unit` is a polite way of saying [the program] knows that the infinite sequence of substitutions (which is what we deal with all the time) is actually one of the very small infinite sequences, because it's always of size 1. Because it's always of size 1, we're able to do magical tricks with it, because we don't actually have to form the infinite sequence with `cons` (which you can see happens with `choice`). So you end up with this kind of optimization: don't use unneccesary heap allocation. This came from the work of... I forget his name right now, but it's in the references in the book (at the beginning of Connecting the Wires). And [his observes] this property that you end up with no additional cons cells. You've already done enough cons cells to do the unify, but you know that `unify` only returns one substitution [at most]. It's tricked into thinking that it's an infinite sequence. We do this "tricking" thing a few times.

`choice` is just your standard way of extending the [stream of] substitutions
. We are `cons`-ing `a` onto a function of no arguments, `f`. When we want the [tail], we'll go after it by invoking `f`.

`case-inf` (and all these other little operators, `mzero`, `inc`, `unit`, etc.) were the brainchild of Ken Shan. You can see that's just a clever macro. It's been edited a little bit. We added an extra `procedure?` line (which was the brainchild of Oleg), which gives us the ability to do a kind of on-demand depth-first search such that you only go down paths that are making some progress. So that when you get your first element, it used the least amount of resources.

So we have these four lines. `(not a-inf)` is basically guaranteeing that we've failed. `(procedure? a-inf)` is an opportunity to switch whose in charge of the resources. The next line `(not (and ...))` basically says that we only have a singleton, so we can do some nice things there. Finally, in `else`, we know we one of those cons cells, so we rip it apart and run the code in that. So it's a kind of pattern matcher without actually seeing the patterns. It's all done with the magic of macros.

Next comes another little story. You can look at this `run` and say it's very cute, very short, and very nice. Everybody loves it. But that `(fresh (x) ...)` that's sitting there wasn't there. You can look at The Reasoned Schemer and you will see that we didn't figure this out. Why didn't we want to figure it out in the first place? If you look at that code, you'll see there's an invocation of something called `var`. Of course, `var` just creates a variable. That would be the variable we create for `x`, and then we would do something that would simulate the behavior that we have here now.

But also there was a place in `fresh` that used `var`. Every time we wanted to change the representation of a variable (and we had a number of varieties after publishing the book), we had to change two places. I can tell you that gets very old very fast. I put on myself the desire to have one `var`. You would have thought I'd have it figured out after 2 and a half years, but no such luck. One night just before I was about to go out to dinner, I called Will and said, "I have an idea. Maybe this one will work." I vaguely described it. I don't remember any words I used. Sure enough, by the time I got back from dinner, Will had this working. We haven't looked back since; it's exactly what we want. You might wonder why we don't just call `(reify x s)` here (because after all, that's what we want to do; we just want to reify).

Any ideas in the audience why it's not just a simple call to `(reify x s)`? The reason is [not] simple. That `(reify x s)` could, in principle, return `#f` if you didn't do something special about it. We wouldn't want to return `#f` if the `(cons ... '())` wasn't wrapping it. Because that would actually cause failure and you wouldn't get any answers. There's a `lambdag@` there which just means I know this is a goal, and so that was a problem: what to do about that `#f`. So we wrapped `(cons ... '())` around it, which at least will distinguish it from failure, since it is a list of one element. Seems like a reasonable plan. Not quite, until you look at `take`.

If you look at the third line of `take`, you'll see that if we have a singleton, we return a singleton. That's the secret. Even though we're returning a list of one element (which is what `a` will be), we're coming in through it via this `cons` so that when we do the recursion we end up with exactly the right answers. So if we're doing something like `(run 1 (q) (== q #f))`, we will indeed get `(#f)` which is what we want for that particular case. Otherwise, we'll do the sort of normal things.

The idea of doing `(and n (zero? n)` at the beginning and `(and n (- n 1))` at the end was another little gem of Oleg's. I had a more complicated piece of code there at one time for managing those numbers. We wanted to have only one operator that did the `run`, and if you wanted to do `run*` just pass in `#f`. If you think about what happens when `n` is `#f` you get false, which is exactly what you want in that case, and the same thing is true in [the recursive call to] `take`. It was a very small amount of energy consumed. Remember: the philosophy that Will, Oleg, and I have is that `reify` is allowed to be expensive--no one else is--because its job is to do something only once per answer.

**Will:** I think the line about sticking `reify` in a list is actually intuitively very simple. It's exactly what you see in `assq`. If you're looking up a value in an association list in Scheme using `assq`, you get back the entire pair. So if you're looking up `x` and `x` is bound to 5, you get bock `(x . 5)`--you don't just get 5 back. The whole reason is that if `assq` *can't* find `x` in the association list, it returns `#f`. So if `x` had been bound to `#f`, you looked up `x`, and you got back `#f`, then you couldn't tell whether `x` is absent from the association list or if `x` is actually bound to `#f`. That's why `assq` returns the entire pair instead of just the value associated with the value you're looking up. It's exactly the same intuition.

**Dan:** Absolutely the case. I couldn't agree more. The problem was finding that out: that we were messing up by returning `#f` in very rare circumstances and then not actually being able to do `(== q #f)` and getting what we wanted.

Moving down. The definition of `==` is actually a violation of types, because in principle at this point, we should actually be saying that if the unify succeeds then we do a `unit` and otherwise an `mzero`. But it seems so crazy to waste all that energy, so we didn't.

For `fresh`, there's not a lot to say. Will got this idea in his mind (and it was a really good one) that we ought to get as many programs to succeed as possible. You see this little `inc` guy in there? We have this collection of little, crazy programs that will go into infinite loops if `inc` ins't in there. We decided that we'll pay a tiny price of an `inc` just to make sure that a lot of really strange programs terminate, even though nobody should ever write them. Will has a whole collection of those and I'm sure he'd be happy to share them with you.

`bind*` is an example of what I was trying to get at when I said I wanted to have everything that looked finite to be hiding in a macro somewhere. `bind*` is one of those. I'm going to open up a Scheme window. Let's take a look.

We're gonna load this file now. We're calling it `miniC` because I gave it a name in a hurry and we didn't want to clobber anything that I could imagine.

```scheme
> (load "miniC.scm")
```

I just want to make sure it's running before we do anything magical.

```scheme
> (run 1 (q) (== 3 q))
(3)
```

By the way, I did a little count before we started. It's 220 lines of code with 40 definitions. For those of you who are curious how much work you have to do to actually get a good sense of what's involved in this very small version (which would have been what I wanted in the first edition), it's not a lot of work.

What I want to do now is something very simple.

```scheme
> (run 1 (q) (== 3 q) (== 4 q))
```

Everyone knows what that's going to do, so I won't discuss that. But I am going to expand it. Here we go:

```scheme
> (print-gensym #f)
> (expand '(run 1 (q) (== 3 q) (== 4 q)))
(take
  1
  (lambda ()
    (let ([s empty-s])
      (lambda ()
        (let ([q (#2%vector 'q)])
          (bind
            (bind ((== 3 q) s) (== 4 q))
            (lambda (s) (#2%cons (reify q s) '()))))))))
```

So this is what it expands to. Not very complicated-looking. We're doing a `take` of 1. That part is pretty natural. Then we have a thunk--a failure continuation. Then we initialize the substitution `s` to the empty substitution `empty-s`. Then we have another thunk. Then we lexically bind `q` to a logic variable `(vector 'q)`. Then we do this "and" operation. Forget the outer `bind` for a moment. We're going to generate `(== 3 q)` to `s`, so it'll take the first substitution. Then it returns a substitution that will be fed into `(== 4 q)`. When that's all done (this is the last part of the `run` where the reifier exists lives). Within `(lambda (s) ...)`, cons the reifier onto the empty list, and that's the second argument to `bind`. You can think of `bind` as a kind of "and" if you want. It's "and" for success and failure instead of `and` for Booleans. You can see that it's not very complicated. It's down to only being functions. For `bind*`, you really don't have to understand what it does, because you really only have to understand what the functions do. And there one is: `bind`. It feeds a substitution in, and I'll feed the result of that substitution into the next one, and so on. It's not a lot of code still.

Let me go back to the other code now. I just wanted to show you that you can expand it, watch it work, and see what's it's actually doing. It's not doing very much. (That was our goal, after all). Back to the code.

`bind` is actually a function, right? Think "and" for a moment. If you're in an "and" and you failed (which is represented by that empty list `()`), then you keep failing. It's the same way `and` works: if you give  `and` a `#f`, you get back a `#f`. The second line (case `(f)`) is only if you want to switch between where the resources are going. Then in the singleton case, we just apply `g` to the single substitution that came out with that `a`, because we know it's a singleton substitution. Finally in the last case `(a f)`, we're basically appending the result of `(g a)` and then building something that (when it gets its turn) will deal the invocation of `f` and invoke this `bind` again. But you don't know when it gets its turn--that's the main thing. Because it doesn't get its turn until `mplus` decides it wants to invoke the thunk in its second argument.

`mplus` is tied to `conde`, so let's take a look at `conde`. Once again, we have this little `inc` guy hanging out for virtually no reason. (Again, it's because we wanted to make as many programs as possbile run, and there are these weird recursive programs that we want to make sure run as well). You may notice that we have what looks like one-too-many occurences of `...` everywhere. That's because `(g0 g ...)` says we're gonna have at least one goal. The `...` at end of `(g1 g^ ...) ...` says we're going to have at least one *clause* . If we didn't have `(g1 g^ ...)`, you would be entitled to write `(conde)`. After some number of hours (or minutes!) of discussion, we decided that we would go with this and be user-friendly, which is not one of my strong suits. It is definitely one of Will's. So we have the extra lines. That's the story of `conde`'s syntax. But yes, we could have gotten rid of [the second call to] `bind*`.

You'll notice that we go right into the `bind*` here, whereas we could have used `fresh`. This was a tougher decision. In the long run, we thought that it would be better not to use `fresh` there, especially since we had an inkling that there would be people from time to time who would want to have a special fresh, for readability, for each `conde` line. That turned out to be correct.

`mplus` is our append, but `mplus*` is saying "I see that you're calling append with a bunch of arguments. But it's always a finite number, so let's play that game of turning everything that's a finite number into something that's a macro." That's why `mplus*` and `conde` are both macros.

We can look at what `mplus` does. You don't have to look very long to recognize that this is just a variant of `append`. `a-inf` represents an infinite sequence of substitutions (most of the time) and `f` is the other sequence of substitutions. So you have these two sequences of substitutions. The empty list in the first case means I've run out of the first list. If you were writing append, [then this is the `null?` case]. But instead of `(f)` you would just say `s`, the second argument of `append`. But since ours is a lazy second argument, you have to invoke it. So that's our first appearance of how `mplus` works.

You can ignore the `(f^)` case because that's just flipping the two thunks that are floating around because it's getting ready to change whose getting the resources.

**Will:** Actually Dan, that's the most important line in the entire implementation!

**Dan:** I'll come back to it in a minute. I just want to point out that `mplus` is like `append`. But you're right, it is the most important line.

The next line `(a)` is that special case of only one. So we can just cons that onto `f`. When it's time to look at `f`, we will, and that will happen inside `take`.

Finally, the last line `(a f^)` says I want something to cons onto the `f`, so we cons it on (which is what `choice` does). Then it makes a variant of [the thunk] but it swaps whose in charge. So that when you are inside `take` and invoke the thunk, you have `f` and `f^` switched. This is Oleg's work. This is how we get the ability to swap control. That happens both in the `(f^)` case and the `(a f^)` case. You'll notice that there's a `(choice a (lambdaf@ () ...))` in the latter and an `(inc ...)` in the former. Remember that `(inc ...)` just creates a `(lambdaf@ () ...)`. It's hard to argue why one is `inc` and the other one is `choice`, but it's because we're `cons`-ing in the first case and just [delaying] in the second case. We know we want to swap whose in control in both those places, but we also want them to be asleep so that the trampoline (so to speak) that's floating around in here will switch whose in charge of the resources.

Will, do you want to say any more about `mplus`?

**Will:** Do you want to talk about the interleaving search and the diagonalization metaphor?

**Dan**: We could, but it's almost easiest to think about it as a distributed trampoline.

**Will:** Can you please define some of these terms?

**Dan:** A trampoline is a way of taking two arbitrary expressions (as long as they've been CPSed), making them thunks (all tail calls have to be thunked), and then you can give control back and forth and see which one terminates. This requires you to have your code CPSed and all the tail calls thunked.

**Will:** When we were talking at Clojure West, some people said they didn't understand CPS. I think CPS + trampolines + distributed may be helpful only to advanced Schemers.

The original intuition was the diagonalization argument for having a one-to-one correspondence between the integers and the rational numbers. That's something that goes back to...

**Dan:** Cantor, in the 1890s!

**Will:** Cantor, yes.

**Dan:** The way I think about [the countability of the rationals] (perhaps so that other people can gain some insight) is that you write across the top of a piece of carpet: the integers from 1 to infinity. And then down the side: the numbers 1 to infinity. Then you roll the carpet up starting from the upper left hand corner. Each roll that you make represents a small finite number of values. You can always append a small number of values to an infinite number of values. So that's the whole process: just roll it up one step at a time and then you have all the rationals.

**Will:** That's the basic intuition. That allows you to deal with infinities in two directions, in a sense. The problem with depth-first search (which is used in the original version of Prolog) is that if you have one goal that can produce infinitely many answers, you can keep going down that one path and never give other conjuncts a chance to get into the action. If you want to have an interleaving search, one reasonable way to do it is this diagonalization approach where you're toggling back and forth. In fact, Oleg sent us a formula for an early version of this implementation where you can calculate the order of any answer based on the diagonalization. That's how I still think of it: every time you see one of these `inc`s, it's time to change direction and give a chance for another goal to run. That gets you off the depth-first search path.

**Dan:** We only have a couple more items here: `conda` and `condu`. They each call a little "if" helper. They only differ in one way: in the first argument to `bind*`. If you look at `ifu`, you'll notice something very strange: the first argument to `bind*` in the last case (where you have both an answer and more to do `(a f)`): [in `ifu`] you have `(unit a)` and [in `ifa`] you have the whole `a-inf`. What it's basically saying is that `f` is ignored. That's kind of like what happens when you use continuations: all of a sudden your future is gone! So `f` is the rest of the `conda`/`condu` lines, so to speak. As soon as you get success, you've given up `f`. There's no more lines, metaphorically, in your `conda` or `condu`. In the `bind*` for `ifu` you only get the one line, and only the first successful substitution out of that one line. Whereas in `ifa`, you get the whole sequence of substitutions. So that's the big difference; otherwise these codes are identical. I tried very hard to come up with a way to make this code a little bit cleaner, but I finally gave up and just copied, because I didn't see any other way to do it.

One of the metaphors that Will, Oleg, and I have is we wanted the code to be easily accessible, very short, and to do one thing reasonably well. We weren't concerned about putting in a pile of features. I do these things for my students. I just wanted my students to be able to say, "I think I can understand this with a little bit of energy." It turns out, this is not the code that I had originally before I met Will or Oleg. (I had already met Oleg, but it was just small sentences in meetings.)

**Will:** Before, you said something about the idempotent substitution model. What in the world is that?

**Dan:** The version that we have, with `walk` (which does no heap allocation) is not the conceptual model of how Prolog works. I think Prolog may have arrived before idempotent substitutions. They were probably trying to explain how Prolog worked, mathematically. [John] Alan Robinson may have talked about idempotent substitutions in his early '60s paper on unification. In any event, it corresponds to using `walk*` at every step: after you get back a subsbtitution you want to get rid of all the bound variables (those which have values) so that there's only fresh variables left. Remember, `walk*` removes all of the variables in the substitution which have values, so you're only left with unbound variables in the substitutions.

**Will:** Do you want to say what "idempotent" means?

**Dan:** "Idempotent" means that if you *apply* the substitution multiples times to the same term, you get the same term back. When you do it the first time you might get some changes, but if you do it a second time you will not get any changes (it will be identical). If you do it a third time it will also be identical. That is the standard meaning of idempotency in mathematics.

As I pointed out, `walk*` does a lot of heap allocation. In my earlier notes that Will got to see, I was doing only idempotent substitutions. It's a tricky exercise to implement them. I worked through [Foundations of Logic Programming by J. W. Loyd](https://link.springer.com/book/10.1007/978-3-642-96826-6), which is a formal description of logic programming. I've worked through it enough times following very assiduously (exactly what he was saying). Trying to trace through exactly what was going on was quite tricky. Trying to get the code to make sense was also quite tricky.

Eventually I figured it out, and I was happy to have done that. I was teaching my graduates and undergraduates this material, and one day--out of the blue--I received a 30+ page critique of my 100+ pages of notes that I made available to my students. In my naivete, I had forgotten (or didn't really realize) that anything I put out there for my students on the web is available to anyone running around. It's my understanding that someone from Lambda the Ultimate found and posted it. (You could probably find it now if you're desparate and have nothing better to do.)

Oleg [Kiselyov] got his hands on it and sent me this nice note criticizing almost everything I had come up with in my version [of logic programming] (which didn't have a name yet). Oleg gave it a name: we called it Kanren. Oleg started doing a lot of nice improvements to Kanren. It was running faster; Oleg came up with all sorts of neat hacks. Then Will came to Indiana University and decided to work with me. When I got to the logic part of my course, I don't think I made any sense to anybody. That's my recollection.

**Will:** My recollection is that you handed out to these poor students who didn't know about macros yet (including myself) this 100 page document full of Scheme macros explaining logic programming (which you hadn't explained). You said, "Okay, now start hacking the implementation."

**Dan:** I think that was pretty accurate, unfortunately. Those were busy days.

I was working with David Mack, who seemed to be grokking it, and Will, who had communicated as politely as he could, "I'm not getting this. This is not making any sense to me." When I finally got Will's message, I went home and thought really hard about how to take all the parts that are confusing in Kanren and turn them inside out (simplify them) using the metaphor that I'm willing to allow the user to work a little harder to make the language a whole lot simpler. The next day I came in and got Will's attention. We did not have it where it is now. But I had enough of his attention to where we could start drafting The Reasoned Schemer. I can't forget [to mention] that when I was developing the (then-unnamed) logic language, Steve Ganz pointed out to me an extraordinary observation. I won't go into the details of the observation, but it certainly got rid of the issue concerning [logic] variables and lexical scope. That's what I wanted all along. It wasn't coming out the way I wanted [until Steve's observation].

Then Mitch Wand came to Bloomington to work with me on [Essentials](https://mitpress.mit.edu/9780262560672/essentials-of-programming-languages/). While he was here, he said that if we get done with our goals for this trip, then he has comments to make about the first 60-or-so pages of our draft [of The Reasoned Schemer]. Mitch made a lot of good observations--removing more things--which has gotten us down to *this* very small implementation which should have been in The Reasoned Schemer: it would have only been `run`, `run*`, `==`, `conde`, `conda`, `condu`, and `fresh`. Of course `conda` and `condu` barely made it into the last chapter of the book. It would have been a lot tighter and more elegant.

It's only 220 lines of physical code. You look at some of these lines: [the definition of] `lambdag@` could easily fit on one line. A lot of these are long because I have certain syntactic rules when I write code.

**Will:** Nada was able to dig up some of those links, like the Lambda the Ultimate link and the original 30-page thing. I put the links in the group chat.

**Dan:** I never saw the 30-page thing except for an email, that's great!

**Will:** I still have the 100-pager on my laptop. I should put it somewhere. I've got some ideas where to put it.

**Dan:** I've used my allotment here. Let's switch over to something else.

**Will:** Thank you very much Dan. We can move on to other miniKanren topics. Hopefully this discussion will help people who look at the implementation in the book and are stymied. I wonder if this would have helped David when he was porting to core.logic.

I will open it up to general discussion. I can start. Nada, Dan, Matt Might (my boss at Utah), and I just submitted a paper to ICFP (International Conference on Functional Programming) on the relational CESK machine, which Dan and I talked about at the Clojure West miniconfo. We were able to clean it up and get it working. In the process, we learned quite a few things. Nada and I could talk about that.

**Claire:** I've been working on a blog post about cKanren. Nothing ready yet. I was curious to see how the work on `absento` was going regarding getting things to fail when they should.

**Jason:** `absento` can work with arbitrary terms in either position. That's live; that's a go!

**Will:** Claire, did you mean `presento`?

**Claire:** I think it was the combination of `presento` and `absento`, like the cases we were talking about in the Google group.

**Jason:** I think problems can emerge even with plain disequality constraints.

**Will:** If you have disequality [mixed with `absento`], that's enough. As David pointed out, you can have:

```scheme
> (run 1 (q)
    (fresh (l a b c d x y z)
      (== `(,x ,y ,z) l)
      (presento a l)
      (presento b l)
      (presento c l)
      (presento d l)
      (=/= a b) (=/= a c) (=/= a d)
      (=/= b c) (=/= b d)
      (=/= c d)
      (symbolo a) (symbolo b) (symbolo c) (symbolo d)))
'()
```

The system would have to be smart enough to solve this.

**Claire:** Has the system gotten any smarter yet? Is this example still a problem?

**Jason:** I don't have a `presento` implementation that handles this or any of the other cases we've been playing with. It's still being rapidly being written and erased from the chalkboard. We're far from the coding stage.

**Will:** Does `presento` even make sense just with core miniKanren (without disequality)?

**Jason:** With just `presento` and `==`, I think that *seems* to work. They don't harm each other. It's when you have `=/=` (and perhaps `symbolo` and `numbero` constraints). Those are what seem to make the search expensive. Just my intuition right now.

**Will:** I'm actually still worried about `presento` [even with only `==`]. Without something else like shape constraints, I'm afraid of situations like using `presento` to say:

```scheme
> (run 1 (q)
    (fresh (z)
      (== q '((a . 1)
              (b . 2)
              (y . ,z)))
      (presento '(x . 5) q)))
(((a . 1)
  (b . 2)
  (y . (x . 5))))
```


Some binding `(x . 5)` exists in an association list which is completely ground except for one association `(y . ,z)` for some fresh `z`. `presento` will happily say that `,z` has to contain `(x . 5)`.


You could argue that what you're asking for is not correct, but I'm still trying to think of uses for `presento` in the simplest cases that aren't gonna get you in trouble. Maybe if you had types or shape constraints. Otherwise, I can't think of a use-case that makes sense. Maybe there are some.

**Jason**: I don't know a use-case [for `presento`], but I also don't have a great use-case for the fully general `absento`, either. Maybe that itself was a bridge too far.

**Will:** Maybe. Although with `absento` it *is* useful to have different types of atomic values. The CESK machine uses both numbers and symbols. But yes, we haven't found a use-case for arbitrary terms. Even for `presento` [where the first argument is] restricted to be symbols, I'm afraid you could get into trouble; I'm not sure. It feels like a very different type of constraint from anything else we've had before.

**Jason:** Hm. I don't know.

**Dan:** It is a hard problem. That's clear. We have been working on it a fair amount of hours. Every time we think we're getting closer, the horizon shifts. It's very tricky.

**Will:** I was talking to Matt Might about using an abstracted version of the [relational] CESK machine for static analysis (which was its original purpose). He was thinking that instead of doing the CESK machines, it might be better (for static analysis) to do the constraint solving approach using set constraints. He thinks if we had constraint logic programming over sets, that would be very useful. I know Nada wants it for type inference. David wants it for various problems as well.

**Dan:** You have to keep in mind that `presento` and `absento` are for exactly that, but for data structures.

[Will shakes his head.]

No?

**Will:** They're not even close to being the same. Think about the operations you have on sets: intersection, membership, union, difference, etc. We don't have operations like that yet.

**Dan:** We'd be using things like `subseto`.

**Will:** I disagree that they are closely related. If you had a fully general `presento`, you could probably use it along with `absento` to emulate constraints over sets.

**Dan:** Yes, that's what I'm talking about.

**Will:** I actually think that's a proof that you can't do it--that `presento` doesn't make sense, at least in the general sense. I could be wrong.

**Jason:** Could you talk a little more about why that leads you to think `presento` doesn't work in general? I missed your reasoning on that.

**Will:** If you look at any of the work on set-based constraints, they all have serious restrictions on the operations. If you look at the cutting-edge work on logic programming with sets, they only give you a handful of primitive operations--you have to derive everything else. Even then, it's a real mess to implement anything. It's ridiculously hard to do; the computational complexity is through the roof. People are really struggling with this problem, even when they have more structure imposed on the data structures and more fixed operations.

The fact that we were able to start playing games using lists with `presento` and `absento` to represent sets: that if we had [a certain primitive] we could implement this-or-that set operator. It started making me nervous, because people doing *just sets* are pulling their hair out to get it to work. I could be wrong, but that's just my intuition. If an operator is powerful enough to let you do set operations on lists, then it's probably too powerful.

**Dan:** When you say too powerful, do you mean unimplementable, or just useless?

**Will:** It may be undecidable. Maybe you could implement it. But I think it would become "the presento system" with other things on it. You'd end up with most of the code trying to deal with `presento`, even in the best case I think. The reification stuff already looks like it would be horrendous. Even having lists causes trouble when you have `=/=` and `presento`. I don't see how this would work with other types of constraints. It seems like if you just had `==` and `presento` over lists, you may be okay. But once you add even just `=/=` it looks like it becomes a nightmare.

**Dan:** We'll let you know! We're working on it.

**Will:** I'm more than happy to be proven wrong about this. I'd like to have it. `presento` is like a genie that when you ask it a wish, it squirms around and...

**Jason:** Fulfills it in the most terrifying way possile. miniKanren itself is like that, too.

**Will:** No, not like `presento`.

**Jason:** Definitely not as bad as `presento`.

**Will:** I can reason about the other constraints in miniKanren. But every time I start thinking about a use-case for `presento`, it seems I would also need shape constraints (and even that probably wouldn't be sufficient). I'd need all these other constraints, but then I'd have to think about how they could interact with `presento`... ugh.

**Dan:** We hear you.

**David:** Even `absento` is pretty weird in the presence of anything that involves a finite domain. Finite domains in general have problems with `presento` and `absento`. For example:

```scheme
> (run 1 (x)
    (fresh (term)
      (boolo x)
      (presento x term)
      (absento #t term)
      (absento #f term)))
```

Even that seems tricky, even if `x` never takes on a value. You've simply said `x` belongs to a domain of values and [simultaneously] that none of those values are possible.

**Will:** Well, we don't have `boolo`. But we could suppose a finite domain over 0 and 1 `zero-or-oneo`.

**David:** Sure.

**Will:** So you'd have:

```scheme
> (run 1 (x)
    (fresh (term)
      (zero-or-oneo x)
      (presento x term)
      (absento 1 term)
      (absento 0 term)))
```

**David:** If you never give `x` a value, you're going to run into a problem.

**Will:** Won't it just turn into a disequality constraint?

**David:** Maybe you're right. You guys do that trick in cKanren where you give something a domain even if it's not part of the answer--you still check that it's satisfiable. You're probably correct.

**Will:** I think it works in the `absento` case. I'm much more worries about the `presento` case. Didn't you [David] or Nada find some issue while in Portland regarding the code in cKanren to check to make sure there is an answer? Weren't you worried about that? I remember that came up in discussions.

**David:** We worried that you might have constraints that are not satisfiable that are not part of the [reified] answer.

**Will:** Right, which is the whole point of that piece of code.

**David:** You can do a `onceo` for any variables that aren't part of the answer. But if the system you designed doesn't know how those constraints should interact with each other, you will get the wrong answers.

**Will:** Wasn't that not a problem with the cKanren system, but rather letting users write [their own] predicate to specify a constraint?

**David:** Yes. This is an easy mistake to make even for the constraint writer! Even if you're writing a [builtin] constraint, you can make this mistake.

**Will:** Oh yeah. Writing constraints, as we've found out, is very difficult. (Not necessarily their implementation--there are things like Constraint Handling Rules that we could adopt.) But my understanding is that CHR doesn't help you figure out the semantics/interactions between constraints. That's the tricky part.

I think Dan's view on things is the CLP approach: make sure the different variables have different domains and constraints and that way you don't have to worry about these interactions. Maybe we are making it too hard on ourselves by allowing a bunch of different constraints on the same variable, because that's where it gets messy. I wonder how general these other systems are in terms of allowing multiple constraints on one variable.

**Claire:** I want to tackle the general problem and not restrict it.

**Will:** I agree. The general problem is more interesting and useful. But it's also much, much harder. Because you have an $n^2$ interaction between the constraints on the variable. We don't have very good tools (or theory) to guide us. Something equivalent to a "code coverage" tool would help us explore and consider all of the possible interactions between different constraint cases. It's very easy to miss a case.

**David:** Actually, maybe my [earlier] complaint about `absento` and finite domains was that it (annoyingly) fails late. It works, but the failure happens very late, whereas you would like to notice it's unsatisfiable as soon as possible. Kind of like what Dan and Jason are trying to do in the more recent versions of miniKanren.

**Will:** Can you explain that more? Why does that failure happen late?

\[Editor's note: in the following discussion between David and Will, they might be mixing up the order of arguments to `absento`, which turns into disequality only when its *second* argument is an atom, not its first.

```scheme
> (run 1 (q) (absento q 'foo))
((_.0 (=/= ((_.0 foo)))))
> (run 1 (q) (absento 'foo q))
((_.0 (absento (foo _.0))))
```

This transcript does not correct the mistake. \]

**David:** Because the failure happens only when you start enumerating the domains, even though you knew it was unsatisfiable long ago.

```scheme
> (run 1 (x)
    (zero-or-oneo x)
    (absento 0 x)
    (absento 1 x))
()
```

If you say that `x` must be either 0 or 1 but then you say 0 or 1 cannot be present, then somehow the system should fail then. But really it will wait until reification to enumerate the domains and then fail very late.

**Will:** I don't remember the implementation of cKanren well enough. Is that true in general? If you have that `x` is in the domain 0 and 1, but put two disequality constraints on `x` right away, does it wait until reification to fail?

```scheme
> (run 1 (x)
    (zero-or-oneo x)
    (=/= 0 x)
    (=/= 1 x))
()
```

**David:** *That* will fail early.

**Will:** I think `absento` actually becomes `=/=` immediately in the case you were describing.

**David:** No, if you say something is absent, you have to wait because you might have some uninstantiated term. It's because `absento` can be applied to any sort of tree.

**Will:** Yes. You could have something like:

```scheme
> (run 1 (x)
    (fresh (t1 t2)
      (zero-or-one x)
      (absento t1 x)
      (absento t2 x)
      ;; ...
      (== t1 0)
      (== t2 1)))
()
```

And that will fail later. Maybe that can be made to fail a little sooner. But you can't fail until you know what `t1` and `t2` are. They might even be the same term `0`, in which case you still have an answer.

**David:** Yes.

**Will:** It be interesting to see if there's an example where it's not failing as quickly as it should. I think should still fail pretty fast here. It shouldn't wait until reification if `t1` and `t2` ever become ground.

**David:** Maybe you missed what I was trying to say. What if that `x` (in `(zero-or-oneo x)`) is not in the answer? It might be some side variable. It might never get ground, which means you won't test if it's satisfiable until the very end.

**Will:** Is that true in cKanren? If you do the simple ground case: `(absento 0 x)` and `(absento 1 x)`, that should be equivalent to saying `(=/= 0 x)` and `(=/= 1 x)`. Doesn't that fail fast even if `x` doesn`t appear in the answer? Or does it wait until reification?

**David:** My example was more like `(absento 0 t)` where `t` is some uninstantiated tree term. Later, some goal inserts a subterm [within `t`?] constrained to the domain 0 and 1. I don't know.

**Will:** `absento` is clever.

**David:** You're right, I think it will fail fast, because the `absento` will be like disequality.

**Will:** Nada, did you want to say anything?

**Nada:** I had the same confusing as David regarding if the constraints are not actually part of the answer are unsatisfiable. Can the system always figure it out? For the current implementation of core.logic, I think not. Because we don't actually check if any constraints are mutually exclusive. But in miniKanren, you really do consider all interactions between constraints. That's a difference in design/philosophy between the two systems.

**David:** We do want to fix core.logic! This problem is something we want to fix.

**Nada:** But if we allow user-specified constraints, it's really hard to fix. We could [only] fix it in the trusted [builtin] constraints.

**David:** Yes.

**Will:** What's a trusted constraint?

**David:** We would have a set of constraints that we ship with which we've tested, which are similar to miniKanren's, and which (we're fairly confident) interact well together, even if these constraints are applied to terms that don't appear in the answer. If it's some side constraint we should still be able to detect failure.

If we allow users to define their own constraints, there are no such guarantees. In fact, as we're seeing, it's pretty hard to craft these constraints to interact properly. Users [defining their own constraints] could easily make mistakes. We would want to be able to run in an [unsafe] mode which emits a warning when a user-defined constraint is used.

**Will:** I can see that. Or, you might be able to have something like an "isolated constraint". You could enforce that such a constraint (+ unification) are the only two constraints, for example, apply to a variable. And that if you use other constraints then you'd get an error at runtime. You might be able to do things like that.

**David:** That's better because it's more flexible. You'd detect multiple incompatible constraints at the var level instead of at the whole-program level.

**Will:** Yeah. I think it's easier to implement with attributed variables than with our current architecture. But I'm sure we can do it.

Austin, you always have good questions. Got anything for us?

**Austin:** The big issue on my mind as I'm learning this is about negation. I'm really curious how the disequality constraints factor in with negation. For instance, I think I have a bug right now where I need something to fail (negation as failure) but instead I'm succeeding with a disequality. This is all very new to me. Anything to do with negation seems to be my biggest stumbling block.

I'm looking at a lot of Prolog stuff too--what [is the Prolog version of] `conda` and `condu`? Do the semantics change?

**Will:** That's a good question. Negation turns out to be, in my mind, the trickiest aspect of logic programming. In Prolog, they have cut. I think it's only one operator, but they have different ways of thinking about it: red cuts, green cuts, etc.; which ones are safe and which ones aren't (that is, which ones prune the search tree too aggressively and are unsound). A cut `!/0` is a global operator, in some sense. We don't exactly have cuts in miniKanren, but `conda` and `condu` can simulate uses of cuts. You might have to do a bigger transformation on your program in order to get all the cut behaviors.

In general, if ou want to model negation as failure, you want to use either `conda` or `condu`. The main difference between `conda` and `condu` is that in `condu`, the first goal in the clause can only produce at most one answer--there's no backtracking even in the first goal.

The `conda`s and `condu`s are unsound in general; you can use them in ways that will lose answers that should exist. You'll get back `()` from a `run` instead of the answers that exist.

There are ways to make sure that your uses of `conda` and `condu` are sound. In particular, Oleg talks about this "g-rule" (ground rule) saying that the special first-place goal in each clause of a `conda` or `condu` (which controls the control flow) had better not ground (or instantiate) any of the variables in that call, otherwise it could be unsound.

The disequality constraints are different. Disequality is really the dual of unification: it makes sense logically and there's lots of math on it. It's a very weak form of negation compared to negation-as-failure.

For example, you can say that "`x` is not 5" or "`x` is not the empty list", but there's no way to say that "`x` is not a pair." You could say that `x` is not a particular pair, like `(=/= x '(,y . ,z)`, but that's just saying `x` is not *that particular* pair. If you later say `(== y 3)` and `(== z 4)`, then you're just saying `(=/= x '(3 . 4))`--`x` could still be any other pair.

So disequality is very weak. It can't express very much. But that's why it works well, actually. If it were more powerful, then you'd run into trouble trying to solve the constraints (or soundness problems). Disequality constraints are a very weak form of negation. These other constraints we have (finite domain constraints or the set constraints we want) can each be looked at as some encoded form of negation--even things like `symbolo` or `numbero`. If you know some `x` is either a symbol or a number, you can have `(symbolo x)` in one `conde` clause and `(numbero x)` in another and the clauses will be mutually exclusive.

But in general, we don't have a "super-negation" operation. I don't think it can exist and still be sound and implementable.

**Austin:** The situation where it keeps arising for me is when trying to write the non-overlapping clauses of a `conde` without using some form of negation to create that separation.

**Will:** Yes. This was a big stumbling block in our interpreter stuff. Even for the lambda calculus: you have variables, procedure application, and lambdas. Lambda, itself, is like a tag. You can unify against a list whose first element is `'lambda`, and that makes sure it's a lambda expression. But before we introduced the `symbolo` constraint, there was to way to say something was a variable (just a symbol). We actually need the `symbolo` constraint to enforce that something is a variable.

We used to tag expressions. We had a `'var` tag so that variables would actually be `(var ,x)`. And procedure application would be `(app ,operator-expression ,operand-expression)`. The problem with that was when we wanted to generate quines and deal with `'quote`. Inside the quotes you had to have to have `'app` tags, `'var` tags, and other really weird ways. It made the answers almost impossible to interpret, parse, and unparse. It was a real mess.

`symbolo` let us get away with not tagging variables. And we just removed the `'app` tag for application. There's a neat trick there: in our interpreter, application actually *does* overlap with `quote`, `*`, and things like that. If you have an expression like `(quote ,x)`, application will also be tried, it just happens that the variable lookup with fail: `quote` isn't a variable bound in the environment (so it would fail). That's a case where we actually violate the non-overlapping principle at the "pattern matching" level but enforce it at the "variable lookup" level.

In general, things like tagging or using special constraints like `symbolo` or `numbero` (or using other constraints useful to you) are the first line of defense in making sure things are non-overlapping. If you're writing a [relational] reducer, where you have an expression you want to keep reducing until it can't be reduced anymore, you can just keep reducing it until you get an actual value, like a number or closure expression--that's another technique you can use.

But you have to look at the individual problem. If you explain your problem in more detail, or post your code to the mailing list, we can help you with that.

**Nada:** I have a problem where I run into this. It relates to the issue where you can't have the negative constraint which asserts that a term is not a pair. Let's say you have a `conda` where your first clause is matching against a pair and the other clause matches when you don't have a pair. Is there any way to transform this `conda` into a `conde`?

**Will:** Maybe. Sometimes you know that you either have a pair or a number, symbol, or some other atomic type. Then you can enumerate (have separate clauses) for those. But if you *really* want to say something is not a pair, you need a separate constraint for that. We've had that constraint in the past: `not-pairo`. We've had it in the language a couple times; Dan has advocated for it a number of times. But it seems like every time we use it, at some point we find a way to get rid of it. If I find myself using `not-pairo`, I'm probably doing something else wrong. But it's an easy constraint to write--the only thing you have to look out for are all these interactions with other constraints.

In the chat, Claire says she thinks cKanren ships with `never-trueo`. What is that?

**Claire:** You can write `(not-pairo x)` as `(never-trueo pair? x)`.

**Will:** Uh oh. This sounds like the core.logic thing where you can define an arbitrary constraint with a predicate.

**Claire:** Yes, it takes a Scheme predicate.

**David:** Yes it's the same. It allows you to convert an arbitrary Scheme predicate into a constraint. We have a similar facility [in core.logic]. Though by including it, you're allowing people to do *very unsound things*.

**Claire:** Yup.

**Will:** Okay. I think you can definitely implement `not-pairo` using `never-trueo`. You'd have to think hard about how it interacts with other things. It seems like `=/=` and `not-pairo` work fine together, but for `absento` or the finite domain constraints you'd have to think through those interactions. Actually, it would probably be fine with finite domain constraints; I'm not sure how it would interact with `absento`. We haven't thought about that. Maybe it would just work. It seems like a fairly safe constraint compared to other constraints we've written. `not-pairo` is a pretty weak constraint, actually. Basically, you're saying it has to be an atomic value.

We've talked about adding an `atomico` or `atomo` constraint that says its one of the well-known atomic values. In fact, if your only constraints are symbols and numbers, you can define `atomo` as:

```scheme
(defrel (atomo x)
  (conde
    [(symbolo x)]
    [(numbero x)]))
```

**Jason:** With booleans and the empty list as well, right?

**Will:** Well...

**Nada:** In my case it was actually more complicated. I was really looking at doing algebraic simplification. In my `conda` I actually had a special term I was looking for (like `(+ x 0)`, for example) and I wanted the other clause to be *not* that. In some cases, it's really hard to translate from relational to non-relational. If you go the constraint route, you have to add a lot of ad-hoc constraints.

**Dan:** Can I get back to the discussion of `pairo` and `not-pairo`? Will, do you think both of those should exist, just one, or none?

**Will:** Every time you ask me about `pairo`, I point out that you can just define it using `==`.

```scheme
(defrel (pairo x)
  (fresh (a d)
    (== x `(,a . ,d))))
```

`pairo` is trivial to define. But like I said, every time I find myself using `not-pairo`, I find out later that I can write it in a simpler way and not need it.

**Dan:** My point was [you're working with?] a diminished space of values.

**Will:** Here's the argument for `not-pairo`:

As you pointed out for `atomo`, if you have booleans and the empty list as well, you'd define `atomo` as:

```scheme
(defrel (atomo x)
  (conde
    [(symbolo x)]
    [(numbero x)]
    [(== '() x)]
    [(== #t x)]
    [(== #f x)]))
```

But depending on what terms are in your [relational interpreter's] language, you may or may not have all those things. If you're doing an interpreter for the lambda calculus, you don't have booleans or numbers. So your definition of what it means to "not be a pair" changes depending on the input language you want to interpret.

The nice thing about having a `not-pairo` constraint is that it wouldn't be "fragile" with respect to what you consider atomic values. That's an argument for it. The other argument is: although it's true that every time I use it I find a way out of it, I don't necessarily find a way out initially. Sometimes I use it in the beginning and then figure out later how to get rid of it. That's another argument to have it: just to be expressive with the idea to get rid of it at some point. It's like how sometimes I'll use a `conda` or `condu` just to get something working and then figure out how to get rid of that later.

**Dan:** Legitimate answer, but I'm not sure I agree with it.

**Will:** What part do you disagree with? I hope you don't disagree about `pairo` [being redundant].

**Jason:** I did want to ask about `pairo`, if you'll indulge me before we move on. We've talked often about how implementing things with `conde` and `==` can be too strict [eager], and how constraints have a more lazy factor. Does that apply here as well? Maybe I want a `pairo` constraint that does not enlarge the substitution. Would that ever be a concern? Maybe in a recursive program?

**Will:** I can't imagine it. Every constraint does change the constraint *store*. It may not affect the substitution, but if the constraint isn't already subsumed, then you have to add it to the overall constraint store.

**Jason:** That's true.

**Will:** Furthermore, I just see it as a type of unification constraint. You're just giving some "shape" to a term. I don't know why you would want that to be particularly lazy.

**Jason:** I don't have a use-case for it, either.

**Dan:** One possibility would be a cleaner implementation of `absento`.

**Will:** I can't imagine making that lazy and not making all of unification lazy. And I really don't think you want to make all of unification lazy, because then you're never going to be able to fail fast, I don't think. Maybe you could make unification lazy and interact with other constraints. But I think, with unification, you either have to make everything lazy, or not. I don't see an advantage to making it lazy.

**David:** I don't totally agree. Something that I don't like about [the user-level] formulation of `pairo` is that you end up instantiating stuff, and that's not *really* what you want to do. A lot of programs are non-terminating because you instantiate things. You really do want to say it's a constraint that gets lazily applied. There are definitely cases where I've seen user programs that end up writing something like this, whereas the termination behavior would be better if they expressed whether something was a list (or not) as a constraint, not as a goal.

**Will:** Can you give an example of that? I can't think of one. The thing that makes `pairo` different from most constraints is that there aren't multiple choices. Normally you'd want to make a constraint lazy (like a finite domain constraint) because you don't want to enumerate a domain. But here there's only one answer. You don't need a `conde` to express it. You don't need to enumerate things. I don't know of a case where you wouldn't want to use [a user-level] `pairo`. Can you think of an example where it's too aggressive?

**David:** Especially given our discussion around `presento` and `absento`, I think there's some utility in lazy constraints that say you have an atomic thing versus a tree-like thing. Again, I haven't completely thought that through. But you guys are working with programs that manipulate very deep tree [terms] where being able to fail faster via a lazy constraint might be better than instantiating terms accidentally through goals.

**Will:** The `absento` and `presento` are cases where if you try to simplify those constraints, you'd end up with *many* simplified constraints. You could end up with things like disjunction.

In fact, one of the hard things about trying to implement `presento` is that you end up with a disjunction in general. It's sorta like having a `conde` inside the constraint. Most of these constraints (like the finite domain constraints) have a disjunction-like feel. But for `symbolo`, `numbero`, or `pairo`, you have one possible answer. I don't see the advantage of making that lazy.

**David:** It might be true. Based on what Claire said, I think there's something not dissimilar from finite domains. When you're able to assert that something is or is not a tree term, that's sort of like a domain over trees where you can say these terms are either going to be atomic or trees, like a flag. Like when you say a term is 0 or 1, this term is either atomic or not atomic. What Claire said made me think some of the issues with `presento` may be solvable in that way. I don't know; just an idea floating around in my head.

**Will:** My argument for not having a separate constraint for `pairo` is that you can already express `pairo` using `==`. Then you would have two different ways (one as a lazy constraint, one using unification) to express exactly the same information. And `not-pairo` would have to see if the variable was already associated with a pair in the store, anyway. It would already have to do that work, onless you got rid of unification.

I think *this* is interesting: if you want to go to constraint systems that don't necessarily have unification as an operator (instead doing finite domain and other operations), then `pairo` makes sense. But to me, that feels very different than anything we've implement so far.

**David:** More stuff to think about.

**Will:** Definitely. Nada, did you want to say anything about the paper? There were a couple interesting things. We learned a bunch of things between the confo talk and finishing the paper. Maybe we could share some of the things we learned that maybe weren't clear or hadn't been fleshed out.

**Nada:** One subtle thing was that when you did the confo talk, you were basically overconstraining. The trick of reconnecting the input and output was overconstraining the result, because the system has to work for any kind of continuation, but we were assuming it was the empty continuation. In the final version, we just do the trick for the empty continuation, which means that it doesn't always run fast backwards in the general case.

**Will:** Right. If you have the 4-element CESK machine, then we're not taking advantage of this trick. We lose to the fact that this is in CPS form and that the input and output are not connected well. The search times just go up exponentially, whereas you don't with the simple evaluator. Really, it's only the continuation we have to know [ground?]. And it doesn't even have to be the empty continuation, but we do have to know what the continuation is in order to play this trick.

Have you thought any more about how this `v-out` trick works for small-step [interpreters], or if it works at all?

**Nada:** No, I haven't thought about it. The trick as we present it in the paper really seems to be related to CPS. We made this connection clear in the paper by using `remember*o` [`rember*o`?] as an example: where you have some situations that work fine but become divergent when you do the CPS transform. We can get back the old non-divergent behavior by using this same trick.

But in the small-step case, it's really different. It's not [slow?] because of a CPS form, but really because we have these intermediary steps. And there, what we do is basically unfold from the small-step to the big-step. It has a different feel to it. I don't really see a connection. The CPS trick, in some ways, is a bit deeper; it's not just about unfolding. You really just have to keep some connection that the CPS removed. Whereas small-step to big-step is just an unfolding.

**Will:** An idea I had was: the problem we ran into with the continuations is that if you don't know the exact starting continuation then this trick doesn't work, because you have to take the derivative of the expected output with respect to both the expression *and* the continuation. You only have to look at the current expression--you don't have to look at the whole tree at once. But you have to know the whole continuation you're starting with. Maybe that's a property of how the interpreters is structured. Maybe we could flip it around and make the interpreter work on continuation application first and expression evaluation second. Maybe that would allow us to look at one part of the contuation at a time if we know the whole expression, if that makes sense. Then, I started thinking you still have the same problem, just flipped around.

Maybe there's a way to combine (or take the cross-product) to deal with the continuation and expression--dispatching on both at the same time. Obviously, your code would grow. I wonder if by doing something like that you would be able to be a little more general and use this trick for the full CESK machine. I'm not sure.

**Nada:** The only thing I worry about, when you do this generalization step, is whether you do too much computation up front, doing more work than you need. That's the feel I had when I went from the small-step to the big-step: the unfolding was doing all the work up front and then at every small step it was just repeating all the big-step work. I guess that's something we have to experiment with. In addition to the repetiton [of code], there's a risk of doing things many more times than you need.

**Will:** Yeah, I think that's why you have a problem using this approach when the continuation is a fresh variable. You could try finding/searching/instantiating the full continuation, but I think in that process you end up doing the work of the evaluator anyway.

**Nada:** And you might be doing it for all possible continuations.

**Will:** Exactly. At that point I think you've already done too much work.

It works well if you know the continuation. You don't have to know the environment or the store, right? In that sense, we're being a little more constraining than we have to with the regular evaluator. We could leave the environment and the store free.

**Nada:** Just a small comment on having a `pairo` constraint: one thing we did find useful in core.logic when doing this [TAPL](https://www.cis.upenn.edu/~bcpierce/tapl/) stuff was having--not a `pairo` constraint--a `listo` constraint which checks that something is a well-formed list such that if you print it out, it would not have any dots. That one is actually useful.

**Will:** Like a `proper-listo`.

**Nada:** Yes.

**Will:** That feels like the shape constraint stuff. You can't express `proper-listo` via unification without introducing recursion. That's something that feels like it should be a lazy constraint, whereas `pairo` should not. I'd like to see it.

Or you could imagine a shape constraint saying "this is an association list mapping symbols to arbitrary terms". that might be helpful. It would be cool to be able to express what the shape looks like somehow: "a proper list where each element is a pair with a `symbolo` constraint in its `car` and no constraint on its `cdr`." At that point you start getting into the tradeoffs between a static type system and a shape constraint system. They still feel different to me.

**Dan:** It seems that for a `proper-listo` constraint, if you manage to walk down to the end and have a dotted logic variable `x` at the end, you can throw out all your work and just do `(proper-listo x)`, and that's it. You shouldn't have to do it for the whole data structure. All you need to know is what's at the end: if `()` is at the end then it's a proper list, and if not, it better be a `proper-listo` of that last term.

**Nada:** You're rigth. You also want to check if the `car` [`cdr`?] is a list, then it should also be a `proper-listo`.

**Will:** Beyond that, for Oleg numbers you might want to say you have a proper list where each element is from the finite domain {0, 1}. But even that's not enough to express all the properties of an Oleg number, because the last element can't be a 0. But it would give you the ability to express more things. The question on my mind is if you can't express all of the features that you need, then is it a good idea to express most of them? Or do you just not worry about it? Oleg's opinion of the Oleg numbers was that the user has to be responsible to not instantiate the number to illegal values. Because we only had unification (at the time, at least) there was no way to enforce it other than enumerating ground values , which defeats the point.

The question is: can we express things in a way that's lazy enough to be useful, but also expressive enough to ensure that the terms are properly typed? And if we get to that point, are we better off just with a static type system (even though that's more restrictive than we might want)?

**David:** Nada, it's funny because at one point I submitted `proper-listo` as a lazy constraint to your repo. But I think you removed it.

**Nada:** I removed it because now it's part of core.logic. You have the [`seqc`](https://clojuredocs.org/clojure.core.logic/seqc) constraint.

**David:** Oh right! Do you use that?

**Nada:** Yeah.

**Will:** I'm going to end the broadcast now. Thanks everyone.
