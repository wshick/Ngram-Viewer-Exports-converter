# Ngram Viewer Exports American English V3 to usable character frequency corpus
Script for converting [Google Books Ngram Viewer Exports V3](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) [American English V20200217 1-grams](https://storage.googleapis.com/books/ngrams/books/20200217/eng-us/eng-us-1-ngrams_exports.html) from a word-frequency-by-year-since-1472 corpus to a word-frequency-since-2000 corpus, with other changes to reduce file size and better facilitate analysis of n-character frequencies later.

This script is probably compatible with other English Ngram Viewer Exports corpora with minor modification--and with other-language corpora with more modification--but this is untested. If you're reading this, you almost certainly know more Python than me, so you can figure it out for your use case. (This isn't sardonic--this is literally the second thing I have ever made with Python.)

## V3 1-grams File Format
What I may refer to as a "1-gram" is really
```
1gram TAB year,match_count,volume_count TAB year,match_count,volume_count [...] NEWLINE
```
as defined in Google's V3 corpora, where
- `1gram` = 1 "word" (consecutive non-whitespace characters),
	- *Google's usage of "-gram" as "word" is less common than the typical usage of "-gram" as "character".*
- `match_count` = number of `1gram` occurrences per year, and
- `volume_count` = number of sources `1gram` occurred in per year.

Each corpus file is *millions* of 1-grams. >99% of this data is useless to letter frequency analysis for keyboard layouts; hence the need for this script.

## Converted File Format, and how we got there
After running this script, the converted file format is simply
```
1gram TAB total_matches NEWLINE
```
where `total_matches` is the sum of all the `1gram`'s `match_count`s after 1999. However, the script also removes a lot of other data to increase relevance for keyboard layout generation and significantly reduce file size:
1. `year,match_count,volume_count`s are removed if `year`<2000. 1-grams with no `match_count`s because of this operation are removed.
	- My opinion is that character frequencies pre-2000 are largely irrelevant to typists of today (as of writing this, 2025).
2. `1gram`s containing non-ASCII characters, digits, and underscores are removed.
	- Non-ASCII characters are uncommon enough in American English that I decided they'd be irrelevant for character frequencies, especially since most US keyboards can't type them.
	- Digits within `1gram`s are almost always Optical Character Recognition (OCR) errors on Google's part. Their software sometimes misidentifies `O`s as `0`s, `l`s as `1`s, and so on.
	- Removing 1-grams containing underscores is an overly-broad method of removing Speech Tags (detailed in the final section). I seem to have discovered that speechtagged 1-grams are subsets of non-speechtagged 1-grams, so they wouldn't be necessary to count separately.
3. `1gram`s not containing letters are removed.
	- There's a lot of junk data, like ~20 lines of increasing numbers of `.`s.
4. `year`s and `volume_count`s are removed, and all the remaining `match_count`s for each `1gram` are summed to `total_matches`.
5. All letters in `1gram`s are lowercased, and duplicates are combined into one 1-gram with their `total_matches` summed.
	- Case is irrelevant because capital and lowercase letters occupy the same keys on a keyboard, on different layers (unshifted vs shifted).
6. All 1-grams are sorted.

## Instructions (WIP)
0. Download [Python](https://www.python.org/downloads/).
1. Download all the compressed corpus files from [American English V20200217 1-grams](https://storage.googleapis.com/books/ngrams/books/20200217/eng-us/eng-us-1-ngrams_exports.html) EXCEPT "<u>Total counts for 1-grams</u>".
	- To choose another corpus, go to [Google Books Ngram Viewer Exports V3](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html)--but this will almost certainly require modification of the script to get something usable and useful.
2. Extract all the compressed corpus files.
3. Create a single NEW, EMPTY folder anywhere convenient, such as your Documents or Desktop. Place all the extracted corpus files in it.
	- DO NOT PUT ANYTHING OTHER THAN THE CORPUS FILES IN THIS FOLDER. This script iterates on all untyped files in that folder and all subfolders, and is highly destructive.
4. Download [NgramViewerExportsEnUSv3converter.py]().
5. Open your command prompt / terminal / powershell and run `cd ` to whatever path you stored the *script*. Then, run `python NgramViewerExportsEnUSv3converter.py`.
	- The script will ask you to input the path of the folder you put the corpus files in. DO NOT USE THIS SCRIPT ON ANY OTHER FOLDER. Again, this script iterates on all untyped files in that folder and all subfolders, and is highly destructive.
6. Watch it go! It'll take a while.

When the script is finished running, you'll end up with a corpus <1% the size of the original files.

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
