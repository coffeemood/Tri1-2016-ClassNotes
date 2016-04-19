# SIT384 - Optimization: Breaking Codes 

### Intro to Optimization 

So far we've only used different algorithms to solve different problems, and use their default results as the working data, with no modification or optimization involved. 

In this chapter we will build a **code-breaking system**, where we seek to find an optimal way to decrypt an encrypted text.

First we take a look at the standard height-weight function. Only this time we have `a` as the slope and `b` as the intercept of the `lm` function, as seen from our previous linear regression. 

This will give us the predicted weight of a person whose height is *zero*

```r
height.to.weight <- function(height, a, b)
{
  return(a + b * height)
}
```

`lm` has its own error function (squared error), let's take a look at it and see the reasonings behind this

```r
heights.weights <- read.csv(file.path('data', '01_heights_weights_genders.csv'))

coef(lm(Weight ~ Height, data = heights.weights))
#(Intercept) Height
#-350.737192 7.717288
```

*If we wanted to analyse the data, we would want to take the mean. However, for optimization purposes, we can go with sum*

```r
squared.error <- function(heights.weights, a, b)
{
  predictions <- with(heights.weights, height.to.weight(Height, a, b))
  errors <- with(heights.weights, Weight - predictions)
  return(sum(errors ^ 2))
}

#We take a look at how different value of a & b affect the squared error value 
for (a in seq(-1, 1, by = 1))
{
  for (b in seq(-1, 1, by = 1))
  {
    print(c(a,b,squared.error(heights.weights, a, b)))
  }
}
```


| a  |  b  | Squared Error | 
| --- | --- | --------- | 
| -1 |  -1 | 536271759 |
| -1 |  0 | 274177183 |
| -1 |  1 | 100471706 |
|  0 |  -1 |531705601 | 
|  0 |  0 | 270938376 | 
|  0 |  1 | 98560250 |
|  1 |  -1 | 527159442 |
|  1 |  0 | 267719569 |
|  1 |  1 | 96668794 |


> As we can derive from looking at the result, some a & b do produce smaller SE. 

This leads to our next step, which is to reate an `objective function`, a metric we could measure and optimise 

- We could try the brute-force method and run this test on a large sample of different `a` and `b` values, but there are two problems: 
    + Step value is unclear (1,2,3) or (0.1,0.2,0.3)? This kind of optimization often leads to infinite loops
    + Exponential growth as we decide to go in-depth (**Curse of Dimensionality**)

#### Introducing `optim` 

The `optim` a function that comes with many popular optimization algorithms

To demonstrate how this works, let's take a look at how `optim` fit the linear regression 

```r
optim(c(0, 0),
      function (x)
      {
        squared.error(heights.weights, x[1], x[2])
      })
```

- The function call took in two objects: 
    + A vector of two values (which in this case is our `a` and `b`)
    + A function that takes in said vector, in our case fitting `squared.error()`

Let's inspect the result 

```r
$par
[1] -350.786736    7.718158

$value
[1] 1492936

$counts
function gradient
     111       NA

$convergence
[1] 0

$message
NULL
```

`par` is similar to the values returned when we call `lm`
`value` is the supposedly lowest value of squared error that `optim` predicts
`counts` is the number of time optim ran test on the function given 
`convergence` is the exit status, meaning an optimisation has been found
`message` lists anything about the process that is worth mentioning

#### Let `b` be zero, let's find `a`

In this example, suppose we set `b` to be zero and want to find the best possible value for `a`, we will use `sapply` to perform a grid search as mentioned before. 

After that, we will do the same for `b` to determine whether or not `optim` could find a single best value

```r
a.error <- function(a)
{
  return(squared.error(heights.weights, a, 0))
}
curve(sapply(x, function (a) {a.error(a)}), from = -1000, to = 1000)
```

![squarederrorvsa](http://i.imgur.com/8WaULNm.png)

From the chart we can see that there is a single optimum value for a. 

```r
b.error <- function(b)
{
  return(squared.error(heights.weights, 0, b))
}

curve(sapply(x, function (b) {b.error(b)}), from = -1000, to = 1000)
```

![squarederrorvsb](http://i.imgur.com/eQahkuk.png)

Looking at this, we can safely conclude that there exist a **global optimum** for the pair of `a` and `b`

### Ridge Regression

Ridge Regression is different from regular regression in that it takes into consideration the value of coefficients (namely `a` and `b`) 

Reintroducing `lambda`, which will be used as a **hyperparmeter** that will regularlize between getting an optimal value and preventing over-fit (minimizing a & b)

```r
ridge.error <- function(heights.weights, a, b, lambda)
{
  predictions <- with(heights.weights, height.to.weight(Height, a, b))
  errors <- with(heights.weights, Weight - predictions)
  return(sum(errors ^ 2) + lambda * (a ^ 2 + b ^ 2))
}
```

We will go with the value of 1 for `lambda`. After this, we can just go ahead and apply `optim` to the ridge error function

```r
lambda <- 1

optim(c(0, 0),
      function (x)
      {
        ridge.error(heights.weights, x[1], x[2], lambda)
      })
```

```r
$par
[1] -340.434108    7.562524

$value
[1] 1612443

$counts
function gradient
     115       NA

$convergence
[1] 0

$message
NULL
```

Aside from the slope & interception, we can also take a look at the singled out examples for a & b just to see how ridge regression perform 

```r
a.ridge.error <- function(a, lambda)
{
  return(ridge.error(heights.weights, a, 0, lambda))
}
curve(sapply(x, function (a) {a.ridge.error(a, lambda)}), from = -1000, to = 1000)

b.ridge.error <- function(b, lambda)
{
  return(ridge.error(heights.weights, 0, b, lambda))
}
curve(sapply(x, function (b) {b.ridge.error(b, lambda)}), from = -1000, to = 1000)
```

![a-ridge](http://i.imgur.com/RzH7FNf.png)
![b-ridge](http://i.imgur.com/RsMaflN.png)

One last example is to apply absolute value to the error function, creating a sharper curve at the optimum point. However, this doesn't work well with `optim` because it the crust is to sharp, therefore `optim` can't decide which direction to move toward, therefore it's hard to find a global optimum 

```r
absolute.error <- function(heights.weights, a, b)
{
  predictions <- with(heights.weights, height.to.weight(Height, a, b))
  errors <- with(heights.weights, Weight - predictions)
  return(sum(abs(errors)))
}

a.absolute.error <- function(a)
{
  return(absolute.error(heights.weights, a, 0))
}

curve(sapply(x, function (a) {a.absolute.error(a)}), from = -1000, to = 1000)
```

![abs-ridge](http://i.imgur.com/cMVbszk.png)

### Code Breaking as Optimization

In this chapter we will take a look at a simple Caesar Cipher using ROT13, and how to find an optimal way to decipher the text. 

First we need to create a list of alphabet letters and create the ciphers

```r
english.letters <- c('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k',
                     'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v',
                     'w', 'x', 'y', 'z')

caesar.cipher <- list()

inverse.caesar.cipher <- list()

for (index in 1:length(english.letters))
{
  caesar.cipher[[english.letters[index]]] <- english.letters[index %% 26 + 1]
  inverse.caesar.cipher[[english.letters[index %% 26 + 1]]] <- english.letters[index]
}

print(caesar.cipher)
```

```r
apply.cipher.to.string <- function(string, cipher)
{
  output <- ''

  for (i in 1:nchar(string))
  {
  output <- paste(output, cipher[[substr(string, i, i)]], sep = '')
  }
  
  return(output)
}

apply.cipher.to.text <- function(text, cipher)
{
  output <- c()
  
  for (string in text)
  {
    output <- c(output, apply.cipher.to.string(string, cipher))
  }
  
  return(output)
}

apply.cipher.to.text(c('sample', 'text'), caesar.cipher)
```

> Sample Cipher: [1] "tbnqmf" "ufyu"

##### Building the Decryption Algorithm 

Now that we have the ciphers, we need to figure out a way to **incrementally rule out** bad algorithms and move toward better ones, and eventually an optimal deciphering rule. 

- The process will be **three-fold**: 
    + Define a measure of quality for the rule
    + Defind an algorithm for creating new rules (modifying old one)
    + Defind an algorithm for moving toward optimum

A common idea for quality of cipher is to test whether or not a text makes sense, as in written in English language or just random nonsense.

##### Introducing the Metric 

To do this, we will store the words in a `lexical database`, then them indivdually and multiply them to get a final probability value. However, we need to be careful because if word were to be recognised as *nonsensical* by the computer, it will return a value of zero, which would ruin our calculation. Therefore, we will apply a *very small* value, called the `epsilon`.

Another problem that might come up is that we can't use `optim` like we did for the regression problem, because decryption rules aren't fixed values and they won't produce a smooth graph for us to work with. Therefore, we'll use Metropolis

With this in mind, we will proceed to continually improve a proposed decryption rule, for a numerous number of times, then judge the output to see whether or not the result is in English, or just random text.

##### Method of Introducing New Rule 

Each time we propose a modification to the existing rule, it will only be a small one (Disturb it in one place and not the whole logic). 

- Furthermore, we don't want to use **greedy optimization** where we always go with the rule that produces higher probability. Instead, we will go by: 
    + If `P(new rule)` > `P(old rule)`, go with **new rule**
    + If `P(new rule)` < `P(old rule)`:
        * Go with old rule *sometimes*
        * Go with new rule `P(new)/P(old)`% of the time

> This is so that we don't blindly follow probability and ensure that we use all rules > 0% of the time

##### The Toolset 

We now want to put the proposed plan into action and generate the functions for our ciphers 

```r
generate.random.cipher <- function()
{
  cipher <- list()
  
  inputs <- english.letters
  
  outputs <- english.letters[sample(1:length(english.letters), length(english.letters))]
  
  for (index in 1:length(english.letters))
  {
    cipher[[inputs[index]]] <- outputs[index]
  }
  
  return(cipher)
}

modify.cipher <- function(cipher, input, output)
{
  new.cipher <- cipher
  
  new.cipher[[input]] <- output
  
  old.output <- cipher[[input]]
  
  collateral.input <- names(which(sapply(names(cipher),
                                         function (key) {cipher[[key]]}) == output))
  
  new.cipher[[collateral.input]] <- old.output
  
  return(new.cipher)
}

propose.modified.cipher <- function(cipher)
{
  input <- sample(names(cipher), 1)
  
  output <- sample(english.letters, 1)
  
  return(modify.cipher(cipher, input, output))
}
```

Next, we're going to load the `lexical database`, which was created from counting their occurences on Wikipedia 

```r
load(file.path('data', 'lexical_database.Rdata'))
```

Here's the probability of some simple words

| Word | Probability | 
| :---: | :-----------: |
| a | 0.01617576 |
| the | 0.05278924 | 
| he | 0.003205034 | 
| she | 0.0007412179 | 
| data | 0.0002168354 | 

Our next step is to write a function to collect individual word's probability, or apply the `epsilon` if not found. 

One problem that comes up is using raw value cause the computation to be **unstable**, therefore, we'll use `log` instead

```r
one.gram.probability <- function(one.gram, lexical.database = list())
{
  lexical.probability <- lexical.database[[one.gram]]
  
  if (is.null(lexical.probability) || is.na(lexical.probability))
  {
  return(.Machine$double.eps)
  }
  else
  {
  return(lexical.probability)
  }
}

log.probability.of.text <- function(text, cipher, lexical.database = list())
{
  log.probability <- 0.0
  
  for (string in text)
  {
    decrypted.string <- apply.cipher.to.string(string, cipher)
    log.probability <- log.probability +
    log(one.gram.probability(decrypted.string, lexical.database))
  }
  
  return(log.probability)
}
```

Finally, we create a metropolis function to continually pick out a new cipher for trials, as described before

```r
metropolis.step <- function(text, cipher, lexical.database = list())
{
  proposed.cipher <- propose.modified.cipher(cipher)
  
  lp1 <- log.probability.of.text(text, cipher, lexical.database)
  lp2 <- log.probability.of.text(text, proposed.cipher, lexical.database)
  
  if (lp2 > lp1)
  {
    return(proposed.cipher)
  }
  else
  {
    a <- exp(lp2 - lp1)
    x <- runif(1)
    
    if (x < a)
    {
      return(proposed.cipher)
    }
    else
    {
      return(cipher)
    }
  }
}
```

The last thing to do now is to set seed, apply the function 50,000 times while storing the log value of each iteration in the process, just so we could see how we perform 

```r
decrypted.text <- c('here', 'is', 'some', 'sample', 'text')

encrypted.text <- apply.cipher.to.text(decrypted.text, caesar.cipher)

set.seed(1)

cipher <- generate.random.cipher()

results <- data.frame()

number.of.iterations <- 50000

for (iteration in 1:number.of.iterations)
{
  log.probability <- log.probability.of.text(encrypted.text,
                                             cipher,
                                             lexical.database)
  
  current.decrypted.text <- paste(apply.cipher.to.text(encrypted.text,
                                                       cipher),
                                  collapse = ' ')
  
  correct.text <- as.numeric(current.decrypted.text == paste(decrypted.text,
                                                             collapse = ' '))

  results <- rbind(results,
                   data.frame(Iteration = iteration,
                              LogProbability = log.probability,
                              CurrentDecryptedText = current.decrypted.text,
                              CorrectText = correct.text))
  
  cipher <- metropolis.step(encrypted.text, cipher, lexical.database)
}

#Plotting

ggplot(data = results, aes(x = Iteration, y = LogProbability)) + geom_line(size=.25, colour = "blue") + geom_point(aes(color = CorrectText), size = .5) + scale_colour_gradient(low = "blue", high = "red")

```

![chart](http://i.imgur.com/oT8XfJC.png)

- There are a few things to take from this example: 
    + The correct text was achieved at step **45,609** (see chart), but the algorithm continued nonetheless, which demonstrate that **the best rule isn't always the most applicable one** in human terms
    + We had a good seed value of 1, if we set the seed to some other values, it might have taken many more steps in order to reach the same result
    + Because our algorithm is non-greedy, it can drop a good rule without second thoughts, which causes a lot of variances in our result

