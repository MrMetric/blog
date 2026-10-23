+++
date = "2026-10-20"
draft = true
title = "DEFLATE optimization"
slug = "deflate"
+++

Optimizing DEFLATE streams is a topic I've found myself caring about a lot, mainly because I have a lot of files that include DEFLATE streams, and I know a lot of people with whom I need to share such files. Most of the time, that means PNG images and ZIP archives.

For the purposes of talking about it here, I'll take it as a given that optimizing DEFLATE streams is worthwhile.

And, when I say "optimizing", I mean reducing the filesize. We're not talking about improving runtime performance of compressors or decompressors. We *do* care about measuring how long compressors take on sample files, but not for the sake of just picking the fastest one. We'll use the results as an indicator of how long we can expect to wait to achieve some size improvement, since the primary goal is *smaller output*.

There are compression formats where the decompressor can get slower when the compressed data is smaller, but DEFLATE nominally isn't one of those, so we'll completely ignore the decompressor side of things.

That means we can make common files smaller, for Free™, with the only cost being one-time upfront computation. Well... *how much* computation? Turns out, anywhere from a trivial amount to endless grinding, depending on how much you care about diminishing returns. I'm one of those people who will leave an optimizer running for *days*, on a single file! You don't need to do that to see significant savings, though, because most software doesn't do a very good job of compressing data to DEFLATE streams.

Let's see some examples.

[enwik8](https://mattmahoney.net/dc/textdata.html) is a file commonly used in compression benchmarks. It's exactly 100 MB (100,000,000 bytes), and since it's part of an old Wikipedia dump, it has a lot of text, both free-form English prose and basic structured data such as XML. I had Claude Fable 5.1 do some basic testing, and here's what he came back with (times are median of 3 runs; my CPU is an AMD Ryzen 5950X).

| Pipeline                    | Bytes          | Ratio  | Time    |
| --------------------------- | -------------: | -----: | ------: |
| gzip -1                     |   42,265,725   | 42.27% |  1.10 s |
| gzip -6                     |   36,518,322   | 36.52% |  4.03 s |
| gzip -9                     |   36,445,241   | 36.45% |  5.20 s |
| gzip -9 > defluff           |   36,444,728   | 36.44% |  6.72 s |
| gzip -9 > DeflOpt           |   36,444,655   | 36.44% |  6.99 s |
| gzip -9 > defluff > DeflOpt |   36,444,596   | 36.44% |  8.38 s |
| gzip -9 > DeflOpt > defluff |   36,444,597   | 36.44% |  8.50 s |
| gzip -9 > unflate --fast    | **36,438,451** | 36.44% | 33.00 s |

[DeflOpt](https://web.archive.org/web/20140209022101/www.walbeehm.com/download/) is version 2.07, from September 5th, 2007! Sadly, its creator, Ben Jos Walbeehm, died in 2013 ([discussion on encode.su](https://encode.su/threads/455-Ben-Jos-Walbeehm-s-DeflOpt-what-does-it-actually-do)). He never published the source code, nor explained how it works.

[defluff](https://encode.su/threads/1214-defluff-a-deflate-huffman-optimizer) is version 0.3.2, published on April 7th, 2011. Its creator, Joachim Henke, never published the source code, nor explained how it works. I hope he is still in good health.

[unflate](https://github.com/MrMetric/unflate) is my own project. To avoid the curse striking a third time, I've published its source code *before* posting about it anywhere.

When running an optimizer, notice that the result depends on the ordering, and no tool provides the best result in one call. If we start with `gzip -9`, running defluff and DeflOpt exhaustively to find the best result, we get:

| Best ordering               | Bytes      | Saved | Calls | Grew | Tied | Outputs | Search  |
| --------------------------- | ---------: | ----: | ----: | ---: | ---: | ------: | ------: |
| DeflOpt > defluff > DeflOpt | 36,444,593 |   648 |    24 |    2 |   12 |      12 | 35.43 s |

