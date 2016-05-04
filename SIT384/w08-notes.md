# SIT384 - Week 8 - MDS: Visualy Exploring US Senators Similartities 

#### Customer Product Rating in R 

Using Transposed Matrix Multiplication, we are able to determine the correlation between customers.

The first example we will look at is a product review dataset of 4 different customers. Each product review can be 0 (skip), 1 (thumbs up), or -1 (thumbs down). We will randomly fill in the matrix using the `sample` function.

```r
set.seed(851982) # To make sure results are consistent
ex.matrix <- matrix(sample(c(-1, 0, 1), 24, replace = TRUE),
                    nrow = 4,
                    ncol = 6)
row.names(ex.matrix) <- c('A', 'B', 'C', 'D')
colnames(ex.matrix) <- c('P1', 'P2', 'P3', 'P4', 'P5', 'P6')
```

##### The need for distance metrics 

After getting the example matrix, we want to generate a distance metric, this can be done through multiplying the matrix with its transposed version. This is done through calling the `%*%` operator. 

```r
ex.mult <- ex.matrix %*% t(ex.matrix)
ex.mult

#A B C D
#A 2 -1 -1 1
#B -1 4 0 -1
#C -1 0 3 -1
#D 1 -1 -1 3
```

Since the rating coding scheme is based on 0,1 & -1, we don't have a lot of **Divergence**, hence making our correlation limited. We want to introduce a way to extrapolate the results to get deeper insights into the information collected. 

To do this, we will make use of `Euclidean distance`, which is calculated through subtracting the distance vectors, square the differences, sum them and then taking the square root. 

```r
sqrt(sum((ex.mult[1, ] - ex.mult[4, ]) ^ 2))
#[1] 2.236068

ex.dist <- dist(ex.mult)
ex.dist
```

|  |  A     |   B |  C |
|---|--------|-----|---|  
| **B** | 6.244998 | | | 
| **C** | 5.477226 | 5.000000 | |
| **D** | 2.236068 | 6.782330 | 6.082763 | 


#### Classical MDS

We need a way of visualising the correlation between data, so we will plot the different points using the function `cmdscalle()`

```r
ex.mds <- cmdscale(ex.dist)
plot(ex.mds, type = 'n')
text(ex.mds, c('A', 'B', 'C', 'D'))
```

![dist](http://i.imgur.com/yrxIcD3.png)

The diagram gives us a quick look into the distance between customers' correlations, however with the exact numerical plotting it's challenging to interpret a larger data set 

#### US Congress case study 

In this example, we will use the same MDS set up to look into correlations between members of congress in two parties, through analysing the rollcall data 

```r
library('foreign') #Used to convert exotic data files into R data frames 
library('ggplot2')

data.dir <- file.path("data", "roll_call")
data.files <- list.files(data.dir)

rollcall.data <- lapply(data.files,
                        function(f)
                        {
                          read.dta(file.path(data.dir, f), convert.factors = FALSE)
                        })

dim(rollcall.data[[1]])
head(rollcall.data[[1]])

#cong id state dist lstate party eh1 eh2 name V1 V2 V3 ... V638
#1 101 99908 99 0 USA 200 0 0 BUSH 1 1 1 ... 1
#2 101 14659 41 0 ALABAMA 100 0 1 SHELBY, RIC 1 1 1 ... 6
#3 101 14705 41 0 ALABAMA 100 0 1 HEFLIN, HOW 1 1 1 ... 6
#4 101 12109 81 0 ALASKA 200 0 1 STEVENS, TH 1 1 1 ... 1
#5 101 14907 81 0 ALASKA 200 0 1 MURKOWSKI, 1 1 1 ... 6
#6 101 14502 61 0 ARIZONA 100 0 1 DECONCINI, 1 1 1 ... 6
```

![cheatsheet](http://i.imgur.com/xijXYjX.png)

In order to simplify the records and have an easier time in analysis, we divide the rollcall into 3 groupings

![groupings](http://i.imgur.com/RUg1shd.png)

We then move on to extracting the simplified version of the votes, as well as their names, party affiliation. 

We want to perform 3 `ifelse` call to the data frame and convert the results to matrix for better analysis. 

> We called subset(state < 99) to excluse the Vice President of the US. 


```r
rollcall.simplified <- function(df)
{
  no.pres <- subset(df, state < 99)
  
  for(i in 10:ncol(no.pres))
  {
    no.pres[,i] <- ifelse(no.pres[,i] > 6, 0, no.pres[,i])
    no.pres[,i] <- ifelse(no.pres[,i] > 0 & no.pres[,i] < 4, 1, no.pres[,i])
    no.pres[,i] <- ifelse(no.pres[,i] > 1, -1, no.pres[,i])
  }
  
  return(as.matrix(no.pres[,10:ncol(no.pres)]))
}

rollcall.simple <- lapply(rollcall.data, rollcall.simplified)
```

##### Performing MDS Clustering 

We will follow the following steps to explore MDS Clustering on the data from congress:

+ Creating senator-by-senator distance matrix
+ `lapply` multiplication transposed matrix
+ Apply `cmdscale` for clustering 
+ Attach appropriate data (name, party, congress) to the data frame 
+ Plot the clustering

```r
rollcall.dist <- lapply(rollcall.simple, function(m) dist(m %*% t(m)))

rollcall.mds <- lapply(rollcall.dist,
                       function(d) as.data.frame((cmdscale(d, k = 2)) * -1))

congresses <- 101:111

for(i in 1:length(rollcall.mds))
{
  names(rollcall.mds[[i]]) <- c("x", "y")
  
  congress <- subset(rollcall.data[[i]], state < 99)
  
  congress.names <- sapply(as.character(congress$name),
                           function(n) strsplit(n, "[, ]")[[1]][1])
  
  rollcall.mds[[i]] <- transform(rollcall.mds[[i]],
                                 name = congress.names,
                                 party = as.factor(congress$party),
                                 congress = congresses[i])
}

head(rollcall.mds[[1]])

#x y name party congress
#2 -11.44068 293.0001 SHELBY 100 101
#3 283.82580 132.4369 HEFLIN 100 101
#4 885.85564 430.3451 STEVENS 200 101
#5 1714.21327 185.5262 MURKOWSKI 200 101
#6 -843.58421 220.1038 DECONCINI 100 101
#7 1594.50998 225.8166 MCCAIN 200 101

cong.110 <- rollcall.mds[[9]]

base.110 <- ggplot(cong.110, aes(x = x, y = y)) +
  scale_size(range = c(2,2), guide = 'none') +
  scale_alpha(guide = 'none') +
  theme_bw() +
  theme(axis.ticks = element_blank(),
        axis.text.x = element_blank(),
        axis.text.y = element_blank(),
        panel.grid.major = element_blank()) +
  ggtitle("Roll Call Vote MDS Clustering for 110th U.S. Senate") +
  xlab("") +
  ylab("") +
  scale_shape(name = "Party", breaks = c("100", "200", "328"),
              labels = c("Dem.", "Rep.", "Ind."), solid = FALSE) +
  scale_color_manual(name = "Party", values = c("100" = "black",
                                                "200" = "dimgray",
                                                "328"="grey"),
                     breaks = c("100", "200", "328"),
                     labels = c("Dem.", "Rep.", "Ind."))

print(base.110 + geom_point(aes(shape = party,
                                alpha = 0.75,
                                size = 2))) #By Party
print(base.110 + geom_text(aes(color = party,
                               alpha = 0.75,
                               label = cong.110$name,
                               size = 2))) #By Name
```

From the graph below, we can see that each part forms a clear, divided cluster with their member belonging to a side of the plot 

![clustering](http://i.imgur.com/bOyQnVu.png)
![clusterbyname](http://i.imgur.com/3j8OOob.png)

Finally, we can also view the change in clustering from congress to congress over time 

![clustering-over-time](http://i.imgur.com/hcgoKnh.png)
