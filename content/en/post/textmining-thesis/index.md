+++
title = "Text mining my Thesis"
subtitle = ""

# Add a summary to display on homepage (optional).
summary = "My first text mining exercise, using text from my Ph.D. thesis"

date = 2019-04-25T17:52:51+02:00
draft = false

# Code highlighting
highlight = true
highlight_languages = ["r"]

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Adriano Fantini"]

# Is this a featured post? (true/false)
featured = true

# Tags and categories
# For example, use `tags = []` for no tags, or the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["text mining", "thesis", "R"]
categories = ["textmining", "R"]

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""
+++

A simple text mining exercise with data from my Ph.D. thesis, using R's beautiful [`tidytext`](https://github.com/juliasilge/tidytext) package and examples from the related [book](https://www.tidytextmining.com/). I have never mined text before, so I'll stick to the examples from the book, but using my own data. 

I'll also use `dplyr` for handling `tibbles` (a sort of `data.frame`) and `ggplot2` to plot the results. This report is generated with `reprex`, since I have not yet gotten around to use `knitr`.

---

Ready? Let's start by loading the packages we will need:

``` r
library(readr)
library(tidytext)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
library(furrr)
#> Loading required package: future
library(ggplot2)
```

We the collect the data. We will use the `.tex` files from my Ph.D. thesis, which has 6 chapters that we ingest separately (but in parallel, because we are cool and we live in a bright [`future`](https://github.com/HenrikBengtsson/future)).
Notice that `format = "latex"` is necessary to ingest `LaTeX` data:

```r
input_files = paste0("https://raw.githubusercontent.com/adrfantini/PhD-thesis/master/chapters/", c(
    "1_intro.tex",
    "2_obs.tex",
    "3_itaobs.tex",
    "4_methods.tex",
    "5_validation.tex",
    "6_conclusions.tex"
))

plan(multiprocess) # Yay multiprocess!

a = future_map_dfr(input_files, function(x) {
    tibble(text = read_lines(x)) %>%
        unnest_tokens(word, text, format = "latex") %>%
        mutate(word = tolower(word)) %>%
        anti_join(stop_words) %>%
        mutate(chapter=basename(x))
})
#> Joining, by = "word"
#> Joining, by = "word"
#> Joining, by = "word"
#> Joining, by = "word"
#> Joining, by = "word"
#> Joining, by = "word"
```

We now have a nice `tibble` with all the words, and can now take a look at which words are more common, like so:

```r
a %>% count(word, sort = TRUE)
#> # A tibble: 2,413 x 2
#>    word              n
#>    <chr>         <int>
#>  1 precipitation   251
#>  2 flood           196
#>  3 data            176
#>  4 model           134
#>  5 fig             121
#>  6 discharge       109
#>  7 italy           106
#>  8 station         105
#>  9 datasets         98
#> 10 chym             97
#> # … with 2,403 more rows
```

_Precipitation_ easily comes on top. No surprises.

Let's now get the most common 10 words by chapter, and plot them:

```r
a_n = a %>%
    group_by(chapter) %>%
    count(word, sort = TRUE)

a_short = a_n %>%
    top_n(10, n) %>%
    ungroup

a_short %>%
    mutate(word = reorder(word, n)) %>%
    ggplot(aes(x = word, y = n, fill = chapter)) +
        geom_col(show.legend = FALSE) +
        facet_wrap(~chapter, scales = 'free') +
        coord_flip()
```

![](https://i.imgur.com/bhs2ekc.png)

Uh-oh! What's going on? The ordering of some words on the Y axis is wrong, but we indeed reordered them by `n` with `mutate(word = reorder(word, n))`! This is because different input files might have different word rankings, and here we are ranking by global rank. We can fix this by means of a small trick: creating a new variable (column) `lab`, which identifies a word-file combination univocally. We merge the labels using the dummy separator `+`, since we know our filenames and our words do not include that character, and later we only plot the part of the label after the `+`:

```r
a_short = a_short %>%
    mutate(lab = paste(chapter, word, sep = '+'))

a_short %>%
    mutate(lab = reorder(lab, n)) %>%
    ggplot(aes(x = lab, y = n, fill = chapter)) +
        geom_col(show.legend = FALSE) +
        facet_wrap(~chapter, scales = 'free') +
        scale_x_discrete(labels = function(b) {strsplit(b, split = '+', fixed = TRUE) %>% sapply(function(x) x[2])}) +
        coord_flip()
```

![](https://i.imgur.com/rGwAbyW.png)

Much better!

Now, let's look at how well [Zipf's law](https://en.wikipedia.org/wiki/Zipf%27s_law) holds for this small corpus of files. We can check by following the example in [chapter 3.2](https://www.tidytextmining.com/tfidf.html#zipfs-law) of the text mining book:

``` r
a_n_rank = a_n %>%
    group_by(chapter) %>%
    mutate(rank = row_number(),
          `term frequency` = n/sum(n))

# Zipf's law!
a_n_rank %>%
    ggplot(aes(rank, `term frequency`, color = chapter)) +
        geom_line(size = 1.1, alpha = 0.8) +
        scale_x_log10() +
        scale_y_log10()
```

![](https://i.imgur.com/g7GCeVz.png)

Not exactly linear, right? Well, that is to be expected with such a small amount of words and in a technical publication.

The book also [shows](https://www.tidytextmining.com/tfidf.html#tfidf) how to extract the `tf-idf` metric. This is the ratio of the _term frequency_ in a given chapter and its frequency in the whole collection. Basically, this shows which terms are more common in a particular chapter compared to other chapters.

This is quite easy following the examples; we'll take the top 12 words for each chapter this time:

``` r
tf_idf = a %>%
    group_by(chapter) %>%
    count(word, sort = TRUE) %>%
    bind_tf_idf(word, chapter, n) %>%
    arrange(desc(tf_idf))

tf_idf %>%
    group_by(chapter) %>%
    top_n(12, tf_idf) %>%
    slice(1:12) %>%
    ungroup() %>%
    mutate(word = reorder(word, tf_idf)) %>%
    ggplot(aes(word, tf_idf, fill = chapter)) +
        geom_col(show.legend = FALSE) +
        facet_wrap(~chapter, scales = "free") +
        ylab("tf-idf") +
        coord_flip()
```

![](https://i.imgur.com/lwQk7Mb.png)

Oh no, the ordering issue strikes again! But... since the plot comes out _almost_ correct and I'm too lazy to fix it right now, this will have to do , sorry :grin:. (If this were a textbook I would say _the exercise is left for the reader_)

What this plot shows is that in chapter 1 I really talk a lot about _risk_, which is a term that I never use in other chapters. This is because my thesis deals with _hazard_, which is a different concept from _risk_, and in chapter 1 I make this distinction very clear. _Interpolation_ is very prominent in chapter 3, since this is where I describe the gridding procedures used to create the observational precipitation dataset that I use. All in all, most of what is shown is what I would expect, knowing what is in every chapter. How boring!

Speaking of boring, how positive is each chapter? We can assign a positivity score to each word (yes, there are tables, and yes, Trump's tweets are found to be [quite](https://www.kaggle.com/erikbruin/text-mining-the-clinton-and-trump-election-tweets) [negative](http://varianceexplained.org/r/trump-tweets/)), and average them:

``` r
a_sent = a_n %>%
    inner_join(get_sentiments("afinn"), by = "word") %>%
    group_by(chapter) %>%
    summarize(score = sum(score * n) / sum(n))

a_sent %>%
    ggplot(aes(chapter, score, fill = score > 0)) +
        geom_col(show.legend = FALSE) +
        coord_flip() +
        ylab("Average sentiment score")
```

![](https://i.imgur.com/qOuiOBU.png)

It turns out that the first three chapters are a bit negative, with the first (the introduction) being worst. But I recover and start thinking in positive terms (or rather, selling my work with fancy words :grin:) in the second half of the thesis, peaking with very positively-written conclusions.

---

Well, this wraps up this rather short and quite uneventful diversion into text mining, I hope you enjoyed it!
