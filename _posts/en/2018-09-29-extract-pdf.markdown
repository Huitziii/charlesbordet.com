---
layout: single
permalink: /en/extract-pdf/
title: "How to Extract and Clean Data From PDF Files in R"
description: "R is a powerful tool for text mining. But you still need to get the data first, and it can easily become a pain when you deal with PDF files. This article shows you how to extract what you need and get the data ready for analysis!"
date: 2018-09-29
lang: en
categories: [r, en]
ref: extract-pdf
header:
    image: /assets/images/extract-pdf/featured.jpg
---

Do you need to extract the right data from a list of PDF files but right now you’re stuck?

If yes, you’ve come to the right place.

*Note: This article treats PDF documents that are machine-readable. If that’s not your case, I recommend you use Adobe Acrobat Pro that will do it automatically for you. Then, come back here.*

In this article, you will learn:

* How to extract the content of a PDF file in R (two techniques)
* How to clean the raw document so that you can isolate the data you want

After explaining the tools I’m using, I will show you a couple examples so that you can easily replicate it on your problem.

*This article was first published on [Medium](https://medium.com/@CharlesBordet/how-to-extract-and-clean-data-from-pdf-files-in-r-da11964e252e), before I had a website.*

## Why PDF files?

When I started to work as a freelance data scientist, I did several jobs consisting in only extracting data from PDF files.

My clients usually had two options: Either do it manually (or hire someone to do it), or try to find a way to automate it.

The first way being really tedious and costly when the number of files increases, they turned to the second solution for which I helped them.

For example, a client had thousands of invoices that all had the same structure and wanted to get important data from it:

* the number of sold items,
* the profits made at each transaction,
* the data from his customers

Having everything in PDF files isn’t handy at all. Instead, he wanted a clean spreadsheet where he could easily find who bought what and when and make calculations from it.

Another classical example is when you want to do data analysis from reports or official documents. You will usually find those saved under PDF files rather than freely accessible on webpages.

Similarly, I needed to extract thousands of speeches made at the U.N. General Assembly.

So, how do you even get started?

## Two techniques to extract raw text from PDF files

### Use `pdftools::pdf_text`

The first technique requires you to install the `pdftools` package from CRAN:

{% highlight R %}
install.packages("pdftools")
{% endhighlight %}

A quick glance at the [documentation](https://cran.rstudio.com/web/packages/pdftools/pdftools.pdf) will show you the few functions of the package, the most important of which being `pdf_text`.

For this article, I will use an official record from the UN that you can find on this [link](https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf)

{% highlight R %}
library(pdftools)
download.file("https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf",
              "./71_PV.62.pdf")
text <- pdf_text("./71_PV.62.pdf")
{% endhighlight %}

This function will directly import the raw text in a *character* vector with spaces to show the white space and `\n` to show the line breaks.

Having a full page in one element of a vector is not the most practical. Using `strsplit` will help you separate lines from each other:

{% highlight R %}
text2 <- strsplit(text, "\n")
head(text2[[1]])
# [1] "                 United Nations                                                                                            A /71/PV.62"  
# [2] "                 General Assembly                                                                                       Official Records"
# [3] "                 Seventy-first session"                                                                                                  
# [4] "                 62nd plenary meeting"                                                                                                   
# [5] "                 Tuesday, 13 December 2016, 10 a.m."                                                                                     
# [6] "                 New York"
{% endhighlight %}

If you want to know more about the functions of the `pdftools` package, I recommend you read [Introducing pdftools - A fast and portable PDF extractor](https://ropensci.org/blog/2016/03/01/pdftools-and-jeroen), written by the author himself.

### Use the `tm` package

`tm` is the go-to package when it comes to doing text mining/analysis in R.

{% highlight R %}
install.packages("tm")
{% endhighlight %}

For our problem, it will help us import a PDF document in R while keeping its structure intact. Plus, it makes it ready for any text analysis you want to do later.

The `readPDF` function from the `tm` package doesn’t actually read a PDF file like `pdf_text` from the previous example we did. Instead, **it will help you create your own function**, the benefit of it being that you can choose whatever PDF extracting engine you want.

By default, it will use `xpdf`, available at [http://www.xpdfreader.com/download.html](http://www.xpdfreader.com/download.html)

You have to:

* Download the archive from the website (under the *Xpdf tools* section).
* Unzip it.
* Make sure it is in the PATH of your computer.

Then, you can create your PDF extracting function:

{% highlight R %}
library(tm)
read <- readPDF(control = list(text = "-layout"))
{% endhighlight %}

The control argument enables you to set up parameters as you would write them in the command line. Think of the above function as writing `xpdf -layout` in the shell.

Then, you’re ready to import the PDF document:

{% highlight R %}
document <- Corpus(URISource("./71_PV.62.pdf"), readerControl = list(reader = read))
doc <- content(document[[1]])
head(doc)
# [1] "                United Nations                                                                                              A /71/PV.62"  
# [2] "                General Assembly                                                                                         Official Records"
# [3] "                Seventy-first session"                                                                                                    
# [4] ""                                                                                                                                         
# [5] ""                                                                                                                                         
# [6] "                62nd plenary meeting"
{% endhighlight %}

Notice the difference with the excerpt from the first method. New empty lines appeared, corresponding more closely to the document. This can help to identify where the header stops in this case.

Another difference is how pages are managed. With the second method, you get the whole text at once, with page breaks symbolized with the `\f` symbol. With the first method, you simply had a list where 1 page = 1 element.

{% highlight R %}
page_breaks <- grep("\\f", doc)
doc[page_breaks[1]]
## [1] "\fA/71/PV.62\t13/12/2016"
{% endhighlight %}

This is the first line of the second page, with an added `\f` in front of it.

## Extract the right information

Naturally, you don’t want to stop there. Once you have the PDF document in R, you want to extract the actual pieces of text that interest you, and get rid of the rest.

That’s what this part is about.

I will use a few common tools for string manipulation in R:

* The `grep` and `grepl` functions.
* Base string manipulation functions (such as `str_split`).
* The `stringr` package.

My goal is to extract all the speeches from the speakers of the document we’ve worked on so far ([this one](https://github.com/Huitziii/crispy-pdf/raw/master/71_PV.62.pdf)), but I don’t care about the speeches from the president.

Here are the steps I will follow:

1. Clean the headers and footers on all pages.
2. Get the two columns together.
3. Find the rows of the speakers.
4. Extract the correct rows.

I will use regular expressions (regex) regularly in the code. If you have absolutely no knowledge of it, I recommend you follow a tutorial about it, because it is essential as soon as you start managing text data.

If you have some basic knowledge, that should be enough. I’m not a big expert either.

### 1. Clean the headers and footers on all pages.

Notice how each page contains text at the top and at the bottom that will interfere with our extraction.

{% highlight R %}
# Remove header of the first page
president_row <- grep("^President:", doc)[1]
doc <- doc[(president_row + 1):length(doc)]

# Remove footer on first page
footer_row_1 <- grep("This record contains the text of speeches ", doc)[1]
footer_row_2 <- grep("\\f", doc)[1] - 1
doc <- doc[- c(footer_row_1:footer_row_2)]

# Remove headers on other pages
header_rows <- grep("^\\f", doc) # Remember: \f are for page breaks
doc[header_rows] <- "page" # I put a marker here that will be useful later
doc <- doc[- (header_rows - 1)]
{% endhighlight %}

Now, our document is a bit cleaner. Next step is to do something about the two columns, which is super annoying.

### 2. Get the two columns together.

My idea (there might be better ones) is to use the `str_split` function to split the rows every time two spaces appear (i.e. it’s not a normal sentence).

Then, because sometimes there are multiple spaces together at the beginning of the rows, I detect where there is text, where there is not, and I pick the elements with text.

It’s a bit arbitrary, you’ll see, but it works:

{% highlight R %}
library(stringr)
doc_split <- strsplit(doc, "  ") # Split each row whene there are 2 spaces
doc_split <- lapply(doc_split, function(x) {
    # For each element, extract:
    #    - doc1 that is the first column. 
    #    - doc2 that is the second column.
    doc1 <- x[1:8][x[1:8] != ""][1] # The first piece of text that's not empty
    if (is.na(doc1)) doc1 <- ""
    # doc2 takes the next non-empty piece of text
    doc2 <- x[x != ""] 
    if (doc1 != "") doc2 <- doc2[-1]
    if (length(doc2) == 0) doc2 <- ""
    # Sometimes there is more text needed to be extracted. 
    # I try to give it to either doc1 or doc2 depending on the size of it.
    while (sum(nchar(doc2)) > 65) {
        doc1 <- paste(doc1, doc2[1], collapse = " ")
        doc2 <- doc2[-1]
    }
    # Clean it before returning it
    doc2 <- paste(doc2, collapse = " ")
    doc1 <- str_trim(doc1) # stringr::str_trim trim the spaces before/after
    doc2 <- str_trim(doc2)
    list(doc1 = doc1, doc2 = doc2)
})
doc1 <- sapply(doc_split, `[[`, 1) # First column
doc2 <- sapply(doc_split, `[[`, 2) # Second column
{% endhighlight %}

Now, let’s put it together, thanks to the marker `page` that we added earlier:

{% highlight R %}
# Vector of the page breaks coordinates:
pages_rows <- c(0, which(doc1 == "page"), length(doc1))
doc <- c()
# Page by page, we fill a new vector:
for (i in 1:(length(pages_rows) - 1)) {
    doc <- c(doc, c(doc1[(pages_rows[i] + 1):pages_rows[i + 1]],
                    doc2[(pages_rows[i] + 1):pages_rows[i + 1]]))
}
doc <- doc[doc != "page"]
{% endhighlight %}

Now that we have a nice clean vector of all text lines in the right order, we can start extracting the speeches.

### 3. Find the rows of the speakers

This is where you must look into the document to spot some patterns that would help us detect where the speeches start and end.

It’s actually fairly easy since all speakers are introduced with "Mr." or "Mrs.". And the president is always called "The President:" or "The Acting President:"

Let’s get these rows:

{% highlight R %}
speakers_rows <- grep("Mrs?\\..+\\(", doc)
president_rows <- c(grep("The President:", doc),
                    grep("The Acting President:", doc))
all_rows <- sort(c(speakers_rows, president_rows))
{% endhighlight %}

Now it’s easy. We know where the speeches start, and they always end with someone else speaking (whether another speaker or the president).

{% highlight R %}
speeches <- list()
for (i in 1:(length(all_rows) - 1)) {
    start <- all_rows[i]
    if (!start %in% speakers_rows) next
    stop <- all_rows[i + 1] - 1
    
    speeches <- append(speeches, list(doc[start:stop]))
}
speeches[[11]]
##  [1] "Mr. Hallak (Syrian Arab Republic) (spoke in"              
##  [2] "Arabic): Yesterday in his statement (see A/71/PV.61),"    
##  [3] "my colleague the representative of the Republic of Korea" 
##  [4] "made unprecedented allegations about my country that"     
##  [5] "we have not read in any report and that have not appeared"
##  [6] "in any document. We ask our colleague to provide us"      
##  [7] "with further information concerning those allegations"    
##  [8] "and to indicate if they have been corroborated through"   
##  [9] "bilateral channels. We would have preferred it if, rather"
## [10] "than accusing us, our colleague from South Korea had"     
## [11] "dispelled and disavowed information referring to the"     
## [12] "existence of nuclear weapons in my country, which"        
## [13] "would constitute a flagrant violation of the Treaty on"   
## [14] "the Non-Proliferation of Nuclear Weapons."                
## [15] "Ms. Kharashun (Belarus) (spoke in Russian):"              
## [16] "I would just like to underscore in my statement the"      
## [17] "untiring commitment of Belarus to the international"      
## [18] "norms and standards concerning nuclear energy, as"        
## [19] "well as the priority nature for us of ensuring nuclear"   
## [20] "safety and security and transparency in carrying out"     
## [21] "the construction of our first nuclear power plant. We"    
## [22] "stand ready for dialogue with all international partners,"
## [23] "including our neighbours. For my country, which"          
## [24] "suffered the greatest impact of the Chernobyl, nuclear"   
## [25] "security is of primary importance."                       
## [26] "As I noted earlier in my statement, Belarus uses"         
## [27] "the tools provided by the International Atomic Energy"    
## [28] ""                                                         
## [29] "Agency to countries that are embarking on nuclear"        
## [30] ""                                                         
## [31] "programmes for the first time. We have hosted the"        
## [32] ""                                                         
## [33] "assessment missions of the Agency on numerous"            
## [34] ""                                                         
## [35] "occasions."                                               
## [36] ""
{% endhighlight %}

Finally, we could get all the speeches in a list. We can now analyze what each country representative talk about, how this evolves over more documents, over the years, depending on the topic discussed, etc.

Now, one could argue that for one document, it would be easier to extract it in a semi-manually way (by specifying the row numbers manually, for example). This is true.

But the idea here is to replicate this same process over hundreds, or even thousands, of such documents.

This is where the fun begins, as they will all have their specificities, the format might evolve, sometimes stuff is misspelled, etc. In fact, even with this example, the extraction is not perfect! You can try to improve it if you want.