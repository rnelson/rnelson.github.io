---
layout: post
title: "The Graph Class Assignment"
description: ""
category: 
tags: school, vcs, unix
---

# Ultra 2

I've long been interested in the various enterprise UNIX systems that were starting to be phased out right around the time I got into computers. I acquired a few via eBay and my favorite was a [Sun Ultra 2](https://en.wikipedia.org/wiki/Ultra_1). It was a little beast of a machine, despite being almost a decade old when I got it. Dual 64-bit UltraSPARC II CPUs at 200 MHz each, 512 MB RAM, 4 GB + 9 GB SCSI hard disks.

# CSC 300

[My undergraduate program](https://www.sdstate.edu/programs/undergraduate/computer-science-bs) had two introductory Computer Science classes (150 and 250, basically a CS I and CS II class) followed by CSC 300: Data Structures. All three were in the primary language used by the program at the time: C++. I took Data Structures the fall of my sophomore year.

The professor I had for the first two classes did a great job of helping students acclimate to a university environment after all of the years before. We were treated as autonomous adults, but also slightly coddled as we were knew to the whole college thing.

Data Structures, however, was run by Ken. Gone was all coddling, replaced with THE WORST PROFESSOR EVER. I vowed to never take another class with that guy once that semester was over.

# Assignment 8

The 8th homework assignment was to build a class that represented [a graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)). We were provided an object file (for linux-alpha) that provided a bit of adjacent implementation, and a header file to define the class. The implementation of the class itself was up to us. Easy peasy!

Having started out with QBasic the summer after 7th grade, I'd spent a lot of years doing some programming before I got to college, so most of my classes were incredibly easy for me. I would typically start assignments the day they were assigned and be done within an hour or two. During the Data Structures class, however, a few friends started having far more questions on the assignments and it was easier if I worked on them at the same time instead of having been done for a week and a half. For the first 7 assignments, this worked flawlessly.

# Night 1

I don't remember exactly which day assignment 8 was due, but let's call it Wednesday.

Monday evening, two friends came over to work on the assignment. We would all work independently near each other, being able to bounce questions off of one another. There was a very strict you-must-work-alone rule in the class, but we chose to interpret that as do your own work, not "you cannot ask your peers questions." Two friends showed up around 9pm and we got to work.

That's a lie.

We watched TV. The Daily Show, Family Guy, whatever else was on Comedy Central between 9pm and midnight those days. Around midnight, we turned the TV off and started some music because we had barely touched code.

We worked through the night, all of us struggling to solve the problem. At some point, we got into a nice groove that fit with our skills at the time. We switched to one person's laptop. I'd write some code and hand the laptop to one of the other two. He would run a sample through it and start debugging (but not with a debugger, that was still pretty much beyond us at the time). If he couldn't find and fix anything within a few minutes, he'd hand the laptop off to the third person who took my whiteboard and ran a sample through by hand with all of us watching to see if we could figure out where it worked.

Sure, we'd abandoned that working alone thing, but at least we were making process. That's all that matters, right[^ken_footnote]? ;)

Around 4am, I went to bed and they continued working for a couple of hours out in the dining room before leaving. Clearly, we needed to work on it on Tuesday to meet the deadline.

# Night 2

Tuesday evening, about 8pm this time, those two friends showed up at my place again. Having learned our lesson, we left the TV turned off this time and stuck to quiet background music. Already exhausted from being up so late the night before then doing our usual daily class/work schedules, we started right away with the laptop passing solution.

I was seldom up until midnight, let alone past midnight, so at about 11pm my brain would usually change from its normal state to something resembling Jell-O. Thankfully, we started about 3 hours earlier, so we would share the files that we were working on and while the other two tried to debug our then current state of the assignment I'd copy the latest onto my own laptop and start working on the next set of changes. It sounds counterproductive, but the bugs we were able to find were so small that it saved us time while I was able to be a bit more productive. A few hours in, however, Jell-O struck and we went back to a single laptop.

Looking back at the code today, we did a pretty good job of testing. We had `simplemain.cpp` to give us a very simple, easy to debug set of inputs to find major problems, and `complexmain.cpp` to put our code through the paces to really test it out. Showing my frustration from some point that night, there is also `dsdfjdsfhdfsuiefwuiwefuihvjkxcvjnkxcvjkvhjdsfuiohwefuohefuiohweruirui3244uerruihdfjkhxcvjkcvxnxcvzjknsdjhkdsfhewfuiewruewruioefwouifuiodfhjvchjkcvxjhkdfshjfdahfhjkewhjkweuhdsfhjvhjkv.cpp`:

```c++
#include <iostream>
#include "nelsonr8.h"

int main() {
  {
    Graph g;
    g.addArc(1, 2);
    g.bFS(1);g.dFS(1);
  }

  return 0;
}
```

# Midnight

It happened.

Right around midnight, everything we threw at the code worked perfectly.

All edges and vertices were linked together as expected, DFS/BFS worked.

WE. WERE. DONE!

Then some idiot (spoiler: it was me) suggested that the other two stop packing their stuff up. Before we quit for the night, we should run the professor's sample through just to be safe.

# Oh no.

It didn't work. Bet you didn't see that coming!

We immediately grabbed the whiteboard and tried to figure out where we went wrong. How could this one example not work when everything else did?

Fast forward through many hours of trying to fix it, and around 6:45am, I went to bed while the other two continued for the half an hour that I slept. My alarm went off as they were packing up to leave. It still didn't work. Class started at 11am and the assignment needed to be done by then. We could turn it in late for a percentage penalty for each class session it was late, but I wasn't giving up yet.

Ken was really good at responding to student questions via email, but obviously he was sleeping while we were stuck on things and I didn't want to risk him being unable to answer me in time, so I was going to go see him during his office hours.

# Run run run

At this point in my life, I was not a runner. I had never been a runner. I was one of those kids who would walk 80% of the mile at the end of the school year when we were all forced to do it. Running hurt and was not fun.

The bike I had at the time was too small for me and my knees hurt when I used it, so just before Ken's office hour started, I ran to campus. Well, I ran for a quarter of a block then stopped for 2 minutes to catch my breath, walked half a block, then repeat. It was not pleasant, but thankfully I lived close to campus.

I started explaining to Ken that I'd worked on the assignment all night, with it seemingly working at one point until I gave it his example. I understood the concept of a graph just fine, but _something_ was wrong with my code.

Before even looking at my code, he dug through one of his desk drawers and pulled out a copy of the assignment. "Oh, that's the wrong answer."

I was **mad**. I was also in shock, so he probably just saw a blank look on my face.

A common practice among professors is to reuse assignments each semester, but tweak them slightly. At the very least change the values used to test to ensure students aren't just hard-coding solutions for a given input.

When he went to prepare the assignment for us that semester, Ken mistakenly included the inputs from one set and the outputs from another. Two hours before class, nobody had brought it to his attention.

# Run run run again

This means our code worked perfectly eight and a half hours earlier. None of us were smart enough to make a backup at the point we thought it was perfect, though, so my options were to figure out what was missing from an older version or what we'd broken trying to "fix" the working one.

Either way, I would need to print my code out to turn it in (because that's a thing we did) so I did my run-gasp-walk thing to get back home and try to work. I had about 2 hours, factoring in that I had to get back to campus for class, to figure out how to make it work again, email the code, and print a copy.

I worked directly at the desk the printer was on to avoid ANY distractions. No music, nothing. Just me and my laptop, staring at Xcode trying to find the problem. Somehow, I did it.

Missing the option to force Xcode to print in black and white, I waited forever for the inkjet to print off pretty colored code, fired off an email, and rushed back to campus. I think maybe half of the class turned something in that day? A lot of people took the late penalty to get some extra time to work on the assignment. To this day, I wonder how many ran into the mismatched sample and didn't say anything.

# CVS? SVN?

When I was done with work that afternoon, I went home with a goal. I knew that source control was a thing but didn't know much beyond that. My Ultra 2 was going to become a source control server that night. I read through blog posts comparing CVS and Subversion, looked at the documentation for each, and settled on Subversion. Never again was I going to have an assignment working at midnight and spend hours breaking it and not be able to get back to the time it had been working.

## Apache

I can say with absolute certainty that I did not have public access to that `svnserve` process with anonymous read access to `/svn/school`. And I **DEFINITELY** didn't set up Apache on my FreeBSD machine and set up a `crontab` entry to pull down a copy of `svn://aten/svn/school` and make it browsable on port 80 for trusted friends to use _as a reference_ if they got stuck on assignments.

Nope. Never happened. No such things were active for the remainder of my undergrad.

# Ken, Part II

As a junior, I needed to add a Computer Science class to my schedule. Being a small program, most classes were only taught once a year, and a lot of the optional ones you just had to take _something_ that would fit into your schedule.

I broke my vow and signed up for GUI Programming (MFC, VB6, AWT/Swing), taught by Ken. Being a little further away from those early college days and not being salty about an honest mistake, I figured I'd give the guy another chance.

As it turns out, he is one of the two professors that I absolutely loved from my undergrad. I don't actually use it for anything but Messenger and Events these days so I can't say I'm truly keeping in touch with him, but all these years later I'm friends with Ken on Facebook.

[^ken_footnote]: Ken, if you're reading this, please don't find a way to make SDSU rescind my degree for working together with friends!
