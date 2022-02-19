---
layout: post
title:  "Practical Guide to Solving Hard Problems"
thumbnail: "/images/2022/hard_problems_thumb.jpg"
---

I sometimes find myself in a position of needing to write some code
that I'm just not sure how to write. Been there have you?
Here are the steps I take when I'm stumped.
No huge revelations here, just hard-earned advice.

1. Think hard about the problem for a few weeks before typing any code.

2. Type in a function or write a class that has the inputs and outputs you need.

3. Break the function down into multiple steps with clear objectives. You may not know how to achieve those objectives, but that’s a problem for your future self. Right now, you’re just trying to write out the high-level algorithm.

4. Create a function for each of those steps and `throw new NotImplementedException()` in each of them. Their names should be long and explanatory and there should be no question about what’s expected of them. It’s *really* OK if you don’t actually know how to write 'em.

5. Now, go implement a few of those functions. You know they’re not *all* hard. Some may even be fun! Build up your confidence and implement the easy ones. It feels good to make progress and it lets the analytical part of your brain run in the background for a bit while you focus on nitty grid number types and file IO.

6. Time to tackle some of those harder functions. Go into each of those and break the problem down into steps just like you did before. You’re right, I’m gonna say it: Rinse and repeat. Keep breaking those hard problems down into steps. Turn each of those steps into a function with a clear name. Implement the easy ones. Then break the hard ones down into steps again. Do this over and over again. You'll be surprised how much you can actually get done.

7. Pretty soon (haha) you will have an 80% complete solution with just a few pesky functions left that throw NotImplemented. Now go scour your favorite package repository, or code repository, or question and answer site, or artificial intelligence programming assistant for implementations. Chances are you’re not the first person to need this particular function or widget. Find some giants, climb on top of them, and scream "Holy shit, there are a lot of smart programmers in the world!"

8. OK, you’ve scoured the inter webs and yet you still have a few pesky NotImplemented exceptions. It’s time to check on those scientists. Enter every SEO permutation of your problem statement into arXiv. Surely others have worked on problems related to one you are trying to solve. They will most likely offer insights or perspective shifts that can help you reframe your problem into something solvable. Do that. Reframe your problem and knock out those NotImplementeds.

9. Now you’re in trouble. If you still have a few NotImplemented exceptions, and there are no giants upon which to stand nor academics obsessing over this particular field, then it’s all up to you. Think big. Think outside the box. Your career depends on it. (Just kidding, I hope.) Perhaps a bath will help you think?

I think these are steps all programmers take, but sometimes it’s good to spell it out.

I especially value the functional decomposition. Functions are a powerful abstraction, not just for writing less code, but for thinking about problems.

And please don’t misinterpret my use of the word “functions” to mean only those things functional programmers like. I mean any data transformer: from lowly lambdas to state-bearing IO-processing monolith objects.

Thanks for reading! Now go solve those hard problems!
