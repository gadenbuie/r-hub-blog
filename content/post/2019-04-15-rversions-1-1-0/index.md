---
title: rversions 1.1.0 is on CRAN!
date: '2019-04-15'
authors:
  - Maëlle Salmon
slug: rversions-1-1-0
tags:
  - rversions
  - R package
  - release
output: 
  html_document:
    keep_md: true
---




Version 1.1.0 of the `rversions` package has been [released on CRAN](https://cran.r-project.org/package=rversions)! `rversions` queries the main R SVN repository to find the versions r-release and r-oldrel refer to, and also all previous R versions and their release dates.

Get `rversions`'s latest version from 
a [safe](https://rud.is/b/2019/03/03/cran-mirror-security/) CRAN mirror near
you:

```r
install.packages("rversions")
```

This release, compared to `rversions` 1.0.3, brings 

* :sparkles: caching :sparkles: of the R versions within each R session (less useless queries!) thanks to a contribution by [Rich FitzJohn](https://github.com/richfitz); 

* :sparkles: version nicknames :sparkles: information to the output of `rversions` functions;

* a documentation website built with `pkgdown`; 

* and a [new home for `rversions`' source](https://github.com/r-hub/rversions) from METACRAN to R-hub! :house:

That's it, but since it's the first time we blog about `rversions`, we'll use the release as an opportunity to introduce it with all due respect! How does `rversions` work and what is it useful for?

# Where does `rversions` get its information from?

As you might have guessed, R is developed under version control. The R Core team uses SVN, i.e. [Apache Subversion](https://en.wikipedia.org/wiki/Apache_Subversion). The versions `rversions` outputs are derived from SVN _tags_ in the [main R SVN repository](https://svn.r-project.org/R/). In SVN lingo, ["tagging refers to labeling the repository at a certain point in time so that it can be easily found in the future."](https://en.wikipedia.org/wiki/Apache_Subversion#Branching_and_tagging) so it might remind you of git tags (or of GitHub releases). In more details,

- [r-versions are coming from SVN tags](https://github.com/metacran/rversions/blob/36deaf5c4ffd648c7524d8e3225871e6d97d9ccf/R/rversions.R#L79)

```r 
head(rversions::r_versions())
```

```
  version                date nickname
1    0.60 1997-12-04 08:47:58     <NA>
2    0.61 1997-12-21 13:09:22     <NA>
3  0.61.1 1998-01-10 00:31:55     <NA>
4  0.61.2 1998-03-14 19:25:55     <NA>
5  0.61.3 1998-05-02 07:58:17     <NA>
6    0.62 1998-06-14 12:56:20     <NA>
```

- r-release is the latest SVN tag

```r 
rversions::r_release()
```

```
    version                date  nickname
117   4.0.0 2020-04-24 07:05:34 Arbor Day
```

- r-oldrel is the latest of the previous minor version

```r 
rversions::r_oldrel()
```

```
    version                date             nickname
116   3.6.3 2020-02-29 08:05:16 Holding the Windsock
```

- r-release-tarball is the latest SVN tag that has a corresponding tar.gz to download

```r 
rversions::r_release_tarball()
```

```
    version                date
116   3.6.3 2020-02-29 08:05:16
                                                       URL             nickname
116 https://cran.r-project.org/src/base/R-3/R-3.6.3.tar.gz Holding the Windsock
```

- Windows, macOS are the same: latest SVN tag that has a file to download

```r 
rversions::r_release_win()
```

```
    version                date
117   4.0.0 2020-04-24 07:05:34
                                                            URL  nickname
117 https://cran.r-project.org/bin/windows/base/R-4.0.0-win.exe Arbor Day
```

```r 
rversions::r_release_macos()
```

```
    version                date
117   4.0.0 2020-04-24 07:05:34
                                                  URL  nickname
117 https://cran.r-project.org/bin/macosx/R-4.0.0.pkg Arbor Day
```

Version nicknames are extracted from specific files from R 2.15.1, e.g. https://svn.r-project.org/R/tags/R-2-15-1/VERSION-NICK. Before that there were only 3 version nicknames, which we [got from Lucy D'Agostino McGowan's excellent blog post about R release names](https://livefreeordichotomize.com/2017/09/28/r-release-names/), where you can also read about their meaning.

# Where is `rversions` used?

`rversions` is used in R packages but also public services for R package developers, including R-hub of course!

### In R packages

* `rversions` is used in `devtools`, in [`dr_devtools()`](https://devtools.r-lib.org/reference/dr_devtools.html) to compare the installed R version to the current R release version.

* [`provisionr`](https://github.com/mrc-ide/provisionr) imports `rversions` and uses it in `check_r_version()`, a function that checks and coerces something into an R version.

### In other tools

In the tools mentioned below, `rversions` is used via its [very own web service](https://github.com/r-hub/rversions.app#readme).

- [r-appveyor service uses `rversions` to determine what versions r-release and r-oldrel are](https://github.com/krlmlr/r-appveyor/blob/master/scripts/appveyor-tool.ps1#L77-L87).

- R-hub uses it to do the same on [Solaris](https://github.com/r-hub/solarischeck/blob/d3edc6077f52081a53b2ed8b9ec40e17176fdb43/run.sh#L105), [macOS](https://github.com/r-hub/macoscheck/blob/846f87d1ad1c4b4aeb283a986e084913f203879f/run.sh#L160) and [Windows](https://github.com/r-hub/wincheck/blob/591ea8d9a28cd5044c063e88f46ef0507fa16fc7/run.ps1#L114).

- [R-hub's Windows install script uses it as well, to decide what versions of R to install](https://github.com/r-hub/rhub-server/blob/master/jenkins/scripts/windows-2008-setup.ps1#L74-L79).


# Can we also _play_ with `rversions` data?

At this point, you have no doubt `rversions` is useful, but what about we use it to get a glimpse at R's release schedule over time?


```r 
library("ggplot2")
versions <- rversions::r_versions()
versions <- dplyr::mutate(versions, date = anytime::anytime(date))
versions <- tidyr::separate(versions, version,
                            into = c("major", "minor", "patch"))
```

```
Warning: Expected 3 pieces. Missing pieces filled with `NA` in 13 rows [1, 2, 6,
11, 15, 18, 20, 22, 23, 25, 27, 31, 33].
```

```r 
versions <- dplyr::mutate(versions, patch = ifelse(is.na(patch), 0, patch))
versions <- dplyr::mutate(versions, release = dplyr::case_when(
  major != dplyr::lag(major) ~ "major",
  minor != dplyr::lag(minor) ~ "minor",
  TRUE ~ "patch"
))

ggplot(versions) +
  geom_segment(aes(x = date, xend = date,
                 col = release), 
                   y = 0, yend = 1) +
  viridis::scale_colour_viridis(discrete = TRUE) +
  theme_minimal() +
  hrbrthemes::theme_ipsum(base_size = 16,
                          axis_title_size = 16) +
  xlab("Time") + 
  theme(
  axis.text.y = element_blank(),
  axis.ticks.y = element_blank(),
  axis.title.y = element_blank(),
  legend.position = "bottom") +
  ggtitle("R releases over time",
          subtitle = "Data obtained via the R-hub rversions R package")
```
{{<figure src="unnamed-chunk-1-1.png" >}}

With the R version being x.y.z, we define a major release as a change in x, a minor release as a change in y and a patch release as a change in z. The figure above shows that the frequency of minor releases decreased in 2013, from twice to once a year. The frequency of patch releases seems irregular, as is the frequency of _major_ releases: not sure when we should expect R 4.0.0!

# Conclusion

The best place to get to know `rversions` is [its brand-new documentation website built with `pkgdown`](https://r-hub.github.io/rversions), and the best place to provide feedback or contribute is [its GitHub repo](https://github.com/r-hub/rversions). Thanks to all folks who chimed in in the issue tracker and pull request tab!
