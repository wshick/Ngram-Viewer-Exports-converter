# Ngram Viewer Exports corpora to ngram frequency corpus
Script for converting [Google Books Ngram Viewer Exports V3](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) [American English V20200217 1-grams](https://storage.googleapis.com/books/ngrams/books/20200217/eng-us/eng-us-1-ngrams_exports.html) from a word-frequency-by-year-since-1472 corpus to a word-frequency-since-2000 corpus, with other changes to reduce file size and better facilitate analysis of n-character frequencies later.

This script is probably compatible with other English Ngram Viewer Exports corpora with minor modification--and with other-language corpora with more modification--but this is untested. If you're reading this, you almost certainly know more Python than me, so you can figure it out for your use case.

## V3 1-grams File Format
What I may refer to as a "1-gram" is really
```
1gramTAByear,match_count,volume_countTAByear,match_count,volume_count[...]NEWLINE
```
as defined in Google's V3 corpora, where
- `1gram` = 1 "word" (consecutive non-whitespace characters),
  - *Google's usage of "-gram" as "word" is less common than the typical usage of "-gram" as "character".*
- `match_count` = number of `1gram` occurrences per year, and
- `volume_count` = number of sources `1gram` occurred in per year.

## Instructions (WIP)
1. download python
2. download script
3. commandprompt

---

# Deprecated notes
*The following notes ended up irrelevant to my final implementation, but I found them worth knowing anyway.*

## Part-of-Speech Tags
from https://books.google.com/ngrams/info

|       Tag | Part of Speech     |
| ---------:|:------------------ |
|    `_ADJ` | adjective          |
|    `_ADP` | adposition         |
|    `_ADV` | adverb             |
|   `_CONJ` | conjunction        |
|    `_DET` | determiner/article |
|   `_NOUN` | noun               |
|    `_NUM` | numeral            |
|    `_PRT` | particle           |
|   `_PRON` | pronoun            |
|  `_PROPN` | proper noun        |
|   `_VERB` | verb               |
| `_START_` | sentence start     |
|  `_ROOT_` | root of parse tree |
|   `_END_` | sentence end       |

- There also seems to be an undefined speech tag `_X` in the corpus.

```python
# Speech tag removal

print("      Discarding 1-grams speech-tagged as numerals...")
filecontent = re.sub(r'(?m)^\S*_NUM\t.*$', "", filecontent)

print("      Discarding other speech tags...")
filecontent = re.sub(r'(_NOUN)|(_PROPN)|(_VERB)|(_ADJ)|(_ADV)|(_PRON)|(_DET)|(_ADP)|(_CONJ)|(_PRT)\t', "\t", filecontent)

print("      Discarding undefined speech tag `_X`...")
filecontent = re.sub(r'_X_?\t', "\t", filecontent)
```
