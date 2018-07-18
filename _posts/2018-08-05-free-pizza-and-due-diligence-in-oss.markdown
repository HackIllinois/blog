---
layout: post
title:  "Free Pizza and Due Diligence in OSS"
date:   2018-08-05 19:00:00
categories: spotlight
tags: TBD
image: TBD
author: Kyle Begovich
---

Hi all. I'd like to experiment with some content that's much more of my observations of the Open Source community that is pretty agnostic to the HackIllinois event. First and foremost though:

## Free Pizza

Let’s say that both you and I are software developer students. We get to chatting one day after class and you tell me about a cool project you’re working on, either alone or with some friends. I really like this project, so I decide to buy you guys a pizza or two to reward this good work and to encourage it to continue. Now, this aforementioned pizza was well-intentioned, but I accidentally sent you a pizza with pineapple on it! Nobody wants pineapple on their pizza. Unfortunately for you and your team, you accept the pizza as the gracious gift that it is, and don’t realize until you take a bite or two that this pizza was more of a curse than a gift! If only you hadn’t accepted this pizza from such an incompetent person!

The point of this story isn’t to be dismissive of pineapple on pizza or to showcase my incompetency in ordering pizza. Rather, it is to highlight this common instance and some of the other cases that can happen in software development (or food service).


## OSS Considerations

Some people don’t realize they’re giving you buggy code
Some people may intentionally try to give you malicious code
Some people are good and genuine contributors and are trying to help you

Let’s go through these points one by one.
The first case is what we saw in the pineapple story, but it is generally much more subtle than an extra pizza topping. A bug in a pull request will almost never be a breaking change upfront: it may be a dependency on some depreciated software, a dependency on a current tool that will be shut down later, it may even be code that works totally fine but is really hard to be scaled later and leads to hacky workarounds that break things. Any of these situations and dozens more can happen, and as the network of contributors grows, tracking down who’s responsible for what change and getting a fix may be harder than git blaming them and waiting for a patch.

The second case is something that’s pretty rare, but it can have a huge impact if it gets merged. There’s an interesting article on such a scenario [here](https://hackernoon.com/im-harvesting-credit-card-numbers-and-passwords-from-your-site-here-s-how-9a8cb347c5b5) that highlights how a nefarious individual could try to do this. This is generally the main concern of any highly risk-averse group, and it would make logical sense to simply not open source anything that is critical to operations or may have access to private data. I’m not necessarily opposed to this, as I don’t want my credit card at risk because my bank was too hasty to open source their software. Even over-the-fence OSS would still pose a risk, as any security vulnerabilities could now be broadcast instead of staying private. The security protocols in place on many projects mitigate this risk, but they don’t remove it completely, and there are some very dedicated hackers out there. There are levels of abstraction, though, where certain components of a project can be more trusted than others, and you can still get the benefits of the OSS community without incurring risk.

The final case is the most common of them, and in established domains, it’s often the only one that actually gets approved. This should be the goal of any project. Many contributors fall into this category for a variety of reasons, from interest to investment in a project and from dependency to earning revenue on a project. Good developers offering their help to you, for free, and it’s a valuable product. You’d be silly to not accept their help! Right? Well, identifying these people and contributions is really hard. Requiring unit test cases and regression testing won’t save you from malicious code, and spending your time going over code line-by-line prior to accepting it is just impractical for some circumstances. So where’s the line? Do you just slap a license on your code that takes away your liability? Do you never accept anything from anybody? Do you write a basic API to interface with your code and only let people interact with you through that? Am I going to keep asking questions without answering them? The suspense is killing me!

The answer, in its inglorious, unsatisfying entirety is that it depends. A government or big bank may never share certain projects simply because the risk is too high. An independent developer or small group project may want to move fast and just add a pre-written line or license to their project and not think twice. If your code is widely used or your name is [Douglas Crockford](https://www.youtube.com/watch?v=-hCimLnIsDA), declare “Do No Evil” and move on from there. Maybe your data isn’t sensitive but the architecture or other functionalities are trade secrets. Let users interface with the database but not submit code, perhaps through an API. There’s no right answer for everybody, so don’t listen to anybody claiming there is, but there are a few good rules of thumb.


## Due Diligence

If you work somewhere with a legal department, talk to them.
If you’re a student or just getting into this industry, your software portfolio and presence is built by public contributions, not private ones.
Some actions are reversible, and others aren’t. Take notice when working with the later.

Understand the level of security you need and act accordingly. Do your due diligence. Care about building a community of developers. Eat your vegetables. Floss, it is essential for good dental hygiene. Never eat pears, they’re too squishy and they always make your chin wet. Remember to like, comment, and subscribe. And finally, don’t put pineapple on somebody’s pizza unless they specifically ask for it.


