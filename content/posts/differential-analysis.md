+++
date = "2026-10-22"
draft = true
title = "Differential Analysis"
+++

Differential analysis is my favorite testing method, right alongside fuzzing, which is my favorite testing method. Unfortunately, I've found most software projects don't seem to take testing very seriously, even when the developers or maintainers believe they're taking testing seriously.

Dan Luu has an excellent writeup on this, dated March 2015: ["
Given that we spend little effort on testing, how should we test software?"](https://danluu.com/testing/). If you think that's too old and therefore probably irrelevant because AIs can do all the testing now, here's his September 2026 writeup on that: ["How well do agents use test and verification techniques?"](https://danluu.com/agentic-testing/).

I won't repeat everything he said. Go read his words — they mirror my own thoughts quite well. (Why yes, I *do* use em dashes, and I was doing it before it became an AI vibeslop smell. I'm not going to stop just because some clanker makes people think they can't write fluent English anymore.)

The TL;DR summary is this:
 * software testing standards are mediocre versus hardware testing standards
 * throwing random inputs at software essentially *always* finds bugs
 * current AIs have no idea how to test software effectively, even with human guidance

If you care about testing at all, you've probably heard of fuzzing already (and if not: go try it!). That's why this post is about differential analysis, not fuzzing. The two techniques go together like peanut butter and chocolate, and it's so natural that I personally view them as parts of the same unified technique.

Differential analysis is when you take multiple implementations of the same thing, and pit them against each other. My favorite example of this is ["Parsing JSON is a Minefield"](https://seriot.ch/security/parsing_json.html), from October 2016 to March 2018. In that experiment, Nicolas Seriot wrote specific tests by hand, generating a nifty comparison matrix showing when a parser conforms to the JSON standard, by the following metrics:
 * does it accept/reject valid input?
 * does it accept/reject invalid input?
 * how does it handle implementation-defined behavior?

The way you combine that with fuzzing is conceptually pretty simple:
```python
def compare(input):
	output1 = parser1(input)
	output2 = parser2(input)
	assert(output1 == output2)
```
Imagine `parser1` is Rust's [Serde JSON](https://github.com/serde-rs/json), `parser2` is .NET's [Newtonsoft.Json](https://github.com/JamesNK/Newtonsoft.Json), and the input is `[1,2`. The comparison function crashes, triggering the fuzzer (TODO: report issue and link it). Oops.

Now imagine the input is (TODO: float parsing issue). Crashed again. Oooops.

The funny thing about modern AI assistance is that I didn't even need to set up the analysis myself. I had Claude Fable 5.1 download a bunch of JSON parsers and run each one's test suite against all of them. I did it with PNG decoders to, and oh boy are there a lot of bugs lurking in that realm.
