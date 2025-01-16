# miniKanren Hangout #5, 14 April 2013
Youtube link: [https://youtu.be/VSpq3qK9L88](https://youtu.be/VSpq3qK9L88)

**Will:** Hi everyone. Welcome to the fifth miniKanren hangout. I'm Will. Today we're going to try some miniKanren live coding (or at least live translation). We've got some of the usual suspects here. Maybe we should start out talking with Austin. We were talking about some things that I think would be of interest to general miniKanren programmers, and could also help feed into today's exercise of translating Scheme code into miniKanren doing live coding. The point of this isn't just to write more miniKanren code; the real point is to talk about how to think about the translation process, how to think about goals, which problems you run into, and *what* to think about. Hopefully it will lead to good questions and thinking about writing miniKanren in practice.

Austin, did you want to recap the things you were thinking about when working on miniKanren recently and the areas where you had some issues?

**Austin:** You summarized it well. How do you think on the level of a miniKanren programmer? How do you understand the semantics of goals? How do you reason about goals without reasoning about their operational semantics? I'm curious what the idioms are. Do you think of *this* goals as a value generator, and another goal as an assertion? That kind of thing.

**Will:** This reminds me of something Dan was talking about the other day that we may not have made explicit, but which is always in our minds (mine, Dan's and Oleg's). When we write these programs, we always want the relations to work in what a Prolog programmer would call "all modes". That is, you can allow all of the variables passed into a relation to be fresh logic variables and the relation should still work just fine. That's the gold standard. Everything being fresh should work. You should be able to do a `run 10` with all fresh variables fed into your relation, and that should mean something. You might have the situation where there are finitely many answers (say, only 6) and when you do a `run 10` your program could go into an infinite loop, because it can't find the 7th (nonexistent) answer. But if you do a `run 6` it should work just fine. This idea that you should be able to *always* feed in *all* fresh variables to *all* of your goals is really the standard we want.

There may be some times where for a particular goal you want to restrict it a little bit, like if you have a little bit of \[mode\] information you want to take advantage of. In those cases you might want to use a `conda`/`condu` which is equivalent to Prolog's cut `!/0`, or you might want to use some additional miniKanren operators like `project`. But that's getting away from the relational paradigm. I see that as an optimization (or compromise you might have to make) if you can't write it directly in the relational style. But most of the time, you can get away with writing everything to support all fresh variables. I always think of the goals as being runnable on fresh variables.

In fact, if I write a complex relation, often I don't know how to test it \[on ground terms\]. If you are writing something like an interpreter and you have a complicated term that you want to feed in (or you have a special representation of environments) and you get one of those terms wrong syntactically, then your program will just fail or diverge. So what I normally do is just a `run 10` will all fresh variables and look at the answers that come out, making sure they have the right shape. Also, that gives me some test data I can feed in: if I know running with all arguments fresh gives me a ground term, then I can stick that in and leave everything else fresh and see answers I can generate. So it's also helpful for testing miniKanren programs.

Okay, so today is "stump the chump day". I'm the chump. The "stump the chump" game works like this: you find a piece of purely functional Scheme code (obviously not 300 pages worth, but something relatively short and self-contained). If no one here can present it, I can try to find something online. I'm trying to translate code I haven't seen before from pure Scheme (no side effects, no `set!`, no `call/cc`) into miniKanren and have it run backwards. I'll go through that whole process and try to explain what I'm thinking about the whole time. If people have any questions, I'd be happy to answer. It's a kind of "thinking aloud" exercise. Maybe we'll run into problems, maybe we won't. We might see opportunities to use constraints or nominal logic programming, etc. This is just to try to give people a chance to see what an experienced miniKanren programmer thinks about when going through this process.

Before we get into that, does anyone have any general questions or things to share? Claire, I know you've been really busy with some miniKanren related things. Did you want to say a few words about what you're working on and their status?

**Claire:** Sure. Last week I implemented a macro in Scheme that will let you define a lazy goal, which basically waits until the end to evaluate the goal. This actually lets you define different searches through the search space, so this kinda solves this problem: if you put the recursive call first \[and unifications later\] and it'll diverge because it doesn't have all the information when it goes depth-first into that recursive call. Now \[with lazy goals\] you can have it wait until all the unifications have happened before it tries to evaluate that goal. That's implemented in cKanren now. You can decide what kind of search strategy you want, which goals can be lazy, and switch between depth-first search, bread-first search (through all the lazy goals), and some weird hybrid search that mixes bread-first and depth-first. I've been getting that to work this week.

**Will:** That's really cool stuff. You were saying before that you might be able to do this per goal: having different search strategies in the same program, right?

**Claire:** Yes. I don't really know what that would mean now, but as soon as I know what that would do, I can implement that.

**Will:** Yes. Ciao Prolog lets you have multiple search strategies in the same program. (I think at the module level). It seems like a very powerful technique which could be cool for doing things like running relational interpreters backwards, for example. Maybe you want to search for terms that have a certain complexity or look. Or maybe you want to have a probabilistic search strategy/heuristic guided search. It would be really neat to change those things around.

You're also working on the CLP(set) stuff and Constraint Handling Rules. Do you want to mention anything about that?

**Claire:** I haven't done much work with the CLP(set) stuff since last weekend. It's still not working entirely, but unions definitely work. And I'm working on some macros that will let you define constraints at a higher level, kind of like Constraint Handling Rules. You can define under what circumstances your constraint will fail, under what circumstances you can ignore the constraint, what other constraints might trigger, and other interactions between constraints. So if you had `(=/= a b)` and `(=/= b a)` in your constraint store, you can drop one of those. You'll be able to specify things like that at a very high level. Hopefully that will make it easier to define constraints.

There was one more cool thing about CHR but now I forget.

**Will:** Did you talk about how making recursion lazy partially solves the problem of "commutative and"? Unification would be the only non-lazy goal, right?

**Claire:** Yes. If you do it right and make all your \[potentially nonterminating\] recursive calls lazy, you can really write your goals in whatever order you want, which is awesome. It doesn't work so well when you have, like, 5 or more recursive calls. In the case of quines, I think there's a case of 3 recursive calls in that evaluator. *That* doesn't work so well because if you give all 3 of those a fair chance to evaluate, you well get an explosion in the number of `conde`s, and I'm pretty sure that's what slows it down a lot. Using depth-first search is the fastest way of doing that, but it's not as fast as the original code yet. There are a lot of dispatches that have to happen (which wouldn't if you were evaluating things \[strictly\] as soon as you encounter them).

**Will:** Cool stuff. Claire is working on four major things.

**Claire:** I remember now! cKanren has actual documentation.

**Will:** Documentation!? That's not the miniKanren way.

**Claire:** I know right? In my repository there are now explanations about how to install and use cKanren.

**Will:** You are going to ruin our reputation.

I thought there were four things. You ported all the stuff to Racket. You've been working on CLP(set)...

**Claire:** Attributed variables!

**Will:** Attributed variables, constraint handling rules...

**Claire:** General \[generic\] unification!

**Will:** General unification, and lazy recursion (which helps to solve commutative conjunction). That's like 6 things.

**Claire:** Oh, and now the constraint store is a hash table so it goes faster.

**Will:** Alright, seven things. Cool. I look forward to seeing the polished version of all those things, the sets and all that. That's really going to make miniKanren much more powerful and useful. Oh yeah, documentation! That counts too. Those things will make miniKanren much better. Nathan, is there anything on your mind?

**Nathan:** I've been spending time trying to parse the difference between CHR and the new constraint handling stuff and where term rewriting fits in. I'm interested to see where this goes. I ended up getting myself really confused because a "constraint" in CHR is more like a fact in Kanren, as far as I can tell. But there is some overlap in different ways. They're more high level and at the same time low level than Kanren in some ways. I got myself confused on what it would mean and what term rewriting would look like in a system that can go forwards and backwards, if that makes sense. When a rule fires in CHR it's super forward-chained and will never go back: would it make sense to have term rewriting in Kanren in the first place? That's kind of what I was doing some reading about and trying to wrap my head around, but I was getting confused by the whole thing.

**Claire:** The idea in my mind is that these CHR-like rules will actually expand to what cKanren does now, and cKanren runs backwards just fine. I'm not anticipating that being a problem.

**Nathan:** Would these rules do something similar to term rewriting? Like simplifying if you have a "less than" constraint that subsumes another "less than" constraint: would it remove that constraint?

**Claire:** Yes.

**Nathan:** That removal itself wouldn't "undo" \[from running backwards\], right? That's sorta outside the search, right?

**Claire:** All of these are still functional data structures, so that won't affect backtracking in any way. The fact that you have been able to refine your constraint store in one branch is not going to affect any other.

**Nathan:** Okay. So it wouldn't "undo".

**Claire:** The set constraints were originally very CHR-like. I had to translate them into what would work in cKanren. I'm excited to go back to the original mathematical formulation of those things, because it's very term-rewrite-y.

**Nathan:** I'm interested in where that goes.

**Claire:** I'm working on it right now in the background, so hopefully not very long.

**Will:** The term rewriting also makes me think that there might be some overlap with things like [PLT Redex](https://redex.racket-lang.org/). It's sort of like that space also. The whole general area of term rewriting and executable semantics is a really cool area and really interesting.

\[Discussion of the IU 3d printing club's antenna is omitted\].

[19:35](https://youtu.be/VSpq3qK9L88&t=1175) I guess it's time to play stump the chump. Does anyone want to challenge me to translate some code into miniKanren, or must I be reduced to finding my own challenge? \[From the chat:\] "Write miniKanren in miniKanren." That's something I've wanted to do for a while. I don't know if I could live code that. It is true that there's a purely functional Scheme implementation of miniKanren, so I suppose that's fair game. Let me think about this. \[Will reads the chat.\] No? Okay, Claire retracts the challenge.

Zippers and tree rewriting? Maybe. What works well for this game is a fairly small amount of self-contained, purely functional code--that's something we could translate. It doesn't have to be Scheme. It could be mathematical rules or something like that. Things in Matt Might's blog posts have been pretty good.

\[Reading chat:\] "Anything that uses nominal logic." Okay. Things like interpreters, type inferencers, and other things which involve lexical scope are good candidates for nominal logic.

I could try doing miniKanren-in-miniKanren. I don't know. I was thinking about making some kind of metainterpreter for miniKanren, but I guess we could just try implementing the Scheme version, but that's a little longer. I don't know how long it would take; I don't know if we could do it all. I'm trying to do something where I can do the whole translation in one sitting and show you. But I'm tempted to try that, but it would be pushing it time-wise.

That's something I might try, anyway: taking the implementation at the back of The Reasoned Schemer and trying to miniKanrenize that. That's a fallback; we'll try that if we can't come up with something more appropriate. It also has "fail" written all over it. I'm not even sure what it means, but it would be kind of cool. We'd have substitutions floating around and stuff.

How about this: let's look for a couple minutes for anything else. If not, we'll actually try miniKanren-in-miniKanren and see how well it works. I can at least probably start translating some of the helpers and see how that goes. We could see if there are any cool Matt Might blog posts. I've translated a few of Matt's stuff. Or if there's a paper that has a rewriting system, those are usually straightforward to translate. Let's see what Matt has.

**Austin:** Another thing that I'm interested in is miniKanren's position on extralogical predicates like `findall/3`.

**Will:** Things like `setof/3`, `bagof/3`, and `findall/3` in Prolog are all interesting and useful. You can do universal quantification (to some extent) using those. I was talking to Ken Shan about that a fair amount this weekend. My stance on it is that those are impure.

I'm trying to remember what `findall/3` does (versus `setof/3` and `bagof/3`). Do you remember the semantics for that operator?

**Austin:** `findall/3` takes a template and a goal, and it instantiates the template as many times it can using the goal and puts all the results in a list, which is the third argument.

**Will:** Is it sort of like a mapping operator?

**Austin:** The way I understand it is you're really searching the extension of the search space at that point where you write the `findall/3`. Like if I was doing an ancestry query and I wanted to find all the sons of somebody.

**Will:** It sounds very similar to `setof/3` and `bagof/3`. I think they are related, but I don't remember exactly. I'm not much of a Prologger. I've read a lot of Prolog stuff, but I haven't spent a lot of time using it. I can tell you about `setof/3` and `bagof/3` which I think are similar. The idea of `setof/3` and `bagof/3` is this:

If you do something like `(membero x ls)`, it'll give you one member at a time. But maybe you want all of the `x`s at the same time. Right now you'd have to do a `run*` and look at the answer. `setof/3` and `bagof/3` would let you run that goal "locally" in a sense and accumulate the answers in a term which you can then use and inspect. I think that behavior is similar to `findall/3`. They're very useful and very cool. The problem in my mind is that they're not relational.

To me, anything "relational" means you can swap the goal ordering and the semantics don't change. At least with `setof/3` and `bagof/3`, the answers you get back/accumulate depend on the groundness of the terms. The order in which you make those calls can change the answer you get. I don't know of a way to make those relational. It's sorta like the `copy_term/2` problem I described in an earlier hangout: depending on the position of `copy_term/2` in the program, the answer you get back can be different. It turns out for `copy_term/2` there are some times where you can use tricks like special forms of tagging plus nominal unification to make a safe version of `copy_term/2`. In our alphaLeanTAP paper, we're only going in one direction but we can still use it safely and relationally. There might be restricted uses of `findall/3`, `copy_term/2`, `setof/3`, and `bagof/3`, but I think the general versions of those operators are non-relational: swapping them around can change the semantics of your program.

**Austin:** The context that is coming up for me is finite failure. I need to show that all possible extensions would fail. I've been trying to understand how you might do that without `findall/3`. I'm not sure if you can do it at the user-level of miniKanren or if a different goal constructor can tune this part of the search. It also relates to what you said about `membero`: oftentimes I want to be able to say that `x` is a member of a list \[without instantiating its position in\] the entire list. I want the next operation to be able to work with the list, but I want to lazily search each branch to find all the elements of that list.

**Will:** Encoding negation is generally just tricky. It sounds like you want to do a universal quantification to show the set is empty, that kind of thing. There are probably ways to encode this in miniKanren without using these operators, but it might be kinda tricky. It's on a case-by-case basis. It's gets very tricky with negation in general. We don't like using negation as failure because that means using `conda`, `condu`, or that kind of thing. Negation is the nasty beast of logic programming. That's why [answer set programming](https://en.wikipedia.org/wiki/Answer_set_programming) is taking off: it's probably the best story (yet) about how to do negation in logic programming, but that's for things that are finite.

I don't have a simple answer other than to say in general negation and Prolog-style programming is just hard: keeping your programs declarative (and relational) and getting decent notions of negation are tricky. That's one of the reasons we've been going to Constraint Logic Programming: the constraints give us sorta limited forms of the nagation. You can use finite domain constraints, and by continuing to shrink over the domain you can prove that no answer exists (by doing a `run*` and getting back the empty set)`. But once again, you're looking outside the program to see if there are no answers.

If you want to say something like "if there are no answers that exist, then do something" then you're getting into negation-as-failure type issues. `findall/2` would let you say "if the set of thnigs that satisfy this goal is empty, then do something". But you're starting to dance between what's relational and what's not, unless that's not really a concern of yours (like if running backwards isn't really a concern).

I think `setof/3` and `bagof/3` are actually quite easy to implement in miniKanren. I did it a few years ago.

**Austin:** So what is the context, then, in which you're considering using those extralogical predicates, if not negation?

**Will:** Oh! The context where I would like them is to be able to encode universal quantification, but I don't know how to do it in a purely relational way. In practice, I just don't use those. Every once in a while I think of the problem some more, but I don't have any great insight or solution in how to jury-rig those operators to make them more relational. I don't know what's possible, I just try to write programs that don't use those.

**Austin:** So you're not interested in miniKanren having this meta layer of extralogical goals or anything like that as part of the system?

**Will:** I'm pretty sure you can encode those things right now. We *do* have some extralogical operators: `conda`, `condu`, and `project`. Just those three things alone are actually really powerful: you can encode some pretty crazy stsuff with them. I don't have any problem with it existing, it may be worth implementing, and I think core.logic *should* have all that stuff. I think it's easy to encode all those things just on top of miniKanren right now: you can always get your hands on the substitution if you write a low-level goal.

Personally, my main interest is not having to write those. My main interest is how to write interesting programs that do the same work without having to resort to extralogical operatiors. The extralogical operators, to some extent, destroy the declarativeness. That may require ingenuity or new constraints. Instead of having a dirty operator which works in general but inhibits your ability to run backwards, the approach I'm more interested in is special constraints for particular cases, or using a very limited form of negation to encode the properties you want. It's harder and it probably requires more ingenuity, but my main concern is being able to run backwards.

That's philosophically how I see one of the big differences between Prolog and miniKanren. The Prolog programmers may start out with a logical specification, but as soon as they start running into trouble with expressing things, negation, or performance, they'll just start using extralogical operators or `assert/1` and `retract/1` to add or remove things from a global database. Real Prolog programs tend to have a lot of that stuff in it. At that point, I think the reasoning about the programs is harder. At some point once you've added enough extralogical features, I personally start questioning how close that is to logic programming and questioning why is it better to use impure features of logic programming versus, say, functional programming. These are things that we could argue. Prolog has a long history and it's been very successful in AI programming; they get results. But then again, people can code things in C and get results, too. I'm looking for higher levels of abstraction. Well, you could argue that `findall/3` and `bagof/3` have high levels of abstraction, but they break what I consider the most important property of logic programming, which I'm not willing to give up.

Anyways, I'll get off my soapbox. If you want those, we can implement them. It's not too hard. It's probably a simple macro.

Nathan, did you want to do finite state machines? They're a little different than regex. It's a different way of looking at it. Just brainstorming.

So the candidates we have are finite state machines, implementing miniKanren in miniKanren, and just trying to grab something off of the blog.

Claire, do you have any Racket code sitting around? Any cool programs worth translating? Do you have a CPSer, or anything like that.

**Claire:** No. Let's see. I have an implementation of a functional queue that I was using this week.

**Will:** Would that be interesting to run backwards?

**Claire:** I don't know. Maybe.

**Will:** Nathan, do you have some code for the FSMs that would be ready to go?

**Nathan:** Not in front of me. I was just thinking of standard string recognition or something like that.

**Will:** Well, I do have some finite state machine stuff laying around that we could show. But it would kinda be cheating since I've already done a lot in that space. I feel like I've already seen the code to some extent.

Obviously, I've seen the miniKanren code before, but I've never tried translating the miniKanren implementation into miniKanren! Maybe we should just try that and see what happens. I smell disaster, but it would be interesting, at least. Maybe it would be okay. Let's try it; let's at least take a look at the code and see if it makes sense to translate it into miniKanren. Sometimes I'll look at code and think woah, I don't even want to think about translating that. Other times I'll look at code and think oh yeah taht'll be easy (and sometimes it'll turn out to be harder). Let's see which version we want to translate. Probably not the one from the book.

Oh! We could maybe do Oleg's sokuza-kanren. Oleg has this super minimal version of miniKanren. That might be short enough to implement. Let's look at the website and find something we could translate that's not too bad.

Here's [regular miniKanren](https://github.com/miniKanren/miniKanren), let's look at that real quick. These macros are going to be interesting. I'd have to translate `reify`. Forget this version, it's really complicated.

**Claire:** Isn't there a [simple-miniKanren](https://github.com/miniKanren/simple-miniKanren). That's the one you sent me to.

**Will:** Yeah, let's see if this one is any better.

```scheme
(define-syntax rhs
  (syntax-rules ()
    ((_ x) (cdr x))))

(define-syntax lhs
  (syntax-rules ()
    ((_ x) (car x))))

(define-syntax size-s
  (syntax-rules ()
    ((_ x) (length x))))
```

**Claire:** When did those all become macros?

**Will:** I don't remember haha. Well, we could start off with `unify`. It might be kinda fun to have a unifier that runs backwards. `unify` is kind of like the heart of the system anyways.

Even simple-miniKanren has a lot of code; some of the stuff we wouldn't be able to translate. We could try doing a unifier in miniKanren, that would be really fun. But I want to look at [sokuza-kanren](https://github.com/miniKanren/sokuza-kanren), Oleg's really simple, stripped down version. That one might be better. Let's see what we've got. Hopefully this one doesn't have as many macros.

```scheme
(define (conj f1 f2)
  (lambda (x)
    (apply append (map f2 (f1 x)))))
```

Ew, `apply`. Hm.

\[Will reaches the end of the code.\] Is that it? That's the whole thing. I don't know if there's enough here to be interesting. Oh no, he's using higher order stuff--I guess that's just trying to show how the stream works.

Let's start with unification in simple-miniKanren and go from there. Let's find `unify` in there.

```scheme
(define unify
  (lambda (u v s)
    (let ((u (walk u s))
          (v (walk v s)))
      (cond
        ((eq? u v) s)
        ((var? u) (ext-s-check u v s))
        ((var? v) (ext-s-check v u s))
        ((and (pair? u) (pair? v))
         (let ((s (unify
                    (car u) (car v) s)))
           (and s (unify
                    (cdr u) (cdr v) s))))
        ((equal? u v) s)
        (else #f)))))
```

Okay, it [depends on] `ext-s`, `ext-s-check`, `occurs-check`, and `walk`. Actually, `unify` has quite a bit in it. If we could implement `unify`, that would be pretty good.

So I'm proposing that we implemented the unification in miniKanren, does that sound good? Okay, let's see how it goes. Even `unify` has a few helpers. It would be fun to have a unifier that runs backwards; we've never had that before.

Austin, you wanted to see nominal unification, right? That might come up here, I'm not sure. We should keep and eye out for that.

What I'm going to do is copy in some of the Scheme definitions from simple-miniKanren. `unify` uses `ext-s-check`:

```scheme
(define ext-s-check
  (lambda (x v s)
    ((occurs-check x v s) #f)
    (else (ext-s x v s))))
```

`ext-s-check` uses `ext-s` and the occurs check:

```scheme
(define ext-s
  (lambda (x v s)
    (cons `(,x . ,v) s)))

(define occurs-check
  (lambda (x v s)
    (let ((v (walk v s)))
      (cond
        ((var? v) (eq? v x))
        ((pair? v)
         (or
           (occurs-check x (car v) s)
           (occurs-check x (cdr v) s)))
        (else #f)))))
```

We're going to need `walk`.

```scheme
(define walk
  (lambda (u S)
    (cond
      ((and (var? u) (assq u S)) =>
       (lambda (pr) (walk (rhs pr))))
      (else u))))
```

And `var?`

```scheme
(define-syntax var?
  (syntax-rules ()
    ((_ x) (vector? x))))
```

`rhs` and `lhs` aren`t very interesting, but we might as well grab those.

```scheme
(define-syntax rhs
  (syntax-rules ()
    ((_ x) (cdr x))))

(define-syntax lhs
  (syntax-rules ()
    ((_ x) (car x))))
```

We better put the macros at the top of the file. I think that's it. We should be able to load this and do a couple unification examples.

```scheme
> (load "unifier.scm")
> (unify 5 5 '())
()
> (unify 5 6 '())
#f
```

That works! Let's see if we can make a var as well.

```scheme
> (unify 5 (var 'x) '())
((#(x) . 5))
```

Yeah. Okay, cool. That seems to work. Do people understand what the interface to unification is? If not, I'll just show you real quick.

`unify` takes two arbitrary terms, `t1` and `t2`, and a substitution which is represented as an association list. So `'()` is the empty substitution.
- If two terms are identical, then you don't need to extend the substitution. That's why we get back the new substitution as empty when we do `(unify 5 5 '())`.
- If we try unifying two terms which cannot be made equivalent in any extension oft he current substitution, then it just fails with `#f`.
- If we try unifying, say, 5 and the variable `(var 'x)` in the empty substitution, that succeeds. And the new substitution `((#(x) . 5))` is saying 5 must be substituted in for `x`.

So thsoe things seem to work. Let's do some more interesting unifications. If we unify two variables, for example:

```scheme
> (unify (var 'x) (var 'y) '())
((#(x) . #(y)))
```

Now `x` and `y` are associated. We can also try something with a nonempty substitution. We could say something like:

```scheme
> (unify (var 'x) 5 `((,(var 'x) . 6)))
```

Actually we have to be careful here. This is a good trick question. Any thoughts about what this unification should do? Will it succeed or fail?

**Austin:** Fail.

**Will:** You think it'll fail, because `x` can't be 5 and 6 at the same time, right?

**Austin:** Right.

**Will:** *I* think it will succeed. Anyone else think it will succeed? This could be a trick question. I think that will actually succeed. Claire, what do you think?

**Claire:** I think it should succeed.

**Will:** And why is that?

**Claire:** Because they're different variables [according to `eq?`].

**Will:** Yes!

**Claire:** I haven't looked at the code, but I think that still holds.

**Will:** I think so, too. Let's see.

```scheme
> (unify (var 'x) 5 `((,(var 'x) . 6)))
((#(x) . 5) (#(x) . 6))
```

Yeah. Because these `x`s are actually different. If we want it to fail we need to let-bind the logic variable.

```scheme
> (let ((x (var 'x)))
    (unify x 5 `((,x . 6))))
#f
```

Now those `x`s refer to the same `(var 'x)`, and sure enough, that fails. The thing is, every call to `var` creates a new variable. Even if they have the same name, that doesn't matter. The vars themselves in our representation are vectors. It's pointer equality. You're getting structures that are `equal?` but not `eq?`. In fact, I can show that.

```scheme
> (equal? (var 'x) (var 'x))
#t
> (eq? (var 'x) (var 'x))
#f
```

They're equal, but not pointer-equal. Does that make sense Austin?

**Austin:** Yes.

**Will:** Okay, cool. So that actually might be kinda hard to express in the miniKanren version of miniKanren. We might have to thread through a counter (or something like that) so that we can get both the `equal?` and `eq?` behavior. That something we'll have to think about.

Anyway, that's the interface for 'unify`. The first question I can think of is: what would it mean to run `unify` backwards? If you were to make `unifyo` (a relation instead of a function), what would that look like? Any thoughts about that? How would this change if it's a `unifyo`?

**Austin:** The third argument would be the result.

**Will:** We would actually have a *fourth* argument. We would add one more argument. We'd have "term 1", "term 2", the original substitution, and the *new* substitution--the potentially extended substitution, or `#f`. Sorta like the "answer". This is fun. You'd be able to feed in an extended substitution and ask what the original substitution in which these two terms unified and it'll start generating substitutions. Or, you'll be able to set the result to `#f` (meaning the terms didn't unify) so give me two terms that *don't* unify in this current substitution. You'll be able to play those sorts of games, which is interesting. I think `unifyo` does make sense.

You'd be able to do something like:

```scheme
> (run* (q) (unifyo 5 6 '() q))
(#f)
> (run* (q) (unifyo 5 5 '() q))
(())
```

It would get more interesting if you do things saying the output is some extended substitution.

```scheme
> (run* (q) (unifyo 5 t2 q `((,(var 'x) . 6))))
```

You could play those sorts of games. That sounds kinda fun; we'll have to see how it works.

In the process of writing this, we'll have to write some simpler things like `walk`, which does variable lookup in a triangular substitution. We'll have some "mini" relations as well. Hopefully that will work well. Let's give it a shot.

I'll add `#!eof` in front of all the Scheme code. That's Chez Scheme's indication that the end of the file has been reached (logically, for Scheme). Anything belowe the `!#eof` will be ignored.

`unify` uses `walk`, `ext-s-check`, and all these other things. I think the first thing to do is probably `walk`, which is an important operator in the system. We'll define `walko`.

It takes some term `u` and a substitution `S`. And now it's going to give us back a walked term `u^`. We have a `cond`, so that can turn into a `conde`. We have this `else`. `else`s are problems.

Also, we're playing some games with Scheme's `cond` using this arrow `=>` notation. Syntactically, what this means is that this whole expression `(and (var? u) (assq u S)) ` is in the test position for this `cond` clause. If this test returns a value other than `#f` (that is, any "true" value in Scheme), then that true value gets passed to the function that's created on the right hand side of this arrow. If the `and` expression returned, say, 42, then this function `(lambda (pr) (walk (rhs pr) S))` would be passed 42, bound to the `pr` argument. This `=>` is a shorthand, Schemely way of doing things. We don't have to do it that way. I'm going to rewrite this code in a slightly less complicated style that will be easier to translate.

```scheme
(define walk
  (lambda (u S)
    (let ((pr (and (var? u) (assq u S))))
      (cond
        (pr (walk (rhs pr) S))
        (else u)))))
```

Okay. If `pr` isn't false, then we're going to walk it. Claire, does that transformation look right to you?

**Claire:** Yes.

**Will:** I should really test all this stuff and be careful. But in the interest of time I'll keep things going.

So let's translate this simpler program instead. Notice I got rid of the `lambda` and the `=>` stuff. We don't want `lambda`s floating around in our programs. It's okay to have a `lambda` right after `define`, but if you just have higher-order functions floating around, that's not going to work well in miniKanren. Unification doesn't know about procedures--you can't unify procedures.

Now we have this `let` version, but we still have `var?` and this `assq` thing. `assq` is fine, it shouldn't be too hard. We're going to have to come up with a reasonable representation of variables, though. Tagged lists are probably the easiest thing to do. Should we just go ahead and do that? Yeah. We'll need to be able to create variables and emulate `var?`.

A variable is just a vector in simple-miniKanren. We need to be able to create variables that are distinct from each other. So I think what we'll do here is define a `var` constructor that takes an `x` and returns a tagged list. We can keep the name in.

```scheme
(define var
  (lambda (x)
    `(var ,x)))
```

This will allow us to create different vars. Austin, this is an example of where you might want nominal logic. If you were doing this with nominal logic programming, we could represent the vars as noms (or atoms) in alphaProlog, which are implicitly tagged. And we could use the "fresh nom" operator to make sure that we're getting back unique variables (instead of having to worry about feeding in different values of `x` to make unify differently). I'll start out doing this `var` constructor approach, but this is something that arguably could be cleaned up itno something nice and nominal and could be made simpler. But nominal logic introduces some issues with reification and interpreting answers which could be a little triciker to understand.

[Reading chat] Yes, if only I had cKanren installed! Then I could mix and match. Claire is going to solve all your problems by next week.

So that's `var`, which just creates a tagged data structure. When we think about unification, if you create two vars (using numbers as indices) `(var 5)` and `(var 5)`, then those variables would unify. But `(var 5)` and `(var 'x)` would not.

So we can construct variables that are the same, variables that are different, variables that will unify, and variables that won't. That's really all we need for vars. As far as making sure we can construct new vars, we could do something like pass through a counter saying what the current variable is. We could even do something like make the counter a Peano numeral, like the symbol `'z` or a list of `s`s, successors of `'z`. We could make those unique and increment as needed. We could use Oleg's arithmetic. There are games we could paly, but right now we won't worry about it.

So we have the `var` constructor. Let's do `var?o`. A term is a variable if it unifies to something that looks like a var.

```scheme
(define var?o
  (lambda (t)
    (fresh (x)
      (== `(var ,x) t))))
```

This is actually a goal, whereas `var` is a Scheme function [a term constructor] which is not very interesting. We could make it into a goal if we wanted to, but that seems kinda overkill to me. But `var?o` we want to actually make a goal which tries unifying to a tagged list.

If we added a second, "output" argument to `var?o`, we would have to have some form of negation [to ensure nonoverlapping `conde` clauses]. We would have to ensure this is not a pair, or not a list of length 2, or if it is a list, its first element is not equal to the `'var` tag. But we don't really need to do this. The idea of returning a `#t` or `#f` isn't very miniKanren-y. Oleg certainly would object to this. In Oleg's mind, if you had somethign that's a predicate in Scheme that returns true or false, the correct way to encode that is as a goal that either succeeds or fails. If it succeeds, then you continue the computation. If it fails, then you just cut off that branch of the computation. So we'll encode it according to the way Oleg likes to do things, which I think has benefits. It definitely has benifits in terms of simplicity, so we'll just do it that way.

So we have `var?o` and the ability to create vars, but we need to be able to look things up, which is what this `assq` does. So `assqo` shouldn't be hard to write. It's like as `assoc` type thing. It's `assq` instead of `assoc` (which is the `equal?` version) is for two reasons. One, it could be more efficient. But the real reason is because we're using vectors and pointer equality in Scheme. We actually *need* to use `assq` instead of `assoc` for two different vectors that were constructed at different times but happen to have the same symbol inside of them. They would look the same, so we would mistake two variables that we want to make different. Since we're not using pointer equality, we can just define `assoco`.

`assoco` is like something out of the Reasoned Schemer. It's pretty easy warm-up relation. 

[1:10:20](https://youtu.be/VSpq3qK9L88?t=4220)

We basically want to look up a variable `x` in a substitution `S` and get back some output term bound to `x` called `t`. If the substitution is empty `'()` (remembering that the substitution is just an association list), then we will fail.

```scheme
(define assoco
  (lambda (x S t)
    (conde
      [(== S '()) fail]
      [])))
```

There's no reason to include this clause because it's failing; we don't need to have a `fail` goal. What we *really* want to to is unify `S` with an association list that *might* contain `x`.

What we'll do is pattern match outside the `conde` for shorter code. This is standard miniKanren, matching on an association list.

```scheme
(define assoco
  (lambda (x S t)
    (fresh (y t^ rest)
      (== `((,y . ,t^) . ,rest) S)
      (conde
        []
        []))))
```

 We're saying that `y` is bound to some term `t^` at the beginning of the substitution, and we have the `rest` of the substitution.

```scheme
(define assoco
  (lambda (x S t)
    (fresh (y t^ rest)
      (== `((,y . ,t^) . ,rest) S)
      (conde
        [(== y x) (== t^ t)]
        []))))
```

If `y` and `x` happen to be the same, then we know that `t^` and `t` are the same; we found the term we're looking up.

```scheme
(define assoco
  (lambda (x S t)
    (fresh (y t^ rest)
      (== `((,y . ,t^) . ,rest) S)
      (conde
        [(== y x) (== t^ t)]
        [(=/= y x) (assoco x rest t)]))))
```

If however `y` and `x` are different, then we need to recur down the `cdr` of association list, `rest`, using the same `x` and still finding the same `t`.

That should be it. Any questions about what I've done so far?

**Nathan:** I'm wondering why we need the regular Scheme `var` constructor. Why can't we get away with a single predicate that both checks for var-ness but also provides a var? Could the predicate return the label as its "truthiness" and also be a predicate that creates vars?

**Will:** That's a good question. That's a stylistic thing. Let's do it that way. Instead of making `var?o`, let's just call this `varo`.

```scheme
(define varo
  (lambda (t)
    (fresh (x)
      (== `(var ,x) t))))
```

Now what this does is ensure that `t` is a var. It will both instantiate `t` to a var if it's fresh, and if `t` is already a var it will succeed. If `t` is not a var (or cannot be unified with a var) it will fail. This is purely a stylistic thing. You could definitely make it work either way. It depends on whetehr you want more Scheme-ly constructors.

It's sorta like when you transform a program into continuation passing style (CPS): any function that could go into an infinite loop, you have to CPS it (create the continuations and pass things around). However, if you have a simple function in Scheme, like a data constructor, that you know terminates and just returns a data structure and does no more work, then you don't have to CPS it. But you could still CPS it if you wanted to.

This is very reminiscent of that situation. Sometimes when you have very simple data structure, like doing some `cons`es, you can just do the `cons`es in Scheme, or you could treat it like a goal. That's more of a stylistic thing. From a performance standpoint you should just do the `cons` in Scheme and not do any unifications. From a simplicity standpoint, you could argue that it's better just to have one goal that does everything. That's the tradeoff as far as I see it. It certainly works either way so it doesn't really matter. That's a good question.

**Claire:** You're going to need to pass in a name to `varo`, or else all vars could unify with each other.

**Will:** Oh, good point! Maybe there is an isuue. Let's go back and look. Maybe we could salvage it.

We would have to do something more like this:

```scheme
(define varo
  (lambda (t x)
      (== `(var ,x) t)))
```

We could leave the `x` fresh if `t` is already a var. Now `varo` is enforcing (or checking) that we have a *specific* var, not just a var in general. This actually is a little different. Claire brought up a good point. Does that difference make sense Nathan?

**Nathan:** Yes. It's a slightly different predicate.

**Will:** I don't know that we always want to be passing in an `x`. It may be better to go back to the previous approach.

**Nathan:** So in this one here, the label would be a kind of "witness" to the var-ness?

**Will:** Yes, exactly. It would let you check if something is a particular var, also, not just a var in general. I don't know if we need that ability. We could try it. Whenever we do these sorts of translations, I try out different approaches. I feel that this interface is a little simpler. I could be wrong.

Now I think we're ready to translate `walk` into `walko`. Since `walk` has a `let`, the simplest thing to do is introduce `pr` as a fresh variable and then unify it with something.

```scheme
(define walko
  (lambda (u S u^)
    (fresh (pr)
      (== pr ))

    (let ((pr (and (var? u) (assq u S))))
      (cond
        (pr (walk (rhs pr) S))
        (else u)))))
```

Well, hm. Maybe that's not really what we want to do. We want to see if it's a var first, *and then* look it up. Here, let's think about what the intent of the code is: we're trying to do a variable lookup in the substitution. The reason we have the `and` is that we only want to look somethnig up if it's a variable. If it's not a variable, then we just want to return it. I think we want a conjunction here.

We want to see if `u` is a variable or not and then tread it differently. What I really want to do, probably, is something like this:

```scheme
(define walko
  (lambda (u S u^)
    (conde
      [(var?o u) ]
      [else (== u^ u)])))
```

If `u` is a variable, we want to do something. If `u` is *not* a variable (here's the `else`) now it gets interesting. We want `u^` to be `u`--we basically want to leave it unchanged if there's nothing to look up. Of course, this `else` could be an issue. We need to figure out how to deal with this because we can't have `else` in `conde`. We'll try to get rid of that `else` in a bit. We sort of want to have a `not-var?o`, the negation of `var?o`. We'll hold off on that for a second.

If something *is* a var, then we want to look it up. We want to use our friend `assoco`. In particular, we want to look up `u` in `S` and get some term `t`, let's say.

```scheme
(define walko
  (lambda (u S u^)
    (conde
      [(var?o u)
       (fresh (t)
         (assoco u S t))]
      [else (== u^ u)])))
```

In that case, we want to walk the right hand side of `t`. `assoco` is supposed to give us a pair... Oh right! I messed this up. This is actually not `assoco`; it's more like a variable lookup. The difference is that when you're looking up in an association list with `assq`, the thing you return is not the right hand side; you actually return the entire pair. Well, we could just return the right-hand side in the lookup function itself. I'm going to call it `lookupo`.

```scheme
(define lookupo
  (lambda (x S t)
    (fresh (y t^ rest)
      (== `((,y . ,t^) . ,rest) S)
      (conde
        [(== y x) (== `(,y . ,t^) pr)]
        [(=/= y x) (lookupo x rest t)]))))
```

There's no reason to return the whole pair, `assq` is just something Scheme happened to have built in. We used it to make the code a little shorter in the miniKanren implementation, but it wasn't really necessary.

Now Claire is saying that this version would have worked even as `assqo`. Why was that, Claire?

**Claire:** The reason `assq` in Scheme returns the entire pair is because if you have a `#f` on the right hand side of an association, there's no way to distinguish "I found it and it's associated to `#f`" from "I didn't find it and I'll return `#f`". But there's no "failure value" in miniKanren that could be stored on the right hand side in an association list; miniKanren will just fail if it didn't find it rather than returning `#f`. It won't be a problem if you just return the right hand side.

**Will:** That's right. We don't have a base case for `lookupo` because we never get to the empty list. We have a difference in how we're dealing with the failure to look something up if it's not found in the environment. That's what we have to think about.

In fact, we need to think about that carefully. Let's think about this. If we're looking up a some term `u` in the substitution `S` to get `t`, it will only succeed if `u` is actually in `S`. If not, this whole `conde` clause `[(var?o u) (lookupo u S t)]` is just gonna fail, so we might get this `[else (== u^ u)]` clause instead. Once again, we run into issues of encoding negation: what happens when `lookupo` fails. We're going to have to think about it a little bit. I think it'll be the tricikest part of translating `walk`.

We know `u` is a variable, we look up `u` in `S`, we get back some term `t`, and then we have to walk the right hand side of the pair (which is just `t` in the miniKanren version) in `S`, and that's our answer. Basically, we have to call `walko` again, recursively. Now we're walking `t` in `S` and the value we get back is `u^`. 

```scheme
(define walko
  (lambda (u S u^)
    (conde
      [(var?o u)
       (fresh (t)
         (lookupo u S t)
         (walko t S u^))]
      [else (== u^ u)])))
```

That's fine. If this whole thing fails, (if it ins't true) then the output should be unchanged, the same as the input term: `(== u u^)`. We have to think about what that negation means.

First of all, if `u` isn't a variable then we just want to leave it unchanged. Maybe `u` is 5; we're looking 5 up. We know 5 isn't going to be bound in the substitution because 5 isn't even a variable, so we just return it. If `u` is not a variable then we want to leave it unchanged. And also, if `u` is a variable and is not in the substitution, then we want to leave it unchanged. So we have to be able to encode these two pieces of negative information.

Does anyone have any thoughts about how we should encode that? This is the interesting, trickier part if we want to keep everything relational. We could obviously encode this using `conda` or `condu`, which would be equivalent to using a cut in Prolog. Any thoughts about how to encode this?

Let's do one thing at a time. How could we encode `u` not being a variable? Could we write a goal that ensures `u` isn't a variable? What would that look like?

**Nathan:** Could we explicitly tag our literals? So that we'd have a tag for logic variables and a tag for literals. Then we could dispatch on the tag, or look up the tag of a term.

**Will:** We definitely could do something like that (`5` is a literal, `'foo` is a literal, that sort of thing) and just tag everything. We could also tag by type, like an `'int` tag.

We have to decide what we have in our language, too. We could have numbers using the `numbero` constraint in miniKanren. `numbero` is a constraint we use in the miniKanren for quine generation--we can tell if something is a Scheme number. We could do something like restrict our language of terms (that we're unifying over) just to numbers and variables. Although we probably want to have lists, because unification isn't interesting unless you have pairs of terms.

So we might want to have Scheme numbers, pairs, and these tagged variables. I think that's sufficient to do interesting things. Let's say those are our three types of things. Everything is either a variable, a pair (which is not a tagged variable), or a Scheme number. Those are our three types of terms. Oh yeah, we'll probably want to have the empty list also. In fact, let's add a comment here.

```scheme
;; terms in our language
;; 1. Scheme numbers: 5, 137, etc.
;; 2. tagged variables: `(var ,x) for some x
;; 3. (non-tagged variable) pairs: (5 . 6), (() . (5 . ()))
;; 4. empty list: ()
```

So we'll have 4 values in our language. `not-var?o` will have to distinguish between variables and non-variables.

We need a `conde` here because we have multiple cases. If `t` is not a variable, then we have 3 cases.

```scheme
(define not-var?o
  (lambda (t)
    (conde
      [(== '() t)]
      [(numbero t)]
      [(fresh (a d)
         (== `(,a . ,d) t)
         (=/= 'var a))])))
```

If `t` is the empty list, then it's not a var. Or if `t` is a number (we have a special `numbero` constraint in miniKanren). And the other case is if `t` is a pair (and furthermore, the `car` of that pair had better not be the `'var` tag).

We have to be kinda careful of this `'var` tag thing, especially running backwards. Really, we probably want to treat var as some special gensym, some special reserved thing you can't type in. Nominal logic can help with that.

Actually this is a place we could use the `absento` constraint `(absento 'var a)`, that something doesn't occur in something else--No, actually! we want to be careful here, because pairs can actually contain tagged vars.

So that's `not-var?o`. Notice that the negative version is a lot more complicated than `var?o`. That is not atypical. If we tag all the other terms, we could make it a little simpler. But tagging the empty list, for example, seems overkill to me.

So that's one part of the negation. We can use it in our definition of `walko`.

[1:33:13](https://youtu.be/VSpq3qK9L88?t=5593)

If `u` is not a var, then leave it as-is.

```scheme
(define walko
  (lambda (u S u^)
    (conde
      [(var?o u)
       (fresh (t)
         (lookupo u S t)
         (walko t S u^))]
      [(not-var?o u) (== u^ u)]
      [(var?o u)
       ??? lookupo failed
       (== u^ u)])))
```

Now we want to express that we have a var, but lookup failed; that it wasn't in the substitution at all. Any ideas how we might encode that?

**Nathan:** Use a sentinel value.

**Will:** What would the sentinel value look like Nathan? Is there more of an idea of how to use that?

**Nathan:** Something like `'not-here`. I guess that would be a new term we'd have to worry about handling. I guess we'd only have to handle it in this one goal. You'd be returning boxed values, `\`(here ,value)` and `(not-here)`.

**Will:** Oh, I see! This would be instead of having `lookupo` fail?

**Nathan:** Yes.

**Will:** Gotcha. Yeah, we could change `lookupo`. Maybe that's the easiest thing to do. There are a couple ways to do this (I can think of at least three). But let's do it this way. We can add a special case for when `S` is the empty list. When `S` is empty, we can return `#f`. If we're going to do this, we'll have to go back to that game where we're returning the full pair. That's actually more like the original Scheme code. We can do it that way. Let's return the entire pair and take the right hand side of it.

```
(define lookupo
  (lambda (x S t)
    (conde
      [(== '() S) (== #f t)]
      [(fresh (y t^ rest)
         (== `((,y . ,t^) . ,rest) S)
         (conde
           [(== y x) (== `(,y . ,t^) t)]
           [(=/= y x) (lookupo x rest t)]))])))
```

Claire says that's gross [in the chat}. Claire, do you have a better suggestion? I actually have an idea.

**Claire:** Can't you just do a `not-ino`?

**Will:** Ah, like an `absento` constraint?

**Claire:** Not exactly, because it would only not be in the `car`. It should still be able to appear in the right hand side of the association.

**Will:** This is like the trick I used in the relational CESK machine where I split up the environment into two lists: a list of variables and a list of terms they are bound to. It makes the representation of the environments a little more complicated, but maybe that's worth showing because it's kinda cool. So then we can use the `absento` constraint.

We can do it the Scheme way (more like the original code where we return a `#f` value if the environment is empty. Or we can do it in this other manner which is just kinda cool. 

I've never seen this split environment approach before. Probably someone invented it in Prolog 30 years ago. The idea is this: Normally our substitution representation is just an association list, right?

```scheme
; S: ((x . 5) (y . 6))
```

Instead, we're going to come up with another representation where the same information can be represented like this:

```scheme
; S ((x y) (5 6))
```

You match up the `x` and `5` by position. You end up `cdr`-ing down both lists at the same time. Why is that helpful? It's helpful because then you can use a constraint on the variable list. There's an `absento` constraint (similar to the # (`hash`) constraint in nominal logic) saying that the variable doesn't appear within the term at all. (In nominal logic, `hash` means the variable doesn't occur free within the term).

For example, if we wanted to make sure that `z` doesn't appear in the substitution `((x y) (5 6))`, we could just have this absento constraint saying that `z` doesn`t appear in the first list. If `z` does appear, it would just fail. It lets us use a lazy constraint to represent this information.

One of the advantages of this is that we can represent the fact that `z` doesn`t appear in an unbound substitution (a fresh logic variable) without having to instantiate the substitution an unbounded number of times. It lets us be more efficient in how we actually represent the information. I'll try that just because that's a neat representation trick. (But we could definitely do it the other way.)

Our `lookupo` will have to change because the `S` looks a little different. It's slightly more complicated, but not terribly.

```scheme
(define lookupo
  (lambda (x S t)
    (fresh (y y-rest t^ t-rest)
      (== `((,y . ,y-rest) (,t^ . ,t-rest)) S)
      (conde
        [(== y x) (== t^ t)]
        [(=/= y x) (lookupo x `(,y-rest ,t-rest) t)]))))
```

We've got a list of variables and a list of terms. We still need to check if `x` and `y` are the same, and if so `t` is `t^`. Otherwise we're still looking up in the rest of the substitution, which we have to rebuild. Our representation is a little clumsier to work with. You can write helpers for all this if you want to abstract the representation. But it'll make it easier for us to express negation.

So in `walko`, where we now want to say that looking up `u` in `S` fails, all we have to use is this `absento` constraint that says `u` does not appear in the first list in `S`. So we have to break apart `S` a little bit into the variables `y*` and the term values `t*`. We want to say that `u` is a variable, but `u` is *absent* in the list of variables in that current substitution (if that's the case lookup will fail) so we can leave it unchanged.

```scheme
(define walko
  (lambda (u S u^)
    (conde
      [(var?o u)
       (fresh (t)
         (lookupo u S t)
         (walko t S u^))]
      [(not-var?o u) (== u^ u)]
      [(fresh (y* t*)
         (var?o u)
         (== `(,y* ,t*) S)
         (absento u y*)
         (== u^ u))])))
```

That's a representation trick. It turns out when doing logic programming, representation tricks are really important. You can often get much better behavior.

We should be able to run it now. Hopefully this will work. We should get 5 back.

```scheme
> (run* (q)
    (fresh (v1 S)
      (== (var 'x) v1)
      (== `((,v1) (5)) S)
      (walko v1 S q)))
()
```

Empty! Here's a traditional debugging question. Maybe you've run into this before Austin: you write this great relational program, and you run in, and then it just gives you back the empty list. How should we debug this? Any thoughts?

**Claire:** Try a simpler test case.

**Will:** Yes. The simplest possible test case is to let `walko` run with all fresh variables.

```scheme
> (run 5 (q)
    (fresh (t S t^)
      (walko t S t^)
      (== `(,t ,S ,t^) q)))
()
```

That gives us back the empty list! That's a serious problem. We need to go back and look at `walko` and see what's going on there.

Let's see here. You'd think at least the `not-var?o` case would succeed, because it enumerates values. I'm actually surprised by that. Let's try something simpler. Why does the `not-var?o` call not succeed?

```scheme
> (run* (q) (not-var?o q))
()
```

Okay, that's failing too. Did I download a bum version of miniKanren? let's make sure miniKanren's working right.

```scheme
> (run* (q) (== 5 5))
((var q))
```

Oh! I know what's going on. Do you know what the problem is, Claire?

**Claire:** No.

**Will:** This is so funny. Let me show you what's going on. This is hilarious.

\[Will navigates to `mk.scm`\]
```scheme
(define var (lambda (dummy) (vector dummy)))
```

I redefined miniKanren's builtin `var`! This is the reason for namespaces and modules. I just trashed miniKanren's ability to construct new variables. Now miniKanren is constructing *my* vars instead. We have to change this. Let's call ours `make-var`.

```scheme
(define make-var (lambda (x) `(var ,x)))

(define var?o
  (lambda (t)
    (fresh (x)
      (== (make-var x) t))))
```

Alright. That could help explain things. because nothing was working.

```scheme
> (run* (q) (== 5 5)
(_.0)
> (run* (q) (== 5 q))
(5)
```

That's better. Let's go back and try the tests.

```scheme
> (run 5 (q)
    (fresh (t S t^)
      (walko t S t^)
      (== `(,t ,S ,t^) q)))
((() _.0 ())
 ((_.0 _.1 _.0) (num _.0))
 (((_.0 . _.1) _.2 (_.0 . _.1)) (=/= ((_.0 var))))
 ((var _.0) (((var _.0) . _.1) (() . _2)) ())
 (((var _.0) (((var _.0) ._1) (_.2 . _.3)) _.2) (num _.2)))
```

Hey, that looks better! Success. Let's see if that makes sense.

- `(() _.0 ())` Here we're walking the empty list, which is a legal term. The substitution is *any* substitution. So it just walks to itself, because the empty list is not a variable.
- `((_.0 _.1 _.0) (num _.0))` Here we say if some `_.0` is a number, then in *any* substitution `_.1`, it just walks to itself.
- `(((_.0 . _.1) _.2 (_.0 . _.1)) (=/= ((_.0 var))))` Here we have a pair. We know it's a pair and not a variable because it has this disequality on the `car`. If you have a pair in any substitution, then it walks to itself.
- Next we have the var cases. In this first case `((var _.0) (((var _.0) . _.1) (() . _2)) ())` in the substitution we have the variable we're looking up as the first thing in the substitution bound to the empty list term `()`, so we get back the empty list.
- In the second case `(((var _.0) (((var _.0) ._1) (_.2 . _.3)) _.2) (num _.2))` we have the variable bound to a number in the substitution and we get back that number.

We could try it with `run 10` and get back more complicated answers. So that appears to work.

Let's try this next example. We'll walk `(var 'x)` in this substitution where `(var 'x)` should be bound to 5 and we expect to get back 5.

```scheme
> (run* (q)
    (fresh (v1 S)
      (== (var 'x) v1)
      (== `((,v1) (5)) S)
      (walko v1 S q)))
(()
 (_.0 (num _.0))
 ((_.0 . _.1) (=/= ((_.0 var))))
 5)
```

That answer is more complicated than I expected. What does this mean? Why are we getting four answers? How could we possibly get back these other cases?

**Claire:** Will, could you see what `not-var?o` does on a var.

**Will:** Yeah, maybe there's something there. We should only get back `(5)`. The other answers are bogus.

```scheme
> (run* (q) (not-var?o (var 'x)))
(_.0 _.0 _.0)
```

Oh! 

**Claire:** That seems incorrect.

**Will:** It does seem incorrect.

**Claire:** You're calling `var`, not `make-var`!

**Will:** Right, thaht's miniKanren's `var`! my program is wrong.

```scheme
> (run* (q)
    (fresh (v1 S)
      (== (make-var 'x) v1)
      (== `((,v1) (5)) S)
      (walko v1 S q)))
(5)
```

Yes. Alright, it seems to work. The lesson for today is that namespaces are good, modules are good, maybe even R6RS libraries are tolerable. So that actually worked just fine. We just got tripped up with a miniKanren-internal definition for `var`. Really, that shouldn't be exported if we were using the R6RS or the Racket version. Your Racket version wouldn't export `var`, right?

**Claire:** Nope!

**Will:** That's a software engineering mistake.

**Claire:** And if you tried to redefine `var` while importing `var`, it would complain. It wouldn`t just overwrite it.

**Will:** That would have been more handy. Well, I haven't gotten through all of unification, but walk is kinda the key thing in unify. The rest of unification might not be too hard to do.

My challenge to anyone watching is to implement the rest of unification based on this implementation of `walko`. I'll put this online somewhere, but you should be able to recreate it pretty easily. I think you can do a unify that runs backwards now that we have a walk that runs backwards. Once we have that, I guess we can do the rest of miniKanren, maybe. I'll play around with it some. It seems like a fun exercise. Any closing thoughts Claire?

**Claire:** Not especially.

**Will:** Anyway, that's it for now. Now I'm going to go hack on HAM radio and 3D printing. Talk to you all later.
