# miniKanren Hangout #5, 14 April 2013
Youtube link: [https://youtu.be/VSpq3qK9L88](https://youtu.be/VSpq3qK9L88)

**Will:** Hi everyone. Welcome to the fifth miniKanren hangout. I'm Will. Today we're going to try some miniKanren live coding (or at least live translation). We've got some of the usual suspects here. Maybe we should start out talking with Austin. We were talking about some things that I think would be of interest to general miniKanren programmers, and could also help feed into today's exercise of translating Scheme code into miniKanren doing live coding. The point of this isn't just to write more miniKanren code; the real point is to talk about how to think about the translation process, how to think about goals, which problems you run into, and *what* to think about. Hopefully it will lead to good questions and thinking about writing miniKanren in practice.

Austin, did you want to recap the things you were thinking about when working on miniKanren recently and the areas where you had some issues?

**Austin:** You summarized it well. How do you think on the level of a miniKanren programmer? How do you understand the semantics of goals? How do you reason about goals without reasoning about their operational semantics? I'm curious what the idioms are. Do you think of *this* goals as a value generator, and another goal as an assertion? That kind of thing.

**Will:** This reminds me of something Dan was talking about the other day that we may not have made explicit, but which is always in our minds (mine, Dan's and Oleg's). When we write these programs, we always want the relations to work in what a Prolog programmer would call "all modes". That is, you can allow all of the variables passed into a relation to be fresh logic variables and the relation should still work just fine. That's the gold standard. Everything being fresh should work. You should be able to do a `run 10` with all fresh variables fed into your relation, and that should mean something. You might have the situation where there are finitely many answers (say, only 6) and when you do a `run 10` your program could go into an infinite loop, because it can't find the 7th (nonexistent) answer. But if you do a `run 6` it should work just fine. This idea that you should be able to *always* feed in *all* fresh variables to *all* of your goals is really the standard we want.

There may be some times where for a particular goal you want to restrict it a little bit, like if you have a little bit of [mode] information you want to take advantage of. In those cases you might want to use a `conda`/`condu` which is equivalent to Prolog's cut `!/0`, or you might want to use some additional miniKanren operators like `project`. But that's getting away from the relational paradigm. I see that as an optimization (or compromise you might have to make) if you can't write it directly in the relational style. But most of the time, you can get away with writing everything to support all fresh variables. I always think of the goals as being runnable on fresh variables.

In fact, if I write a complex relation, often I don't know how to test it [on ground terms]. xIf you are doing something like an interpreter and you have a complicated term that you want to feed in (or you have a special representation of environments) and you get one of those terms wrong syntactically, then your program will just fail or diverge. So what I normally do is just a `run 10` will all fresh variables and look at the answers that come out, making sure they have the right shape. Also, that gives me some test data I can feed in: if I know running with all arguments fresh gives me a ground term, then I can stick that in and leave everything else fresh and see answers I can generate. So it's also helpful for testing miniKanren programs.

Okay, so today is "stump the chump day". I'm the chump. The "stump the chump" game works like this: you find a piece of purely functional Scheme code (obviously not 300 pages worth, but something relatively short and self-contained). If no one here can present it, I can try to find something online. I'm trying to translate code I haven't seen before from pure Scheme (no side effects, no `set!`, no `call/cc`) into miniKanren and have it run backwards. I'll go through that whole process and try to explain what I'm thinking about the whole time. If people have any questions, I'd be happy to answer. It's a kind of "thinking aloud" exercise. Maybe we'll run into problems, maybe we won't. We might see opportunities to use constraints or nominal logic programming, etc. This is just to try to give people a chance to see what an experienced miniKanren programmer thinks about when going through this process.

Before we get into that, does anyone have any general questions or things to share? Claire, I know you've been really busy with some miniKanren related things. Did you want to say a few words about what you're working on and their status?

**Claire:** Sure. Last week I implemented a macro in Scheme that will let you define a lazy goal, which basically waits until the end to evaluate the goal. This actually lets you define different searches through the search space, so this kinda solves this problem: if you put the recursive call first [and unifications later] and it'll diverge because it doesn't have all the information when it goes depth-first into that recursive call. Now [with lazy goals] you can have it wait until all the unifications have happened before it tries to evaluate that goal. That's implemented in cKanren now. You can decide what kind of search strategy you want, which goals can be lazy, and switch between depth-first search, bread-first search (through all the lazy goals), and some weird hybrid search that mixes bread-first and depth-first. I've been getting that to work this week.

**Will:** That's really cool stuff. You were saying before that you might be able to do this per goal: having different search strategies in the same program, right?

**Claire:** Yes. I don't really know what that would mean now, but as soon as I know what that would do, I can implement that.

**Will:** Yes. Ciao Prolog lets you have multiple search strategies in the same program. (I think at the module level). It seems like a very powerful technique which could be cool for doing things like running relational interpreters backwards, for example. Maybe you want to search for terms that have a certain complexity or look. Or maybe you want to have a probabilistic search strategy/heuristic guided search. It would be really neat to change those things around.

You're also working on the CLP(set) stuff and Constraint Handling Rules. Do you want to mention anything about that?

**Claire:** I haven't done much work with the CLP(set) stuff since last weekend. It's still not working entirely, but unions definitely work. And I'm working on some macros that will let you define constraints at a higher level, kind of like Constraint Handling Rules. You can define under what circumstances your constraint will fail, under what circumstances you can ignore the constraint, what other constraints might trigger, and other interactions between constraints. So if you had `(=/= a b)` and `(=/= b a)` in your constraint store, you can drop one of those. You'll be able to specify things like that at a very high level. Hopefully that will make it easier to define constraints.

There was one more cool thing about CHR but now I forget.

**Will:** Did you talk about how making recursion lazy partially solves the problem of "commutative and"? Unification would be the only non-lazy goal, right?

**Claire:** Yes. If you do it right and make all your [potentially nonterminating] recursive calls lazy, you can really write your goals in whatever order you want, which is awesome. It doesn't work so well when you have, like, 5 or more recursive calls. In the case of quines, I think there's a case of 3 recursive calls in that evaluator. *That* doesn't work so well because if you give all 3 of those a fair chance to evaluate, you well get an explosion in the number of `conde`s, and I'm pretty sure that's what slows it down a lot. Using depth-first search is the fastest way of doing that, but it's not as fast as the original code yet. There are a lot of dispatches that have to happen (which wouldn't if you were evaluating things [strictly] as soon as you encounter them).

**Will:** Cool stuff. Claire is working on four major things.

**Claire:** I remember now! cKanren has actual documentation.

**Will:** Documentation!? That's not the miniKanren way.

**Claire:** I know right? In my repository there are now explanations about how to install and use cKanren.

**Will:** You are going to ruin our reputation.

I thought there were four things. You ported all the stuff to Racket. You've been working on CLP(set)...

**Claire:** Attributed variables!

**Will:** Attributed variables, constraint handling rules...

**Claire:** General [generic] unification!

**Will:** General unification, and lazy recursion (which helps to solve commutative conjunction). That's like 6 things.

**Claire:** Oh, and now the constraint store is a hash table so it goes faster.

**Will:** Alright, sever things. Cool. I look forward to seeing the polished version of all those things, the sets and all that. That's really going to make miniKanren much more powerful and useful. Oh yeah, documentation! That counts too. Those things will make miniKanren much better. Nathan, is there anything on your mind?

**Nathan:** I've been spending time trying to parse the difference between CHR and the new constraint handling stuff and where term rewriting fits in. I'm interested to see where this goes. I ended up getting myself really confused because a "constraint" in CHR is more like a fact in Kanren, as far as I can tell. But there is some overlap in different ways. They're more high level and at the same time low level than Kanren in some ways. I got myself confused on what it would mean and what term rewriting would look like in a system that can go forwards and backwards, if that makes sense. When a rule fires in CHR it's super forward-chained and will never go back: would it make sense to have term rewriting in Kanren in the first place? That's kind of what I was doing some reading about and trying to wrap my head around, but I was getting confused by the whole thing.

**Claire:** The idea in my mind is that these CHR-like rules will actually expand to what cKanren does now, and cKanren runs backwards just fine. I'm not anticipating that being a problem.

**Nathan:** Would these rules do something similar to term rewriting? Like simplifying if you have a "less than" constraint that subsumes another "less than" constraint: would it remove that constraint?

**Claire:** Yes.

**Nathan:** That removal itself wouldn't "undo" [from running backwards], right? That's sorta outside the search, right?

**Claire:** All of these are still functional data structures, so that won't affect backtracking in any way. The fact that you have been able to refine your constraint store in one branch is not going to affect any other.

**Nathan:** Okay. So it wouldn't "undo".

**Claire:** The set constraints were originally very CHR-like. I had to translate them into what would work in cKanren. I'm excited to go back to the original mathematical formulation of those things, because it's very term-rewrite-y.

**Nathan:** I'm interested in where that goes.

**Claire:** I'm working on it right now in the background, so hopefully not very long.

**Will:** The term rewriting also makes me think that there might be some overlap with things like [PLT Redex](https://redex.racket-lang.org/). It's sort of like that space also. The whole general area of term rewriting and executable semantics is a really cool area and really interesting.

[Discussion of the IU 3d printing club's antenna is omitted].

[19:35](https://youtu.be/VSpq3qK9L88&t=1175) I guess it's time to play stump the chump. Does anyone want to challenge me to translate some code into miniKanren, or must I be reduced to finding my own challenge? [From the chat:] "Write miniKanren in miniKanren." That's something I've wanted to do for a while. I don't know if I could live code that. It is true that there's a purely functional Scheme implementation of miniKanren, so I suppose that's fair game. Let me think about this.  [From the chat:] "No". Okay, Claire retracts the challenge.
