# Desktop tools

_Last update: 5/19/2022_

Here is a list of tools for the desktop. Unless otherwise noted, these tools are supported only on MacOS.

---

## [srcfind](https://github.com/jcampbellGoPuff/main-repo/tree/main/desktop-tools/srcfind)

- Type: CLI (zsh script)
- Environment: MacOS
- Provided for: faceted search of source files in Github project
- Current version: 0.4 (alpha)

### Description: faceted command line search of multiple terms in a source hierarchy

Developers often search through source files to find strings in an effort to locate functions. However, a search for just one string often returns too many matches to be useful. It can often be true that we need find source files that contain all of more than one string to pinpoint the function we are seeking.

For example, suppose you were working in Node JS and wanted to find uses of the `lodash` function `reduce`. A search for just `lodash` would probably return a lot of files. The same would be true if you searched for `reduce`. The natural way to deal with this is to make the search more specific, such as by searching for `lodash.reduce` or `_.reduce`. However, you cannot be sure that this would find all the occurrences you want to change. As an example, it is possible that the the function `reduce` was imported from the `lodash` module individually, such as with this code:

```
const { reduce } = require('lodash')
```

which allows the developer to call only `reduce` without any prefix.

A more reliable way to find all potential occurrences is to first find all the files that contain `lodash`, and then, from only those files, search for the word `reduce`. Although IDE's provide this functionality by allowing users to search open files, it is convenient to be able to do this search on a command line. That is what `srcfind` provides.

---

### Sample case - why faceted search is tough with standard UNIX commands

Here is an example of files to search and some places within them that hold the strings `forEach` and `iterate`:

| Name               | Contains                     |
| :----------------- | ---------------------------- |
| src/array-utils.js | arr.forEach()                |
|                    | // call forEach              |
|                    | // iterates through array    |
| app/search.js      | items.forEach(v=> v)         |
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

But the above has issues, beyond having to type a long, typo-vulnerable command line:

- You will search files you probably don't care about, such as hidden files and software installation folders
- The above will match binary files
- The above command will break if file names have spaces.
- You run some risk of exceeding the number of arguments allowed in a command if the `find` command returns too many path names.

For the reasons noted above, the above `egrep` command would not work quite right. The subcommand in `$(...)` would list `test/test all.js` as it should, but the pieces of the file path would be treated as separate arguments to `egrep`. Here is the result you would see:

```
./src/array-utils.js
./app/search.js
./test/test: No such file or directory
all.js: No such file or directory
```

If `test/test all.js` contained `iterate` as well as `forEach`, it would not be included in the results.

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

This means the file names will be searched only when they are recognized by `git`, when they are not binary (i.e., non-textual) files, and the NULL-delimiting options in `git ls-files`, `xargs` and `egrep` will be provided to handle file names that contain spaces.

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

Because this script delegates its matching to `egrep`, the `egrep`-style of regular expression syntax is fully supported. All the flags to that program are generally supported as well.

---

### Caveats

Right now, `srcfind` is specialized to Github projects and has other limitations. I will make updates to support more use cases as I go along.
