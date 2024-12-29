**Video Title:** miniKanren hangout #1, March 10, 2013  
**Link:** [YouTube Video](https://www.youtube.com/watch/vRrgaibcTYs)

**Speakers:**
- **Lil Bird** (early designer of miniKanren)
- **Ambrose** (spoken on core.logic at Clojure Conj)
- **David** (involved with core.logic)
- **William** (PhD student, miniKanren developer)
- **Dan** (mentioned, not speaking)
- **Oleg** (contributor to miniKanren)
- **Joe Near** (undergraduate, worked on miniKanren)
- **Christopher** (joined late)

**Transcript:**

---

**Lil Bird:**
Yeah, it took me a while just to figure out how to post a URL. I couldn't figure out how to find you after you mentioned it the first time. I was searching for your name and couldn't find anything. Yeah, it took me a while to figure out just where to get the URL. So, I think Google Plus has some interface issues. Alright, well, since this is being recorded for posterity, we have a little intro. My name's Lil Bird, and I'm one of the early designers of miniKanren, which is a relational or logic programming language. This is the first Google Hangout for miniKanren, which I'd like to turn into a weekly event. This is somewhat about meeting people who are interested in miniKanren, and also, we would like to do some miniKanren things shortly. 

We've done a quick round of introductions; it sounds like we've got a mixture of experience levels, from someone like Ambrose, who has spoken on core.logic at Clojure Conj, to folks who are completely new to logic programming, and that's fine. So, part of this is we're trying to figure out what the structure of this hangout should be. We have a two-hour block, and the question is, how do we want to break that up? One idea was thinking of having more intro material in the beginning, and the second hour could be more hard-core things. I'm sure people will be joining and leaving, but in the beginning, maybe intros, then Q&A, and maybe submitting miniKanren tutorial information and talking about some techniques that would be generally useful to people. Then, maybe afterwards, going through a paper or a book like "Types and Programming Languages" and trying to translate Scheme to miniKanren or whatever. That's the basic structure and the kind of things we were hoping to talk about today.

It sounds like most people are happy to just kind of go with the flow, and I'm happy to do that too. We just kind of see what happens organically, and we don't have to have a super plan right now. I think it'll just kind of evolve over time. 

Well, hey, what I'd love to hear is just your background with the whole logic programming thing, how you got involved with it, your genesis for your paper, how long you worked on that, and where it took you.

**William:**
Yeah, that's a good question. In fact, one of the things I was thinking about talking about at some point was sort of like the history of miniKanren. Dan would really have to be here to do full justice to it, but he might be able to hang out sometimes. 

I started at Indiana University as a PhD student in 2003, or maybe it was 2004, I forget. Let's say I graduated in 2009; it took me six years. When I started, I had picked the school based partly on a visit to ICFP (International Conference on Functional Programming) which was held in Pittsburgh in 2002, where I met Dan and a bunch of other people in the logic programming community. I was trying to figure out which graduate school to go to; Dan just seemed to have lots of energy and was really excited about what he was working on. So that was one of the main reasons I picked IU.

So, I show up the first day of Dan's class for 521, which is the advanced programming language graduate class. I walk up to him after class like, "Dan, I'm so glad to be here and learn about Scheme programming," and he says, "I don't do Scheme anymore. I do logic programming now." I was like, "Damn, something you could have told me!" It was quite a shock, but I'm glad I did end up there. 

Dan had been working on logic programming for a long time, and he didn't like Prolog because it didn't have lexical scope. In Prolog, all the variables are global in this giant database, and you can leverage the power from manipulating that database, asserting and retracting global facts, and all sorts of things like that. But if you believe in lexical scope, then you're very unhappy with Prolog. So, Dan wasn't happy with Prolog; he refused to use it, and over the years, he had worked on a couple of logic programming languages. There had been other work in Scheme logic programming, but it never really took off.

When I started, Dan had been working on this new language for a while, and shortly before I arrived at IU, he posted notes on his website, and somehow Oleg found them. Oleg apparently wrote him this really long email pointing out all the mistakes or things he could do better. That was a friendly letter, and it got Dan's attention because Oleg had done a lot of Prolog. At this time, Oleg wasn't very well-known, but now everyone knows who he is in the academic functional programming community.

Based on Oleg's notes, Dan decided it would be good to collaborate with him, and with a number of Oleg's suggested changes and improvements, what ended up being called Cameron, which was the predecessor to miniKanren, was born. Then I showed up as a new grad student shortly after they had started working together. 

Dan is a very good teacher; he moves very quickly, especially if he thinks his students are good. The time I took his graduate class, there were some really good students, and he just went at the speed of light. A couple of weeks in, he handed out this paper which was like a hundred pages, a collaboration between him and Oleg. It was the implementation of a version of Cameron using Scheme macros. I didn't know macros or logic programming, so I had no idea what this was about. By the end, he handed out an assignment to do something with this language, like the zebra problem, which is a famous logic puzzle. 

Eventually, I figured out how to do the zebra problem, but I was underwhelmed because it was just kind of like this naive search. I didn't think it really showed off anything interesting. I still don't think it's a good example of logic programming. Personally, I think something like type inference or an interpreter is much more exciting or, you know, something like that. 

We tried teaching it in 311, which is the undergrad PL class, and it was a complete disaster. I still have the notes somewhere, but there were like four types of conjunction and two or three types of disjunction, and it was really a mess. But over time, we started cleaning it up, and when we wrote the book over the summer, that was kind of the big thing. 

**Ambrose:**
I guess I first heard of core.logic when it moved to contrib, and when that first mailing list post came out. I just kept bugging David more and more about this logic stuff, and I just made it a goal to give a talk at Clojure Conj. I had a little type inference in there, and it just so happened that my talk was just before a little unsession from Dan and Will, which was amazing.

**David:**
Voice now know what's going on? Okay, the apparently chat has or some of the Google stuff works better in Chrome for some reason.

**William:**
I'm just going to load miniKanren here. Yeah, it works. Okay, good. So, when people see that, yeah, now go on. So, let's try doing a little example here. 

... [Here, William goes into a detailed explanation of miniKanren syntax and operations, including examples of running queries, showing how relational programming works, and discussing the concepts of purity and relational operators.]

**David:**
So, you have to understand that before I can explain copy term. If you understand that, we can do copy term now.

**William:**
Okay, so this, by the way, if this is the only thing I said all of today, that would still be worth it because it turns out that whenever we're trying to design a new operator for miniKanren or a new constraint, the way I always think about whether or not the design is correct is whether or not it violates this property of being able to swap conjunctions and disjunctions and getting the same semantics.

So, if you're not familiar with miniKanren, basically a `run*` means I'm doing a query, I want to know which values are bound to `Q`, which is the query variable, and I want all of the possible values that's what the star means. `fresh` introduces new logic variables, so we've got `x` and `y`, and then I'm performing two unifications, so `equis` equals unification, and I'm unifying `y` with 5. You can think of unification as two-way pattern matching; it's like pattern matching where you can have variables on either side. So, I'm going to unify with 5, and now I'm going to unify `x` with 6, and then at the end, I'm going to unify `q` with this list containing `x` and `y`, and I get back one answer. Again, there's an extra set of parens wrapped around this because you could potentially have multiple answers, but we have one answer back; it's a list containing 6 and 5.

Now, the thing that makes relational programming interesting is that we can swap the order of those goals. So now the unification of `x` comes first, and we get the same answer back. In fact, I can move the unification of `q` up, and we do the same answer back. I can also do some things like add disjunction, so I can have a `conde`, which is sort of like Scheme's `cond` in a way, except it tries all the clauses. So now I can do something like this:

```scheme
(run 1 q 
  (conde 
    ((== x 6)) 
    ((== y 5)))
  (== q (list x y)))
```

So, I do `run 1` first, so I want one answer back. The `conde` means I'm going to try either one of these clauses, okay? So, in practice, the way it's going to happen is since I'm only going to get one answer back, I'll try the first clause, so `x` will be associated with 6, but in this clause, `y` is not associated with anything, so `y` is going to remain fresh. Then in the output, `q` will be associated with `x`, which would be 6, `y` will be fresh, and we get back this answer. So, we have 6 in the place of `x`, the first item in the list, and then `y` is fresh, which we represent as a special value underscore dot some number.

If I want to, I can change this to a `run*` to say give me all the answers, so I get back two answers now, and this is a proof by the way, there are only two answers, no more. In the second answer, we're trying the `y` being unified to 5, and we're forgetting about the association between `x` and 6, or you might call it refreshing it, so we're pretending as if this had never happened. So, for the second answer, `y` is 5, `x` is not any value that we know, that's still fresh, so that's why it's represented as this underscore zero. So, there are two answers and two answers only.

Now, the important thing is I can still play this game of swapping goals around, and I still get the same answer. I can even go beyond that; I can say alright, well, let's associate `x` with something, in this case, so let's say `x` is 4 here, and `y` is 7. Okay, now we get these two answers back where neither one has fresh logic variables. Within the `conde`, which in the thin `conde` clause, we have a conjunction, so both of these have to hold, both these goals. So this unification of `x + 4` and this unification `y` with 5. So `conde` basically is a disjunction of conjunctions, and we can swap the goals inside of `conde` clauses, we can swap the `conde` clauses themselves, we can swap the goals within the `fresh`, and even though the order of the answers comes back different, we should be thinking of these answers as a set of answers, and we still get the same set of answers. So, to me, this is what makes programming relational or not relational is whether or not you can swap the conjunction, disjunction, and get the same answer back. And to me, this is the most important and the fundamental property of miniKanren programming and core.logic programming. Anything that violates this property, any sort of operator where you can't swap things around and still get the same set of answers is impure. That's my view of purity.

So, that's the basic idea of purity. Any questions about how that works before I talk about copy term? 

... [The conversation continues, but due to the length and complexity, here are key points from the remaining discussion:]

- **Copy Term Discussion:** William explains the concept of copy term in logic programming, highlighting why it's not inherently relational. He discusses how they managed to create a relational version in specific contexts like the Alpha Lean Tap paper, using nominal logic and tagging.

- **Debugging:** There's a significant discussion on debugging miniKanren programs, with William sharing his approach of writing one relation at a time and crafting test programs. He mentions the challenge of debugging due to the language's nature and suggests possible future improvements like not discarding failures but wrapping them in a special type to track failures.

- **Future Topics and Hangouts:** They discuss potential topics for future hangouts, including nominal logic, probabilistic logic programming, and how to better integrate miniKanren and core.logic communities. The idea of having a permanent URL for these hangouts and setting up a Google Group for better communication is floated.

- **Technical Challenges:** William describes a current problem he's working on related to small-step interpreters in miniKanren, where he wants to execute steps until no more rules apply without using non-relational constructs like `conde` or `condi`, which would compromise the relational purity of the program.

- **Closing:** The hangout concludes with plans to continue these discussions weekly, potentially around events like Clojure West, and the possibility of live streaming from conferences for those who can't attend in person.
