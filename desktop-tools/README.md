# Desktop tools

*Last update: 10/7/2021*

Here is a list of tools for the desktop. Unless otherwise noted, these tools are supported only on MacOS.

## srcfind

* Type: CLI (zsh script)
* Environment: MacOS

#### What it does

`srcfind` performs a faceted expression search on plain files in a directory hierarchy through the use of multiple calls to `egrep`.  This provides a convenient means of locating all files that contain occurrences of multiple strings. For example, suppose you had a directory of source code. Two files in the directory are:


| Name               | Contains                     |
| :------------------- | ------------------------------ |
| src/array-utils.js | arr.forEach()                |
|                    | // call forEach              |
|                    | // iterates through array    |
| app/search.js      | items.forEach(v=>v)          |
|                    | const iterate = (items) => { |

You needed to find all files that contain both the strings `forEach` and `iterate`.

```
# List all files that contain the strings "forEach" and "iterate"

egrep -l $(find . -type f -print | egrep -l forEach) iterate

# if the above works right, it will output because both files contained both strings..
./src/array-utils.js 
./app/search.js 
```

but this is a lot to type with a lot of commands and options to remember. Plus, this has added problems:

* You will search files you probably don't care about, such as hidden files and directories, software installation folders, and binary files
* This will break if file names have spaces
* You run some risk of exceeding the number of arguments allowed in a command

`srcfind` provides a driver for the same operation that saves you typing and avoids these difficulties.

```
srcfind forEach iterate

# outputs same as above
./src/array-utils.js 
./app/search.js
```

In the above case, `srcfind` will run the equivalent of:

```
find . \ 
    -type d '(' -name node_modules -o -name dist -o -name '.?*' ')' -prune \
    -o '(' -type f ! -name '.*' ! -name package-lock.json ! -name yarn.lock ! -name '*.log' ')' -print0 |
xargs -0 egrep '--binary-files=without-match' --files-with-matches foreach
```

The extra arguments to `find` and `egrep`, as the application of the file lists through the use of `xargs` with NULL bytes, provides the features that avoid the problems.

#### Features

If you want to supply your own options to `egrep`, you can provide them after the search patterns. For example, to find the occurrences of `forEach` and `iterate` that are individual words, you can provide `-w`.  Add flags will have the added function of showing you the matches.

```
srcfind forEach iterate -w

# outputs only app/search.js, which contains 'iterate' and 'forEach' as words.
# array-utils.js is not included because it contains no word-bound occurrence of 'iterate'
./app/search.js:items.forEach(v=>v)
./app/search.js:// const iterate = (items) => {
```

If you want to see the matches without providing a flag to `egrep`, just add '--' to the command line:

```
srcfind forEach iterate --

# outputs
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

### Caveats

Right now, this is utility is specialized to Node projects.  We can keep going.
