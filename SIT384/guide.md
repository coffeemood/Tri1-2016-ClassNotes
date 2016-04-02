# Guide for grabbing Dr. Zhang's papers 

Hey guys, this is a quick guide for using R & bash to grab mosts of Dr. Zhang's papers. I wasn't able to get all 60+ but it was enough to work with. 

1. Download [this](https://cran.r-project.org/web/packages/scholar/index.html) R package (*I'm not walking you through this*)

2. Load the package, and get his publications using the `get_publications()` function

```r
library(scholar)
jun = c("QmsjV8QAAAAJ")
jun_publishes = get_publications()
#Export all of the pubids to a text file (Destination of your choice) 
write(as.character(jun_publishes$pubid), file="~/~/links.txt")
```

3. Run a bash script to grab all the pdfs link 

```bash
#!/bin/bash

base='https://scholar.google.com.au/citations?view_op=view_citation&hl=vi&user=QmsjV8QAAAAJ&citation_for_view=QmsjV8QAAAAJ:'
while read line; do
{
    curl -s -L $base$line | pup -c 'div a attr{href}' | grep pdf >> pdfs.txt
} done < links.txt
echo "All done :)"
```

> Remember to place the links file in the same folder as the bash script 

4. Finally, just download the pdfs

```bash
while read line; do 
{
    echo "Download from $line"
    curl -s -L $line -O 
} done < pdfs.txt 
```

Now you get the documents AND the metadata from the scholar package. Have fun :) 

> p/s: Shout out to Simon for the R package! 

