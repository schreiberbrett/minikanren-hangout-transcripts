# Transcript of miniKanren hangout #1, 10 March 2013

**Will Byrd:** My name's Will Byrd and I'm one of the designers of miniKanren, which is a relational (or logical) programming language. And this is the first Google Hangout for miniKanren, which I'd like to turn into a weekly event. So this is a somewhat "meeting people who are interested" and "organizational" hangout, and also I would like to do some miniKanren-y things shortly.

So we've done a quick round of introductions. It sounds like we have a mixture of experience levels in terms of everything from someone like Ambrose [Bonnaire Sergeant] who has spoken on core.logic at Clojure Conj to folks that are completely new to logic programming, and that's fine. So part of this is we're trying to figure out what the structure of this hangout should be. We have a 2-hour block and the question is how to we want to break that up. One idea I was thinking was we have more introductory material in the beginning -- the second hour could be more hardcore things. I'm sure people will be joining and leaving. But in the beginning, maybe, intros if there's anyone new, Q&A, and maybe some miniKanren tutorial information, and talking about some techniques that would be generally useful to people. And then maybe afterwards going through a paper (or a book like Types and Programming Languages) and try and translate Scheme to miniKanren, or whatever.

So that's the basic structure and the things I was hoping to talk about today. So it sounds like most people are happy to go with the flow, and I'm happy to do that, too, and see what happens organically. We don't have to have a super-plan right now. I think it'll evolve over time.

**Austin:** Will, you know what I'd love to hear? Just your background with the whole logic programming thing: how you got involved with it, the genesis for your paper and how long you worked on that, and where it took you.

**Will:** That's a good question. In fact, one of the things I was thinking about talking about at some point is sort of the "history of miniKanren". Dan [Friedman] would really have to be here to do it full justice. I think sometimes he'll be able to hang out. And maybe Oleg [Kiselyov] too.

Let's see. I started at IU, Indiana University, as a PhD student in 2003, I think. May have been 2004. I graduated in 2009. It took me six years. I think I started in 2003. When I started, I had picked the school partly based on a visit to ICFP, the International Conference of Functional Programming, which was held in Pittsburgh in 2002. I met Dan and I met a bunch of other people in the logic programming community. I was figuring out which graduate school I wanted to go to. Dan just seemed to have lots of energy and was really excited what he was working on. So that was one of the main reasons I picked IU. I show up the first day of Dan's class for 521 [CSCI-B 521?], which was the Advanced Programming Languages class. I walk up to him after class like "Dan, I'm so glad to be here and learn about Scheme programming!" And Dan says, "I don't do Scheme programming anymore!" Or rather, he says, "I don't do functional programming anymore. I do logic programming now." I was thinking dammit, something you could've told me in Pittsburgh, because then I wouldn't have come here. Had I known, I would not have gone to IU. But I'm glad I did in hindsight. But it wasn't what I wanted to hear at the time, at least.

So it turned out that Dan had been interested in logic programming languages for a long time. He had messed around with Prolog and he didn't like the fact that Prolog didn't have lexical scope. In Prolog, all the variables are global. They're in this giant database. You get a lot of power from manipulating that database and asserting and retracting global facts (and all sorts of things like that). But if you believe in lexical scope, if you think that's an important concept, then immediately you are very unhappy with Prolog. So Dan wasn't happy with Prolog. He refused to use it. And over the years he had worked on a couple logic programming languages. There had been other work in Scheme [for] logic programming, but it had never really taken off. So when I started, he had been working on this new language for a while. And shortly before I arrived at IU, I guess he posted notes on the course website, and somehow Oleg found them. Oleg, apparently, wrote [Dan] this really long email pointing out all the mistakes, or things he could do better.

**Austin:** Where was Oleg at the time?

**Will:** Oleg, I think, was working in California. Anyway, it was a friendly letter. It wasn't a bad letter.  It got Dan's attention, because Oleg has done a lot of Prolog. And Oleg is also Oleg, if you've met him. This is before Oleg got famous. Now everyone knows who Oleg is in the academic functional programming community. But at the time, he wasn't very well known.

Based on Oleg's notes, Dan decided it would be good to collaborate with this guy. Dan's language with a number of Oleg's suggested changes and improvements ended up being what we would now call "Kanren" which was the predecessor to miniKanren. And then I showed up as a new grad student shortly after they had started working together. And I don't think Dan had met Oleg when I showed up.

So I start this class, and if you know Dan, he's a very good teacher [but] he moves quickly, especially if he has students he thinks are good. And at the time I took his graduate class, there were some really good students in the class. He just went at the speed of light. A couple weeks in, he handed out this document (which I still have somewhere). It's like a hundred pages. He and Oleg had worked on this thing, I guess. And it was basically an implementation of a version of Kanren using Scheme macros. I didn't know macros; I didn't know logic programming; I had no idea what this was about. Dan hands this out to everyone and was like "Okay, the assignment is to do X with this language in this system." I think most of the students in the class were like "What in the world?" [Dan] assigned the Zebra problem, which is a famous logic puzzle: The Spaniard lives in a house next door to the Italian, and the Italian owns a greyhound, and the Frenchman drinks wine, etc. There are all these constraints, and the question is: who owns the zebra as their pet? It's a famous logic puzzle and he wanted us to implement it in miniKanren. Eventually I figured out how to do it but I wasn't very excited. I was underwhelmed, because it was kinda just this naive search. I didn't think it really showed off anything interesting. I still don't think it's a good example of logic programming, personally. I think something like a type inferencer is much more exciting, or something like an interpreter. Although those are harder to explain to people who are not programmers.

So I implemented [the zebra problem] and Dan tried to explain the 100 pages of macros to me, and I didn't understand it very well. Dan had another student named David [Mack?]. This guy was super hardcore. Really brilliant, really nice guy. So Dan, David Mack, and I started meeting to look at this Kanren stuff. Oleg had added all these optimizations to the original Kanren, including something called head-let, which was this optimizing macro that did Prolog-style pattern matching. It did a number of optimizations, and if you didn't know Prolog very well, it was almost impossible to understand. So David [Mack] figured that out. And eventually we came up with the idea to write a book over the summer. Or maybe Dan already had that in mind. But the idea was David Mack, Dan, and I would write a book. And I think Oleg was gonna help, I forget if he was originally gonna be involved. I guess that was gonna be the four of us. That was gonna be The Reasoned Schemer but we didn't have a title for it yet. I was still having trouble understanding full Kanren. It was Prolog-y. In some ways it was closer to core.logic with the "rel" stuff and the "fact" stuff. It felt more Prolog-y than core.logic, for sure. If you didn't know Prolog, [Kanren] didn't make much sense.

I kept pushing Dan for a simplified version of it. But I wanted an interpreter, not a simplified DSL. That was largely from my frustration of not understanding what he was trying to do. So eventually Dan relented and implemented a simplified version of miniKanren as a bunch of Scheme macros. He didn't do an interpreter, which in hindsight was probably good that he did it as macros not an interpreter but eventually that became miniKanren. The original miniKanren was really messy. It had a bunch of [special] forms that weren't right. It had this "all^i" and it had something like four types of conjunction, and none of us could keep them straight. We tried teaching it in CSCI-C 311 which is the undergrad PL class. It was a complete disaster. I still have the notes somewhere. There were four types of conjunction and 2 (or 3) types of disjunction. It was really a mess. But over time we started cleaning it up. When we wrote the book over the summer, that was the big thing. And even then, the version of miniKanren in The Reasoned Schemer is still more complicated than the current version. It has this condi, which is this interleaving version of conde, and things like that. We definitely have made improvements [since]. But by the end of that summer or shortly thereafter we had the basic miniKanren implementation. Since then, we've been extending it in different ways and cleaning up the implementation, getting into constraint logic programming.

And of course, the other big thing that happened is when David Nolen came along and sent me an email that said "Hey! I read your dissertation." And that was really cool. To me, that's when miniKanren really took off. I don't think miniKanren would have ever had much influence beyond the people who read [The Reasoned Schemer] if it hadn't been for David. David, you can say something about how you got started. What's the background of core.logic? I know core.logic wasn't the only miniKanren implementation [in Clojure], right? After David explains it, I'd like to hear your story too, Ambrose.

**Austin:** Has there been anything else published on miniKanren since your dissertation?

**Will:** There are a number of papers. Dan [Friedman], Eric Holk, and I have a paper that just came out on writing relational interpreters [miniKanren, Live and Untagged](https://www.schemeworkshop.org/2012/papers/byrd-holk-friedman-paper-sfp12.pdf). It's basically about generating quines. I think that's the most exciting thing we've done, actually. And I know Fogus is going to include something on core.logic in the second edition of his book [The Joy of Clojure Second Edition](https://www.manning.com/books/the-joy-of-clojure-second-edition).

**David Nolen:** So the same thing happened to me. I saw [Jim Dewey?]. He did a version of miniKanren in Clojure in fall 2010. I saw the Zebra problem and I had never seen that before. I wanted to know how it worked. I asked "I want to know how this works." And he says, "I can explain it to you." And I was like, "But how did you figure it out?" And he said, "I read this book called The Reasoned Schemer. You shouldn't read that." He told me not to read it!

**Will:** I have to admit, it's a hard book to read.

**David:** So I picked up a copy of it and I spent probably two months trying to figure it out. I think I finally figured out what it meant by working them out on paper. But I still didn't really understand how the system works. I was actually blown away that Will had written a dissertation on it. You get to the end of the book and Chung-chieh Shan spends 1 page trying to explain the monadic part. It's incomprehensible. You see the macros at the end of the book, also incomprehensible. And I was thinking there has to be more information. So I found Will's dissertation and I read it a couple times. I always thought the unification stuff is pretty easy, the substitution stuff is pretty easy. The hardest part for me was understanding the goal stuff. Which is funny because the goal stuff is tiny; it's not really that much code. It's just a bit abstract and I wasn't familiar with monadic-style code.

**Will:** By the goal stuff, do you mean the monadic threading of the substitution? Or do you mean the dealing with the streams via conjunction and disjunction?

**David:** I understood the passing of the substitution (as a value) around through the closures. But it was harder to understand exactly how conjunction and disjunction worked. So I wrote a version. I literally ported the Scheme. I did Clojure stuff, but I had only been doing it for a year and half. So I was curious: "Is Clojure good enough? This looks like a pretty fancy Scheme program. I wonder if I can port this." I spent a couple weeks porting it, and it worked. And then I started looking at how to integrate it better with Clojure and how to optimize it. Basically, any IU paper that was ever published on miniKanren, I downloaded. I found a couple things that you guys had done on miniKanren performance. I saw that you guys had a version of miniKanren that ran faster because you swapped out an implementation of the substitution [Bender, Kuper, Byrd, Friedman](https://users.soe.ucsc.edu/~lkuper/papers/walk.pdf). So a lot of the early core.logic work was just trying out things that you guys had written about. Anything else people want to know?

**Will:** So at some point you went beyond just copying miniKanren, or porting the Scheme code, right? You ended up with this extensible unifier, and you added back some of the things that were in the original Kanren, or similar things, like facts. Where did that come from?

**David:** So I put it out there as a project, and I think one person tried it. For three months or four months after it was out, maybe one person was interested in it. I think what happened was I started getting questions on StackOverflow and on the mailing lists where somebody was like "Oh neat, this is like Prolog." And people started buying Prolog books and then realizing they couldn't go through the Prolog books. I thought I should add some of the Prolog stuff back. That's why we added the fact and defrel stuff. The [extensible] unification stuff which is really cool was sorta an accident. When I had started working on miniKanren, I saw that [Clojure] has more recursive data structures than Scheme does. miniKanren only works on lists in Scheme. I wanted core.logic to work on hashmaps and sets and all this other stuff. I realized: the deeper the case statement got in the unifier, (Scheme miniKanren's unify only has five or six cases). I needed to add four or five more and I noticed that performance was degraded because of that. I thought what if we make that operation polymorphic? It was really just an optimization. It was four or five months later I realized oh this is actually really cool. This kinda solves a problem around the fact that in Prolog, unification is [isn't?] extensible.

**Will:** Yeah, that is really nice. And also, I guess, it turned out to pay dividends when it came to nominal unification and things like that, right?

**David:** Yeah. I'm glad you posed that challenge: will alphaKanren work? Can you actually cleanly extend core.logic to do nominal logic? It was exciting to see it actually does work. We didn't need to muck around with the internals very much because there are enough hooks in place for that to work.

**Will:** Cool. So then after I got David's email and sent him the code from my dissertation (some of which wasn't in very good shape) I saw that Clojure Conj was coming up, and I saw that Ambrose was giving a talk and David was also giving a talk, I think. So I thought it'd be fun to go there. So I got tickets. I convinced Dan to come. We went as civilians, I guess. Ambrose gave a really nice talk. Ambrose, do you want to say something about how you got involved?

**Ambrose:** Sure. I first heard of core.logic when it moved to contrib and when that first mailing list came out. I kept bugging David about this cool core.logic stuff. I made it a goal to make it a Conj talk. I had a tiny type inferencer in there. It just so happened that my talk was just before a little un-session from Dan and Will which was just amazing. They made it segue perfectly. I was really pleased with that. My hacking with miniKanren was really in 2011, early 2012. Lately, I've been doing [core.typed](https://github.com/clojure/core.typed) in Clojure. I'm excited to get back into [miniKanren].

**Will:** Have you been going through any cool typed [PL] books? Have you been going through [Pierce](https://www.cis.upenn.edu/~bcpierce/tapl/) or [Harper](https://www.cs.cmu.edu/~rwh/pfpl.html) or anything like that?

*Ambrose:* A bit of Pierce's book. But it's mostly the Typed Racket papers. That's it really. There's a lot of them. Just implementing and porting has taken up my 2012.

*Will:* That's just a quick recap. As far as Oleg's involvement, like I said, he had written this email to Dan shortly before I arrived at IU. I had went to Lightweight Languages 3 at MIT (I had gone to Lightweight Languages 2, also). I convinced Dan to go with me. This was at MIT, and the Scheme Workshop was being held at Northeastern the same weekend, or whatever. So Dan decided to go up with me and we got to meet Oleg. We sat in a hotel room and hacked miniKanren. I was a fly on the wall because I didn't know anything. I was like, "I'll be quiet, you just hack". It was actually kind of intimidating, but very cool to see. Oleg ended up visiting IU a few times and visiting Ken [Chung-chieh Shan] at least once and got to meet him a little bit more. I hope Oleg will make it to one of these Clojure conferences; I was glad to see him come to Strange Loop.

That's a bit of history. Obviously I could go into it more, but that's the broad brush strokes, I guess. Does that help, Austin?

*Austin:* When did you decide that was gonna be the topic of your dissertation? How did you decide to write about going through alphaLeanTAP? And what's happened since?

*Will:* I guess as is the case for many people I didn't plan my dissertation topic in advance. It was more like in hindsight: I have a bunch of papers and I want to graduate, and most of them are on the same topic. If I want to graduate this decade, I should probably staple them together and write a dissertation. It used to be more the case that people would literally staple three papers together and write an introduction. That's not quite the case anymore. When I wrote my dissertation I was hoping to the LaTeX version [of that]: import three papers and then write an intro. But I ended up having to rewrite everything from scratch to some extent. I'm glad I spent the time on it, but it was a lot of work to write that dissertation. I think [the topic] was partly an accident.

We kept hammering on these papers. In fact the first paper that came out, [the Scheme Workshop paper from 2006](https://www.schemeworkshop.org/2006/12-byrd.pdf) was the first miniKanren paper. I was, at the time, deeply ashamed of the paper. I had an idea and it didn't quite work out, and this is the paper we ended up with. But it seems like people like that paper, mainly because it had a cleaned up implementation in the back. I think a lot of people learned the miniKanren implementation from that paper. 

And then we had the nominal logic stuff. That came about because there was a problem I was trying to solve: the "copy term problem". So `copy_term/2` is an impure operator in Prolog. You can take a term (in Scheme, this would be a list that contains fresh logic variables and other things like booleans and stuff like that). But the idea is you want to create a copy of that term, but you want to replace all the fresh variables and you leave everything else alone. And it turns out that this isn't actually a pure operator. To me, a pure relational operator is one where you can reorder the conjuncts and disjuncts in any way you want, and that's not the case with `copy_term`. So for months I had been trying to think if there's a declarative way of doing `copy_term`. And I came up with a bunch of really weird stuff and I never was able to figure it out. I think it was Roshin James (and maybe Michael Adams) had mentioned something about this nominal logic stuff that Andy Pitts was doing at Cambridge. So I looked at that a little bit, and that looked really cool.

And then at Trends in Functional Programming in New York (I forget which year that was) I met one of Andy Pitts' students who had worked on a language called ML/SOS which was like a metalanguage for structural operational semantics [Lakin and Pitts 2009](https://www.cl.cam.ac.uk/~amp12/papers/residb/residb.pdf). I met Matt Lakin, who is a really great guy, he was finishing up his PhD. We invited Andy Pitts to come out and visit IU Bloomington and talk about nominal logic, but he was busy at the time. But we were able to get Matt out, who showed us a bunch of cool stuff: their ML/SOS language. And there's also an alphaProlog language. And the idea of nominal logic is that you can do unification modulo alpha equivalence, which is just a fancy way of saying: if you have two terms that have a notion of scoping in them, you can unify them and do variable replacement, or alpha renaming. This turns out to be really handy when writing interpreters and type inferencers and so forth. So that was cool.

But I still didn't know how to solve `copy_term`. And then a really good undergraduate, Joseph Near (who I guess is finishing up his PhD at MIT now) spent a summer working with Dan and two other undergrads on trying to do an automatic theorem prover. Dan wanted to do it in some flavor of miniKanren. It was actually based on this leanTAP tableau-based theorem prover. The cool thing about the original leanTAP paper is that the entire implementation of the theorem prover is the abstract of the paper; it's only a few lines of Prolog code. So we did a miniKanren version of leanTAP and decided to do a nominal logic (alphaKanren) version of it. When Joe Near was working on that, he realized that by using a combination of nominal logic and tagging, you could do a copy_term (because the original leanTAP paper uses copy_term). Joe figured out if you use tagging and nominal logic--in *that particular instance*-- a fully relational version of copy_term. To me, that's the big technical innovation in that paper [Near, Byrd, and Friedman](http://webyrd.net/alphaleantap/alphatap.pdf). It was cool. Once you have that type inferencer, you can say: here's a theorem, give me a proof. Or here's a proof, give me a theorem that matches a proof. Or you can start enumerating pairs of theorems and their corresponding proofs. Or you can partially-instantiate a theorem and get the proof. You can play all sorts of cool games like that. But to me, the main thing was being able to do that relational `copy_term`.

**David:** Will, do you want to explain why `copy_term` can't be relational?

**Will:** Okay. First, is it clear what I mean by saying an operation can be relational? Let's try doing a little example here.

```scheme
> (run* 1 (q)
    (fresh (x y)
      (== y 5)
      (== x 6)
      (== `(,x ,y) q)))
((6 5))
```

So let's do a `run*`. If you're not familiar with miniKanren, `run*` means: I'm doing a query, I want to know which values are bound to `q` (which is the query variable), and I want all of the possible values (which is what the star in `run*` means). `fresh` introduces new logic variables. We have `x` and `y`. I'm performing two unifications with `==`. I'm unifying `y` with `5`. You can think of unification as two-way pattern matching where you can have variables on either side. And I'm going to unify `x` with 6. And at the end I'm going to unify `q` with this list containing `x` and `y`. And I get back one answer. (There's an extra set of parentheses wrapped around this, because you could potentially have multiple answers.) We have one answer back: the list `(6 5)`.

The thing that makes relational programming interesting is that we can swap the order of these goals.

```scheme
> (run* 1 (q)
    (fresh (x y)
      (== x 6)
      (== y 5)
      (== `(,x ,y) q)))
((6 5))
```

So now the unification of `x` comes first and we get the same answer back. In fact, I could move the unification of `q` up:

```scheme
> (run* 1 (q)
    (fresh (x y)
      (== `(,x ,y) q)
      (== x 6)
      (== y 5)))
((6 5))
```

And we get the same answer back. I can also do things like add disjunction. So I can have a `conde` which is sort of like Scheme`s `cond`, in a way, except it tries all the clauses. I want to do a `run 1` first.

```scheme
> (run 1 (q)
    (fresh (x y)
      (conde
        [(== x 6)]
        [(== y 5)])
      (== `(,x ,y) q)))
```
I want 1 answer back. The `conde` means I'm going to try either one of these clauses. In practice, since I'm only going to get one answer back, I'll try the first clause. So `x` will be associated with `6`. But in this clause `y` is not associated with anything, so `y` is going to remain fresh. And then in the output, we get back this answer.

```scheme
((6 _.0))
```

So we have `6` in the place of `x`, the first item in the list. And `y` is fresh, which we represent as this special value: underscore-dot-some number.

If I want to, I can change this to a `run*` to get all the answers.

```scheme
> (run 1 (q)
    (fresh (x y)
      (conde
        [(== x 6)]
        [(== y 5)])
      (== `(,x ,y) q)))
((6 _.0)
 (_.0 5))
```

I get back two answers now. This is a proof, by the way, that there are *only* two answers, no more. In the second answer, we're unifying `y` with `5`, and we're forgetting about the association between `x` and `6`; you might call it "re-freshing" it. We're pretending as if [this first disjunct] had never happened. So for the second answer, `y` is `5`, `x` is not any value that we know of -- it's still fresh. So that's why `x` now is represented as this `_.0`. So there are two answers and two answers only.

The important thing is that I can still play this game of swapping goals around, and I still get the same answer (up to ordering).

```scheme
> (run 1 (q)
    (fresh (x y)
      (== `(,x ,y) q))
      (conde
        [(== y 5)]
        [(== x 6)]))
((_.0 5) (6 _.0))
```

I can even go beyond that. Let's associate `x` with something in this first case. Let's say `x` is `4` here, and `y` is 7 in the second case.

```scheme
> (run 1 (q)
    (fresh (x y)
      (== `(,x ,y) q))
      (conde
        [(== x 4) (== y 5)]
        [(== x 6) (== y 7)]))
((4 5)
 (6 7))
```

Now we get these two answers back where neither one has fresh logic variables. Within each `conde` clause, we have a conjunction, so both of these goals have to hold in the first clause: `(== x 4)` and `(== y 5)`. So `conde` is a disjunction of conjunctions. And we can swap the goals inside of the `conde` clauses; we can swap the `conde` clauses themselves; we can swap the goals within the `fresh`.

```scheme
> (run 1 (q)
    (fresh (x y)
      (conde
        [(== y 7) (== x 6)]
        [(== x 4) (== y 5)])
      (== `(,x ,y) q)))
((6 7)
 (4 5))
```

And even though the order of the answers comes back different, we should be thinking of these answers as a set of answers. We still get the same set of answers. To me, this is what makes programming relational or non-relational: whether or not you can swap the conjuncts and disjuncts and get the same answer back. To me, this is the most important and fundamental property of miniKanren programming and core.logic programming.  Anything that violates this property--any sort of operator where you *can't* swap things around and still get the same set of answers--is impure. That's my view of purity. So that's the basic idea of purity. Before I talk about `copy_term`, are there any questions about how that works? I know that's a lot, I may have lost some people. You have to understand that before I can explain `copy_term`, but if you understand that, we can do `copy_term`. If this is the only thing I said today, it would still be worth it, because whenever we're trying to design a new operator for miniKanren (or a new constraint), the way I always think about whether or not the design is correct is whether or not it violates this property of being able to swap conjuncts and disjuncts and being able to get the same semantics. Anything that violates that I don't want in the language.

There are these operators `conda` and `condu` which are closer to the way Scheme's `cond` works. They basically prune the search tree--you don't necessarily get all the answers back. The equivalent operator (called "cut" !/1 in Prolog) is used a great deal in Prolog programming. There are good reasons to use cuts for efficiency and to prevent divergence. However, if you are not really careful, the cut operator is unsound: you can *lose* answers that actually exist. Furthermore, once you start using cut, in general, you start losing the ability to swap conjuncts and disjuncts. So that's why I don't like `conda` and `condu`. Oleg and I kinda have disagreements about this. There are still some cases where I don't know how to get rid of `conda` and `condu`, and I can give you an easy example of that, which I'm beating my head against right now. Anyway, `conda` and `condu` are examples of operators in miniKanren that are in the book, in all the implementations that we have, and in core.logic that are not relational operators in my mind.

Another operator that we had trouble with was this `copy_term` operator. I don't have an implementation of `copy_term` but I can kinda fake it in Emacs. I can show you how it works and why it's not relational.

Let's pretend we're doing language design and we want to design a `copy_term` operator. First of all, we want to have a term. A term is just any legal value that miniKanren can use in unification, for example.

Suppose term 1 is a list with a boolean and a number and a sublist and another boolean. And in the sublist we're gonna have the symbol foo and the variable y and the variable z, let's say. And at the end let's have a variable w. So that's a term.

```scheme
> (run 1 (q)
    (fresh (t1 t2 y z w)
      (== `(#t 5 (foo ,y ,z) #f ,w) t1)))
((_.0))
```

When we run this, since we didn't associate anything with `q`, we get back `_.0`. So this is a legal miniKanren program. We have this term `t1`. If we want to see `t1` [in the solution] we can unify it with `q`.

```scheme
> (run 1 (q)
    (fresh (t1 t2 y z w)
      (== `(#t 5 (foo ,y ,z) #f ,w) t1)
      (== t1 q)))
(((#t 5 (foo _.0 _.1) #f _.2)))
```

So there's our term. Now it may be that, for whatever reason, we want to make a copy of `t1` where the variables are replaced, but everything else is the same.

**David:** Can you show how `copy_term` is different from just unifying `t1` with `t2`? Just so people understand that something different from unification has to happen.

**Will:** That's a good idea. Before I unify them, let's just reify `t1` and `t2`.

```scheme
> (run 1 (q)
    (fresh (t1 t2 y z w)
      (== `(#t 5 (foo ,y ,z) #f ,w) t1)
      (== `(,t1 ,t2) q)))
(((#t 5 (foo _.0 _.1) #f _.2) _.3))
```

So `t2`, as we might expect, at this point isn't associated with anything.  So this second part, `_.3`, is `t2`, still fresh. So we might have this idea that, in order to make a copy of `t1` we just have to unify `t1` and `t2` together. So let's try that.

```scheme
> (run 1 (q)
    (fresh (t1 t2 y z w)
      (== `(#t 5 (foo ,y ,z) #f ,w) t1)
      (== t1 t2)
      (== `(,t1 ,t2) q)))
(((#t 5 (foo _.0 _.1) #f _.2)
  (#t 5 (foo _.0 _.1) #f _.2)))
```

The tricky part here: do you see those `_.0`s, `_.1`s, and `_.2`s? They represent the same variable in both cases. So why might that be a problem?  The problem comes up when we want to use the term `t1` as a template. Maybe we want `t1` to represent something where we're gonna fill in parts of the structure later, multiple times. And we want to keep the original template around. And let's say we're only going to instantiate the copies of the template. That turns out to be problematic.

Let me try to justify that a little bit more. Instead of just a term, maybe we have a lambda expression. 

```scheme
> (run 1 (q)
    (fresh (t1 t2 x body)
      (== `(lambda (,x) ,body) t1)
      (== t1 t2)
      (== `(,t1 ,t2) q)))
(((lambda (_.0) _.1) (lambda (_.0) _.1)))
```

Like that. In fact, this is exactly the context in which this came up. I was trying to write interpreters. I basically wanted to do a substitution style interpreter and substitute into the `t1`. It turned out I was having problems because I was damaging the original term, whereas I wanted to be able to reuse it.

Let's say our term `t1` represents the identity function like `(lambda (,x) ,x)`. We might think we can get away with something like procedure application. Suppose I'm going to implement an interpreter and I want to implement procedure application. And I want to be able to apply `t1` to some value like `5`. I've got this brilliant idea: how am I going to implement application? Easy! I'm going to do it by unifying `x` with `5`, because that's the variable. Then, I could just take the body and that will be my answer. Because unification, in effect, will perform variable substitution for me. So now that I'm really clever about things. I'll say something like `(lambda (,ignore) ,answer)` unified with `t1` as well.

```scheme
> (run 1 (q)
    (fresh (t1 t2 x ignore answer)
      (== `(lambda (,x) ,x) t1)
      (== x 5)
      (== `(lambda (,ignore) ,answer) t1)
      (== t1 t2)
      (== `(,t1 ,t2 ,answer) q)))
(((lambda (5) 5)
  (lambda (5) 5)
  5))
```

Look what I just did here. I want the identity function applied to 5. The way I do this is by unifying the logic variable representing the argument being passed in to the value being passed in. And then I just grab the body. Which is fine, it actually works. I get back 5, which is the correct answer. The problem is I've damaged my original term. Now, if I wanted to apply the lambda term `(lambda (,x) ,x)` to a *different* term, 6, also, then it doesn't work anymore. If I want to apply the same lambda term to two different values, it fails.

Brilliant idea: what if I make a copy of `t1` first? With that duplicate term, `t2`, then I can use `t2`, apply that to 5, keep `t1` as my template, and every time I want to do an application of `t1`, I'll just make one more copy of it and then do a different application. I'll just use unification of variables to values to do the application. In order to do that, I need to be able to make a copy of `t1` that replaces the fresh logic variables with new logic variables. You can see, just by unifying `t1` and `t2`, that doesn't work. I get back the term that's already been instantiated.

Instead I want to somehow produce a term that's like a copy of `t1` that has `x^` in place of `x`.

```scheme
> (run 1 (q)
    (fresh (t1 t2 x x^ ignore answer)
      (== `(lambda (,x) ,x) t1)
      (== x 5)
      (== `(lambda (,ignore) ,answer) t1)
      (== `(lambda (,x^) ,x^) t2)
      (== `(,t1 ,t2 ,answer) q)))
(((lambda (5) 5)
  (lambda (_.0) _.0)
  5))
```

Here, `t1` becomes damaged through this application technique. But `t2` can now act as a template. I can just keep making copies of that every time I want to do an application, and I can still use this trick of unifying a logic variable representing a lexical variable to get back the `5`. Now, there are some scoping issues and there are other reasons that this could be problematic, but in a nutshell, this doesn't work anyway because I don't have an easy way of copying the term and replacing the variables. And that's exactly what `copy_term` is supposed to do. If I had `copy_term`, which I'll call `copy-termo`, the idea is I could do a `copy_term` here before I unify `x` with `5`. That would create the equivalent of `(lambda (x^) x^)` where `x` has been replaced with this new variable `x^`, and I can do exactly what I wanted to do before.

```scheme
> (run 1 (q)
    (fresh (t1 t2 x x^ ignore answer)
      (== `(lambda (,x) ,x) t1)
      (copy-termo t1 t2)
      (== x 5)
      (== `(lambda (,ignore) ,answer) t1)
      (== `(,t1 ,t2 ,answer) q)))
```

So that's the setup. So why doesn't this work? It *does* work as long as you don't care if `copy_term` isn't relational. You could do this in Prolog. We could implement a miniKanren version of `copy_term`. It's not too hard to implement, in fact, it's quite easy. It's a couple lines of code.

The problem is this: like I said before, the semantics of the miniKanren code should be the same no matter how you reorder the goals. So what happens if we move the `copy_term` up front?

```scheme
> (run 1 (q)
    (fresh (t1 t2 x x^ ignore answer)
      (copy-termo t1 t2)
      (== `(lambda (,x) ,x) t1)
      (== x 5)
      (== `(lambda (,ignore) ,answer) t1)
      (== `(,t1 ,t2 ,answer) q)))
```

Well, now we're copying `t1` while it's still a fresh variable. Then `t2` is just a new fresh variable; it has no structure at all. So then when we unify `t1` with this list, `t2` doesn't get unified with the lambda term at all; it doesn't have any of the structure. So when we would look at our answer, `t2` would just be a fresh variable, it wouldn't do anything.

Also, if we move `copy_term` down below the unification of `x` and `5`, now `t1` is a ground list! It's `(lambda (5) 5)`. So the `copy_term` would produce a new term `t2` which is this same list `(lambda (5) 5)`.

```scheme
> (run 1 (q)
    (fresh (t1 t2 x x^ ignore answer)
      (== `(lambda (,x) ,x) t1)
      (== x 5)
      (copy-termo t1 t2)
      (== `(lambda (,ignore) ,answer) t1)
      (== `(,t1 ,t2 ,answer) q)))
```

So we have this problem where what `copy_term` does depends on where we place it in our program. Actually I should be careful, because there are lots of [pure relational] constraints that will behave differently depending on where you put it. But the point is, the answer you get at the end has to be the same with any ordering, and `copy_term` breaks that.

The cool trick in the alphaLeanTAP paper is that there's a way to use nominal logic (plus tagging) in the context of the alphaLeanTAP prover to make a relational version of `copy_term`. It's not completely general. It doesn't work with just any terms. But in the context of that prover, we were able to come up with a relational version. And to me, that was the cool part of the paper. I haven't figured out how to apply that in other context yet, but that at least will give you some idea of the sorts of problems that we come up with when trying to create operators in miniKanren.

**Jamie:** That was cool. I have a different question about the reordering property that you want. Can you talk about recursion? You have to put the recursive call last (rule of thumb).

**Will:** That's a good point. I kinda lied about the semantics--that you can reorder things in any way and that you always get the same answers back. There are several caveats with that. Let me show you some things where this doesn't actually work.

miniKanren has this crazy numeric system that maybe you've seen. This is really a tour-de-force from Oleg. But the idea is that you can build numbers using `build-num`. So we have some representation of 3 and some representation of 4, and we add them together.

```scheme
> (run 1 (q) (pluso (build-num 3) (build-num 4) q))
((1 1 1))
```

We get 7. This representation of 7 is a list of binary digits in little-endian order.

```scheme
> (run 1 (q) (pluso (build-num 2) (build-num 2) q))
((0 0 1))
```

Here's a representation of 4. The least significant digit comes first; the most significant digit comes last. So this is like 100 in binary.

The cool thing about `pluso` is that we can feed in variables. And once we   do that, we can get back an arbitrary number of answers.

Let's do `*o` instead. We want to find numbers where the product is 6. We'll look at two numbers `x` and `y`. Let's go ahead and get all answers, pairs of values for `x` and `y`.

```scheme
> (run* (q)
    (fresh (x y)
      (*o x y (build-num 6))
      (== `(,x ,y) q)))
(((1) (0 1 1))
 ((0 1 1) (1))
 ((0 1) (1 1))
 ((1 1) (0 1)))
```

- 1 * 6 = 6
- 6 * 1 = 6
- 2 * 3 = 6
- 3 * 2 = 6
So those are all the ways to get 6. We just did factoring through (relational) multiplication.

If we want to generate more answers, we can leave the output fresh and do `(*o x y z)`. If we have `run*` here, we'll diverge, so let's get 10 answers back.

```scheme
> (run 10 (q)
    (fresh (x y z)
      (*o x y z)
      (== `(,x ,y ,z) q)))
((() _0 ())
 ((_0 . _1) () ())
 ((1) (_0 . _1) (_0 . _1))
 ((_0 _1 . _2) (1) (_0 _1 . _2))
 ((0 1) (_0 _1 . _2) (0 _0 _1 . _2))
 ((0 0 1) (_0 _1 . _2) (0 0 _0 _1 . _2))
 ((1 _0 . _1) (0 1) (0 1 _0 . _1))
 ((0 0 0 1) (_0 _1 . _2) (0 0 0 _0 _1 . _2))
 ((1 _0 . _1) (0 0 1) (0 0 1 _0 . _1))
 ((0 1 _0 . _1) (0 1) (0 0 1 _0 . _1)))
```

Here are 10 answers. It doesn't matter what they are. The important thing is the 10 answers we get back depend on the implementation of `*o` and they depend on miniKanren's search strategy--the fact that it uses interleaving search instead of depth-first search, and things like that.

Really, there are an infinite set of solutions (to x * y = z) and we only get some of them back. If we reordered goals inside the definition of `*o`, for example, and did a `run 10`, we might get 10 different answers back, or some answers and not others. So this is a restriction on the property I claim: that reordering the goals will give us back the same set of answers. That's only true if we get back the entire set of answers. So that's one way in which the notion of swapping conjuncts and disjuncts--you have to be careful when you define what it means to be pure, or relational.

The other big caveat, which is what Jamie is talking about, is when you have a program that goes into an infinite loop, or diverges. Divergence basically means that there isn't a solution, but miniKanren can't figure out there isn't a solution.

Let's try creating a situation where there's an infinite loop and miniKanren can't figure it out. Let's define a helper function here called `alwayso`.

```scheme
> (define alwayso
    (lambda (q)
      (conde
        [g]
        [(alwayso g)])))
> (run 5 (q) (alwayso (== q 5))
(5 5 5 5 5)
```

`alwayso` is a higher-order goal, basically. (A goal is something like `==`, `conde`, `fresh`, or something built with those.) `alwayso` is recursive: it either runs the goal or it recurs. This is the same as writing a `conde` that has an unbounded number of calls to the goal `g`.

```scheme
> (conde
    [g]
    [g]
    ...)
```

When I say `(run 5 (q) (alwayso (== q 5))`, that would be the same as if I had a conde that had `(== q 5)` in each of the clauses an arbitrary number of times.

```scheme
> (conde
     [(== q 5)]
     [(== q 5)]
     ...)
```

If I want 5 answers, I'll get 5 answers. If I want a million answers, I'll get a million answers. If I do a `run*`, it'll diverge.

Anyway, `alwayso` runs this goal over and over. The important thing is that I can always try to get another answer back. Let's try to cause some trouble for miniKanren.

With unification, if we want to unify `q` with 6 twice:

```scheme
> (run 1 (q)
    (== q 6)
    (== q 6))
(6)
```

That works fine, because that information is consistent. If we instead want to unify `q` with 5 and then unify `q` with 6, that's not consistent. There's no value of `q` that will make it both 5 and 6 at the same time; that doesn't make sense. It doesn't matter how you reorder these things.

```scheme
> (run 1 (q)
    (== q 6)
    (== q 5))
()
```

So we get back the empty list. That means there is no answer.

Let's try adding an `alwayso` here.

```scheme
> (run 1 (q)
    (== q 6)
    (== q 5)
    (alwayso (== q 7)))
()
```

That too has no answer, because `q` can't be 6, 5, and 7 at the same time.

Now what happens if I move the `alwayso` up front?

```scheme
> (run 1 (q)
    (alwayso (== q 7))
    (== q 6)
    (== q 5))
...
```

It doesn't come back. We're in an infinite loop. miniKanren is trying to associate `q` with 7, which succeeds. Then it tries associating `q` with 6, which fails. But because `alwayso` can succeed an unbounded number of times, we try associating `q` with 7 again, and then we try associating `q` with 6, and it fails. We go back and associate `q` with 7 which succeeds, and then we go back and associate `q` with 6 and that fails, and we just keep toggling back and forth between these two clauses forever. In fact, I can get rid of that third goal and we still diverge:

```scheme
> (run 1 (q)
    (alwayso (== q 7))
    (== q 6))
...
```

So this is really interesting. When we have the `alwayso` up top, we go in an infinite loop. But if I move the `alwayso` to the bottom, it fails with no more answers. It fails immediately.

So this is the other big caveat to this notion of declarativeness. In miniKanren it works like this: you can reorder `conde` clauses any way you want without changing the semantics. If there's a finite set of answers to be produced, you'll get back the same finite set, maybe in a different order, as long as you do a `run*`. If you have a finite amount of answers you can change the `conde` clause ordering all you want. You're changing the disjuncts, the "or"s.

If you want to change the ordering in a `fresh`, in a `run`, or within a particular `conde` clause, those all represent conjuncts (those are "and"s, basically). If there is no answer at all and you do a `run 1`, or if there are only 4 answers and you do a `run 5` or a `run*`. If you change the ordering of conjuncts, one of two things happens:
1. Either your program will fail (or produce four answers and no more) and terminate in a finite amount of time.
2. Or, it will go in an infinite loop.
And these represent the same information, in a sense. But of course, we can't tell, in general, whether or not a program goes into an infinite loop. It's unfortunate. So we always want to fail as quickly as possible.

So the art of writing miniKanren programs is trying to make sure that you fail fast. And unification (by itself) is something that fails fast. We want to put calls to `==` first, because maybe we'll end up in a system where we have inconsistent information and we'll just fail. We won't try going through this infinite search tree.

The trick, of course, is when you have *two* goals that could potentially go into an infinite loop. Like if you had two calls to `alwayso` in your program. You can't put both of them last. 

This ordering diverges:

```scheme
> (run 1 (q)
    (== q 6)
    (alwayso (== q 5))
    (alwayso (== q 7)))
...
```

This ordering diverges:

```scheme
> (run 1 (q)
    (== q 6)
    (alwayso (== q 7))
    (alwayso (== q 5)))
...
```

This ordering diverges:

```scheme
> (run 1 (q)
    (alwayso (== q 7))
    (== q 6)
    (alwayso (== q 5)))
...
```

And so forth. So now we're stuck. When you have multiple goals within conjunction (a `fresh`, `run`, or `conde` clause) that can each produce an unbounded number of answers, you have the potential for divergence in a serious way. Then it becomes tricky to try to write your programs. You might have to write special bounds like Oleg did for the number system, use lazy constraints, or use other constraints.

One thing to remember is that this only comes up when you're searching for an answer that doesn't exist. If you're looking for 5 answers and there's 6 answers (or 5 answers, or a million answers) then it doesn't matter how you order your conjuncts or disjuncts. You'll always find those answers, at least in theory. It might take a million years, or terabytes of memory, but you'll find the answers if they exist. This is only a problem if you're looking for answers that don't exist. We call this the "commutative conjunction" problem. Conjunction in miniKanren is only commutative up to divergence; your program might diverge instead of failing finitely. This is a tricky problem we've been working on for years. There may be some partial solutions, but it's a hard problem.

**David:** As far as I know, it's also not solved for Prolog. Or for anyone else that wants to do this type of logic programming.

**Will:** Yeah, I've never seen a solution. I mean, I should be careful. We actually have working versions of commutative conjunction, but they're exponential (in cost) in the number of conjuncts. It's not an undecidable problem, because what you can do is run all of the orderings. You can make copies of the miniKanren program, basically, with all possible orderings of the conjuncts, and then run those programs in parallel. But obviously this is expensive. It's a problem that's theoretically solvable, but the performance is bad.

**Jamie:** But the case like the two-`alwayso` example you showed, that would be undecidable, right?

**Will:** It depends on your level of granularity. If the problem is just trying to reorder the goals, it's not undecidable in the sense that you couldn't try all the orderings. Of course, miniKanren programs can diverge. There's nothing you're gonna be able to do, ultimately, because it's a Turing-complete language. What I'm saying is you could try all possible orderings (like at runtime, for example) at exponential cost. There are still other ways to get into an infinite loop. It turns out there are tons of ways to get in an infinite loop in miniKanren. This is just one possible way. Even if you had a commutative conjunction, you could still get into an infinite loop in other ways.

It's certainly undecidable if you tried to unroll, or tear apart, infinite goals like those `alwayso`s. There might be some solutions (or partial solutions) using clause learning. Ken Shan pointed us to that. I found a paper the other night on trying to dynamically reorder goals for best behavior, and stuff like that. So there are tricks you can use. But I haven't seen a real nice solution to this problem.

**David:** Part of this problem is that neither miniKanren nor core.logic do upfront static analysis. There are a lot of papers on doing static analysis to see if you can guarantee termination.

**Will:** Also in languages like Mercury, you're required as a programmer to put on type annotations onto your goals. But in addition, you put on mode annotations to say "this argument is always instantiated", "this argument is always a fresh variable", or "this one could be in or out, either way". You can put on additional annotations saying if the goal is finitary: could this goal produce just one answer, a finite number of answers, or an infinite number of answers. So between the type system, knowing the modes, and knowing whether each goal is finitary, the [Mercury] compiler can do lots of static analysis with that information, a lot of optimizations. It can compile the relations down to just functions, and thigs like that. With miniKanren and core.logic, we don't really have any of that mechanism. And these are embedded languages: we don't have a full compiler, we're not doing full analyses. There are lots of things that we aren't doing right now. You could definitely imagine much more efficient versions of miniKanren or core.logic with lots of analyses. It would be cool if someone interested in compiler hacking would like to write a compiler for miniKanren will all sorts of optimizations and analyses. Ciao Prolog actually does lots of cool stuff. Jamie, does that answer your question?

**Jamie:** Yeah!

**Will:** So I don't have a good solution to that. Actually, since you're at Northeastern, I've got a question for you. I've used miniKanren a few times to implement reduction systems for doing semantics or trying to understand a paper on semantics. PLT Redex, obviously, is designed for those purposes.

**Jamie:** I've been saying for ages now that you and Robby [Findler?] need to talk to each other.

**Will:** Yeah, we do. Anyway, I was just curious if you've been playing around with Redex or if you had any thoughts around how Redex could use ideas from logic programming. One thing I thought Redex could potentially use is nominal logic. How do you normally deal with scoping in Redex?

**Jamie:** My head is not in that space right now. I haven't messed around in either Redex or miniKanren in a while.

**Will:** What would be a good thing to approach next time?

**Austin:** I got two things I am strongly interested in. One is the implementation, with all the monads, the choice, and all that. Second, really on my mind, is how do we debug these programs? How can we trace lvars?

**Will:** Oh dear.

**David:** I can say things that I want to do. Prolog has a long history of doing meta-interpreters. The way that serious Prolog programmers will debug their programs is they will write a meta-interpreter and run their program inside the meta-interpreter. That's a way they can get traces and lots of debugging information. They can see the proof tree. You can actually do that. There's nothing stopping you from working in that way. Nada Amin, who does core.logic stuff now, has tried that and it's worked out. But still, I would prefer debugging to be inside core.logic. What's nice is that the monad that you're passing around can hold whatever you want. In fact, every time Dan, Will, and I add a feature to either core.logic or miniKanren, we're just adding stuff inside the store. There's more stuff going in there that holds more information. So you could in theory keep a program trace there. I want to start up a substitution and run through the monadic interfaces, but we actually collect extra information as it gets passed along. So my idea was to provide a "debug" substitution. (Debugging is going to be costly, because you're tracing the execution of the whole program, so normally you don't want to run with it on.) If you run with "debug" on, you can run your program just like normal. One big issue is that miniKanren currently discards failures. Whenever you get a failure, the backtracking monad just throws it away, which is not what you want.

**Will:** This is really *fun* when you're trying to debug type inferencer  problems, or if you're just trying to use it as a type checker. If your program doesn't type check, what do you get back? The empty list!

**David:** Yeah. So something that's on the table is we're gonna stop throwing away failures. So when you run in debug mode, we won't discard failures. We'll actually wrap the substitution in a special "failure" type which will appear in the stream. Then you can say "I want the fifth failure". And then you can say "Give me stack trace when that failure occurred." I've been wanting this for, like, 2 years. So we're definitely starting to work on it. Does that answer your question?

**Austin:** Yeah. I was curious about actual implementation issues. I was wondering if we could annotate the substitutions with metadata, or try to keep track of the graph.

**David:** We're gonna let you do that, basically. The important thing is that we don't discard the ones that failed. Currently, we're discarding them and there's no way to get to the failures.

**Austin:** I'm also curious how this can be presented. I have an implementation right now where I can trace it out like a stack trace, but due to the nature of how [the search] is actually executing, it's not of great help. For instance, you could be following a branch that, at the same level of the graph, it jumps to a different branch. It gets hard to mentally see what's happening. I feel like that presentation could be a lot better.

**David:** Yes, even if we record the graph, that will be a challenge of how to present it to the user.

**Austin:** Yeah.

**Will:** As far as the way I debug miniKanren programs, I kinda don't. I just write them correctly the first time. What I mean by that is I don't spend time debugging miniKanren programs. If it doesn't work exactly the way I want it to, I look at the code. Dan and I spent a fair amount of time trying to develop tools for miniKanren. (We have some; I think Dan occasionally uses them.) But I found it very difficult to do in a productive way. I don't debug in general, I should say. It's kinda not my philosophy. I try to do correctness-preserving transformations, or try to write code from the math when I can. If I'm really having trouble with code, I either throw the code away and start over again for that part of it, or I look at it really carefully and try to compare it to the rules.

There are a few really trick gotchas in miniKanren. I'll show you the hardest thing in the world to debug in miniKanren, to the extent that we thought about changing the language. This one does get me. Very rarely. But when I do, I spend an hour looking for it.

Suppose I define `fooo`:

```scheme
> (define fooo
    (lambda (x y)
       (== x 5)))
> (run* (q) (fresh (x y) (fooo x y) (== `(,x ,y) q)))
((5 _.0))
```

And then I go into `fooo` a week later, a month later, or a year later, and I decide I actually want to define `(== x 5)` and `(== y 7)`.

```scheme
> (define fooo
    (lambda (x y)
       (== x 5)
       (== y 7)))
> (run* (q) (fresh (x y) (fooo x y) (== `(,x ,y) q)))
((_.0 7))
```

Wait--what happened to the `(== x 5)`? Do you see the problem? The problem is I didn't wrap a `(fresh () ...)` inside `fooo`.

```scheme
> (define fooo
    (lambda (x y)
      (fresh ()
        (== x 5)
        (== y 7))))
```

The way miniKanren actually works is it ignores all the goals except the last one inside a lambda. So it actually throws away the `(== x 5)`. When you're working on a big program and you make that mistake, you will pull your hair out, because things that you *know* work won't work anymore. So that, to me, is the biggest gotcha.

Short of that, the way I normally develop is: I will write one relation at a time, I will write test programs as I go; I will write any complicated relation one clause at a time, and I will write a test case for each clause as I go. Doing that is both good practice as far as writing test programs, and it also gives me a lot of confidence as I code. That's just the way I develop things. Because of that, I don't need to debug things in the sense that I wrote a bunch of code that's failing and I don't know where to start. Usually I've got a pretty good sense, because if I screw something up, a bunch of my tests fail.

Now *testing* miniKanren programs is actually really tricky. That's something worth talking about. If you change the ordering of goals when there are an infinite set of answers, and you do a `run 10`, well, you will potentially get 10 different answers back. Or if you're doing nominal logic programming, your answers are correct alpha-equivalent to some other  answer. So that actually turns out to be quite tricky.

But if you can write good test programs and develop a little bit at a time, I think that helps a lot.

**David:** One big complication, even if we improve debugging: one goal of core.logic and miniKanren is how much stuff you can stick in there. So the more constraint frameworks you support, the more difficult it is to debug, because many things are going on inside one program. We'll see. How easy a miniKanren program is to debug is probably dependent on how much of the system you want to use.

In any case, Nada had this weird thing where she was creating a variable and she was associating it with another variable, and we needed to keep them in sync. It was kind of like `copy_term`, but because of this issue, we didn't want to actually copy anything. So I had to build in core.logic a neat behind-the-scenes propagator. So if anything happens to the copied variable, we propagate those changes to the other one. This allows us to be relational. It's also what allows finite domain variables to sit inside a nominal logic program without messing things up.

**Will:** That sounds really cool. I'd like to hear more about that.

**David:** I think you could actually do a relational `copy_term`, if inside your miniKanren system you knew that these variables belonged to a set, and any change in one variable was always copied over to another one. So if there's a unification, it triggers a constraint, and those updates are passed on to what we call the "entangled variable".

**Will:** So you mentioned sets. Does this have anything to do with constraint logic programming over sets [CLP(set)]?

**David:** No, it's just that we had a set of entangled variables. You could say that x and y are entangled. Or that x, y, and z are entangled. It doesn't matter who gets the update, everyone else needs to receive it.

**Will:** Do you want to give a plug for the Google Summer of Code stuff?

**David:** For Google Summer of Code, there's a bunch of cool projects. Specifically for core.logic, we're looking for people interested in doing constraint logic programming over sets. I know Will wants it. I want it. I assume Clojure programmers want it. I think Clojure programmers always feel a little constrained in core.logic because they have to write Scheme-y programs--they have to use lists most of the time. They can't use the other Clojure data structures. So we would like sets (and eventually hashmaps) to work transparently in core.logic. It's a bit of a tricky problem.

And we would like to see some sort of probabilistic logic programming. That would be pretty cool. Those two would be big.

**Will:** Probabilistic logic programming is something I could imagine talking about in these hangouts, and maybe even reading a couple papers and trying to implement some [probabilistic] systems in miniKanren or core.logic. I implemented the SLP [Stochastic Logic Programming?] stuff, but that needs some work. The probabilistic stuff is really cool anyway. That's an area I could imagine spending some time talking about.

David has some things to talk about. I have some things I could talk about. I'm trying to think of ways where we can spread out the fun so that it's not all just one person talking. I think Nada has some things to talk about. Maybe Ambrose would be up for talking about something.

**Ambrose:** I wouldn't mind preparing something. It's 6am right now.

**Will:** One thing I was also thinking of was show-and-tell type stuff. Or we would have some sort of challenge of translating some code into miniKanren. Or if there's a problem that we have that we don't know how to solve very well, and see what people can come up with.

I can actually show you a problem I'm dealing with right now that I don't know the solution to. I'd really like a solution to this. It's a simple problem to explain. I have no idea how to solve this, and I've been thinking about it for a couple years and I have no idea. I don't think it's too hard to explain.

The problem has to do with writing small-step interpreters. There are big-step semantics and small-step semantics when you're writing an interpreter:

A big-step interpreter will take an expression and just return the final value for you. Like if it's some complicated procedure application that has arithmetic inside the bodies. You give it the expression, it gives you back 42, or whatever, in one big swoop.

A small-step interpreters does just a little bit of work at each step, and it's always guaranteed to terminate with each step. It can get stuck, but it's not going to go into an infinite loop. So it'll do a little bit of work. If you have a complicated arithmetic expression, it might just do a little bit of it. Or if you have a bunch of procedure applications it might do one procedure application and then stop. The trick is, when you write an interpreter (or a term reducer) in this [small step] style, you then need to have a driver relation that runs the small step reducer over and over and over again. And this comes up pretty up when you're doing semantics-type-stuff in miniKanren. And not just semantics: but anything where you want to keep running a rule (or set of rules) until you can't apply any more rules -- until there are no more applicable rules left.

The reason this is tricky is that you want to do something basically until you fail--until trying to apply this set of rules would fail. And if the rules *would* fail, then you want to just finish, like I'm done. Any time you have a situation where you want to do something if some other goal fails, then you're entering nonrelational territory very quickly (`conda`, `condu`).

Actually, I can show you some code I've written recently where I ran into this problem last week. It's on my GitHub: [https://github.com/webyrd/relational-progress-sensitive-security](https://github.com/webyrd/relational-progress-sensitive-security) It's the relational version of progress-sensitive security, which is a security paper by some folks at Harvard: ["Precise Enforcement of Progress-Sensitive Security" by Moore, Askarov, and Chong, CCS'12](https://people.seas.harvard.edu/~chong/pubs/ccs12-progress.pdf). I just took the semantics and created a little reducer.

Here's the type inferencer, which is cool because it deals with security levels (like secret vs public) instead of normal values. And here is the reducers, like this arrow relation is the reducer.

```scheme
(define ->o
  (lambda (in-state out-state)
    ; ...
    ))
```

You have an `in-state` and an `out-state`. For example, if you have the `'skip` command, then you just `'stop`. If you have an `'assign` command, then you just stop. Normally, you have a sequence of commands (two commands). There, you do the first command, and if the first command ends up giving you a `'stop`, then you stop, otherwise you go ahead and do the second command. Things like that. So that's all fine, that's easy to translate.

The tricky part is this thing right here, which is actually really short:

```scheme
(define ->*o
  (lambda (in-state out-state)
    (fresh (state)
      ; Dirty conda, playing in the mud!
      ; Wish I knew a cleaner way to do this.
      (conda
        [(->o in-state state)
         (->*o state out-state)]
        [(== #f #f) (== in-state out-state)]))))
```

It's the driver relation to do any number of these small steps, and to keep stepping while there's a rule to be applied. This is what the first clause is saying: if you're going from an `in-state` to a new `state`, that's fine, just recur and keep going.

What we really want to happen, though, is if there isn't a transition to the `in-state` to some other `state` (that is, if that fails) then we just want to be finished. The `out-state` should be what the current `in-state` is. I had to use `conda`, which is like a local version of Prolog's cut. Basically, I'm saying this [second clause] happens if the first test in the `conda` fails. If there is no stepping [to an intermediate `state`], then we're done. I want to write this in terms of a `conde`, and I don't know of a good way to do this. I've thought of using tabling, but I don't know how that would help. You could try encoding these really complicated relations to see if any of the other goals hold. You can kinda do that sometimes, but it's a real pain; it requires doing a lot of duplicate work. Maybe you could use a tool to do that, I'm not sure. This is a problem that I'm facing right now in another project. If anyone has any bright ideas on how to handle this, I'm all ears.

**Ambrose:** Where does that come up as a problem? Obviously, it's not relational.

**Will:** Well that's the big problem. To me, miniKanren is the relational language. I want to try to keep everything relational whenever possible. Because this could be a part of a bigger program. As soon as we go non-relational, it's putting a restriction on what we can do while running backwards. And this problem pops up in lots of guises, this is probably the most common version I've seen. Maybe there's some type of lazy constraint, advanced tabling, or any other sort of approach. I've had this problem with every small-step reducer I've worked with, every type of semantics I've worked with, and for a bunch of other types of programs. Even though I first saw this problem 5 or 6 years ago, I'm still no closer to a good solution today than I was then. That's something I'd like to solve.