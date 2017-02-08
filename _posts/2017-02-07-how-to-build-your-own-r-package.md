---
layout: post
title:  "How to Build Your Own R Package"
date:   2017-02-07 12:04:22 -0500
categories: R
---
It is incredibly simple and painless to create your own R package.  Let’s suppose you want to create an R package for calculating the mean of a dataset.  In order to get started, you’ll need to install the following libraries:

{% highlight r %}
install.packages(c('devtools', 'roxygen2'))
library(devtools)
library(roxygen2)
{% endhighlight %}

Find a suitable directory in which to create a folder to hold your package.  Set your working directory in R and create a new sub-directory for your package.  Note that the name of this directory will be the name of your R package.  In the lines below, I’m setting my working directory to my Documents folder and creating a package called ‘average’:

{% highlight r %}
setwd('C:/Users/Nick/Documents')
create('average')
{% endhighlight %}

Now open a new R script in R Studio or some other IDE.  Create a function and save the script in the R sub-directory of your new package directory:

{% highlight r %}
average <- function(d) {
	return(sum(d)/length(d))
}
{% endhighlight %}

![R Package Directory]({{ site.url }}/assets/images/R Package Directory.png)

Now you can add documentation to the function, using the roxygen2 package to handle everything automatically.  The example below shows sample documentation for the ‘average’ function I created above.  Notice that the functions input arguments are dentoed by “@param” and any text in the documentation that you would like formatted as code can be entered within the brackets of “\code{}”.  Finally, the example at the end of the documentation shows how the function should be run.

{% highlight r %}
#' Mean (Average)
#'
#' This function takes a numeric vector as input and 
#' outputs the mean of the vector. It is the same as
#' running \code{mean()} but takes more time.
#' @param d A numeric vector
#' @export
#' @examples
#' average(iris$Sepal.Width)
{% endhighlight %}

Type the documentation at the top of your R script, above the function, and save it.  Now navigate into the package directory and tell R to add documentation:

{% highlight r %}
setwd('./average')
document()
{% endhighlight %}

That’s it!  Now you can move back up to the parent directory (the Documents folder if you’re following my example) and install it: 

{% highlight r %}
setwd('../')
install('average')
{% endhighlight %}

Go ahead and try calling the function or the documentation.

## How to Install Your Package on Other Computers

To pass the package to anyone else to use, it is easiest to zip and send.  Then they can extract the zip file to their working directory (the Documents folder in the example below), and install it.  They must have devtools installed as a prerequisite:

{% highlight r %}
install.packages('devtools')
library(devtools)
setwd('C:/Users/Nick/Documents')
install('average')
library(average)
{% endhighlight %}