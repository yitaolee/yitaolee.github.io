---
layout: post
title: Problems
date: 2017-07-20 15:32:24.000000000 +09:00
---

#### What's the problem?

[Chumi] is an attendees-focused event hosting platform, it has payment, instant messaging, and circles feed features. Due to the product is pivoting a lot of times, it leaves unneccesary code and structures. Every time when developers need to fix bugs or add new features, it takes a lot of time to check the code and easily causes more issues.

#### technical problems

1.ARC
2.Wrong struc, enum, class usages
3.Unnecessary force unwrapping model properties
4.Lack of proper memory management considerations while doing network calls within 5.view controllers
6.Some bad coding design
7.Lack of tests
...

#### What causes those issues?

TDD is not excueted. No code review. Tight schedule. Too many part-time developers. Lack of coding style guide.  

#### What would I do?

1. Draw a diagram and write a doc for all kinds of files. 
2. follow mvvm, work on model by model. 
3. Work on view-model.
4. Work on view controllers.
5. Write a coding style guide.

#### License

Great thanks to [Dale Anthony](https://github.com/daleanthony) and his [Uno](https://github.com/daleanthony/uno). Vno Jekyll is based on Uno, and contains a lot of modification on page layout, animation, font and some more things I can not remember. Vno Jekyll is followed with Uno and be licensed as [Creative Commons Attribution 4.0 International](http://creativecommons.org/licenses/by/4.0/). See the link for more information.
