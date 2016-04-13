# SIT384 - Week 4 - Regression 

### Regression  

Regression is a simple idea. We take on set of numbers as input and try to **predict** another set of numbers as output. 

Regression is different to classification in that it returns *raw* number and not booleans-like classifcation. 

#### Baseline Model 

A seemingly naive way of predicting future outcome without external inputs is to take the mean value. This might seem counter-intutive, but it proves to be quite accurate in the following example: 

![chart](http://i.imgur.com/6RFGAxh.png)

> The mean value of life-expectancy is clearly shifted from non-smokers to smokers in the chart 

#### Squared Error 

We have: 

- `y` is the true output
- `h` is our prediction of `y`

--> The squared error is `(y-h)^2`  

This is often used to roughly measure our errors, or in otherwords, the quality of our predictions. 

> We took the squared error of the mean longevity, which in this case is the rounded up number 73.

```r
ages <- read.csv('data/longevity.csv') 
guess <- 73
with(ages, mean((AgeAtDeath - guess) ^ 2))
#[1] 32.991
```

This alone doesn't prove anything. We still can't be sure if it's the best possible guess. In order to know this, we need to loop through possible guesses and compare them

```r
ages <- read.csv('data/longevity.csv') 
guess.accuracy <- data.frame()
for (guess in seq(63, 83, by = 1))
{
prediction.error <- with(ages,
mean((AgeAtDeath - guess) ^ 2)) 
guess.accuracy <- rbind(guess.accuracy,
data.frame(Guess = guess,
Error = prediction.error))
}

ggplot(guess.accuracy, aes(x = Guess, y = Error)) + geom_point() +
geom_line()


```

![charted](http://i.imgur.com/9mXxpqg.png)

As we could see 73 returns the smallest possible squared error

#### Root Mean Squared Error (RMSE)

Instead of using the general mean for everything, we can use the mean for smokers and non-smokers seperately when predicting a new person 

> sqrt(MSE)

```r 
ages <- read.csv(file.path('data', 'longevity.csv'))

constant.guess <- with(ages, mean(AgeAtDeath))

with(ages, sqrt(mean((AgeAtDeath - constant.guess) ^ 2)))

smokers.guess <- with(subset(ages, Smokes == 1),
                      mean(AgeAtDeath))

non.smokers.guess <- with(subset(ages, Smokes == 0),
                          mean(AgeAtDeath))
# Using the transform function, we can bind the NewPrediction to ages depends on whether or not a person smokes
ages <- transform(ages,
                  NewPrediction = ifelse(Smokes == 0,
                                         non.smokers.guess,
                                         smokers.guess))

with(ages, sqrt(mean((AgeAtDeath - NewPrediction) ^ 2)))
```

![chartroot]()

> When talking about dummy vars, we want to include non-binary classfication factors but not too many because it can be counter-intuitive 

> 

#### Linear Regression

+ **Two** assumptions 
    * **Separability/Additivity**: Use multiple information in isolation to each other
    * **Monotonicity/Linearity**: 
        - Monotonicity means that change in input leads to change in output (Always changes up/down) 
        - Linearity: The plotted data should directly reflects the input & output (**regression line**).

> Standard Linear Regression deals with line. It's possible to fit into curves and waves but that's more advanced and will come later 

Below, the `geom_smooth` function is used to produce a mapping **linear model** 
```r
library('ggplot2')
heights.weights <- read.csv('data/01_heights_weights_genders.csv', header = TRUE,
sep = ',')
ggplot(heights.weights, aes(x = Height, y = Weight)) + geom_point() +
geom_smooth(method = 'lm') 
```

![scatterheight](http://i.imgur.com/XNVj6S3.png)

**Regression formula used in `lm`**: `A ~ B` --> Predicting A from B  

> Notes on storing global variable: It's better to specifically point out the data set being used instead of storing global variable. This is because sometimes it's easy to lose track of all the data that have been imported. 


##### Deriving Parameters 

`lm` -> Linear model. Takes in two meaningful values & a data frame 

```r
fitted.regression <- lm(Weight ~ Height,
data = heights.weights)
```

`coef` -> Return coefficient for a linear model 

```r
coef(fitted.regression) 

#(Intercept) Height 
#-350.737192 7.717288

intercept <- coef(fitted.regression)[1] 
slope <- coef(fitted.regression)[2]
# predicted.weight <- intercept + slope * observed.height
# predicted.weight == -350.737192 + 7.717288 * observed.height
```

The output above means: 

- A change of 1 inch in height will result in around 7.7 pounds change in weight 
- An interception of -350 means that if a person were to have zero height, her weight would be -350 pounds. 

> From that interception can deduce that this model isn't perfect and only fits for the general population, not outliers.

Next we can extract the difference between true values and predicted values for `residual errors`, or we can use the `residual` function directly 

```r 
true.values <- with(heights.weights, Weight)
errors <- true.values - predict(fitted.regression)
# OR
residuals(fitted.regression)
```

We could also identify a residual error trend by plotting it

```r
plot(fitted.regression, which = 1)
# which = 1 means we are only using the first regression diagnostic plot, there are other plots
```


##### R Squared

Having these residuals is great, but sometimes an outlier prediction will leave our performance really messed up. Furthermore, larger data sets tend to return larger error value so so even the `RMSE` won't fix that. To combat this, we use `R^2`

`R^2` returns value between 0 and 1. If we perform worse than the mean, it returns 0. If we perform perfectly, it returns 1. Never more than that. 

To produce `R^2` we get the mean and model mse, and do a simple arithmetic operation 

```r
mean.mse <- 1.09209343
model.mse <- 0.954544

r2 <- 1 - (model.mse / mean.mse)
r2
```

### Case Study - Predicting Web Traffic 

We will take the case of studying popular websites and do an analysis as to which factor plays the biggest part in determining future web traffic. The few factors we will look at are: `Rank`, `PageViews`, `UniqueVisitors`, `HasAdvertising`, and `IsEnglish`.

> The rank column is interesting because its value isn't weigh in absolute terms but rather used just to specify the order of popularity (Top1000)

- `PageViews` vs. `UniqueVisitors`:
    + PageViews is the measure of traffic that the site has been getting. **This will be the choice for our output**.
    + Through comparing and contrasting PV with UniqeVisitors, we get a sense of which website has many repeat views and which website doesn't

- `HasAdvertising` & `IsEnglish`: 
    + HasAdvertising attempts to show us any correlation between traffic and whether or not a site has advertising. IsEnglish is also similar, that is whether or not a site is in English. 
    + It should be noted that these two are prime examples of correlation doesn't mean causation. Regression could do well in showing us any connection between stats but it cannot tell us which cause which.

##### Plotting PageViews vs UniqueVisitors

```r
top.1000.sites <- read.csv('data/top_1000_sites.tsv', sep = '\t',
stringsAsFactors = FALSE)
ggplot(top.1000.sites, aes(x = log(PageViews), y = log(UniqueVisitors))) + geom_point()
# We use log here because the data can be different from site to site, and with regards to sample we don't want that messing up our data structure. 
```

![scatterplot](http://i.imgur.com/Olc4E3z.png)

```r
ggplot(top.1000.sites, aes(x = log(PageViews), y = log(UniqueVisitors))) + geom_point() +
geom_smooth(method = 'lm', se = FALSE)
# Using geomsmooth(method = lm) to draw a line
```

![smoothedlm](http://i.imgur.com/YFgB4iY.png)

Next step, we want to use `lm` to extract the slope and interception

```r
lm.fit <- lm(log(PageViews) ~ log(UniqueVisitors), data = top.1000.sites)
```

Upon getting the fitted line, we want a summary of it 

![resultfit](http://i.imgur.com/HFFPuss.png)

