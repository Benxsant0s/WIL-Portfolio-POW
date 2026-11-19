# Haskell Wordfreq

A tiny Haskell command-line tool that prints the most common words in a text.

Give it a file (or pipe text in on stdin) and it prints the top *N* words with their counts, most frequent first.

```console
$ echo "the cat and the hat and the bat" | cabal run -v0 wordfreq
the            3
and            2
bat            1
cat            1
hat            1
```

## Features

- Reads from a file or from standard input
- Case-insensitive: `The` and `the` count as the same word
- Ignores punctuation and digits; keeps apostrophes so contractions like `it's` stay intact
- Deterministic output: ties in count are sorted alphabetically
- Pure, testable core logic separated from I/O, with a dependency-free test suite

## Requirements

- [GHC](https://www.haskell.org/ghc/) 8.10 or newer (the package requires `base >= 4.14`)
- [Cabal](https://www.haskell.org/cabal/) 2.4 or newer

The easiest way to install both is [GHCup](https://www.haskell.org/ghcup/).

## Getting started

Clone the repository and build:

```console
git clone <your-repo-url>
cd wordfreq
cabal build
```

Run the tests:

```console
cabal test
```

## Usage

```
wordfreq [FILE [N]]
```

| Argument | Description                                   | Default       |
| -------- | --------------------------------------------- | ------------- |
| `FILE`   | Path to a text file. Omit to read from stdin. | stdin         |
| `N`      | How many top words to show (whole number).    | `10`          |

### Examples

Top 10 words in a file:

```console
cabal run wordfreq -- README.md
```

Top 5 words in a file:

```console
cabal run wordfreq -- README.md 5
```

Read from stdin (top 10):

```console
echo "the cat and the hat and the bat" | cabal run -v0 wordfreq
```

> **Tip:** `--` separates Cabal's own flags from your program's arguments, and `-v0` silences Cabal's build output so only the word counts are printed.

To install the executable onto your `PATH`:

```console
cabal install exe:wordfreq
wordfreq myfile.txt 20
```

### Errors

| Situation                          | Result                                              |
| ---------------------------------- | --------------------------------------------------- |
| `N` is not a whole number          | Exits with `N must be a whole number`               |
| More than two arguments            | Exits with a usage message                          |

## How it works

Text goes through three steps, all implemented as small pure functions in `WordFreq`:

1. **`tokenize`**: lowercases the text, replaces anything that isn't a letter or apostrophe with a space, and splits on whitespace.
2. **`countWords`**: tallies occurrences of each word in a `Data.Map`.
3. **`topN`**: sorts by count (descending), then alphabetically, and takes the first *N*.

## Using it as a library

The `WordFreq` module is exposed by the package, so you can use it from your own Haskell code:

```haskell
import WordFreq (topN)

main :: IO ()
main = print (topN 2 "a b a c b a")
-- [("a",3),("b",2)]
```

| Function                              | Description                                         |
| ------------------------------------- | --------------------------------------------------- |
| `tokenize :: String -> [String]`      | Lowercase and split text into words                 |
| `countWords :: [String] -> Map String Int` | Count occurrences of each word                 |
| `topN :: Int -> String -> [(String, Int)]` | The *N* most frequent words, highest first     |

## Project layout

```
wordfreq/
├── app/Main.hs        # command-line handling and output formatting
├── src/WordFreq.hs    # pure logic: tokenize, count, rank
├── test/Spec.hs       # dependency-free test suite
└── wordfreq.cabal     # project definition
```

## Testing

Tests live in `test/Spec.hs` and use a small hand-rolled `check` helper, so there are no extra test dependencies. They cover tokenization, ranking by count, alphabetical tie-breaking, and empty input.

```console
cabal test
```
