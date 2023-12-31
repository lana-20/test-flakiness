# Test Flakiness, a.k.a. Non-Determinism

A common criticism of GUI tests is that they are "flaky". What does this mean, and how can this challenge be addressed?

If you've spent some time in the test automation world professionally, by now you've certainly heard the accusation that the kind of tests we've been learning about in this course, namely UI tests, are often unstable. People use the term "flaky" to describe this. What does flakiness really mean, where does it come from, how should we think about it, and what can we do about it?

What is Flakiness? |
---- |
Flakiness  =  a test passing one time you run it, then failing another time (without the app code or the test code having changed) |
Also known as "non-determinism" |
Usually, "flaky" or "non-deterministic" means "I don't know what the problem is" |
There's always a reason a test fails erroneously, even if we don't understand it. ∴ A "flaky" test is really the one that passes some times and fails the others, for reasons we don't **yet** undestand. |

Dangers of Flakiness |
---- |
Flakiness is a rather big concern. We don't want our build to be full of the "tests that cry failure". Tests that fail erroneously lead to the lack of quality signal and lack of trust in the testsuite |
Miniscule levels of test instability can be magnified under normal CI circumstances |

Flaky Math  |
---- |
Test cases: 100  |
Average stability rating: 99.9%  |
Environments: 2 Android, 2 iOS, 2 models each  |
Total tests per build: 800  |
Chance of Flake per build: 80%  |
***4 out of 5 builds contaminated by a flake***  |

## Causes of Flakiness
| Cause |  Solution |
| ---- | ---- |
| Race Conditions | Use *Explicit Waits* to ensure your test has verified the app state before taking actions |
| Erroneous test assumptions | Run lots of tests in lots of environments to flush out the potential issues before considering your test 'Done' |
| External instability (3rd party services, etc.) | Isolate or mock the external services as far as possible |
| App flakiness | Send screenshots, videos, and device logs to the developers |

Common Recommendations  |
---- |
Take a zero tolerance approach to flakiness, as far as possible. I.e., do not tolerate asinine ignorance when it comes to erroneous failures |
From the very get-go, use *Best Practices* |
Establish processes to protect your build from gradual contamination by flakiness -- try a "new test purgatory" |


When people say a test is flaky, what do they mean? In the most concise way of putting it, what they mean is that a given test might pass one time you run it, and then fail another time you run it, even if neither the app code nor the test code has changed. This would obviously be a frustrating circumstance! You spend an hour writing your test and having it work, only to find that when you run it in a CI system, it suddenly fails 1%, or 10%, or 50% of the time!

Another word you might hear describing the same scenario is the word "nondeterministic". Basically, you don't know when you run your test whether it will certainly pass or fail, even if the app code and test code remain the same.

But hold on a second here. Is it really nondeterministic? We're dealing with computers after all... Isn't everything that happens with computers precisely determined? More or less. And in fact, when people talk about flakiness or nondeterminism, what I've noticed is that most often what they mean is that they just don't know why something appears unstable. There's no reason that's obvious to them.

We need to be careful here. Just because we can't think of an obvious reason that a test would sometimes pass and sometimes fail, doesn't mean there isn't one. I've seen lots and lots of testers and developers just throw up their hands and call an Appium or Selenium test "flaky" without bothering to actually investigate the cause. We'll return to this point momentarily. But for now, let's set a new definition for flakiness: a flaky test is one that sometimes passes and sometimes fails, for reasons you don't yet understand.

Well, so what? Is flakiness that big of an issue? What if a test only fails 1 time in 100? Is it worth spending any time figuring out why it fails that 1 time?

The answer is usually yes. Allowing any instability in your build is usually a very bad idea. Many of you might recall the story of the 'boy who cried wolf'. To transpose that to our industry, we could talk about the 'test that cries failure'. One problem with flakiness is that a flaky test begins to lose its ability to signal anything about the quality of the feature it's covering. When you've seen a test fail erroneously in the past, it's very tempting to regard any future failures as erroneous as well, and then you're better off not having the test at all!

The other issue is that even when a test has a tiny degree of instability, that becomes multiplied when you consider the context of a build that runs many tests on many different platforms. Let's do a bit of math. Let's say you have a testsuite with 100 different testcases, and that each test has a stability rating of 99.9%, meaning it only fails 1 out of every 1,000 times it's run. Seems pretty stable, right? Now imagine that your build runs testcases on 2 Android versions, 2 iOS versions, and 1 phone and tablet form factor for each of those versions. That means the total number of tests run per build is 800, because each testcase gets run on 8 different devices at the end of the day. Given that each test has a 0.1% chance to fail, and that we're essentially giving 800 chances, that means that there's an 80% chance that at least one test in the build will fail erroneously. This in turn means that 4 out of 5 of your builds will require investigation into a failure which turns out to be a false positive! And how much trust do you think the development team will have in a build that seems to fail more often than not even when nothing is wrong? So you can see, even tiny levels of instability get multiplied when we talk about running tests at any kind of scale.

So we've covered the dangers of flakiness, but what about the causes of it? Once we start digging into the unknown and spend some time hunting down the underlying causes, we end up with an array of options:

1. One of the primary causes of instability that turn out to be fixable from the perspective of the test code is the category of the race condition. We've spent a lot of time talking about race conditions in this course, because they're one of the most important things to be able to sense and investigate. Again, a race condition is when your test assumes that the app is in a certain state, but the app has not yet reached that state (or has reached it and already moved on). The use of explicit waits to check for various app conditions is usually a good strategy, assuming you've clarified the exact nature of the race condition.
2. More generally, there is the category of bad test assumptions. This is when your test code assumes something which is just not the case. Race conditions are only the main example. There are others, for example assuming a certain screen size. This matters especially on Android, where elements that are not on the screen actually don't exist. Your test might reliably find a certain element over and over again, and then when run on a different device type, not be able to find the element at all. Running your test on a variety of platforms and device varieties before committing it to your build is a good way to flush out these kinds of assumptions. This is a practice that we call "test hardening," meaning allowing it to become more robust by adjusting it in the face of lots of different environments.
3. Another cause of test instability is instability in environments that are external to the test. One of the benefits of UI tests is that they are tests that take place in totally real-world environments. But one of the drawbacks is that the real-world environment is not often as clean and predictable as we would like. For example, many mobile and web apps rely on various webservices to function correctly. But what if there are issues with these backend webservices unrelated to the UI you're trying to test? These issues could show up as all kinds of errors. And if the webservice is under your team's control, that would be a good type of bug to catch. But there are also many 3rd party webservices that apps rely on, whether it's a CDN to deliver assets, or some kind of 3rd party API. What if one of those happens to be down, or decides to block your requests because you have 100 tests trying to hit the API at the same time? There are all kinds of cases like this that leave you scratching your head. The ideal solution to this kind of environmental instability is to find a way not to rely on an external environment at all. We get now into some advanced topics that we're not going to look at in detail, but it's worth looking up the concept of mocking external services. This doesn't mean to make fun of them, it means to make your own version of them that you control and that is guaranteed to work reliably. Basically, if you can find a way to create your own fake test environment that your app runs completely within during the course of your test, then you can address this kind of instability.
4. The last cause of test flakiness that we'll look at is a bit subtle, and we could call it "app flakiness". The purpose of a test is to find bugs in an application. But what happens if an app has bugs that don't reliably show up? Apps are so complex nowadays, it's not uncommon to hear of a bug that shows up semi-randomly. You might have a rock-solid test that fails periodically for no other explanation than that the app didn't do the right thing, even though in manual testing the app cannot be seen to exhibit that behavior at all! If you find yourself in this case, congratulations---you've done a great service for your team! In a way, running many tests of an app are a form not just of functional testing but of stress testing, and you can often flush out rare bugs just by running a lot of tests. The solution to this kind of problem is of course to fix the app!

In general what I'm trying to recommend is that we take basically a zero tolerance approach to flakiness. Not that I mean we'll have a test suite that never has an erroneous failure. But what I mean is that we should never settle for an erroneous failure that we don't understand. At the end of the day, with UI testing being so high-fidelity, there are going to be environmental failures that we can't control. But we need to be very sure that we're dealing with a failure that is out of our control. The only way we can be sure of that is to understand the cause of the failure, so it's our job as testers to make progress in that direction, even if we need to rope in some of the app developers to help us understand.

Of course it helps to use all the best practices we've been discussing and will discuss, when it comes to writing Appium and Selenium tests. These will help eliminate some of the more common causes of instability right off the bat.

For everything else, there are practices you can put in place to help give you a good idea of the stability of your tests, for example having a system whereby new tests that are added to a testsuite first get put into a kind of quarantine or purgatory, where they get run many many times and have to pass a certain success threshold before they're even allowed to contribute to a build. This could keep flaky tests from ever polluting your build to begin with. I definitely recommend doing something like this.

That's all we'll talk about in terms of test instability for now. Keep following the best practices to ensure fast and reliable tests overall!
