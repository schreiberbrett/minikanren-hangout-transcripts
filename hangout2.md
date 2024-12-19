# miniKanren hangout #2, 24 March 2013

**Will:** Welcome everyone to the second Google hangout on air. We have an IRC channel now, hash-miniKanren on freenode if you want to join that and ask questions and discuss things.

Format-wise, maybe we can spend the first part of the hangout talking about general miniKanren or logic programming issues, the miniKanren confo at Clojure West. I know Dan's got a question for David Nolen about some optimization stuff. We can talk about all sorts of things. For the second half, I was thinking of talking about the relational CESK machine that Dan and I showed at the confo, because that's something we're writing up now. I can show you the cool things about it and some of the open issues.

Dan [and Jason], did you want to go ahead and ask the question?

**Jason:** Hey David, we're trying to use a model similar to the way core.logic is doing persistent substitutions. We were curious if you could talk more about how you were doing disequality constraints: how you were capturing the prefix and using it.

**David:** With disequality, because we want to use persistent data structures, we have two kinds of unification: user-level unification and internal unification. Internal unification basically doesn't track changes, but if a user calls `==`, we initialize a field to me (more or less) an empty list, and as vars change, because everything goes through `ext-S-no-check`, they get added to that list as being vars that have changed. So that's the prefix.

**Jason:** So your prefix is just a list of changed variables?

**David:** For us, that's all it is. For most constraints, just [tracking] the vars that have changed is enough. We don't care about the values. We actually handle disequality separately. In the case of disequality, we also do maintain a true prefix: the actual vars and their bindings that have changed after disequality. But in our case, we don't base disequality on unification at all. We don't use that trick. Does that make sense?

**Jason:** Yeah.

**David:** So we handle it in two different ways: one way for constraints in general, and a different way for disequality. Mostly this is because in the past, we did what you guys did. We actually would store the substitution in two different representations: one was a persistent hashmap, and one was a list. And that meant we were using twice as much memory just to calculate the prefix. So we changed our minds about how we wanted to do it.

**Jason:** Okay, that sounds like we're gonna have to try that approach. Thank you very much.

**Will:** I wanted to thank Jamie Perconti for giving the solution last time to the thing I was stuck on. I was talking about running a small step interpreter through a fixpoint loop. I had a little driver relation and I was having trouble figuring out how to do that. I was using `conda` and that made me unhappy. Jamie pointed out that I could have a `valueo` relation that says whether or not I've reached a value. By using a `valueo` relation, you can do the obvious recursion and it works just fine. That was the right answer.

Nada [Amin] actually had done that already for the nominal quine stuff for a small step interpreter. Then I realized I had actually done that also in the past, and for some reason I had convinced myself that `conda` was the only way to do that. Thanks Jamie for pointing that out and explaining it.

I've got a couple other tricks with the CESK machine that I can show. Most of them I understand pretty well. There's one that I'm still trying to understand. That's kinda the big open problem for me with this paper. I think I know what's going on, but I don't have all the details. But it looks like this could be a general purpose technique. I don't know how it relates to other techniques out there. The general rule of research is everything was discovered in the '60s or '70s. But I haven't seen this trick used in this manner, and it seems really useful for writing relational interpreters. I'll show that a little bit later, and maybe some people have some suggestions.

Dan and Jason, did you have any other implementation questions for David?

**Dan:** I don't think so. You answered the question. I don't know if we'll be using that idea or rather going back to the model of doing a `walk*` and checking to see we violated the disequality constraint after every unification. I'm not positive which way we'll go.

**David:** One thing we've been talking about, related to cKanren, is changing disequality entirely and basically having a new kind of constraint: a constraint where there's a parent constraint at the root, so that disequality has sub-constraints. And the moment when one of them is violated, it will invalidate all the sub-constraints all the way up to the root. So this is sorta like a disjunctive tree of constraints. I think that could be a lot more efficient, because I think we could do it such that there's a lot less work than what's currently going on with disequality. I'm not sure if that makes sense.

**Dan:** I think it makes sense, I'm not sure that it will be that necessary at this stage, but I can't figure that out right now.

**David:** It's very similar, conceptually, to the kinds of things you guys want to do with `presento` and `absento`, which are very much like pushing constraints into a tree. I'd like disequality to work more in that manner.

**Jason:** Tell me more about the relationship between this "extended disequality" constraint and `absento`. Could I get something that works like `absento` out of that if I just have the same term in the first position?

**David:** The way `absento` works is that it's basically a simple push-down constraint. `absento` is better here. You just push down a constraint looking for a case where no term is this value that you're looking for. Disequality can be done similarly: all you're looking for is the moment at which you know that the two terms are not equal. As soon as that disequality is satisfied, you don't care about any of the other constraints that were placed on any other terms. So there needs to be some way to communicate to the other constraints that were applied elsewhere.

We've been thinking about that because it could potentially involve less work.

**Dan:** Don't you just go back to a past state in that case?

**David:** No, what we would do is that the moment something is not equal (and you know the two terms can never be made equal), these sub-constraints know what the root is, and they tell the root "We're done." And the root says: "Whoever else was trying to apply my constraint, you're also done." So all these constraint just disappear. Though this does mean that reification is more complicated. This may complicate reificiation.

**Jason:** We do something similar in this cycle model that we've been playing around with.

**David:** So [Dan] has talked about this. I have to sit down and really understand what you guys have done. I have a sense that we're converging on the same ideas here.

**Jason:** That might be the case.

**David:** I'll have to look into it.

**Jason:** Likewise.

**Will:** One thing that we can talk about is future topics for hangouts. David said he'd be willing to do a two-part implementation of core.logic. I've been trying to get Claire to do a debugging session. And Nada to do something on the meta-interpreters. That was a big hit at the confo. Maybe Ryan could do a reprise of his introduction to constraint programming. Maybe do a one/two/three part intro to miniKanren and either going through The Reasoned Schemer or a similar progression, or going through the mythical "second edition" which doesn't actually exist but kinda exists in our heads. We could figure out a way to do it. Just to give people a way to start.

One thing to do is a schedule, sign-up, or long term plan. A seminar or reading group where we have different topics.

**Jason:** One of the things I always found neat in miniKanren are the cool little toy examples you show off. I saw somebody do in core.logic a crossword puzzle solver. I'd love seeing a toy usage of miniKanren every so often. Those are always cool and hopefully don't take too much time.

**Dan:** It might be interesting for us to send David the code. He can ask us questions and everyone can learn how our implementation works.

**David:** Instead of talking about core.logic, which is a lot of code, it would be better for me to go over the new miniKanren and learn from that.

**Dan:** Do you want to do that in real time?

**David:** Not today. I have to read the code and figure it out. And then we can talk about it and come to an understanding together.

**Dan:** There's only two or three places to look at to get the gist of what's going on.

**Will:** Since these hangouts are on Youtube, my preference is to have a little more background material and accessible material so people can watch it online and keep up to speed. Dan, maybe you'd be willing to do one or two hangouts to explain the basic implementation in the updated Reasoned Schemer. Not even with disequality constraints. Before we start getting into fixpoints and constraint solving and all that.

**Dan:** Okay. As long as I'm in the country and Jason's on my left, that'd be just fine.

**Will:** Cool. One of the things that been on our mind has been these `absento` and `presento` constraints. We can talk about those for a minute.

**Jason:** The `absento` and `presento` constraints take two terms where [for `absento`,] the first term can never appear in the second. We'd had various iterations on this idea but the most general one is where it's an arbitrary term in each position. I think we've got `absento` worked out and working pretty well. But `presento` is still on the drawing board. We found some neat things that go on there.

**Will:** Most people don't know what `absento` and `presento` is. Can you start from the beginning?

**Jason:** Sure. `absento` is a constraint that takes two terms and mandates that the first appears nowhere in the second. So the first cannot be equal to the second, it can't be in the head of the second, and it can't be in the tail of the second. It can appear nowhere as a subterm of the second term.

**Will:** Why is that useful? Can you give a use-case for it?

**Jason:** I think for a split environment interpreter it ended up being very useful, like the one you guys were doing for the quines program. Rather than having to write a `not-in-envo` miniKanren relation, you can provide an `absento` constraint.

One of the other things it does--and this is my best way of understanding it--is: sometimes I don't want to see all the ways a first term can't appear in the second. If I wrote a [user-level] miniKanren relation doing that, it would start spitting out the infinitely many ways the first term can't appear in the second. And that's not what I'm after. I want all that collected into a single result, and then move on.

**Will:** Yeah, the cool thing about this constraint is that it's lazy. For that use case with environments, whenever we're dealing with an interpreter (or something like that) we want to be able to say something like "the variable `x` does not appear in the environment". In order to do that previously, we had to do these disequality constraints, and that meant we had to basically `cdr`-down the environment list and therefore instantiate the length of that list. [Doing that] you end up an unbounded number of boring answers that all kinda look the same with these different environments with different disequality constraints pushed in. The idea of `absento` is that by being lazy, you can say `(absento 'x env)` where `env` is a fresh variable representing the environment, and that's it. You don't have to do any more work. So that's the cool part of it. Currently, we're using it both with variables and Scheme numbers in the environment for the CESK interpreter, which I'll show later. I currently don't have any use-cases for general terms [as the first argument to `absento`], but it does seem like if you're gonna relax the restrictions of the first term in the `absento` constraint, you may as well go the full way. But it's not really clear when that would be necessary.

**Jason:** I don't know why yet, but for some reason it feels right.

`presento` is sorta the mirror of `absento`. It mandates that the first  term appear as a subterm in the second. Will's come up with an idea where that might end up being useful. But I think your use case still has that number/symbol/var restriction [on the first term].

**Will:** Yeah, we're either doing `(absento <symbol> <term>)` or `(absento <number> <term>)` or `(absento x <term>)` where `x` is a fresh variable representing either a symbol or number.

**Jason:** And with `presento` it's the same, right? Your use case would either be a number or a symbol or something that represents a number or symbol?

**Will:** Maybe. `presento` is much harder to figure out. Every time I talk to anyone about it, I become convinced that we probably don't want it, because it's gonna do too much damage to the system.

**Jason:** I want to hear about that.

**Jamie:** Is it the case that when you're doing a type judgement, if you have a variable rule using `presento`. Like you have that "x has some type `T` has to be present in this environment" `(presento '(,x ,T) env)` . And even if you might know what variable you have, it would do some unification for you and now `T` is matched up with this type in the environment, and now you actually know something.

**Jason:** Yeah, you know it's in there before you go to look it up.

**Will:** Yeah, once again the idea is you're lazy.

**Jamie:** Well, no. You *both* determine that it's in there *and* looking it up, because you have a variable in the position that the type was. And you just said it appears somewhere in that term. I don't know the semantics of `presento`, but it sounds like it would do some unification there.

**Jason:** It works kinda similar to unification. If you're guaranteed the second term is not a pair, it acts just like unification. But if it's an environment, maybe it has no special structure as miniKanren knows it. This could, for all we know, appear first, second...

**Jamie:** Right. There are more ways for "`x` has type T" to appear in an environment than what you've found. Because there could be some other x with type T (unless you've already got uniqueness baked in somewhere).

**Jason:** Yeah, or unless you've done something really nice to tighten it down to make sure this [environment] has to be an alist.

**Will:** Yeah, it seems really hard to use `presento` without also having some notion of shape constraints (which kinda have the feeling of types, but aren't exactly the same thing). Even then, every time I come up with a use-case for it, it seems like there's some way to bust it or it's way too easy to break it. David came up with a number of really tricky cases in Portland. Maybe he's willing share a couple.

**David:** Yeah. The one to be afraid of is: you have a list of three things, you use `presento` four times on that list, all four things are distinct, and all four things are symbols. That should fail.

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

**Jason:** The half-baked implementation I've got in mine should be able to handle that. Did Dan mention the one found? It's fairly nasty as well. I think I've gotten around yours. This one I'm not certain:

If I have a bunch of `presento`s on all fresh variables that makes a cycle, then all those variables are same thing. They end up unifying, right? If `a` is present in `b`, `b` is present in `c`, `c` is present in `a`, then `a` is `b` is `c`.

```scheme
> (run 1 (a b c)
    (presento a b)
    (presento b c)
    (presento c a))
'((_.0 _.0 _.0))
```

So if I've got for example `(presento a b)`, `(presento b c)`, `(absento d a)`, `(presento e a)`, etc., and if I do another `presento`, then in some fashion I end up with very spooky action that causes failure in a way that's difficult to find. It seems like you need some graph search algorithms on the constraints to find that failure. I may have lost some folk with that one. It almost needs a whiteboard.

**David:** It might be useful to put these examples in the mailing list for people to try.

**Will:** Jason, were you able to get that hard case where it's list-instantiated? The one where you have a list of length three with four `presento`s and disequality constraints.

**Jason:** Yeah, that's one you've got to take care of.

**Will:** I think there are probably many along those lines that you'd have to be careful of.

**Jason:** A little reflection makes it look hairy. I'm with you on that.

**Will:** Maybe there's a way to figure it out. I don't want to give up on it yet. It has this feeling. Maybe the way to think about it is in the simplest possible context. Maybe  consider [the interaction between `presento` and] just unification, see if that's helpful. Seems potentially tricky.

Should we talk about the miniKanren confo and Clojure West? Clojure West was three days, and the second night was the miniKanren confo, which was the first time ever. It started out as a joke tweet by Fogus. Some people thought it was exciting and not just a joke. They encouraged him to try it. He talked with Alex Miller and they decided to actually put it on. They were worried if they could get 20 people to show up, but there were 140 people that showed up. Surprising to me how many people were interested. The room was packed. There were recordings of all the talks, hopefully they should be online soon. There were some really interesting talks there. Really cool stuff going on. Nada gave a really interesting talk on nominal logic programming, but at the end was showing off how to use different meta-interpreters to get different behavior. Like if you write a type inferencer, you can use this meta-interpreter that's running on top of the type inferencer to generate derivation trees in LaTeX, found counterexamples, and give you decent typechecking error messages. All sorts of stuff like that. It was kinda like a PLT Redex type thing using meta-interpreters. That was neat. It made me want more to sit down with Robby [Findler] and show him that stuff. Maybe see if there's a way to combine forces somehow with these tools.

**Dan:** I found it really interesting that Nada's metainterpreter required the use of `conda`. Any comments about that Will?

**Will:** What I would be interested in is a meta-metainterpreter where the metainterpreter is relational. What would happen if you did that? I don't know. It seems like there are some interesting games to play there. The interesting thing to me was that the metainterpreter works because the interpreter itself is relational. If the interpreter itself used `conda`, it seems like these games don't work as well. So it's both an argument for having `conda` and an argument for writing relational programs. Both worlds going on, which I thought was really neat.

Also, if you were able to go meta-meta- level and had your metainterpreter be relational (maybe through extra constraints, or something) maybe you'll end up with something like a "tower of interpreters". Seems like there's lots to play with. Obviously, Prolog people have been playing with it for a long time, but I think we might have some new tricks, or approaches, that might pay off with that.

The stuff on typed logic programming that Adam Folzer showed off was really neat. I'm still trying to wrap my head around that typed logic programming. I don't know exactly what that means. Getting a type error during compilation versus failing during runtime, it seems like there are some subtle issues. He was understanding my point at the end which was that you might want a way for the compiler to realize that your program is going to fail, but not necessarily give a type error for that, but instead just you would get back the empty list at compile time instead of run time when there are no answers to your query. I think Adam saw that as a staged evaluation approach. There's probably something to that. Also there are things like gradual typing, but I don't know exactly what that would mean in the context of logic programming. There's been some work on that, but that's an area we haven't really explored, the types aspect. I thought that was really cool.

Were there any talks or ideas you all found interesting?

**Dan:** The last morning [of Clojure West] someone was showing a modern-day Forth, called Factor. It looked very fascinating. I've always been a big fan of Forth as it is. I'm looking forward to having some spare moments to look into how Factor works, what's involved, and why it's modern-day. It's a great place to build a system from the ground up. You're right on top of the machine when you're in Forth or Factor.

**Will:** The cool thing about the Forth interpreter (or any concatenative language) is that you can stream in the program a little bit at a time. That would be really interesting to run in miniKanren. Also, if you had something like `presento` (or perhaps with difference lists), you might somehow be able to say: here are different snippets of the program that appear somewhere in the input expression and fill in the rest of it. PostScript is basically like Forth with graphics operators. You might be able to do something like run backwards: here is a picture, now generate the program. Or here's part of a picture and part of the code, now fill things in. There could be some cool games to play with a relational concatenative interpreter.

Ryan gave a really nice talk on constraint logic programming: an introduction with a number of puzzles. It was a very nice explanation.

David gave a talk about the core.logic extensions for constraint programming (like cKanren-meets-core.logic). That was really neat.

The first talk, Gabe's talk, was about ideas for a different style of logic programming language that had higher order features and things like that. And that's something that we really haven't played with. Something like lambdaProlog. That was interesting. Were there any other talks?

**Ryan:** Just your own.

**Will:** Right. From Nada's talk, the nominal logic stuff may have seemed kinda weird, but certainly people who realized what was going on, when they saw the metainterpreter stuff, that was really neat.

Here's one of the problems that the metainterpreter solves. And these techniques have been known for a long time in the Prolog world. The relational interpreter and inferencer stuff we've been doing works really well with it. One of the problems is that if you write a relational type inferencer and you enter an expression that doesn't typecheck (it could be the wrong type, syntactically invalid, or whatever) you just get back the empty list saying that there is no answer. That could be very frustrating, especially if you make a syntactic error. You don't know what went wrong.

With the meta-interpreter stuff, you can see exactly where things went wrong. You can see the exact derivation tree. So for debugging, it's phenomenal. And for getting type inference error messages back, it's extremely useful. So that's definitely a technique we can push really far.

I've been thinking a fair amount about Austin's question from last time about how do you debug these programs. I gave a kinda sassy answer: "You just make it mathematically correct." That's kinda a lie. You do want to do that, but even if you have the math-y rules, it's still really easy to make mistakes with these programs, like syntactic mistakes. For example, I created an interpreter last week for the CESK machine based on Matt Might's blog post, and it only accepts its input language in a special form called A-normal form. I was trying to generate quines and it wouldn't work. So then I tried accepting quines, and it still wouldn't work. Where's the bug in my program? There wasn't a bug in my program, it was [failing] because you can't have nested lists in A-normal form. It doesn't work; it's not part of the input language. It was failing correctly, but it was really difficult for me to understand why it was failing. It was difficult to distinguish the correct failure versus a bug in my program. The metainterpreter stuff could really help when writing these kinds of programs. And there are a bunch of other techniques. Claire has a bunch of techniques she uses. She helped us with this last weekend. I think between Claire's stuff and Nada's stuff, that could really help with the debugging.

**Ryan:** I had a question for David about why we use integers instead of longs as intervals in core.logic. I did a bit of looking on that yesterday. I think one of the reasons you might have picked integers as the common operation for intervals is that in the plus operator, for example, the ranges of the two values that you're adding actually get added together before if the domain actually needs to get updated. So if you use an integer with that, Clojure auto-promotes that to long, and you're always within the bounds of a long when you add two integers.

**David:** You're right. Now that you're saying this, you're reminding me why.

**Ryan:** So I've got a very simple core.logic program 1 + [1, Long.MAX_VALUE] = 2. What it does is it tries to add the domain of 1 to the domain of [1, Long.MAX_VALUE] and of course rolls the long, even though that value would never take effect, eventually. So it'll roll the long, throw an integer overflow error from Java, and then just dies. So I was thinking through possible ways to fix that and I just wanted to get your opinion on it.

**David:** This was a design decision I made a while ago. IIRC, at one point I experimented with just using BigInts everywhere, and the performance was horrible.

**Ryan:** Yeah, that seems like it would be bad.

**David:** I think I picked that size because it's a large enough range for most operations, and you don't run into this issue of two very large domains getting added together initially. I actually don't have a good solution. I played around with it for a while and then just said it's just gonna be [hacked?] in.

**Ryan:** Here's the code I was talking about. [Pastes in Hangout chat]. If you run something like that, it'll fail badly. I think there are several things you could do. The adding up of those ranges could be delayed, like in this trivial example, there's no actual need to compute that high. It should be constrained before it tries to do that big arithmetic. Or there could be some range checking before adding things together.

**David:** It's true. Something like that's just gonna require a lot of changes, especially to figure out where exactly we can make those operations if we delay them. I think that's why I punted.

**Ryan:** This was the kind of thing that bit me earlier about trying to use that. I was trying to come up with a use-case that presents that bug. That's all I had.

**Will:** I forgot about the logo contest. If you go to miniKanren's Twitter, it's just that egg. We don't have a logo. I've designed a logo or two, some people liked it, some people didn't. Cameron Swords submitted a couple things. We had a very short logo contest at the confo. I was waiting for someone to sabotage it. If people have ideas for a logo, that would be great.

**Dan:** How about some sort of Klein bottle.

**Jason:** A Quine bottle?

**Will:** A Quine bottle, I like. I don't know why a Klein bottle would work, but a Quine bottle would be good.

**Dan:** Just throwing it out.

**Will:** I have a really cute example involving nominal logic programming relating to classic rock. Michael Adams might use it in a paper. It's an joke along the lines of our `'no-closure` and `'no-clojure` tags, but much funnier. I shouldn't divulge it unless Michael doesn't use it.

I might follow through and just create a logo. We could do something like Ken Shan's email signature. Everyone will be embarrassed and then I'd have to fix it.

**Dan:** That's why there weren't any t-shirts [at the confo].

**Will:** That's right.

**Joe:** I've got a question. I'm implementing a version of miniKanren in Haskell and I'm curious what the easiest way to implement the disequality constraint is. Just to say this should never unify with that. What would be a resource to go look that up? Where would be a good place to start?

**David:** Have you looked at Will's dissertation?

**Joe:** I think so. I hadn't dug into it too deeply. I haven't gotten close to that point yet. Just up to the point to start implementing things.

**David:** I think the implementation of disequality in Will's dissertation is pretty easy to follow compared to tabling or alphaKanren. It's like the easiest extension to add to your system. I would look there first to see how they did it.

**Joe:** I will take a look at it again, thanks.

**Will:** I can give you the intuition. In fact, I was talking to Dan and Jason today about how all of this is a really neat trick we use. It causes problem if you aren't building a prefix to the substitution while you are unifying, but it works really well with list representations of substitutions being returned from unification. It works less well if you're doing, say, explicit trees.

Anyway, the idea is that disequality constraints are, in a real sense, the dual of unification. They're both tree constraints. `==` and `=/=` are actually deeply related. They're both types of constraint logic programming. They were both in the original version of Prolog, I think. Certainly, an early version of Prolog had disequality constraints. They're deeply related. There's a paper by [Uber Kammon?] that talks about the disequality constraint stuff. I figured out (sorta out of desperation) a way to do this which turned out to be exactly what was described in the paper, in a way. So this stuff's been known for a long time.

The idea is: If you're unifying two terms in a substitution that's represented as an association list, there are three possible outcomes:

1. If you're unifying something like `(== 5 5)` or `(== x x)`, the unification will succeed without extending the substitution any more; there are no more associations which need to be added. You get back the same substitution you started with.
2. The unification fails. Like if you try to unify `(== 5 6)`, or `(== x y)` when `x` and `y` are bound to different values. Or `(== x '(,x))`, which violates the occurs-check. You get back failures.
3. Unification succeeds and extends the substitution. That's something like unifying `(== x 6)` in a substitution that didn't have a binding for `x`. Now you get back that substitution, extended with a binding between `x` and 6.

So those are the three cases: unification succeeds without extending the substitution, unification fails, or unification succeeds *and* it extends the substitution. And when unification succeeds and extends the substitution, you can imagine the new associations being added on to the list representing the substitution. So if you're using an association list, `((x . 6))` would be the prefix of the new substitution that you got back. It's what's added onto the old substitution.

If you think of unification that way (which is how we implemented it in The Reasoned Schemer) then you can think of the disequality constraint in terms of those three cases. So if you have two terms `t1` and `t2`, and you want to know if they're disequal in the current substitution `s`, what you can do is just unify them in the substitution and look what you get back from the unification. So you can implement disunification with unification: you unify the two terms and once again you have those three cases:

1. If the unification succeeds without extending the substitution, that means the two terms are already equal. That means they are 5 and 5, or `x` and `x`. Or `x` and `y` where `x` and `y` are bound to the same term. Those are simple examples, obviously. But if they're already equal, you fail, because they can't be equal. So if unification succeeds and you get the same substitution back, then the disequality constraint fails.
2. If the unification itself fails, that is, if you try to unify 5 and 6, or `x` and `y` when `x` and `y` are different values, that means that not only isn't the constraint violated, it could never be violated! There is no extension to the substitution which could violate that constraint. So you can throw away that disequality constraint, because you know it will never be violated.
3. The third case is when the unification succeeds, but extends the substitution. Like `(== x 5)` unified on an empty substitution. Now you get back a new substitution where `((x . 5))` is now the prefix on top of the old substitution.

**Joe:** And you can just move the disequality constraint forward a bit?

**Will:** Yes. This means two things. First, it means you didn't violate the disequality constraint. Second, it means `((x . 5))` (or more generally, the prefix could be multiple associations, like `((x . 5) (y . 6))`) that prefix represents a set of disequality constraints that are related which is, in fact, the new "normalized" constraint. You use it to normalize the constraints. That's what's described in my dissertation. That's a clean way to implement disequality just using unification. But you have to have a way to build this prefix. That's what Dan and Jason were running into problems with when we tried a different representation of substitutions. If you're using trees, or something like that, you have to explicitly build the prefix or you have to find some other way to get the prefix, which is potentially extra work.

**Joe:** Right. I think in my case the method you're describing might be a bit easier, because I represent my terms as graphs. It's difficult to describe, but basically the substitution happens immediately on the terms, because they're represented underneath by references. So if I say "x is now bound to y", then boom, it's done and I don't have to worry about it anymore. I think if I can just use unification, that's easy. That clears up my intuition a lot, thanks.

**Will:** Sure. I'd be interested in seeing that. Are you doing the occur check also with unification?

**Joe:** No. That's the main reason why I'm using graphs. I don't need the occurs check; I just allow the circularity. If I end up in a case where I'd normally fail via the occurs check, I just end up with a cycle in the graph. The disadvantage is that my unification algorithm is probably more expensive, because I always have to check if I've travelled back to myself. But when I do unification, you take two graphs, and you walk them, and you zip them together. You end up with a unified graph that has your terms. It works pretty well. I have no idea what the implications are other than that you can unify infinite terms.

**Will:** Unifying infinite terms is a fine thing to do as long as you decide that infinite terms are part of your language. Later versions of Prolog allowed this. I think Prolog II allowed this. Certainly Prolog III allowed it. The tricky part is: if you're not failing when you have a cycle, then even if you handle reification properly, you might run into unsoundness when it comes to the things where you normally want to have finite terms. For example, type inference for the Simply Typed Lambda Calculus is the biggest example we run into: self-application is not supposed to typecheck.

**Joe:** It depends on what you want. The problem is that in the lambda calculus, everything typechecks. You can't generate a non-typeable term, right? If you use the circular types. If you take just the regular lambda calculus (not simply-typed) you can typecheck everything with circular types. You can't make a term that will not typecheck.

**Will:** Well, in particular I'm talking about the simply typed lambda calculus. In STLC, self-application doesn't typecheck, which is why it's strongly normalizing. If you were to write a type inferencer for STLC and you wanted that self-application to fail, you'd probably need some other notion of unification or the occur check. In early versions of miniKanren we actually had two unifications: one that did occur check and one that didn't. Most Prologs don't do the occur check because it's expensive. You do have to be careful, because if you're restricting yourself to terms that are finitary, then unification is actually unsound if you don't do the occur check.

**Joe:** What do you mean by unsound? In the case of self-application you'd just end up with a function which yields a function which yields a function and so forth.

**Will:** This is the big question: whether or not you want to allow those terms. If you want to simulate or model a system where that's not allowed, I don't think you could even do it in your system. I suspect you wouldn't be able to encode STLC rules in your system as it's currently written.

**Joe:** With what I'm playing with, there are a lot of cases where I want the infinite types. What I was planning on doing is handle the infinite types (because I want them for certain cases) and then have another case that only allows them in cases where I've explicitly said they should be in those points. But yeah, I should see if that's feasible.

**Will:** Then you may want to do what we did and have two versions of unification where one does the occur check in the case you want to restrict trees to being finitary, and another version that doesn't do the occur check in general. But you have to be careful when you're using those two variants that you're not writing unsound things.

**David:** I have a Haskell question for Joe. One thing they do in miniKanren for the prefix is they rely on the fact that you can do a pointer equality check on the `cdr` of lists. If you build up a list and call the `cdr`, then because of structural sharing, you can actually do a [constant time] test that tests for the equality between lists. But this is memory equality. Is that something that Haskell allows you to do?

**Joe:** It depends. Generally, no. Generally, equality is done on values. But the system I have is built on top of a monad which holds references. There are a couple reference monads in Haskell, like `IORef` and `STRef` (which is like an `IORef` that is safe to do in pure code). I implemented my own references on top of the `State` monad, which gives me branching for free. So all the interleaving stuff that happens, like all the backtracking, is transparent; I don't have to worry about it. In that part, each of my references (which would be ever logic variable, everything you could freshen) has an integer associated with it; its identifier. And then I can just compare those.

**David:** Cool.

**Will:** Neat. And there are two other Haskell implementations. Adam Folzer is showing off his Molog, which changes every week. Actually, there's this really old Haskell implementation which Amr [Sabry], Ken Chan, and Oleg were hacking on.

**Jason:** Will, is Adam's Haskell version of miniKanren on minikanren.org? I don't think I saw it.

**Will:** I don't think it is. I need to add it. Sounds like its under flux, but I need to add that. I'm happy to add any other implementations people have. Send me links and I'll add it, or feel free to add it yourself, because the page is in GitHub. You can edit and send a pull request and I'll update the site.

**Austin:** Real quick, I can give a demo of this debugging tool I've been working on. The basic idea is that I can see what's going on inside the search as it's happening and inject hooks to either print diagnostic information, or trigger an error if I detect something is wrong. I'm using some Clojure facilities here to swap out the root definition of a goal with the instrumented version. It's pretty easy to call a function when the goal is first entered, but it's very tricky if you want to check how that goal changed the current substitution, or the state of the running system. But I think I have that working and I'll just give a quick demo.

So we're tracing `membero`, which just means that we're adding it to our system. It doesn't do anything except log it during the search. Here, we add in these hooks that just println with the information that's coming to our `debug-in` hook and our `debug-out` hook. And it'll look like this, just the raw data. But I can do that and get a trace with better formatting. You can see that this is the execution of `membero`. Here at the top that `x` took on this logic variable and `l` became a vector of 1 and 2 (because we're doing `(membero q [1 2])`. Then it goes here to the inner `cons`, and you can see that we have the same logic variable here as we did in the `x`, `d` takes on a new logic variable, and we get that `a` is restricted to 1, and `d` is restricted to the rest of that list. Anyway, you get the idea.

Where I'm at now is trying to make it useful. With this display for these tiny examples, it's easy to follow. But for something bigger, it's a mess. Especially when you get to these which are the same depth. This example isn't a good representation of this graph which is constantly being interleaved and you're jumping around to different branches. I think there's much better ways for that [situation] to be presented. But I think I've gotten to a point where I can spit out that data, I just need a better tool to display the graph.

A more interesting application here, instead of printing out the output, I'm gonna check if all the variables to a relation are unground when it is first called with that substitution. So instead of a println, I warn about unground args. When I run that, I get a Java error that all args to `flatteno` are fresh. You can see that it's basically like a stack trace from that run. Now I need to figure out how to make this useful, what other extensions would be useful. Really, I just need more experience debugging [relational] programs to see what is the view that you'd want to see. how easily can it be navigated, or can better information be printed about how the substitution is being constrained throughout the search.

**Jason:** That looks like a really cool tool. One thing that might be useful: when I look at these, sometimes I don't want to go the full depth. Maybe specify "after depth 5, I don't want to see it." I know there's something like that for Emacs parens, and I find that sometimes useful, too. Just to see the high level operation.

**Austin:** That's a good idea, too. And when I'm doing the stack trace and working back up, it might also be useful to restrict that as well.

**Will:** Claire, how does this compare to the way you debug things?

**Claire:** The debugger that I have basically just projects all the variables when you come into the goal. It doesn't have any information about the substitution when you come out of a goal. I haven't thought about doing that yet. Someone pointed out on IRC that it would be awesome to output something graphical. Something so that you could actually visualize the output of the program without the ASCII lines and stuff like that. It would be nice if you could do that. I was thinking about doing that in Racket with the cKanren version in Racket. Looks good.

**Will:** There's been a ton of work on trying to debug Prolog programs. A Prolog program using depth-first search is a lot easier to figure out what's going on than interleaving search. There are all these models. There's the [Byrd Box model](https://eu.swi-prolog.org/pldoc/man?section=byrd-box-model) named after Don Byrd, also at Indiana. It has these boxes with arrows coming in and out.

There are a ton of approaches to debug Prolog programs. There's a lot of literature we could look at. Obviously, things that are based explicitly on the WAM may not be applicable. But a lot of the models might be useful. Also the metainterpreter stuff. Although with the metainterpreter stuff, you probably have to have a program that's fairly close to working and which is purely relational. It would be potentially helpful to have some notion of breakpoints or "you've reached a certain depth" or stuff like that. One weakness for our tools right now is we don't have good debugging support.

**Unknown 2:** Some kind of gdb-style: "set a breakpoint here and explore the values of things where they're at currently" would be really helpful, especially for people just trying to figure out the basics. It's okay if the debugging tools are only good on really simple programs, because that's good for people trying to understand how simple programs work.

**Ryan:** I've found SWI-Prolog's debugger very helpful.

**Will:** What did you find helpful about that? How does it work?

**Ryan:** It was graphical based. It allowed me to step through the program while watching a logic variable as it went through all the various unifications and other steps. And for concise programs that do a lot, I found it to be interesting to see what was actually going on as opposed to just the declarative representation.

**Will:** It would be cool if people who have played around with these Prolog debugging tools would be willing to give some demos. If you'd be willing to give a demo of that in one of the hangouts, it would be interesting to see how that works in practice. I haven't used those things.

**Ryan:** Sure, although I'm no expert. I'm sure it's getting pretty bad if I have to resort to a Prolog debugger to figure out what's going on. But I'm willing to show something.

**Will:** David, how do you debug core.logic programs?

**David:** Debugging core.logic programs is very frustrating. I don't know if I have a good solution. Part of the thing is I'm not afraid of the internals at all. My debugging is very primitive. I know exactly the layout of the substitution and I'll just stick in a print goal, and I'll print out the parts of the substitution that I care about. It's very primitive, but so far that's worked out well for me. In the early days, I did use something more similar to what Austin was showing. When I was starting out, that view was useful to me. But I haven't needed something like that recently. Still, having some step-by-step debugging tool is useful for newcomers to relational programming.

**Will:** The biggest technique I use is just writing relations a little bit at a time. Ideally, one relation at a time, one clause at a time, and then adding a test for each clause. Another trick I like to do is run the relation with all arguments fresh: If I see that a clause isn't generating answers then I know something is seriously wrong. That's a crude technique I use. That can be really helpful when working from a specification or program I already have. But it didn't really help me in the example where my program was failing because it wasn't accepting programs in A-normal form. That was something outside just the translation [from functional to relational]. You had to have an understanding of what the program was trying to do. Basically, what I did was I took the input program and I kept breaking it down into smaller and smaller pieces until I had the smallest piece that didn't work anymore. Then I realized, "Oh yeah, that's a nested list. Why is the nested list not working? Oh yeah, because it's not in A-normal form." That [debugging] took me much, much longer than it should have. That's an example where a tool or a better technique would've helped me. Anyone else have any general miniKanren debugging tools or techniques that they want to share? It would be good to have this as a full topic at some point.

**Joe:** There might already be something out there for this, but one of the things I've wanted to use my implementation for is quick-and-dirty type inference, but I also want to be able to produce useful error messages. Right now, I don't have any way of showing the simplified piece that's failing, instead it just says its unable to type your program. Having these kind of debug tools could be used programmatically to spit out more localized error messages, potentially.

**Will:** Hopefully we can get Nada to talk about the metainterpreter stuff. We can try to get a better sense of when the metainterpreter is the appropriate tool versus one of these standard debugging techniques. Once you have the inferencer working, it seems like the metainterpreter is the way to go. It is really flexible and you can use it to do all sorts of cool things, but I'm not sure it's the best tool when you're still developing the inferencer and trying to get it to work correctly.

**Joe:** I don't know much about the metainterpreter. What does it do?

**David:** I can explain. I once wrote on StackOverflow (because I love Lisp) that Lisp was the best language for doing metaprogramming. And someone who was into Prolog was like "That's absurd. Prolog is great for doing metaprogramming." I didn't believe him, but then I started reading Prolog books. Prolog is an extremely uniform term language. Probably the only uniform language that's not Lisp. So it's trivial to write Prolog metainterpreters. A Lisp metainterpreter is quite large; the smallest Prolog metainterpreter is 4 lines of Prolog. It's very short. Once you understand how it works, you can take your programs and do just a surface-level change to them, and you can get a meta-version of your Prolog program.

**Joe:** The piece I was having the most problems with was type inference. It doesn't [beta out?].  For some things, I want it to try multiple branches and exhaustively search, but sometimes I want it to commit such that if it fails past that point it errors with "x doesn't unify with y. You're stuck with that. Fix your program." I don't see how [metainterpretation] helps that problem, but I know nothing about metainterpreters yet.

**David:** It's basically one step [of indirection] so that your goals are interpreted at the meta level. miniKanren has very few syntactical constructs: only `fresh`, `conde`, and `run`. So the metainterpreter says, instead of executing these directly, we use the relational system to run these things with one step of indirection. I can paste a link to Nada Amin's repo. She has an example in there. [www.github.com/namin/minikanren-confo/](www.github.com/namin/minikanren-confo/).

**Will:** My understanding of what Nada is doing is that you can basically wrap every call to `==` (all the unifications in your system) so that the metainterpreter will keep track of all the failures in your system. So if you're doing a type inferencer it'll know where things failed. And then you can use rules to find the deepest part in the tree where it failed. Or if you have some special information about where failure should or shouldn't happen, then it can look for that sort of thing. If you know some properties of the system you're modeling, it turns out to be really helpful. In her case, she was able to show how to generate not only good error messages, but also counterexamples (progress, preservation, proofs, derivation trees in LaTeX). She generated her slides using the metainterpreter itself. The cool thing is it was the same relational type inferencer for everything. You never change the type inferencer at all. And I think it was the same metainterpreter. She would change just one thing in the metainterpreter and get all this different behavior. It's an extremely powerful technique. I'd like to learn how it works. The metainterpreter isn't relational. One question is could it be made relational. It has this "higher order" feature, so maybe you'd need to go to something like lambdaProlog. Seems like there's a lot there we could play with. Metainterpretation is also described in The Art of Prolog, a classic text on Prolog programming.

Any other debugging advice?

**Ryan:** I echo David's suggestions. I frequently log goals in core.logic. If need be, I get into the plumbing and print stuff out there, too. Which is not much different to how I debug regular Clojure programs.

**Will:** I guess the big difference to me between debugging Scheme programs versus miniKanren programs is that `printf` doesn't seem very useful, because of the interleaved search. You're just printing out the things in whatever order the interleaving stuff happens.

The other interesting topic is testing miniKanren or core.logic programs, because that gets tricky. Sometimes you know your programs will diverge because of the ordering in a conjunction. Sometimes you'll make a little change in your program, and you get back the same answers, but they come back in a different order. Over the years, we've written a bunch of macros to handle all that stuff, but I don't think they're online anywhere. We should share some of those and see what tricks other people use. It's frustrating. The constraints make it even worse. You'll have the terms coming back, but now they'll have different constraints. But even just the ordering is tricky. You *could* have a set version that does set-equality on the answers, but if there are an unbounded number of answers, you can't do that. If you do a `run 10` you get a subset of the answers back. So you'd have to have a test macro that asserted these 10 answers should exist, so look for them, but they may not be the first 10 answers if you've reordered things. That also gets tricky. Did you run into that at all in core.logic, David? Do you have any special testing techniques?

**David:** We just put everything into sets when we can--when we know there's a finite set of answers. For the infinite case, there's not much you can do.

**Will:** The only thing you can do in that case is assert you know there's an unbounded set and that these elements should be in the set. So you can deal with it that way. But also if you really reorder things, it might take a really long time to generate answers. There's a notion of how things affect performance when you reorder things, and that could be tricky to figure out.

**Jason:** Will, one other thing you could do if you have an infinite set of answers is just look for success and failure. Pick a couple of representative constituents and just make sure this succeeds, this succeeds, and this succeeds.

**Will:** You can definitely do that, but that doesn't tell you if you're pruning the search tree aggressively enough to generate the answers. It's a lot easier to test to see if an answer is accepted than it is to generate and test an answer. That's always the thing I do when debugging: if I know there's an answer that should exist, I feed that into the system and it better succeed. If it fails, then I know something is wrong. That does become trickier when you're trying to figure out why reordering a program which used to finish immediately now takes a trillion years.

I can show the CESK machine. Should I show the trick we're using for performance, or just some examples?

**Dan:** I always like performance.

**Will:** Here is a Scheme version of the CESK machine. We did this in the hotel the day before the confo. It is a higher-order representation of an interpreter. If you're familiar with how to write interpreters in the style of Essentials of Programming Languages (Dan's book), or in C311 (the course at IU), this is similar to that.

This is the evaluator. The idea is you take an expression `exp`, an environment `env`, a store `s`, and a continuation `k`. Then it's going to produce some answer evaluating the expression in the environment and that store with respect to that continuation.

Normally, you can think of an interpreter as just taking an expression and giving back a value. The interpreter that we used to generate quines in the Scheme workshop paper takes an explicit environment (so you can say, like, `x` is bound to 5 in the environment).

This version also takes a store, so it allows you to easily represent side effects, like `set!`. You might say that instead of `x` being directly bound to 5, you can say that `x` is bound to a location in the store, an address. And that address is, in turn, bound to 5. So it's an extra level of indirection. We can do side effects like `(set! x 7)` by extending the substitution to have that address bound to a new value.

A continuation is a representation of the work that the interpreter still has to do to evaluate the entire program. Thinking about continuations is something that comes up a lot in Scheme, because Scheme has first class continuations. But it's less common to think about continuations in the world of Clojure. When we were talking about continuation stuff, some people in the confo were a little confused by that. You can think of it as a representation of the stack in the heap where you represent the stack as functions. In this case, higher order fuctions--actual procedures.

So those are the four arguments coming in. Normally the way you'd run it is you'd take your expression with an empty environment (in our case it might be the empty list, as is the store). And the initial value for the continuation is just the identity function--a function which just takes a value and returns it. So that's how you run this thing, normally.

We're doing pattern matching on the expression into different cases. We might have a number for example. If you have a number, normally you just return it. But since we have a continuation argument we have to apply the continuation. And furthermore, we don't just have a number as the final answer, we have this `answer` constructor which takes both the value and the current store. We have to pass the store back up when we're giving answers.

So that's the main difference between this and a regular interpreter. You can think of this as a regular environment-passing, store-passing interpreter which has been rewritten in continuation-passing style. If you want to know more about that, there's plenty in Dan's Essentials of Programming Languages book. If people are interested in that, we can talk about it at some point.

Anyway, this is just an interpreter for the lambda calculus. We've got `lambda`, we've got variables, we've got procedure application down here, we also have `set!` for setting a variable to a value, and we have `begin` (which turns out to not really be necessary; you can sequence expressions just using `lambda` and application). And that's the whole thing.

It looks kinda weird. You can see it's [indenting a lot] to the right with all these lambdas. These lambdas are actually continuations. Any lambda with a parameter like `v/s^`, `v1/s^`, or `v2/s^^` is a continuation. That represents the rest of the work to be done. For example, in the case `(begin ,rand1 ,rand2)` which is sequencing these two expressions--first you do `rand1` and then you do `rand2`--we call the evaluator `eval-exp` recursively on `rand1` in the environment being passed in with the current store, and that's going to produce a value. That value is passed in here. Remember our answers have two things: a value and a store. So we get back one of these `answer` things `v1/s^` which is a value and a store. And then we just tear it apart here. (Really, we shouldn't be using `car` and `cdr` here; we should be using something more abstract, something like `answer-value` and `answer-store`). Anyway, we're getting the value `v1` and the store `s^`. Then we're evaluating the second argument in the original environment `env` but with the extended store `s^`. The reason we have to do that is in case `rand1` contains a side effect like a `set!`. Because even though it doesn't affect the lexical environment (in terms of which variables are bound), it might affect the values that the variables are bound to.

**Dan:** In looking at the code, it might turn out to be better to make your continuation take two arguments, instead of having to construct/destruct all these answers. I'm not certain of that, but it has that feel to me right now.

**Will:** When you say better, you mean cleaner?

**Dan:** Yes.

**Will:** Sure. That's a style thing. You could do it either way.

Once you evaluate the second expression `rand2` in the `begin`, (our `begin`s only take two expressions) now you get back this new answer `v2/s^^`, which we tear apart into a value `v2` and a new store `s^^`. And now we can construct a final `(answer v2 s^^)` and then apply the continuation to it. And that's just how you do things in continuation-passing style.

This CESK machine is what Matthias Felleisen wrote about in [his dissertation](https://www2.ccs.neu.edu/racket/pubs/dissertation-felleisen.pdf). This came out in 1988, I think.

**Dan:** Sounds about right.

**Will:** The CESK machine is used for a lot of things. Matt Might, who I'm working for at the University of Utah, is using this for doing static analysis of programs. I'm trying to figure out whether or not a relational version of the CESK machine would be useful for Matt's work.

But this is just a Scheme version. We can just run it. This is all on my Github, by the way: [https://github.com/webyrd/relational-cesk](https://github.com/webyrd/relational-cesk).

**Ryan:** Is Matt Might's blog post the best way to learn about the CESK machine?

**Will:** Matt's got a couple blog posts on it. It's a really good place to start. It's not exactly the machine that we're using, but it's very similar in spirit.

**Dan:** You might want to look first at the CEK machine and build a comfort level for that. Then, push up to the CESK machine. Because it's only the S that you're adding, and it's just changing the way you're treating variables: you're saying variables are now associated with addresses and addresses are associated with values such that you have a double-lookup to get to the value. If you understand the CEK machine, then the change is very simple and you only have to think about one change at at time. You're just adding the store to the metaphor.

**Will:** Sure. Depends on how you want to approach it. Matt's got a really nice [blog post](https://matt.might.net/articles/cesk-machines/) about writing an interpreter in a CESK machine style. He's actually got a couple of these. He doesn't really show how to derive it, but he talks about how it works and what it means to have a continuation, and things like that. The biggest difference between his version and ours is that his version accepts versions in A-normal form (which is very similar to continuation-passing style). He accepts only programs in a restricted form to make it easier to write the machine. That's the big difference. He also has some additional forms like `call/cc`. We also have a version with `let/cc` and `throw`. But it might give you some good intuition to look at Matt's blog post.

Anyway, I'm not going to talk about all the ins-and-outs about that machine. It's a very interesting machine, but the main thing that's cool about it is that you get your hands on the continuation itself. It doesn't have to be the empty continuation. These examples all use the empty continuation, the empty environment, and the empty store, but you don't have to do that; you can use any continuation you want. Furthermore, during an evaluation, the environment, store, and continuation won't be empty--they'll be built up over time. That's a neat thing. It's kinda bizarre to look at the answers you get when you play around with this.

Let me briefly show you what you get when you translate this to miniKanren. Here's the miniKanren version of that code: `apply-ko` (which is the continuation application) and `eval-exp-auxo` are the two main relations in this system.

In order to convert it to miniKanren, we had to make the interpreter representation-independent with respect to continuations, environments, and stores. We also had to make everything into a first-order representation. So for example, instead of using Scheme procedures to represent continuations, we have to use lists. That's a multi-step process, but this is stuff that's bread-and-butter for these sorts of program manipulations. We talk about most of those transformations in the Quine paper for the Scheme workshop.

Let's show some examples. Here's where it gets interesting. We're leaving everything fresh: the expression `expr`, the environment `env`, the store `s`, the continuation `k`, and the output value `val` can all be generated. The weird thing is if you look at an answer like this one, the continuation isn't empty. You actually have this `application-inner-k`.

```scheme
(('_.0
 (_.1 _.2)
 (_.3 _.4)
 (application-inner-k
  (closure _.5 '_.0 (_.6 _.7))
  (empty-k)
  _.0)
 _.0)
 (=/= ((_.5 quote)))
 (sym _.5)
 (absento (closure _.0) '_.1 '_.6))
```

It's a continuation which itself includes a closure and then the empty continuation `empty-k`. I'm not going to get into how to read these, because it's kinda complicated. The fact that we can get our hands on the continuation (which you can think of as a first-order representation of the stack) will let us play a bunch of interesting games: we can run the machine from any state during the derivation of the answer, and things like that. So that's really interesting and it'll take a while to figure out.

Besides just its flexibility, the thing that was really tricky is that you have this explicit continuation argument (which is equivalent to rewriting an environment-passing, store-passing interpreter into continuation-passing style). Normally you've got this nice property that all the calls are tail calls (which is very desirable). The problem is when trying to run backwards, like generating fifty programs that evaluate to `6`, or generating quines (where you say the input expression and the output value have to be the same when the environment and store are empty, and the continuation is the initial continuation). Those games only work when your interpreter has a relatively strong connection between the input expression and the output value. What I mean by strong connection is: you want those terms (or subterms) to be unified as quickly as possible. The CPS'd interpreter is actually the worst-case behavior. If you look at `out` [in `eval-exp-auxo`] it gets passed to `apply-ko` or [recursively] to `eval-exp-auxo`, and it's never actually unified with any of the values here. It's never unified with the expression `expr`, for example. And if you look in `apply-ko`, it's same thing, but with the exception that when the continuation is empty, (meaning there's nothing left to be done in the program) then, and only then, do you unify the out value `out` with the input expression `expr`, which at this point is going to be an atomic value.

What I'm trying to explain here is that CPS-ing the interpreter, which is normally a really nice thing to do in "functional land", has this problem that you break the close association between input and output of a [relational] interpreter, which means that the search space is gigantic. If you're trying to do something like generate a program that evaluates to 6, or generate a quine, the system will not be very constraint. It's going to find, somewhere in the tree, some giant expression, potentially, and evaluate it. And only at the very, very end will it find out that the value of that expression is not the value you're looking for, and then fail, backtrack, and try some other giant expression. It can be really slow. What we really want to do is prune the search tree as aggressively as possible. The way you do that most effectively is by having a strong connection between the input expression and the expected output value.

I'd had an idea of how to do this for a while, in particular with small-step interpreters. I was talking to Nada about it for some small-step interpreter stuff she was working on. Kinda out of desperation, on the day of the confo, I really tried to push this technique. It seemed to work out pretty well. I don't completely understand everything about it, but the intuition (which I don't really have time to get into) is that it's related to something called "parsing with derivatives", which Matt has a [blog post](https://matt.might.net/articles/parsing-with-derivatives/) about. It's also related, potentially, to taking a derivative of a data structure, which Conor McBride has papers about. The idea is that we add an extra argument to the relations. So this `v-out` thing represents the expected output at any given point in time, er, the derivative of the output with respect to both the current expression and the current continuation. The idea is `out` is the final answer, so if we evaluate some giant arithmetic expression, `out` might be 12, whereas `v-out` will potentially always be changing as you go through the evaluator. Really, what `v-out` means is that this is the answer that you would get if the current expression `expr` was all there was to the entire program (in other words, if the continuation `k` was the empty continuation). With `'(quote ,datum)` for example, if you're quoting something like `'(quote 5)`, then you get back 5, so we can associate `v-out` immediately with 5. This can result in failure sooner rather than later, whereas normally with the program, the `'(quote 5)` might be a part of something bigger, like an element inside of a list. So when we do the direct translation, we don't know whether or not `5` is supposed to be the answer we get; we just have to pass around this `out` variable until we get to the application of the empty continuation. By passing around this extra argument `v-out` which is constantly changing in a way that's sorta algebraically specified by the rules for an interpreter, then we can fail fast and prune the search tree. I think I've got at least an intuitive understanding of what these algebraic rules are, but I don't have a complete understanding yet in all the cases. I think it's related to some of the stuff Conor McBride has done. In particular, the list example is easiest to think of.

There's this `list-auxo` helper, because the `list` operator can take any number of arguments. If `list-auxo` is taking the empty list, then the expected output is the empty list, so you can fail right away if it's not. The more interesting case is the pair case: if you're calling `(list 3 4)`, you know that the first expression evaluates to 3 (maybe it's `(+ 1 2)` so the expected output at that time had better evaluate to 3. If not, you can just fail then; you don't have to look at anything else. You definitely don't have to wait until the end of the program to know if you've succeeded or not. In this case, you're taking the derivative of the expected output with respect to the `cons` that's happening in the list. Anyways, that's a high-level explanation of this trick of passing in this extra argument, whose whole purpose is to fail fast--it has nothing to do with the semantics. If you do this properly, you'll only fail faster, but you'd still get the exact same set of answers you'd get normally. That's it in a nutshell. That's what we're going to try to explain in this paper for ICFP and get into the gory details. It'll be a squeeze to fit that into 12 pages.

**David:** I wonder if there's a smaller example than the full CESK machine. Like some smaller example of how this prunes the search. And then you can show how you use this technique within a larger context.

**Will:** Just look at a direct-style interpreter, like the quines interpreter, where you don't necessarily need it. But it should work there, I think. Even something like `appendo` might work as well. Or a type inferencer. I think you're right about trying it on something simpler. The other weird thing is that it seems like sometimes you have to put `v-out` inside the continuation (in the interesting cases for `list` and procedure application). That's got an interesting feel to it. It feels like you're taking the derivative with respect to both the  expression and the continuation.

Claire also asked why sometimes we aren't using it. The answer is I haven't finished cleaning up this code. I think it can be pushed more aggressively. There were a couple times where I wasn't sure exactly what to do because I wasn't sure what the rules should be. So I just left it as-is. The nice thing is if you pass it through `fresh`, [leaving `v-out` fresh?], then it just works; you're not going to run into any problems. But you don't get the aggressiveness of the pruning. I think it can be pruned some more to get better behavior, but I need to understand the behavior better.