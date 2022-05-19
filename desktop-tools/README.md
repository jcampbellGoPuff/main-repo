# Desktop tools

*Last update: 5/19/2022*

Here is a list of tools for the desktop. Unless otherwise noted, these tools are supported only on MacOS.

----

## [srcfind](https://github.com/jcampbellGoPuff/jcampbellGoPuff/blob/main/desktop-tools/srcfind)

* Type: CLI (zsh script)
* Environment: MacOS
* Provided for: faceted search of source files in Github project
* Current version: 0.4 (alpha)

### Description: faceted command line search of multiple terms in a source hierarchy

Developers often search through source files to find strings in an effort to locate functions.  However, a search for just one string often returns too many matches to be useful.  It can often be true that we need find source files that contain all of more than one string to pinpoint the function we are seeking.

For example, searching through files to find an occurrence of the `lodash` function `reduce` in an effort to convert the source from using `lodash`'s versions to the ES6 version, a search for just `lodash` would probably return a lot of files that you don't want to search through by hand.  The same would be true if you searched for `reduce`.  So one way to make the search more specific is to search for `lodash.reduce` or `_.reduce`.  This, however, would likely miss some occurrences of the functions, since the just the function `reduce` can be imported from the `lodash` individually, which means the call would work as just `reduce`.

As in this case, to find the instances of functionality in a source hierarchy, it is often necessary to search for occurrences of more than one term in any source file before it can be considered likely to contain the searched function.  `srcfind` does this for you.

---

### Example - why faceted search is tough with standard UNIX commands

Here is an example of files to search where the below lists the only lines they have that match `forEach` and `iterate`:


| Name               | Contains                     |
| :------------------- | ------------------------------ |
| src/array-utils.js | arr.forEach()                |
|                    | // call forEach              |
|                    | // iterates through array    |
| app/search.js      | items.forEach(v=> v)          |
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

* You will search files you probably don't care about, such as hidden files and software installation folders
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

---

### srcfind hides the complexity & avoids the issues

`srcfind` provides a driver for `egrep` and `find` that will require you to enter only the strings you are seeking, as well as rid you of the issues noted above.
```
srcfind forEach iterate

# outputs same as above, with no errors on "test/test all.js"
./src/array-utils.js 
./app/search.js
```

In the above case, `srcfind` will run the equivalent of:

```
git ls-files -z \
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

The extra arguments to `find` and `egrep` ignore typically uninteresting files that appear in Node.js projects. It also skips binary files, and uses NULL-delimiting options in `xargs` and `egrep` to handle file names with spaces.

---

### Flexibility

You can get `srcfind` to work more specifically for your needs. If you want to supply your own options to `egrep`, you can provide them before the search patterns. For example, to list the files that contain the string `foreach` in any case and `iterate` as a whole word in only lowercase, you can provide the flags individually.

```
srcfind -i foreach -w iterate

# outputs only app/search.js, which contains 'iterate' and 'forEach'.
# array-utils.js is not included because it contains no word-bound occurrence of 'iterate'
./app/search.js
```


You may also see the matches by providing `-S` or `--show-matches` as the first argument.

```
srcfind -S iterate -i foreach

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

Right now, `srcfind` is specialized to Github projects and has other limitations.  I will make updates to support more use cases as I go along.
