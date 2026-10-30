# Hunspell Spell Checking and Morphological Analysis

The `hunspell` function is a high-level wrapper for finding spelling
errors within a text document. It takes a character vector with text
(`plain`, `latex`, `man`, `html` or `xml` format), parses out the words
and returns a list with incorrect words for each line. It effectively
combines `hunspell_parse` with `hunspell_check` in a single step. Other
functions in the package operate on individual words, see details.

## Usage

``` r
hunspell(
  text,
  format = c("text", "man", "latex", "html", "xml"),
  dict = dictionary("en_US"),
  ignore = en_stats
)

hunspell_parse(
  text,
  format = c("text", "man", "latex", "html", "xml"),
  dict = dictionary("en_US")
)

hunspell_check(words, dict = dictionary("en_US"))

hunspell_suggest(words, dict = dictionary("en_US"))

hunspell_analyze(words, dict = dictionary("en_US"))

hunspell_stem(words, dict = dictionary("en_US"))

hunspell_info(dict = dictionary("en_US"))

dictionary(lang = "en_US", affix = NULL, add_words = NULL, cache = TRUE)

list_dictionaries()
```

## Arguments

- text:

  character vector with arbitrary input text

- format:

  input format; supported parsers are `text`, `latex`, `man`, `xml` and
  `html`.

- dict:

  a dictionary object or string which can be passed to `dictionary`.

- ignore:

  character vector with additional approved words added to the
  dictionary

- words:

  character vector with individual words to spell check

- lang:

  dictionary file or language, see details

- affix:

  file path to corresponding affix file. If `NULL` it is is assumed to
  be the same path as `dict` with extension `.aff`.

- add_words:

  a character vector of additional words to add to the dictionary

- cache:

  speed up loading of dictionaries by caching

## Details

Hunspell uses a special dictionary format that defines which stems and
affixes are valid in a given language. The `hunspell_analyze` function
shows how a word breaks down into a valid stem plus affix. The
`hunspell_stem` function is similar but only returns valid stems for a
given word. Stemming can be used to summarize text (e.g in a wordcloud).
The `hunspell_check` function takes a vector of individual words and
tests each one for correctness. Finally `hunspell_suggest` is used to
suggest correct alternatives for each (incorrect) input word.

Because spell checking is usually done on a document, the package
includes some parsers to extract words from various common formats. With
`hunspell_parse` we can parse plain-text, latex and man format. R also
has a few built-in parsers such as
[`RdTextFilter`](https://rdrr.io/r/tools/RdTextFilter.html) and
[`SweaveTeXFilter`](https://rdrr.io/r/tools/SweaveTeXFilter.html), see
also [`?aspell`](https://rdrr.io/r/utils/aspell.html).

The package searches for dictionaries in the working directory as well
as in the standard system locations. `list_dictionaries` provides a list
of all dictionaries it can find. Additional search paths can be
specified by setting the `DICPATH` environment variable. A US English
dictionary (`en_US`) is included with the package; other dictionaries
need to be installed by the system. Most operating systems already
include compatible dictionaries with names such as
[hunspell-en-gb](https://packages.debian.org/sid/hunspell-en-gb) or
[myspell-en-gb](https://packages.debian.org/sid/myspell-en-gb).

To manually install dictionaries, copy the corresponding `.aff` and
`.dic` file to `~/Library/Spelling` or a custom directory specified in
`DICPATH`. Alternatively you can pass the entire path to the `.dic` file
as the `dict` parameter. Some popular sources of dictionaries are
[SCOWL](http://wordlist.aspell.net/dicts/),
[OpenOffice](https://www.mirrorservice.org/sites/download.openoffice.org/contrib/dictionaries/),
[debian](http://archive.ubuntu.com/ubuntu/pool/main/libr/libreoffice-dictionaries/?C=S;O=D),
[github/titoBouzout](https://github.com/titoBouzout/Dictionaries) or
[github/wooorm](https://github.com/wooorm/dictionaries).

Note that `hunspell` uses [`iconv`](https://rdrr.io/r/base/iconv.html)
to convert input text to the encoding used by the dictionary. This will
fail if `text` contains characters which are unsupported by that
particular encoding. For this reason UTF-8 dictionaries are preferable
over legacy 8-bit dictionaries.

## Examples

``` r
# Check individual words
words <- c("beer", "wiskey", "wine")
correct <- hunspell_check(words)
print(correct)
#> [1]  TRUE FALSE  TRUE

# Find suggestions for incorrect words
hunspell_suggest(words[!correct])
#> [[1]]
#> [1] "whiskey"  "whiskery"
#> 

# Extract incorrect from a piece of text
bad <- hunspell("spell checkers are not neccessairy for langauge ninja's")
print(bad[[1]])
#> [1] "neccessairy" "langauge"   
hunspell_suggest(bad[[1]])
#> [[1]]
#> [1] "necessary"   "necessarily"
#> 
#> [[2]]
#> [1] "language" "melange" 
#> 

# Stemming
words <- c("love", "loving", "lovingly", "loved", "lover", "lovely", "love")
hunspell_stem(words)
#> [[1]]
#> [1] "love"
#> 
#> [[2]]
#> [1] "loving" "love"  
#> 
#> [[3]]
#> [1] "loving"
#> 
#> [[4]]
#> [1] "loved" "love" 
#> 
#> [[5]]
#> [1] "lover" "love" 
#> 
#> [[6]]
#> [1] "lovely" "love"  
#> 
#> [[7]]
#> [1] "love"
#> 
hunspell_analyze(words)
#> [[1]]
#> [1] " st:love"
#> 
#> [[2]]
#> [1] " st:loving"    " st:love fl:G"
#> 
#> [[3]]
#> [1] " st:loving fl:Y"
#> 
#> [[4]]
#> [1] " st:loved"     " st:love fl:D"
#> 
#> [[5]]
#> [1] " st:lover"     " st:love fl:R"
#> 
#> [[6]]
#> [1] " st:lovely"    " st:love fl:Y"
#> 
#> [[7]]
#> [1] " st:love"
#> 

# \donttest{
# Check an entire latex document
tmpfile <- file.path(tempdir(), "1406.4806v1.tar.gz")
download.file("https://arxiv.org/e-print/1406.4806v1", tmpfile,  mode = "wb")
untar(tmpfile, exdir = tempdir())
text <- readLines(file.path(tempdir(), "content.tex"), warn = FALSE)
bad_words <- hunspell(text, format = "latex")
sort(unique(unlist(bad_words)))
#>  [1] "CORBA"             "CTRL"              "DCOM"             
#>  [4] "DOM"               "DSL"               "ESC"              
#>  [7] "JRI"               "OAuth"             "OpenCPU"          
#> [10] "RInside"           "RPC"               "RProtoBuf"        
#> [13] "RStudio"           "RinRuby"           "Rserve"           
#> [16] "SIGINT"            "STATA"             "Stateful"         
#> [19] "auth"              "cpu"               "cran"             
#> [22] "cron"              "css"               "csv"              
#> [25] "de"                "dec"               "eol"              
#> [28] "facto"             "grDevices"         "httpuv"           
#> [31] "ignorable"         "interoperable"     "js"               
#> [34] "json"              "jsonlite"          "knitr"            
#> [37] "md"                "memcached"         "mydata"           
#> [40] "myfile"            "nondegenerateness" "ocpu"             
#> [43] "opencpu"           "pandoc"            "pb"               
#> [46] "php"               "png"               "prescripted"      
#> [49] "protobuf"          "rApache"           "rda"              
#> [52] "rds"               "rlm"               "rmd"              
#> [55] "rnorm"             "rnw"               "rpy"              
#> [58] "saveRDS"           "scalability"       "scalable"         
#> [61] "schemas"           "se"                "sep"              
#> [64] "stateful"          "statefulness"      "suboptimal"       
#> [67] "svg"               "sweave"            "tex"              
#> [70] "texi"              "tmp"               "toJSON"           
#> [73] "urlencoded"        "www"               "xyz"              

# Summarize text by stems (e.g. for wordcloud)
allwords <- hunspell_parse(text, format = "latex")
stems <- unlist(hunspell_stem(unlist(allwords)))
words <- head(sort(table(stems), decreasing = TRUE), 200)
# }
```
