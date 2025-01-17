# miniKanren Hangout #6, 21 April 2013
Youtube link: [https://www.youtube.com/watch?v=MTAhbsWUUxM](https://www.youtube.com/watch?v=MTAhbsWUUxM)

**Will:** Hello everyone. Welcome to the 6th weekly miniKanren hangout. I was hoping a couple people would talk about a logic puzzle today, but they haven't arrived. Maybe they'll show up later. Anyway, there's always plenty to talk about in miniKanren Land. I've got a few topics to talk about, myself. Let's start out by going around and seeing how everyone's doing and if they have anythnig te talk about in terms of miniKanren development or things they're working on. I know Claire has been working on some stuff, and dan and Jason have been working on stuff. I'm not sure if they're ready to share what they're working on. Claire, do you have any updates on the features you've been adding to miniKanren?

**Claire:** I don't think there's anything to share, beyond what I posted on the Google Group.

**Will:** Can you give a recap of that?

**Claire:** I can. Let's see. I posted about a number of things. Documentation now exists for cKanren. I implemented a way to do fair conjunction (and in general being able to customize your search strategy through the search space). So now you can define goals to be lazy, and specify if you want to do a breadth-first or depth-first search through those lazy goals.

Older update is being able to do general unification on any structure that implements the Unifiable generic. We have attributed variables, a `set` constraint implementation that is working as far as I can tell. I've started working on a higher-level language to define ways to rewrite the constraint store, kind of like CHR. And I implemented the `normalize-store` procedure that takes two disequality constraints and checks to see if one subsumes the other. So that's now a one-liner. There's a way to name `conde` branches so that if you print out information (like print out the packege) you can actually see which `conde` branch you went through. And there's a `#lang cKanren` now that incorporates all of this now, which is actually distributable and just works as-is. You don't have to do anything crazy to make it work. I think that's it.

**Will:** And it took you a whole week to do that? What else were you doing?

**Claire:** Yeah, I actually haven't gotten much work done *this* week for obvious reasons. But I'm hoping to get back to work after my Systems final.

**Dan:** Am I going to get in trouble because of this, Claire?

**Claire:** I don't think so. Why would you get in trouble?

**Dan:** You're one of my students. \[Your Systems professor\] might be upset with me.

**Claire:** Oh, he knows!

**Will:** He's my academic brother, I'll take him on. He should be happy Claire's doing all this stuff.

Excellent. ALright Dan, what have you been up to?

**Dan:** I'll try to characterize it. Will, Jason, and I talked the other day about writing an interpreter that would run the miniKanren stuff.

**Will:** Something I've only been asking for for 10 years.

**Dan:** I knew it was easy, so I didn't waste my time with it! It turned out to be not so easy, but not so hard either. We did it in two steps, basically. The first step was to design and implement a miniKanren-like language that was devoid of the use of macros. There are basically 5 operators: `run-g`, `fresh-g`, `==`, `project`, `ifu`, `ifa`, two argument `conde-g`, and `conj-g` (which took two goals). So that was the little language, and we implemented that without any macros. It's all functions.

Some of the operators look a little bit like `call/cc` because of the fact that one of the arguments in a function. For example, `fresh-g` took a function as an argument whose formal parameter was the new variable, and the body of the function would be a goal expression. Then when we introduced `conj-g`, which was conjunction, it just took two goal expressions. And so on.

So we have these functions. And then the only task was to simulate those functions inside an interpreter, which turned out to be pretty straightforward. I don't know what else to say about the language. It's a very simple language: only 2 goals in a conjunction, only 1 goal in `fresh`, 2 goals in `conde`, `project` only takes one goal, and `ifu` and `ife` take 3 goals (if the first goal succeeds, then it conjoins it with the second goal; if it fails, it runs the third goal). Everything has to be manually nested. We realize that macros will do that for you for the functional version. It's more of a problem when you rely on macros for the interpreter version, because (thanks to Kent \[Dybvig?\]) not all names are the same. Because if you happen to name some of your interpreter's special forms with the same name as primitves like `cons`, then it macro-expands into `%%cons`, which was very unfriendly. We're thinking about ways of getting around that.

We do have an interpreter that does what we want, and I think it meets the criteria we discussed Monday night. (I think it was Monday night.)

**Will:** Oh, you mean Monday night ten years ago?

**Dan:** It seems like ten years ago.

**Will:** It *was* ten years ago, because I wanted miniKanren to be an interpreter, which is not what you did. You did something better than an interpreter, but now I finally want my interpreter.

Now, of course, part 2 of this exercise is to translate this into miniKanren so that you have something metacircular.

**Dan:** Which miniKanren do you want to translate it into?

**Will:** What do you mean?

**Jason:** You remember the little language Dan described? He didn't mention disequality constraints listed as one of those forms, for instance. You didn't hear `absento`, `presento`, or any of that stuff.

**Will:** Well, start out with the "little book" version. Start out with the minimal language.

**Dan:** We need the whole language to implement it, because you can't look up a lexical variable \[without disequality constraints\].

**Will:** You'll have to figure it out. I mean, this is like writing a metacircular interpreter for a subset of Scheme.

**Dan:** Almost. Not quite. When you write a metacircular interpreter, you have fixed on both languages.

**Will:** Well, you're going to have to fix those languages.

**Dan:** We have full miniKanren with the 5 special forms.

**Jason:** Writing an interpreter for full miniKanren seems like step two. Implementing the rest of those operators (to make it self-hosting) seems like it'll take a while, I think.

**Will:** Okay. Next week then. Sounds good. Austin, how are you doing?

**Austin:** I'm good. I took a week off from logic programming. I think my brain got full. I'm waiting for some new additions to appear.

But I had an interesting problem come up this week that I can summarize. I'm working on an abductive planner and I have five classical AI problems (like [Blocks world](https://en.wikipedia.org/wiki/Blocks_world)) to test it out. For the last few weeks I could only solve around two of those problems--I can't solve all of them. But I made a change this week and suddenly I could solve all of them. I was happy for just a moment, because I didn't know exactly why my change caused it to work. But I soon discovered that it was because I switched my implementation of `findall/3`. The new one is causing solutions to be found because it was reifying fresh variables (but these were failing branches anyway). My other solutions were floundering in the whole negation-as-failure issue. So it was not terminating. It was interesting that this bug helped me see another flaw in my program. It really helped elucidate the whole floundering and negation-as-failure issue. Now I've got two problems to solve.

**Will:** Let me see if I understand correctly. You changed your `findall/3` and it started working, but it was unsound? Is that the problem? Were you succeeding when you shouldn't have?

**Austin:** Previously, it was not terminating. But when I switched it to a version which was grounding fresh variables (because those were negation-as-failure branches), it was able to close \[prune?\] those once it had a reified value of `'_.0` instead of remaining fresh.

**Will:** When you were diverging, were you trying to do a `run *` (or a `run 10` when there were only 9 answers)? Or was it actually diverging while trying to find answers that actually exist?

**Austin:** I haven't been able to investigate it futher, but I believe it was floundering in negation-as-failure where it's assuming that when it's proving the negation of a goal it's that whole "closed world" assumption. Since the variables are fresh and I'm proving the negation, it keeps floundering.

**Will:** Are you paying attention to the G rule when you're doing negation as failure?

**Austin:** What's the G rule?

**Will:** I assume you're using a `conda` or `condu` somewhere.

**Austin:** Yes.

**Will:** The idea is that `conda` and `condu`, just like Scheme's `cond`, have *test positions* for expressions. The first goal in each clause is in test position. The G rule says that in order to use `conda` or `condu` in a way that's sound, you have to make sure that none of the fresh variables in a goal in test position end up being ground/instantiated when you run it.

**Austin:** Is this the Second or Third Commandment in the Reasoned Schemer? Or the "Law of `conda`?

**Will:** Oh! Is there a Law of `conda`? I don't remember! I spend all my time avoiding `conda`.

**Austin:** Oh, it's the Third Commandment \[the Second Commandment in TRS2e\]: "If prior to deptermining the question of a `conda` or `condu` line a variable is fresh, it must remain fresh in that line's question." That's been a hard one for me to digest.

**Will:** Wow, I just learned something from the Reasoned Schemer. Well, I didn't know it was in the book. Good for us for putting that in. Shame on us for putting `conda` and `condu` in.

**Dan:** You didn't internalize it because you knew you'd never need it!

**Will:** I know the rule, I just said it. I didn't know it was in our book! Now I'm ashamed.

**Austin:** And you call it the G rule?

**Will:** Yeah. It's G for "ground". That's what Oleg called it. Oleg is the one who taught us about that nastiness. Anyway, are you respecting the G rule, ensuring things don't become ground? Because what you're talking about sounds like a G rule related issue.

**Austin:** It definitely is. It's a flaw in how I was posing these problems. I was leaving it where variables could stay fresh. Basically, I had an axiom where the head was unifying but the body contains variables that never get grounded. I wasn't respecting one of the restrictions that the author of the interpreter stated.

**Will:** Do you think you have a handle on it now, or do you have to revisit it?

**Austin:** I'll keep plugging away at it. But this problem really helped me see exactly where the flaw was occuring. Exactly what happens when you try to negate a fresh variable.

**Will:** Let's see. I've been working with Michael Adams and Andy Keep on some macro related things in alphaKanren. Hopefully I'll have something to show people not too far in the future. I think the program we're working on is by far the most complicated alphaKanren program out there. It's kinda scary trying to debug this thing. I think we actually found a bug in the unifier yesterday of one version of alphaKanren. We're noticing that a certain unification was failing and it didn't make any sense that it failed. We tried the equivalent in the REPL and it succeeded. Michael and Andy noticed that if you projected one of the two terms being unified that unification succeeds instead of failing (`project` is equivalent to a `walk*`). In other words, if you have some logic variable that contains a term and some other logic variables, a `walk*` makes sure to replace all the logic variables with ground values to the greatest extent possible. When you `walk*` one of the terms, then the unification suddenly succeeded, which should never ever happen. That was bad. I think Claire thinks the problem might have to do with suspensions, which seems plausible. We'll have to look at that clause. We "fixed" it temporarily by making sure the unifier does a `walk*` on each term, which is extreme, but it's always fine to do a `walk*`--it shouldn't ever hurt (unless you have circularities, in which case you might diverge). Anyway, we spent an hour and a half trying to debug that problem before thinking maybe the unifier is broken. Anyway, that work is pretty cool.

Still doing work on latticeKanren.

I've started to think hard about adding probabilistic programming and machine learning aspects to miniKanren. That will be a project in the future. If anyone has thoughts or things they'd like with miniKanren handling probabilities, machine learning, or natural language things...

**Dan:** If it's possible, I'd like Claire to describe in more detail how she does parallel conjunction.

**Claire:** Sure. Let me pull up Emacs.

\[Discussion about Google Hangouts muting microphones omitted.\]

**Will:** While Claire's getting set up, if anyone has thoughts about probabilistic logic programming, machine learning, or handling big data and their tie-ins to miniKanren, I'm all ears.

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

I defined this new form `define-lazy-goal`. Instead of using Scheme's define, you'd use this new macro. All `syms*` does is go through a possibly nested tree and emit all the symbols at the end. If you have a symbol, `out` is a singleton list of that symbol. If you have a pair, then you find all the symbols in the `car` and the `cdr` and append them. I have the goals in a different order just so I can show that it works even when you have the silliest \[recursive\] goal first. Same with the pair case of `appendo` -- it has the recursive call first. I have some tests that show this works okay.

So you can see that writing the body of these goals is the same as plain miniKanren. You just need `define-lazy-goal`.

```scheme
(begin-for-syntax
  (search-strategy 'dfs))
```

I have this, which is basically saying when you do expansions, make sure you use the search strategy "depth-first search". You can also do a search strategy that's breadth-first search. I can change it and it can go just fine.

```scheme
(begin-for-syntax
  (search-strategy 'bfs))
```

I changed the symbol `'dfs` to `'bfs`.

On the right side of my screen is a big macro for `define-lazy-goal`. Most of it is not important--don't try to understand most of what it's doing. It has to do with this complicated case of whatever search strategy you're using.

The most important thing is that it basically takes a normal miniKanren goal and writes it as this "enforce" function instead of actually calling the goal.

[26:45](https://youtu.be/MTAhbsWUUxM?t=1605)

If I were to expand this out, this `syms*.
