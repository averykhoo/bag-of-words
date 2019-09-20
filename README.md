﻿#   Aho-Corasick Find & Replace
-   Given a dict from strings to other strings, replaces all occurrences non-recursively
-   Fast because it does a single pass over the target string
-   (Incompletely) based on [Aho-Corasick string search](https://en.wikipedia.org/wiki/Aho–Corasick_algorithm), 
    but modified to perform replacements
    -   Uses a [trie](https://en.wikipedia.org/wiki/Trie) structure *without* suffix links, 
        so that you can dynamically add/remove stuff


##  TL;DR

### Convert list of strings into a regex:
-   Single line: `print(Trie.fromkeys(['pen', 'pineapple', 'apple', 'pencil'], case_sensitive=False).to_regex())`
```python
from find_replace import Trie

strings = ['pen', 'pineapple', 'apple', 'pencil']

trie = Trie.fromkeys(strings, case_sensitive=False)
pattern = trie.to_regex()
print(pattern)  # '(?:(?:apple|p(?:en(?:cil)?|ineapple)))'
```
-   Why is space being changed to match all whitespace ('\s')?
    -   Fuzzy spaces make life easier
    -   To disable, use `trie.to_regex(fuzzy_spaces=False)`
-   Why is the black shape ('\uFFFD') being changed to match any ('.')?
    -   '\uFFFD' is a char that could have been anything
    -   To disable, use `trie.to_regex(fffd_any=False)`
-   What's happening to my curly quotes?
    -   To disable, use `trie.to_regex(fuzzy_quotes=False)`

### Find occurrences of a list of strings
```python
from find_replace import Trie

target = 'I have a pen... I have an apple...'
strings = ['pen', 'pineapple', 'apple', 'pencil']

trie = Trie.fromkeys(strings)
matches = list(trie.find_all(target))  # `find_all` returns a generator
print(matches)  # ['pen', 'apple']
```
-   How do I make it case insensitive?
    -   Use `trie = Trie.fromkeys(strings, case_sensitive=False)`
-   How do I find overlapping matches?
    -   Use `trie.find_all(target, allow_overlapping=True)`

### Replace occurrences of some strings with other strings
```python
from find_replace import Trie

target = 'I have a pen... I have an apple...'
replacements = {'pen': 'pineapple', 'apple': 'pencil'}

trie = Trie(replacements)
output = trie.process_text(target)
print(output)  # 'I have a pineapple... I have an pencil...'
```
-   How do I make it case insensitive?
    -   Use `trie = Trie(replacements, case_sensitive=False)`
-   Why is it finding substrings (e.g. "java" in "javascript")?
    -   That's just what a string search should do...
    -   To match at word boundaries, specify a tokenizer when creating the trie
        -   First: `from find_replace import space_tokenize` <-- tokenizes on spaces and punctuation
        -   Then: `trie = Trie(replacements, lexer=space_tokenize)` 

### Replace occurrences of some strings with a specific string
```python
from find_replace import Trie

target = 'I have a pen... I have an apple...'
strings = ['pen', 'pineapple', 'apple', 'pencil']

trie = Trie.fromkeys(strings, 'orange')
output = trie.process_text(target)
print(output)  # 'I have a orange... I have an orange...'
```
-   How do I remove instead of replace?
    -   Use `Trie.fromkeys(strings, '')`

##  Advanced Usage

-   `update`: add/modify replacements, 
              e.g. `trie.update({'apple', 'orange', 'orange': 'apple'})`
-   `__setitem__`: add/modify replacements, 
                   e.g. `trie['pen+apple'] = 'apple pen'`
-   `__getitem__`: see what you've inserted, 
                   e.g. `print(trie['pen'])` or `print(trie['apple':'orange'])`
-   `process_file`: replace stuff in some text file to a new file, 
                    e.g. `trie.process_file(input_path, output_path)`
-   `__delitem__`: remove a string-replacement pair, 
                   e.g. `del trie['pen']`

##  To-Do
-   refactor code into multiple files?
-   parallelize file processing to make processing faster, sharing a single trie
-   split into find-only and find+replace
-   unicode_tokenize (like [fts5](https://sqlite.org/fts5.html#unicode61_tokenizer))
-   find a way to convert trie to DFA by computing suffix/failure links, 
    while still allowing in-place updates to the trie

