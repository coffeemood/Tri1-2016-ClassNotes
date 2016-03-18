# SIT384 - Week 2 - Binary Classification/Conditional Probability

## Conditional Probability/Naive Bayesian 

This week we look at conditional probability, more specifically binary classification. We will try to establish data training based on previously specified `spam` OR `ham` data. 

From this we will try our best to arrive at a system that will make an educated guess at whether or not an email is spam or ham. 

### Constructing Training Data

In constructing our classifciation logic, aside from the usual frequency and occurrences counts, we need one more deciding factor which is: **prior**

> In this case, the prior will be the base rate of seeing spam messages. The higher the base rate, the more likely it is f3or us to take `weak evidence` and declare that a message is, in fact, spam. 

- First we need to set the path to our data 

```r
# Load libraries
library('tm')
library('ggplot2')

# Set the global paths
spam.path <- file.path("data", "spam")
spam2.path <- file.path("data", "spam_2")
easyham.path <- file.path("data", "easy_ham")
easyham2.path <- file.path("data", "easy_ham_2")
hardham.path <- file.path("data", "hard_ham")
hardham2.path <- file.path("data", "hard_ham_2")
```


- We then want to write a function that will start collecting plain text from the 2nd line (the first line is usually a blank)

```r
get.msg <- function(path)
{
  con <- file(path, open = "rt", encoding = "latin1")
  text <- readLines(con)
  # The message always begins after the first full line break
  msg <- text[seq(which(text == "")[1] + 1, length(text), 1)]
  close(con)
  return(paste(msg, collapse = "\n"))
}
```

> This function returns a vector, which is then collapsed into a single character element using `paste`, with `\n` for every new line (using `collapse`).

- Use `sapply` to apply the above function to all our spam emails 

```r
# Get all the SPAM-y email into a single vector
spam.docs <- dir(spam.path)
spam.docs <- spam.docs[which(spam.docs != "cmds")]
# There is a cmds file which is a long list of Unix cmds that move files into the directories 
all.spam <- sapply(spam.docs,
                   function(p) get.msg(file.path(spam.path, p)))
```

- We then create a text corpus from this vector (using the `tm` package), which we will use to construct a `Term Document Matrix (tdm)` in order to quantify frequencies. 

```r
get.tdm <- function(doc.vec)
{
  control <- list(stopwords = TRUE,
                  removePunctuation = TRUE,
                  removeNumbers = TRUE,
                  minDocFreq = 2)
  doc.corpus <- Corpus(VectorSource(doc.vec))
  doc.dtm <- TermDocumentMatrix(doc.corpus, control)
  return(doc.dtm)
}
# stopwords -> Eliminate all the stopwords --> see stopwords()
# minDocFreq -> Appear in the document at least twice

spam.tdm <- get.tdm(all.spam)
```

- Finally, we construct the data frame which follows a few steps: 
    + Create a matrix from the tdm 
    + Use `rowSums` to create a frequency vector for all the terms 
    + `cbind` the vector into the data frame (with stringAsFactors=FALSE)
    + Pass every row through a function in `sapply` that would divides the positive elements by the total number of columns in the TDM.
    + Calculate the frequency
    + Add `spam.occurrence` and `spam.density` into the data frame

```r
# Create a data frame that provides the feature set from the training SPAM data
spam.matrix <- as.matrix(spam.tdm)
spam.counts <- rowSums(spam.matrix)
spam.df <- data.frame(cbind(names(spam.counts),
                            as.numeric(spam.counts)),
                      stringsAsFactors = FALSE)
names(spam.df) <- c("term", "frequency")
spam.df$frequency <- as.numeric(spam.df$frequency)
spam.occurrence <- sapply(1:nrow(spam.matrix),
                          function(i)
                          {
                            length(which(spam.matrix[i, ] > 0)) / ncol(spam.matrix)
                          })
spam.density <- spam.df$frequency / sum(spam.df$frequency)

# Add the term density and occurrence rate
spam.df <- transform(spam.df,
                     density = spam.density,
                     occurrence = spam.occurrence)
```

### Constructing Classification 

#### Defining Classifier 

> In training the classifier, we use the pre-calculated occurrences in the training data to arrive at a general probability. I.e: 

|Term| Frequency |
|----|:----:|
|table | 10%|  
|html | 30%| 

| Result | 
|:-------:|
|0.3 * 0.1 = 0.03 (3%) |  

**Note**: A problem arise when we encounter a term that we haven't seen in the training data. One way to combat this is to set a small percentage of *spam probability* to these term (ie. 0.0001%)

In constructing the `classify.email` function, we: 

1. Extract the message that we are inspecting and calculate the frequency of terms with `rowSums`
2. Using `intersect` we look for intersections of the terms from our training data and the actual message 
3. Determine whether the words in the email are **present** in the training set. If **no**, we apply the `c` value that is 1e-6 which was mentioned earlier. If **yes**, we start looking up the occurrence probaiities using the `match` function.
4. Finally, we calculate the `product` of all these values and combine it with our missing terms probabilities to come up with the answer. 

```r
classify.email <- function(path, training.df, prior = 0.5, c = 1e-6)
{
  # Here, we use many of the support functions to get the
  # email text data in a workable format
  msg <- get.msg(path)
  msg.tdm <- get.tdm(msg)
  msg.freq <- rowSums(as.matrix(msg.tdm))
  # Find intersections of words
  msg.match <- intersect(names(msg.freq), training.df$term)
  # Now, we just perform the naive Bayes calculation
  if(length(msg.match) < 1)
  {
    return(prior * c ^ (length(msg.freq)))
  }
  else
  {
    match.probs <- training.df$occurrence[match(msg.match, training.df$term)]
    return(prior * prod(match.probs) * c ^ (length(msg.freq) - length(msg.match)))
  }
}

# prior = 50/50 spam-to-ham probability
# c = 0.0001% assigned for unseen terms
```

Finally, we run our classifier on the `hardham` data set: 

```r
# Run classifer against HARD HAM
hardham.docs <- dir(hardham.path)
hardham.docs <- hardham.docs[which(hardham.docs != "cmds")]

hardham.spamtest <- sapply(hardham.docs,
                           function(p) classify.email(file.path(hardham.path, p), training.df = spam.df))
    
hardham.hamtest <- sapply(hardham.docs,
                          function(p) classify.email(file.path(hardham.path, p), training.df = easyham.df))
    
hardham.res <- ifelse(hardham.spamtest > hardham.hamtest,
                      TRUE,
                      FALSE)
summary(hardham.res)
```

|Mode|FALSE|TRUE|NA's|
| |:--:|:--:|:--:|
|logical|243|6|0|


### Running Classifier Against All Types

When we run all classifier against all email types, we take a simplified approach: 

1. Create a classifier that return the `probability` of **spam** vs **ham**, If one is greater than another then classify the email as such.
2. Repeat the code, make a new data frame consists of everything 
3. Examine result

```r
# Finally, attempt to classify the HARDHAM data using the classifer developed above.
# The rule is to classify a message as SPAM if Pr(email) = SPAM > Pr(email) = HAM
spam.classifier <- function(path)
{
  pr.spam <- classify.email(path, spam.df)
  pr.ham <- classify.email(path, easyham.df)
  return(c(pr.spam, pr.ham, ifelse(pr.spam > pr.ham, 1, 0)))
}

# Get lists of all the email messages
easyham2.docs <- dir(easyham2.path)
easyham2.docs <- easyham2.docs[which(easyham2.docs != "cmds")]

hardham2.docs <- dir(hardham2.path)
hardham2.docs <- hardham2.docs[which(hardham2.docs != "cmds")]

spam2.docs <- dir(spam2.path)
spam2.docs <- spam2.docs[which(spam2.docs != "cmds")]

# Classify them all!
easyham2.class <- suppressWarnings(lapply(easyham2.docs,
                                   function(p)
                                   {
                                     spam.classifier(file.path(easyham2.path, p))
                                   }))
hardham2.class <- suppressWarnings(lapply(hardham2.docs,
                                   function(p)
                                   {
                                     spam.classifier(file.path(hardham2.path, p))
                                   }))
spam2.class <- suppressWarnings(lapply(spam2.docs,
                                function(p)
                                {
                                  spam.classifier(file.path(spam2.path, p))
                                }))

# Create a single, final, data frame with all of the classification data in it
easyham2.matrix <- do.call(rbind, easyham2.class)
easyham2.final <- cbind(easyham2.matrix, "EASYHAM")

hardham2.matrix <- do.call(rbind, hardham2.class)
hardham2.final <- cbind(hardham2.matrix, "HARDHAM")

spam2.matrix <- do.call(rbind, spam2.class)
spam2.final <- cbind(spam2.matrix, "SPAM")

class.matrix <- rbind(easyham2.final, hardham2.final, spam2.final)
class.df <- data.frame(class.matrix, stringsAsFactors = FALSE)
names(class.df) <- c("Pr.SPAM" ,"Pr.HAM", "Class", "Type")
class.df$Pr.SPAM <- as.numeric(class.df$Pr.SPAM)
class.df$Pr.HAM <- as.numeric(class.df$Pr.HAM)
class.df$Class <- as.logical(as.numeric(class.df$Class))
class.df$Type <- as.factor(class.df$Type)
```

--- 

#### Result 

| Email Type | % Classified as ham | % Classified as spam |
| :--- | :--- | :-- | 
| Easy ham | 0.78 | 0.22 |
| Hard ham | 0.73 | 0.27 |
| Spam | 0.15 | 0.85 | 

Looking at the result, we can see two indicators that our training isn't perfect: 

> + Still a lot of hard hams near the spam area 
> + There are both easy & hard ham messages that have much higher relative probability of being ham 

![graph](http://i.imgur.com/m70ciAa.png)


