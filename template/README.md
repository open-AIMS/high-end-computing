This is a template from which to build explorations. Please
remove these opening instructions and populate the sections below.
For those less familiar with markdown, the following might help

- [markdown-cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

- **bold** _italic_ ~~strikethrough~~ font

- a link comprises either: 

  `[<text>]` followed by `(<url>)` - where `<text>` is the in text
  label and `<url>` is the URL.  See the example in the first point
  above.
  
  Or
  
  `[<text>][<number>]` inline, followed by [<number>]: <url> on its own
  line following the current paragraph
  
- inline code appears between a pair of backticks, eg `ls -la`

``` 
ls -la
```
Or, 
```{label, engine='bash', results='markdown', eval=FALSE} 
ls -la
```
where `label` is a unique id for the chunk, `bash` is the engine to
use (could be others like `R`, `python` `c` `java` etc.  In some
contexts this will determine syntax highlighting, linting etc.


- to include a chunk of code, surround the code by a pair of backtick
  triplets.  For example:

- a block of verbatim text
~~~
verbatim text
~~~

- images can be embedded as `![](<image path>)`

Opening instructions end here

* Overview

* Dependencies

* Approach

* Limitations
