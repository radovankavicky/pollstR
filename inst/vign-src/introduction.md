---
title: "pollstR: An R Client for the HuffPost Pollster API"
author: "Jeffrey B. Arnold"
date: "2016-04-20"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{pollstR Introduction}
  %\VignetteEngine{knitr::rmarkdown}
  \usepackage[utf8]{inputenc}
---




This R package is an interface to the Huffington Post [Pollster API](http://elections.huffingtonpost.com/pollster/api), which provides access to opinion polls collected by the Huffington Post.

The package is released under GPL-2 and the API data it accesses is released under the [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.en_US).


# API Overview

The Pollster API has two primary data structures: charts and polls.

*Polls* are individual, dated topline results for a single set of candidates in a single race.
The poll data structure consists of (generally) named candidates and percentage support for each, along with additional information (e.g., polling house, sampling frame, sample size, margin of error, state, etc.).

*Charts* aggregate polls for a particular race or topic (e.g., "2012-president" or "obama-job-approval".
A chart reports aggregated survey estimates of support for the candidates in a given race and, possibly, daily estimates of support.
Each chart is named, reports the number of aggregated polls it presents, a last-updated date, and a "slug" field. The "slug" identifies the chart both in the API and on the Pollster website.

In ``pollstR``, there are three functions in that provide access to the opinion polls and model estimates from Huffpost Pollster.

- ``pollstr_charts``: Get a list of all charts and the current model estimates.
- ``pollstr_chart``: Get a single chart along with historical model estimates.
- ``pollstr_polls``: Get opinion poll data.

## Charts

To get a list of all the charts in the API use the function ``pollstr_charts``,

```r
charts <- pollstr_charts()
str(charts)
```

```
#> List of 2
#>  $ charts   :'data.frame':	100 obs. of  10 variables:
#>   ..$ id           : int [1:100] 742 741 736 735 734 733 732 731 730 729 ...
#>   ..$ title        : chr [1:100] "2016 Delaware Republican Presidential Primary" "2016 Delaware Democratic Presidential Primary" "2016 National House Race" "2016 Florida Senate Republican Primary " ...
#>   ..$ slug         : chr [1:100] "2016-delaware-republican-presidential-primary" "2016-delaware-democratic-presidential-primary" "2016-national-house-race" "2016-florida-senate-republican-primary" ...
#>   ..$ topic        : chr [1:100] "2016-president-gop-primary" "2016-president-dem-primary" "2016-house" "2016-senate-gop-primary" ...
#>   ..$ state        : chr [1:100] "DE" "DE" "US" "FL" ...
#>   ..$ short_title  : chr [1:100] "2016 DE Republican Presidential Primary" "2016 DE Democratic Presidential Primary" "2016 National House Race" "2016 FL Senate Republican Primary " ...
#>   ..$ election_date: Date[1:100], format: "2016-04-26" ...
#>   ..$ poll_count   : int [1:100] 1 1 29 12 14 15 3 6 1 1 ...
#>   ..$ last_updated : POSIXct[1:100], format: "2016-04-20 17:57:46" ...
#>   ..$ url          : chr [1:100] "http://elections.huffingtonpost.com/pollster/2016-delaware-republican-presidential-primary" "http://elections.huffingtonpost.com/pollster/2016-delaware-democratic-presidential-primary" "http://elections.huffingtonpost.com/pollster/2016-national-house-race" "http://elections.huffingtonpost.com/pollster/2016-florida-senate-republican-primary" ...
#>  $ estimates:'data.frame':	331 obs. of  8 variables:
#>   ..$ choice         : chr [1:331] "Democrat" "Republican" "Undecided" "Other" ...
#>   ..$ value          : num [1:331] 43.3 43.5 12.7 5.5 12 8.2 7 4 1.6 1 ...
#>   ..$ lead_confidence: num [1:331] NA NA NA NA NA NA NA NA NA NA ...
#>   ..$ first_name     : chr [1:331] "" "" NA NA ...
#>   ..$ last_name      : chr [1:331] "Democrat" "Republican" NA NA ...
#>   ..$ party          : chr [1:331] NA NA "N/A" "N/A" ...
#>   ..$ incumbent      : logi [1:331] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ slug           : chr [1:331] "2016-national-house-race" "2016-national-house-race" "2016-national-house-race" "2016-national-house-race" ...
#>  - attr(*, "class")= chr "pollstr_charts"
```
This returns a ``list`` with two data frames.
The data frame ``charts`` has data on each chart,
while the data frame ``estimates`` has the current poll-tracking estimates from each chart.

The query can be filtered by state or topic.
For example, to get only charts related to national topics,

```r
us_charts <- pollstr_charts(state = "US")
```

## Chart

To get a particular chart use the function ``pollstr_chart``.
For example, to get the chart for [Barack Obama's Favorable Rating](http://elections.huffingtonpost.com/pollster/obama-favorable-rating), specify its *slug*, ``obama-favorable-rating``.

```r
obama_favorable <- pollstr_chart('obama-favorable-rating')
print(obama_favorable)
```

```
#> Title:       Barack Obama Favorable Rating 
#> Chart Slug:  obama-favorable-rating 
#> Topic:       favorable-ratings 
#> State:       US 
#> Polls:       954 
#> Updated:     1461021144 
#> URL:         http://elections.huffingtonpost.com/pollster/obama-favorable-rating 
#> Estimates:
#>        choice value lead_confidence first_name last_name party incumbent
#> 1   Favorable  47.5              NA         NA        NA   N/A     FALSE
#> 2 Unfavorable  46.4              NA         NA        NA   N/A     FALSE
#> 3   Undecided   2.9              NA         NA        NA   N/A     FALSE
#> 
#> First 6 (of 2572) daily estimates:
#>        choice value       date
#> 1   Favorable  47.5 2016-04-14
#> 2     Neutral  11.7 2016-04-14
#> 3   Undecided   2.9 2016-04-14
#> 4 Unfavorable  46.4 2016-04-14
#> 5   Favorable  47.5 2016-04-11
#> 6   Undecided   2.9 2016-04-11
```
The slug can be found from the results of a ``pollstr_charts`` query.
Alternatively the slug is the path of the url of a chart, http://elections.huffingtonpost.com/pollster/obama-favorable-rating.

The historical estimates of the Huffpost Pollster poll-tracking model are contained in the element ``"estimates_by_date"``,

```r
(ggplot(obama_favorable[["estimates_by_date"]], aes(x = date, y = value, color = choice))
 + geom_line())
```

![plot of chunk obama-favorable-chart](inst/vign-src/figures/obama-favorable-chart-1.png)

## Polls

To get the opinion poll results use the function ``pollstr_polls`.
The polls returned can be filtered by topic, chart, state, or date.

By default, ``pollstr_polls`` only returns 1 page of results (about 10 polls).
To have it return more polls, increase the value of ``max_pages``.
To have it return all polls, set the value of ``max_pages`` to a very high number.
For example, to return all the polls on the favorability of Bararck Obama after March 1, 2014,

```r
obama_favorable_polls <- pollstr_polls(max_pages = 10000, chart = 'obama-favorable-rating', after = "2014-3-1")
str(obama_favorable_polls)	
```

```
#> List of 4
#>  $ polls        :'data.frame':	165 obs. of  9 variables:
#>   ..$ id          : int [1:165] 24305 24266 24229 24194 24187 24181 24141 24084 24016 23910 ...
#>   ..$ pollster    : chr [1:165] "NBC/WSJ" "YouGov/Economist" "AP-GfK (web)" "IBD/TIPP" ...
#>   ..$ start_date  : Date[1:165], format: "2016-04-10" ...
#>   ..$ end_date    : Date[1:165], format: "2016-04-14" ...
#>   ..$ method      : chr [1:165] "Live Phone" "Internet" "Internet" "Live Phone" ...
#>   ..$ source      : chr [1:165] "http://msnbcmedia.msn.com/i/MSNBC/Sections/A_Politics/16229%20NBCWSJ%20April%20Poll.pdf" "https://today.yougov.com/news/2016/04/13/tale-two-conventions/" "http://ap-gfkpoll.com/main/wp-content/uploads/2016/04/March-2016-AP-GfK-Poll-FINAL_Clinton.pdf" "http://www.investors.com/politics/trump-support-fades-as-mistakes-grow-sanders-clinton-tied-ibdtipp-poll/" ...
#>   ..$ last_updated: POSIXct[1:165], format: "2016-04-18 23:11:33" ...
#>   ..$ partisan    : chr [1:165] "Pollster" "Nonpartisan" "Nonpartisan" "Nonpartisan" ...
#>   ..$ affiliation : chr [1:165] "Dem" "None" "None" "None" ...
#>  $ questions    :'data.frame':	15274 obs. of  14 variables:
#>   ..$ question       : chr [1:15274] "2016 National Republican Primary" "2016 National Republican Primary" "2016 National Republican Primary" "2016 National Republican Primary" ...
#>   ..$ chart          : chr [1:15274] "2016-national-gop-primary" "2016-national-gop-primary" "2016-national-gop-primary" "2016-national-gop-primary" ...
#>   ..$ topic          : chr [1:15274] "2016-president-gop-primary" "2016-president-gop-primary" "2016-president-gop-primary" "2016-president-gop-primary" ...
#>   ..$ state          : chr [1:15274] "US" "US" "US" "US" ...
#>   ..$ subpopulation  : chr [1:15274] "Likely Voters - Republican" "Likely Voters - Republican" "Likely Voters - Republican" "Likely Voters - Republican" ...
#>   ..$ observations   : int [1:15274] 310 310 310 310 1000 1000 1000 1000 1000 1000 ...
#>   ..$ margin_of_error: num [1:15274] 5.57 5.57 5.57 5.57 3.1 3.1 3.1 3.1 3.1 3.1 ...
#>   ..$ choice         : chr [1:15274] "Cruz" "Kasich" "None" "Trump" ...
#>   ..$ value          : int [1:15274] 35 24 1 40 50 39 9 2 46 44 ...
#>   ..$ first_name     : chr [1:15274] "Ted" "John" NA "Donald" ...
#>   ..$ last_name      : chr [1:15274] "Cruz" "Kasich" NA "Trump" ...
#>   ..$ party          : chr [1:15274] "Rep" "Rep" NA "Rep" ...
#>   ..$ incumbent      : logi [1:15274] FALSE FALSE NA FALSE FALSE FALSE ...
#>   ..$ id             : int [1:15274] 24305 24305 24305 24305 24305 24305 24305 24305 24305 24305 ...
#>  $ survey_houses:'data.frame':	200 obs. of  3 variables:
#>   ..$ name : chr [1:200] "Hart Research (D)" "Peter Hart (D)" "YouGov" "Gfk" ...
#>   ..$ party: chr [1:200] "Dem" "Dem" "N/A" "N/A" ...
#>   ..$ id   : int [1:200] 24305 24305 24266 24229 24194 24187 24181 24141 24141 24084 ...
#>  $ sponsors     :'data.frame':	176 obs. of  3 variables:
#>   ..$ name : chr [1:176] "NBC News" "Wall Street Journal" "Economist" "Associated Press" ...
#>   ..$ party: chr [1:176] "N/A" "N/A" "N/A" "N/A" ...
#>   ..$ id   : int [1:176] 24305 24305 24266 24229 24194 24187 24187 24181 24141 24084 ...
#>  - attr(*, "class")= chr "pollstr_polls"
```


# Errors

If you encounter an error when using one of the functions it is likely that there was an error in converting the data as returned by the API into data structures more usable in R.
The conversion code is fragile, and can break when there are changes in APIs, or from weird cases that I didn't anticipate.
So if you encounter an error, try running the function with `convert = FALSE`; this will return the data as returned by the API.
If there is no error, then it is problem with the conversion code in this package.
Using `convert = FALSE`, you will still be able to get the data from the Pollster API, but you will need to do the extra data cleaning yourself.
To get the bug fixed, open a new issue on [github](https://github.com/rOpenGov/pollstR/issues).


<!--  LocalWords:  API APIs github
 -->


# Example: Obama's Job Approval Rating

This section shows how to use ``pollstr`` to create a chart similar to those displayed on the Huffpost Pollster website.
I'll use Obama's job approval rating in this example.

The slug or name of the chart is ``obama-job-approval``, which is derived from the chart's URL , http://elections.huffingtonpost.com/pollster/obama-job-approval.
I'll focus on approval in 2013 in order to reduce the time necessary to run this code.

```r
slug <- "obama-job-approval"
start_date <- as.Date("2013-1-1")
end_date <- as.Date("2014-1-1")
```
For the plot, I'll need both Pollster's model estimates and opinion poll estimates.
I get the Pollster model estimates using ``polster_chart``,

```r
chart <- pollstr_chart(slug)
estimates <- chart[["estimates_by_date"]]

estimates <- estimates[estimates$date >= start_date 
                       & estimates$date < end_date, ]
```
and the opinion poll results,

```r
polls <- pollstr_polls(chart = slug, 
                        after = start_date, 
                        before = end_date,
                        max_pages = 1000000)
```
Note that in ``polster_poll`` I set the ``max_pages`` argument to a very large number in order to download all the polls available.
This may take several minutes.

Before continuing, we will need to clean up the opinion poll data.
First, only keep results from national subpopulations ("Adults", "Likely Voters", "Registered Voters").
This will drop subpopulations like Republicans, Democrats, and Independents.

```r
questions <-
    subset(polls[["questions"]],
           chart == slug
           & subpopulation %in% c("Adults", "Likely Voters", "Registered Voters"))
```
Second, I will need to recode the choices into three categories, "Approve", "Disapprove", and "Undecided".

```r
approvalcat <- c("Approve" = "Approve",
                 "Disapprove" = "Disapprove",
                 "Undecided" = "Undecided",
                 "Neither" = "Undecided",
                 "Refused" = NA,
                 "Neutral" = "Undecided",
                 "Strongly Approve" = "Approve",
                 "Somewhat Approve" = "Approve", 
                 "Somewhat Disapprove" = "Disapprove",
                 "Strongly Disapprove" = "Disapprove")

questions2 <-
    (questions
     %.% mutate(choice = plyr::revalue(choice, approvalcat))
     %.% group_by(id, subpopulation, choice)
     %.% summarise(value = sum(value)))
```

```
#> Warning: '%.%' is deprecated.
#> Use '%>%' instead.
#> See help("Deprecated")

#> Warning: '%.%' is deprecated.
#> Use '%>%' instead.
#> See help("Deprecated")

#> Warning: '%.%' is deprecated.
#> Use '%>%' instead.
#> See help("Deprecated")
```
Now merge the question data with the poll metadata,

```r
polldata <- merge(polls$polls, questions2, by = "id")
```

Now, I can plot the opinion poll results along with the Huffpost Pollster trend estimates,

```r
(ggplot()
 + geom_point(data = polldata,
              mapping = aes(y = value, x = end_date, color = choice),
              alpha = 0.3)
 + geom_line(data = estimates,
             mapping = aes(y = value, x = date, color = choice))
 + scale_x_date("date")
 + scale_color_manual(values = c("Approve" = "black", 
                                 "Disapprove" = "red", 
                                 "Undecided" = "blue"))
 )
```

![plot of chunk obama-favorable-chart-2](inst/vign-src/figures/obama-favorable-chart-2-1.png)

<!--  LocalWords:  Huffpost API Huffington CRAN github devtools str
 -->
<!--  LocalWords:  devools jrnold ggplot obama url aes favorability
 -->
<!--  LocalWords:  Bararck
 -->

<!--  LocalWords:  Huffpost API Huffington CRAN github devtools str
 -->
<!--  LocalWords:  devools jrnold ggplot obama url aes favorability
 -->
<!--  LocalWords:  Bararck
 -->