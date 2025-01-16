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


