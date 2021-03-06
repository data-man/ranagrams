# rana
yet another anagram algorithm, this time in Rust

## Usage

The text provided by `--help-long`.
```
USAGE:
    rana [FLAGS] [OPTIONS] <word>

FLAGS:
    -w, --words-in     Returns the set of words composable from the letters in the input phrase
        --strict       When finding --words-in, returns only words that occur in some anagram
        --prove        Like --strict, but emits a phrase proving this word occurs in an anagram.
    -h, --help         Prints help information
        --help-long    Prints *detailed* help information
    -C, --no-cache     Do not cache partial results (this saves memory and costs speed)
    -r, --random       (Partially) shuffle order of discovery
        --ribbit       Ego sum
    -V, --version      Prints version information

OPTIONS:
    -d, --dictionary <file>          A line-delimited list of words usable in anagrams [default: ~/.anagram-dictionary.txt]
    -x, --exclude <word>...          Exclude this word from anagrams
    -i, --include <word>...          Include this word in the anagrams
    -l, --limit <n>                  Only find this many anagrams
    -m, --minimum-word-length <n>    Words in anagrams must be at least this long
    -t, --threads <n>                The number of threads to use during anagram collection [default: 8]

ARGS:
    <word>...    The words for which you want an anagram

Rana generates all the possible anagrams from a given phrase and
dictionary. Note "given some dictionary." Rana does not have a word list
built in. You must tell it what words it may use in an anagram. I have made
myself such a list out of a list of English words I found on the Internet from
which I delted all the words likely to offend people. By default rana will
look in your home directory for a file called .anagrams-dictionary.txt.

In many cases a simple phrase will have hundreds of thousands or millions of
anagrams, setting aside permutations. The phrase "rotten apple", for example,
with a fairly ordinary dictionary of of 109,217 English words, produces 2695
anagrams. Here are 10:

    pone prattle
    plea portent
    pole pattern
    portent pale
    platter pone
    planter poet
    potent paler
    porn palette
    pron palette
    poler patent

Because so many anagrams are available, you are likely to want to focus your
search. Rana provides several options to facilitate this.

--words-in

This will list all the words in your dictionary composable from some subset of
your phrase. You likely want to add --strict to this, so you only get words that
occur in *some* anagram. The --strict version is slower. If you want to verify
that each word listed has some anagram, you can add --prove, or replace --strict
with --prove. This will cause rana to emit the remaining words in an anagram
using the word in question.

--exclude

Discard from your word list particular words.

--include

Include only those phrases which include particular words.

--limit

Only provide a sample of this many phrases.

--random

Shuffle the search order over partial results while searching for anagrams. This
does not provide a fully random sample of the possible anagrams, since only the
results found at any point are shuffled, not all possible results, but this is
a decent way to look at a sample of anagrams when the phrase you've fed in has
many thousands of results. This is particularly useful when paired with --limit.

Caching and Threads

Rana by default uses as many processing threads as there are cores on your
machine. Generally this is what you want, but if you've got a lot of other
things going on, you can limit the number of available threads to reduce the
load your kernel has to deal with.

Rana also uses a dynamic programming algorithm to reduce the complexity of
finding algorithms for large phrases. This is probably unnecessary for short
phrases, though rana provides no lower limit. For larger phrases, like the
complete alphabet, the cache used by the dynamic programming algorithm may grow
so large that the process crashes. If you turn off the cache rana will use
a constant amount of memory, though it may take considerably longer to find all
anagrams.

Text Normalization

Rana attempts to strip away certain characters from your word list and all
other textual input, so it will treat "c-a-t" and " C A T " the same as "cat".
Here is the actual code that does this:

    pub fn normalize(word: &str) -> String {
        word.trim()
            .to_lowercase()
            .chars()
            .filter(|c| c.is_alphabetic())
            .collect::<String>()
    }

I have not tested what this will do for something like ß or Í. You may want to
normalize the text yourself before you give it to rana.

NOTE:

The caching algorithm treats character counts as long base-10 numbers. So, for
example, "cat" might be 111 if "c" is 100s, "a" is 10s, and "t" is 1s. Then
"cad" might be 1110 -- "d" is 1000s, and there is one of them, but there is no
"t", so the 1s place is 0. If you have a phrase, like

    "Dhrtaraashtra uvaaca, dharmakshetre kurukshetre samavetaa yuyutsavah"

which has 15 a's, the "a" count has to be represented as 5 -- 15 mod 10 is 5. In
other words, there isn't space in the a's column of the number for all the a's
in the phrase. It is unlikely that this will ever cause trouble, because for a
particular phrase you are unlikely to be encounter two sets of character counts
during processing that have different counts but the same code. Also, a phrase
this size will consume so much cache space that the process will probably crash
before you encounter this collision.

Another consideration with caching is that this scheme can only accommodate
alphabets up to 38 characters in size.
```

An example use:

```
~ $ rana eat
eat
a et
eta
ate
tea
```

## Installation

Rana is available on `crates.io`, so you can install it with a

    cargo install ranagrams

You can also clone or copy this project to your machine and run
`cargo build --release` in its directory. This will produce an executable called
`target/release/rana`. To use the executable you will need a word list. I have
not checked mine in. You may have a file named something like
`/usr/share/dict/words` on your computer. If so, this will work, though it may
have every individual letter in it along with other words, which means if you
use it as your dictionary that `c a t` will be an anagram of `cat`. You can find
word lists on-line if you simply search for "word list". The list will need to
have all the forms you are interested in. Rana cannot infer "cats" from "cat",
much less "brought" from "bring".

## History and Credits

I have made many variants of this anagram algorithm. This is the first in Rust.
It is more efficient and faster than any of the previous versions. I cannot say
that this is particularly good or idiomatic Rust. It is the fist significant
bit of Rust I've ever written. To the extent that this is good Rust, the credit
is entirely due to @TurkeyMcMac, who knows Rust much better than I do and could
generally tell me when I was doing something particularly stupid.
