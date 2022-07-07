# Desktop tools

_Last update: 7/7/2022_

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
| `src/array-utils.js` | `arr.forEach()`              |
|                    | `// call forEach`              |
|                    | `// iterates through array`    |
| `app/search.js`      | `items.forEach(v=> v)`         |
|                    | `const iterate = (items) => {` |
| `test/test all.js`   | `_.forEach(item1)`            |
|                      | `// iterate over this array`  |

You can search the files that contain occurrences of both string using a fairly simple UNIX command line:

```
# List all files that contain the strings "forEach" and "iterate"

egrep -l $(find . -type f -print | egrep -l forEach) iterate
```

If the above command works right, we would expect to see all the files because they contain both strings:

```
./src/array-utils.js
./app/search.js
./app/test all.js
```

However, this will not work nearly as well as we might expect.  Here are a few things that can go wrong:

- You will search files you probably don't care about, such as hidden files and software installation folders
- You will match binary files, which is usually not the intent
- The expansion of the `find` command might provide too many file names, which will cause the command line to fail.

Even if the above example were attempted in a directory without a lot of data, it is still guaranteed to break because one of the file names returned would be `./test all.js`, which contains a space.  The pieces separated by the spaces would be processed as individual file names, which is clearly not correct.  You would get this result:

```
./src/array-utils.js
./app/search.js
./test/test: No such file or directory
all.js: No such file or directory
```

These problems and others can easily get in the way of the simple and commonplace task of searching files for multiple strings.

---

### srcfind hides the complexity & avoids the issues

Now let's try the above search with `srcfind`

```
srcfind forEach iterate
```

In the above, `srcfind` will run the equivalent of:

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

This means the file names will be searched only when they are recognized by `git`, when they are not binary (i.e., non-textual) files. The NULL-delimiting options in `git ls-files`, `xargs` and `egrep` will be used to handle file names that contain spaces.

The result will then be correct, matching what was attempted in the previous section.

```
src/array-utils.js
app/search.js
test/test all.js
```

---

### Flexibility

Because this script delegates its matching to `egrep`, the `egrep`-style of regular expression syntax is fully supported. All the flags to that program are generally supported as well and can be provided with each pattern separately. For example, to list the files that contain the string `foreach` in any mix of lowercase or uppercase characters that also include `iterate` as a whole word in only lowercase, you can provide flags to specialize each match separately.

```
srcfind -i foreach -w iterate
```
The results will exclude `src/array-utils.js`, because it does not contain `iterate` as a word:
```
app/search.js
test/test all.js
```

`srcfind` will also allow you to see the matches, rather than just the file names that match. To do this, provide `-S` or `--show-matches` as the first argument.

```
srcfind -S iterate -i foreach

# outputs
src/array-utils.js:arr.forEach()
src/array-utils.js:// call forEach
src/array-utils.js:// iterates through array
app/search.js:items.forEach(v=>v)
app/search.js:// const iterate = (items) => {
test/test all.js:_.forEach(item1)
test/test all.js:// iterate over this array
```

---

### Caveats

Right now, `srcfind` is specialized to Github projects and has other limitations. I will make updates to support more use cases as I go along.
