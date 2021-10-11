# Desktop tools

*Last update: 10/11/2021*

Here is a list of tools for the desktop. Unless otherwise noted, these tools are supported only on MacOS.

---

---

---



## srcfind

* Type: CLI (zsh script)
* Environment: MacOS
* Provided for: faceted search of source files in Node JS
* Current version: 0.1 (alpha)

### What it does

Developers often search through source files to find strings in an effort to locate functions.  However, a search for just one string often returns too many matches to be useful.  It can often be true that we need find source files that contain all of more than one string to pinpoint the function we are seeking.

For example, searching through files to find an occurrence of the `lodash` function `reduce` in an effort to convert the source from using `lodash`'s versions to the ES6 version, a search for just `lodash` would probably return a lot of files that you don't want to search through by hand.  The same would be true if you searched for `reduce`.  So one way to make the search more specific is to search for `lodash.reduce` or `_.reduce`.  This, however, would likely miss some occurrences of the functions, since the just the function `reduce` can be imported from the `lodash` individually, which means the call would work as just `reduce`.

As in this case, to find the instances of functionality in a source hierarchy, it is often necessary to search for occurrences of more than one term in any source file before it can be considered likely to contain the searched function.  `srcfind` does this for you.

___

### Example - why faceted search is tough with  standard UNIX commands

Here is an example of files to search where the below lists the only lines they have that match `forEach` and `iterate`:


| Name               | Contains                     |
| :------------------- | ------------------------------ |
| src/array-utils.js | arr.forEach()                |
|                    | // call forEach              |
|                    | // iterates through array    |
| app/search.js      | items.forEach(v=>v)          |
|                    | const iterate = (items) => { |
| test/test all.js   | \_.forEach(item1)            |

You can search the files that contain occurrences of both string using a fairly simple UNIX command line:

```
# List all files that contain the strings "forEach" and "iterate"

egrep -l $(find . -type f -print | egrep -l forEach) iterate
```

If the above command works right, it will output both the files because they each contain both strings:

```
./src/array-utils.js 
./app/search.js 
```

But the above has issues, beyond having to type a long command, typo-vulnerable command line:

* You will search files you probably don't care about, such as hidden files and software installation folders (e.g., `node_modules`, `.github/**`, `.npmrc`).
* The above will match binary files
* The above command will break if file names have spaces.
* You run some risk of exceeding the number of arguments allowed in a command if the `find` command returns too many path names.

In this case, we have a file space in its name, 'test/test all.js', that contains `forEach`.  Because of the space, the above command would not work quite right. The expansion of the `find` command listed the file names with the spaces, but the shell would not treat that specially in the result.  This would mean that the outer `egrep` would not evaluate `./test/test all.js` as a single file. 

```
./src/array-utils.js 
./app/search.js 
./test/test: No such file or directory
all.js: No such file or directory
```

If `test/test all.js` contained `iterate` as well as `forEach`, it would not have been included in the results.
___
### srcfind hides the complexity & avoids the issues

`srcfind` provides a driver for `egrep` and `find` that will match the command above but eliminate the issues descibed above.  With `srcfind`, you need only the strings you seek.

```
srcfind forEach iterate

# outputs same as above, with no errors on "test/test all.js"
./src/array-utils.js 
./app/search.js
```

In the above case, `srcfind` will run the equivalent of:

```
find . \ 
    -type d '(' \
      -name node_modules \
      -o -name dist -o \
      -name '.?*' \
    ')' -prune \
  -o '(' \
  -type f \
    ! -name '.*' \
    ! -name package-lock.json \
    ! -name yarn.lock \
    ! -name '*.log' \
  ')' -print0 |
xargs -0 \
  egrep \
    --binary-files=without-match \
    --files-with-matches \
    --null \
    foreach
xargs -0 \
  egrep \
    --files-with-matches \
    iterate
```

The extra arguments to `find` and `egrep` ignore typically uninteresting files that appear in Node.js projects. It also skips search of binary files, and uses NULL-delimiting in `xargs` and `egrep` to handle file names with spaces.

---

### Flexibility

You can get `srcfind` to work more specifically for your needs. If you want to supply your own options to `egrep`, you can provide them after the search patterns. For example, to find the occurrences of `forEach`and`iterate`that are individual words, you can provide`-w`.  Add flags will have the added function of showing you the matches.

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

---

### Caveats

Right now, `srcfind` is specialized to Node projects and has other limitations.  I will make updates to support more use cases as I go along.
