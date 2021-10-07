# Desktop tools

*Last update: 10/7/2021*

Here is a list of tools for the desktop. Unless otherwise noted, these tools are supported only on MacOS.

## srcfind

* Type: CLI (zsh script)
* Environment: MacOS

This performs a faceted expression search on plain files in a directory hierarchy through the use of multiple calls to `egrep`.  This provides a convenient means of locating all files that contain occurrences of multiple strings. For example, suppose you had a directory of source code and you needed to find all files that contain both the strings `forEach` and `iterate`.  You can enter this command:

```
# List all files that contain the strings "forEach" and "iterate"
# options here provide skipping of node_modules folder, consider only plain,
# non-binary files, support for file names with spaces, etc.
find . -name node_modules -type d -prune -o -type f --binary-files=without-match -print0 |
xargs -0 egrep --files-with-matches --null forEach |
xargs -0 egrep --files-with-matches iterate

# outputs
 # contains 'arr.forEach()', '// call forEach', '// iterates through array' on separate lines
./src/array-utils.js 

# contains 'items.forEach(v=>v)', 'const iterate = (items) => {' on separate lines
./app/search.js 
```
but this is a lot to type with a lot of commands and options to remember. `srcfind` provides a driver for the same operation.  With `srcfind`, you can shorten this greatly:
```
srcfind forEach iterate

# outputs same as above
```
If you want to supply your own options to `egrep`, you can provide them after the search patterns. For example, to find the occurrences of `forEach` and `iterate` that are individual words, you can provide `-w`.  Add flags will have the added function of showing you the matches.
```
srcfind forEach iterate -w

# outputs
# contains 'iterate', 'forEach' as words (no word 'iterate', only 'iterates' in array-utils.js)
./app/search.js:items.forEach(v=>v)
./app/search.js:// const iterate = (items) => {
```
If you want to see the matches without providing a separate flag, just add '-' to the command line:
```
srcfind forEach iterate -

# outputs
# contains 'iterate', 'forEach' (matches 'iterates' in array-utils.js)
./src/array-utils.js:arr.forEach()
./src/array-utils.js:// call forEach
./src/array-utils.js:// iterates through array
./app/search.js:items.forEach(v=>v)
./app/search.js:// const iterate = (items) => {
```
If you want to provide flags without seeing the matches, just add '-l'.
```
srcfind forEach iterate -w -l

# outputs
./app/search.js
```
Because this script delegates its matching to `egrep`, the `egrep`-style of regular expression syntax is fully supported.  All the flags to that program are generally supported as well.