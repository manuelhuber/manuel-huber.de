---
tags: [programming]
titleFormatted: Code <fat>Reviews</fat>
---

Thinking about doing code reviews in your project? Then this is for you:

For a recent lecture about software quality assurance we could pick topics freely. And while I usually jump at the chance to build something I chose a topic where I wouldn't write any code - that's how important I think this topic is. To me **code reviews** are one of the pillars of a proper software development process, like unit tests or a CX server. 

If you disagree, just imagine the following: You're handed legacy code - but instead of having to struggle understanding this ancient crap and fix it you can just walk over to the author and tell them "That's not good enough. Make it readable!". Doesn't this sound great? Well that's what code reviews allow you to do (except you do it right away instead of letting it become legacy code).

Interested yet? No?! Well maybe you're a manager and actually feast on the suffering of coders - then consider this: Code Reviews are one of the cheapest ways to test & fix software. You might know graphs like this one:

![graph1](/assets/2018/08/graph1.jpg)


I've had them explained to me (this business studies stuff is out of my league) and it's basically this: Fixing / changing is more expansive the later you do it. Well guess what: Code reviews fixes bugs and reworks issues right when they are made (besides wrong requirements, misguided business decisions, that terrible CI style guide, ...). Studies have shown that code reviews are one of the cheapest ways to fix bugs. And while you might not want to skip features to allocate time for reviews just keep in mind: These bugs need to be fixed anyways - the question is just will you do it now for cheap or next sprint or after the release at higher cost?

The result of my work is a PDF document that guides you step by step through the process of wanting to do code reviews, defining how to do code reviews and actually start doing them:

  
### [It's on my Github!](https://github.com/manuelhuber/codeReviews/blob/master/Code%20Reviews.pdf){:target="_blank"}


There's also a PowerPoint presentation that may seem like complete non-sense but makes sense when you see it as backdrop to my talk about the topic :D

Happy coding & reviewing!
