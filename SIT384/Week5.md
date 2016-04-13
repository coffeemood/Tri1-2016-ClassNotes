# SIT384 - Week 5- Text Regression 

#### Introducing `poly` for fitting

#### Cross Validation

1. Collect hypothesis
2. Build a model & Gather Data
3. Test it

> We want to remove the data used in the fitting process when it comes to making the prediction 

Therefore we want to *randomly* split the data into two parts. One for fitting data and for testing predictions

```r
n <- length(x)
indices <- sort(sample(1:n, round(0.5 * n)))

training.x <- x[indices] training.y <- y[indices] 
test.x <- x[-indices] test.y <- y[-indices]
training.df <- data.frame(X = training.x, Y = training.y) 
# training.df will be used to fit polynomial regression
test.df <- data.frame(X = test.x, Y = test.y)
```

After splitting the data, we want to use **RMSE** to test the errors

```r
# Defining RMSE
rmse <- function(y, h)
{
  return(sqrt(mean((y - h) ^ 2)))
}
```

Next, we want to recursively apply rmse to the two df. We will only use up to polynomial degree of 12, not 14

```r
performance <- data.frame()

for (d in 1:12) {

poly.fit <- lm(Y ~ poly(X, degree = d), data = training.df)
performance <- rbind(performance, data.frame(Degree = d,
Data = 'Training',
RMSE = rmse(training.y, predict(poly.fit))))
performance <- rbind(performance, data.frame(Degree = d,
Data = 'Test',
RMSE = rmse(test.y, predict(poly.fit,
newdata = test.df))))

}
```

![plottedrmsevsdegree](http://i.imgur.com/03HabIV.png)

The plotted polynomial graph comparing the rmse level as the degree increases show us the picking too high or too low of a degree will lead to inconsistency between performance (Training vs Testing). 

The model is either **not complex enough** at low degrees (Doesn't fit with the data set) or **too complex** at too high a degree (Fit too well with one data set which brings trouble to when used in testing)

> We can see that picking level 6 polynomial is optimal. This is good for avoiding underfit and overfit

#### Regularization 

`glmnet` package comes with many linear models that helps us predict the fitting of our model 

Output: 

- `Df` --> Returns non-zero coefficients 
- `%Dev` --> R^2 of this model
- `Lambda` --> Generally present the complexity of our model (**Hyperparameter**)

![lambdachart](http://i.imgur.com/eLVUP3n.png)

Plotting the lambda value against RMSE, we can see the best lambda value is around 0.05

### Test Regression 

When analysing texts, we often run into the trouble of having more words than observation. This leads to us using more 2-grams, 3-grams data. Because of this, we want to use **regularization** to attempt to prevents overfit 

#### Predicting Bestsellers 

In this example, we attempt to convert the descriptions of each book into word vector, do a little cleaning with the data and fit them in order to predict book sales based on description 

```r
ranks <- read.csv(file.path('data', 'oreilly.csv'),
                  stringsAsFactors = FALSE)

library('tm')

documents <- data.frame(Text = ranks$Long.Desc.)
row.names(documents) <- 1:nrow(documents)

corpus <- Corpus(DataframeSource(documents))
corpus <- tm_map(corpus, tolower)
corpus <- tm_map(corpus, stripWhitespace)
corpus <- tm_map(corpus, removeWords, stopwords('english'))

dtm <- DocumentTermMatrix(corpus)

# Converting the dtm to a matrix
x <- as.matrix(dtm)
y <- rev(1:100)

set.seed(1)

library('glmnet')

```

Upon setting up the data, we want to test it against several lambda value and *randomly* splitting it 50 times for each value to better assess our performance 

```
performance <- data.frame()

for (lambda in c(0.1, 0.25, 0.5, 1, 2, 5))
{
  for (i in 1:50)
  {
    indices <- sample(1:100, 80) #Randomly returning indices for splitting
    
    training.x <- x[indices, ]
    training.y <- y[indices]
    
    test.x <- x[-indices, ]
    test.y <- y[-indices]
    
    glm.fit <- glmnet(training.x, training.y)
    
    predicted.y <- predict(glm.fit, test.x, s = lambda)
    
    rmse <- sqrt(mean((predicted.y - test.y) ^ 2))

    performance <- rbind(performance,
                         data.frame(Lambda = lambda,
                                    Iteration = i,
                                    RMSE = rmse))
  }
}
```

Finally, plot RMSE against Lambda to see which value performed best 

```r
ggplot(performance, aes(x = Lambda, y = RMSE)) +
  stat_summary(fun.data = 'mean_cl_boot', geom = 'errorbar') +
  stat_summary(fun.data = 'mean_cl_boot', geom = 'point')
```

![plottederrorbar](http://i.imgur.com/8DBKPVg.png)

### Logistic Regression 

This is a type of regression used for classification, that is using a specified threshold to classify the predictions

- Main differences 
  + Use `error rates` rather than RMSE 
  + Part of `glmnet`
  + Threshold returns 0/1 for classification

There are two ways we can go about creating this threshold:

1. Using an `ifelse` function that uses 0 as the threshold 
2. Using `inv.logit` function from the `boot` package to extract the numerical value of the predictions

We can then run the predictions through the same lambda/split loop that we did for the last example 

```r
set.seed(1)

performance <- data.frame()

for (i in 1:250)
{
  indices <- sample(1:100, 80)
  
  training.x <- x[indices, ]
  training.y <- y[indices]
  
  test.x <- x[-indices, ]
  test.y <- y[-indices]
  
  for (lambda in c(0.0001, 0.001, 0.0025, 0.005, 0.01, 0.025, 0.5, 0.1))
  {
    glm.fit <- glmnet(training.x, training.y, family = 'binomial')
    
    predicted.y <- ifelse(predict(glm.fit, test.x, s = lambda) > 0, 1, 0)
    
    error.rate <- mean(predicted.y != test.y)

    performance <- rbind(performance,
                         data.frame(Lambda = lambda,
                                    Iteration = i,
                                    ErrorRate = error.rate))
  }
}
```

Finally, plot error rate vs lambda value

```r
ggplot(performance, aes(x = Lambda, y = ErrorRate)) +
  stat_summary(fun.data = 'mean_cl_boot', geom = 'errorbar') +
  stat_summary(fun.data = 'mean_cl_boot', geom = 'point') +
  scale_x_log10()
```

![plottederrorratevslambda](http://s18.postimg.org/4j1uibtsp/Screen_Shot_2016_04_13_at_4_00_54_PM.png)

From the figure we can see that using Error Rates is very much different than using RMSE. This gives us a sense that *regularization* forces us to make use of **simpler model**



