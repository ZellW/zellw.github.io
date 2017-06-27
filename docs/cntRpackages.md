
# Count R Packages
*Date of publication:  Apr 7, 2017*

##Introduction
This markdown file counts how many packages are in the CRAN Repository.  This code is presented to be copied and used as a learning tool (that's why I offer 2 different ways to get to the same result).  

##Foolish First Attempt

Have you ever tried to dig a hole without a shovel because you simply - and ignorantly - thought you could do it quickly without it?  Well, that describes my first effort to count the number of R packages.

As you will learn, there are a few R packages that you will use all the time.  Each time you bypass the opportunity to use them, you will waste time.  I'll prove this below.

Remember, R can do anything - that's why it is my hammer.  Fortunately, it cannot only do almost anything you want, most of the time it does it very well.  Scraping web information is a good example.

Commonly, you will load packages you know you will need.  Don't worry, you will learn these by heart as you use R.  Here is just one example - `reader`.  `readr` makes it easy to read many types of rectangular data, including csv, tsv and fixed-width files. Compared to R base equivalents like read.csv(), readr is much faster and gives more convenient output.  Do yourself a favor and always include `readr` in your R scripts.  Just do it now and you'll thank me later.

We are going to scrape the CRAN page with a list of the available packages.  You can find it [here](https://cran.r-project.org/web/packages/available_packages_by_date.html).  If you are lazy like me (R makes me seem super productive at work), here is a picture of what you would see:

![](../images/CRAN_Pgks.JPG)


Let's get started.  We'll load `readr` and use its function `read_lines` to *read* the CRAN Package URL.  We then see how long the list is by using the `length' function in base R:


```
library(readr)
readByDate <- read_lines("https://cran.r-project.org/web/packages/available_packages_by_date.html")
length(readByDate)
```

```
## [1] 13155
```
We now know there are 13155 lines on HTML on the CRAN Package page.  But is all of that code useful to us?  Of course not. We do not need anything but the table.  So the code in the beginning and at the end can be ignored.  Here's some of the extraneous code at the beginning:

```
head(readByDate,3)
```

```
## [1] "<!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Strict//EN\" \"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd\">"
## [2] "<html xmlns=\"http://www.w3.org/1999/xhtml\">"                                                                    
## [3] "<head>"
```
and here's some code we do not need at the end:

```
tail(readByDate,3)
```

```
## [1] "</table>" "</body>"  "</html>"
```
So now we will get the code of interest that includes the information about the CRAN Packages.  To make sure it all looks right, let's also look at the first and last line of the file we have created:


```
readByDate <- readByDate[18:length(readByDate)-3]
readByDate[1]
```

```
## [1] "<tr> <td> 2017-06-27 </td> <td> <a href=\"../../web/packages/bridgesampling/index.html\">bridgesampling</a> </td> <td> Bridge Sampling for Marginal Likelihoods and Bayes Factors </td> </tr>"
```

```
readByDate[length(readByDate)]
```

```
## [1] "<tr> <td> 2005-10-29 </td> <td> <a href=\"../../web/packages/vioplot/index.html\">vioplot</a> </td> <td> Violin plot </td> </tr>"
```

OK, everything looks OK.  We have scraped the CRAN site and parsed the HTML code to the line we are interested in.  Now we start using some R code.  I know this might be one of your first introductions to R so I will probably use some terms that are new to you.  Sorry about that!  To keep this post small, rather than explain each term, I'll provide you links where you can learn more.  Do not get to concerned about this.  I promise you will know all this and more if you follow my blog!

The first of these concepts is critically important to the world of R - data frames.  A data frame is R’s standard container for data. It is extremely flexible: variables can be numeric, character or factors, or indeed any data type supported by the system. The only constraint is that all variables in the data frame must have the same number of rows. You will become very familiar with data frames.  In fact, you will use them every time you use R!

Below, we create a data frame from the HTML code we have collected:

```
readByDate <- data.frame(readByDate, stringsAsFactors = FALSE )
```
We can now look at the data frame (notice I load a library to assist.  Was this really needed below - no.  But I have gotten into the habit to using the `dplyr` library whenever I can:

```
library(dplyr)
glimpse(readByDate)
```

```
## Observations: 13,138
## Variables: 1
## $ readByDate <chr> "<tr> <td> 2017-06-27 </td> <td> <a href=\"../../we...
```

Whenever importing data, you must review it carefully.  There are always surprises.  (This will be a topic for another blog entry).  The data we have has problems.  Let's look at one example:

```
readByDate[8,1]
```

```
## [1] "<tr> <td> 2017-06-27 </td> <td> <a href=\"../../web/packages/JM/index.html\">JM</a> </td> <td> Joint Modeling of Longitudinal and Survival Data </td> </tr>"
```
We are interested in records that begin with the HTML tags `<tr> <td>` only.  Therefore, we will create a list of Boolean (TRUE or FALSE) values where TRUE indicates the record in the data frame begins with the HTML tags we expect.  


```
library(stringr)
tmpBool <- str_detect(readByDate$readByDate, "<tr> <td>")
```
We can then filter the data frame using the Boolean list so only records that are TRUE (ie, starting with the HTML tags `<tr> <td>`) are kept in the data frame.  FALSE records will be removed.

```
readByDate <- data_frame(readByDate[tmpBool,])
```
We're done!  All we need to do is count how many records remain:

```
nrow(readByDate)
```

```
## [1] 10929
```
So we now know there are 10929 R packages.

Even if the code does not make sense to you, the logic we used as pretty simple.  All we did was count how many records in an HTML start with specific HTML tags.  The code works, but. . . was it the most efficient method?  No, not even close.

Recall  the beginning of this document when I stated there is an R package for everything and often these tools make things easier?  Well, like always, it is true in the case too.

The [rvest package](https://cran.r-project.org/web/packages/rvest/rvest.pdf) provides all the functionality to answer our question - *How namy R packages are there?* in 1 line of code (after loading the package and identifying the URL):


```
library(rvest)
pkgs <- "https://cran.r-project.org/web/packages/available_packages_by_date.html"
pkgs_raw <- read_html(pkgs) %>% html_nodes("table") %>% .[[1]] %>% html_table()
```
Using rvest, the number of records is 10929 - the same number we calculated before!

Again, if there is a package that meets your needs - use it.  It will save time and effort every time!

>FYI:  If you run this code in RStudio, you'll find that my code runs much faster than the `rvest` code.  I guess sometimes there is a tradeoff between simplicity and performance.
