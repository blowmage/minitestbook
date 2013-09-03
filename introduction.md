Introduction
============

Why Testing?
------------

Have you ever had to maintain an application or library and were utterly and completely unable to make sense of it? Have you ever tried to add a feature to an existing codebase but every change you made broke the app in multiple places? Have you noticed that the older a codebase is the harder it is to make changes? Have you ever found yourself several days into a task only to come to the conclusion that the available documentation and comments in the code were horribly outdated at best and incompetent at worst?

I've done all of that. And I've done it to teammates and myself. I used to think that if I could get better at thinking like a computer or acquire a deeper understanding of design patterns I could cure myself (and others) from making such a mess in the code. But these approaches only addressed the surface issues. The core problem was that the code I was working in and contributing was lacking design. Not necessarily lacking design on the high-level architecture scale, but at the micro scale. Lines and methods. Which, coincidentally, is where we spend most of our time.

Why Minitest?
-------------

Minitest is a full-featured testing library that comes in Ruby's standard library. It provides the classical testing API, a spec DSL, mocks, stubs, and performance testing. Minitest is fast. Not only in running tests, but in writing and maintaining them. Minitest is also a considerably smaller codebase than other testing libraries, which means its easier to understand and extend.

Minitest also carries a philosophy that informs it's design. Like Ruby, it acknowledges that there is usually more than one way to accomplish a task. [Something about flexibility vs. guidance.] Minitest is more than a testing library. It embodies a specific philosophy for testing code. It also embodies a larger philosophy of how software should be designed. (Unix, TDD, XP) Minitest embodies these philosophies, but that doesn't mean you can't use Minitest if you have a different view. Minitest is intended to be used, and so it doesn't force these views on you. But, if you are receptive, you will find Minitest is a great ally.

Versions
--------

This book is written using Ruby 2 and Minitest 5. Ruby 2 comes with Minitest 4.3.2, which means you will have to install the latest Minitest from rubygems.

```shell
$ gem install minitest
Fetching: minitest-5.0.6.gem (100%)
Successfully installed minitest-5.0.6
1 gem installed
```

Some details are different between Minitest 5 and 4.3.2, but the general approach is the same.
