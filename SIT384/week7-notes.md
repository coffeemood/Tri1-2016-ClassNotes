# PCA: Building a Market Index 

#### Unsupervised Learning 

We learned before that when we have a correct dataset used to provide feedback in the process of training our predictions, it is called `supervised learning`

Now we want to move on to `unsupervised learning` where there isn't a metric/indicator as to how well we're doing. 

In this chapter we take a look at a data set providing stock price history from 2002 -> 2011. Since this table has too many columns to deal with, we want to apply some sort of dimensionality reduction. In this case, we will make use of an approach called **Principal Components Analysis** (PCA)

The idea is to create new columns and observe the correlation between columns in our data set. Then, we will reduce those correlated columns to one single columns that demonstrates an underlying pattern. 

```r
prices <- read.csv(file.path('data', 'stock_prices.csv'),
                   stringsAsFactors = FALSE)

#Using the package lubridate, we format the dates into proper date variables
library('lubridate')
prices <- transform(prices, Date = ymd(Date))

#Using the `cast` function of the `reshape` pkg, we rearrange the raw data into a proper table
library('reshape')
date.stock.matrix <- cast(prices, Date ~ Stock, value = 'Close')
#                                  ^        ^       
#                                 row      col         

# Removing missing entries
prices <- subset(prices, Date != ymd('2002-02-01'))
prices <- subset(prices, Stock != 'DDR')
date.stock.matrix <- cast(prices, Date ~ Stock, value = 'Close')
```

- After we've finished formatting the table, we want to 
    + Use `cor` to create a matrix of correlations as mentioned before. 
    + Convert that matrix into a single vector
    + Create a density chart 

```r
cor.matrix <- cor(date.stock.matrix[, 2:ncol(date.stock.matrix)])
correlations <- as.numeric(cor.matrix)

ggplot(data.frame(Correlation = correlations),
  aes(x = Correlation, fill = 1)) +
  geom_density() +
  theme(legend.position = 'none')
```

![corrdens](http://i.imgur.com/2PsDJ3Y.png)    

After getting the principal component analysis, we extract the first principal compnent as it covers most of the correlations. We then build another density plot with it

```r
principal.component <- pca$loadings[, 1]

loadings <- as.numeric(principal.component)

ggplot(data.frame(Loading = loadings),
  aes(x = Loading, fill = 1)) +
  geom_density() +
  theme(legend.position = 'none')
```

![pcdens](http://i.imgur.com/9e44mY4.png)

Finally, we will create our market index using the `predict` function

```r
market.index <- predict(pca)[, 1]
```

In order to test our prediction, we will load a preexisting data set of Dow Jones Index, then run a pot comparing their performance. 

```r
dji.prices <- read.csv(file.path('data', 'DJI.csv'),
                       stringsAsFactors = FALSE)
dji.prices <- transform(dji.prices, Date = ymd(Date))

# Removing some stuffs
dji.prices <- subset(dji.prices, Date > ymd('2001-12-31'))
dji.prices <- subset(dji.prices, Date != ymd('2002-02-01'))

# Splitting by index & dates 
dji <- with(dji.prices, rev(Close))
dates <- with(dji.prices, rev(Date))

# Building a comparison data frame
comparison <- data.frame(Date = dates,
                         MarketIndex = market.index,
                         DJI = dji)

ggplot(comparison, aes(x = MarketIndex, y = DJI)) +
  geom_point() +
  geom_smooth(method = 'lm', se = FALSE)
```


![djivsmi](http://i.imgur.com/SkxpGZT.png)
It turned out the our index is inversely correlated with the DJI, as seen in the figure above, so we want to simply flip our data by multiplying the index by **-1**

```r
comparison <- transform(comparison, MarketIndex = -1 * MarketIndex)

ggplot(comparison, aes(x = MarketIndex, y = DJI)) +
  geom_point() +
  geom_smooth(method = 'lm', se = FALSE, color = 'red')
```

![flipped](http://i.imgur.com/0NfcJ9m.png)

Now that we've seen how accurate our predictions are, we'd like to take a closer look at how the prediction actually follows the trends of DJI. 

- To do this, we will make use of the `melt` function which works as follow
    + Take in a data frame
    + Take in an `id.vars` which will be remain in the DF and act as the main iterator
    + Take in `measured variables` to melt into a single column of variable for easy data exploration. In this case, we didn't specify them so the default is used which is all columns other than **Date**

```r
comparison <- transform(comparison, MarketIndex = scale(MarketIndex))
comparison <- transform(comparison, DJI = scale(DJI))

alt.comparison <- melt(comparison, id.vars = 'Date')

names(alt.comparison) <- c('Date', 'Index', 'Price')

ggplot(alt.comparison, aes(x = Date, y = Price, group = Index, color = Index)) +
  geom_point() +
  geom_line()
```

![finalcomparison](http://i.imgur.com/IqS5aNP.png)

As we can see from the comparison, PCA did a fairly good job at accurately predicting market trends, followed the movement of the DJI closely, and there weren't many gaps in between the two lines.