# SIT384 - Week 1 Notes 

This week we are introduced to the course and we took a look at the UFO case study to demonstrate how powerful r is when it comes to analysing data.

## UFO Sighting Case Study 

### Overview 

We are going to work on a comprehensive UFO sighting report dataframe and analyse it.

Loading the data set to study a few possible questions such as:

* Seasonal trends
* Frequencies
* Variation across states 

---

#### Using `read.delim` AND extras

- `read.delim` -> function for reading csv data 
- "stringAsFactors=False" -> Prevent auto conversion from raw to data 
- Specify that "No" header is used
- Set "0" values to NA to prevent errors 

```r
ufo<-read.delim("data/ufo/ufo_awesome.tsv",sep="\t",stringAsFactors=FALSE,header=FALSE,na.strings="")

head(ufo) #Generating header by R
```

#### Setting custom header names

Instead of V1,V2,V3... We want some headers that would help us better understand the column data.

```r
names(ufo)<-c("DateOccured","DateReported","Location","ShortDescription","Duration","LongDesc")
```

> c("...") means constructing a vector

#### Organising Dates: 

We can manually grab date sub-objects and convert them:

```r
ufo$DateOccurred<-as.Date(ufo$DateOccurred,format="%Y%m%d")
ufo$DateReported<-as.Date(ufo$DateReported, format="%Y%m%d")
```

> The command above should produce an error. This is due to some entry being out of format and malformed. 

We then need to to create a "**data vector**" for _good entries and bad entries_, which we will use to reapply into the frame. 

```r
good.rows<-ifelse(nchar(ufo$DateOccurred)>!=8 | nchar(ufo$DateReported)!=8,False,True)
```

The length function gives the number of malformed data
```r
length(which(!good.rows))
```

The final command reapplies the ufo data frame using the vector as conditions
```r
ufo<-ufo[good.rows,]
```

Now we can repeat the formatting codes and it should work

#### Organising locational data: 

We use `lapply` function to format location data using R's regex.

![Organising Location Data](http://i.imgur.com/zKP0cxn.png)

| Name | Usage |
| :------ | :------------:|
| `gsub` | Global Substitution: Used to replace trailing white space |
| `tryCatch` | Use a generic error function to catch errors |
| `strsplit` | Used to catch errors in splitting the delimiter |

**After defining this method, we use `lapply` to apply this sequentualy to all instances in the iterable of locations**

#### Organising location into states: 

1. We aply `rbind` and `city.state` into `do.call` to convert it into a matrix.

2. We then use `transform` to cocatenate the location.matrix into appropriate new rows in the `ufo` list (Converting the US states into lowercase)

3. After this, we construct the `match` function to construct a list of `us.state`

4. The final two lines specify the condition for matching states into the list. If the result is returned positive, the state is added. Otherwise, "NA" will be returned

![Processing Location into States](http://i.imgur.com/iPyGRGC.png)

#### Processing data

We use `summary` to get a sense of the chronological order for these UFO sightings 

We then apply `ggplot2` to the UFO object to get a general trend

##### Building our first `ggplot2` object

1. We start by passing a data frame as our first argument 
2. Then we pass the aesthetic for x-axis to be `DateOccurred`
3. After this, we add a histogram layer with the `geom_histogram` function
4. Finally, since this data spans over a long time, we rescale the x-axis with the `scale_x_date` function 


```r
quick.hist <- ggplot(ufo.us, aes(x = DateOccurred)) +
  geom_histogram() #+ 
  #scale_x_date(breaks = "50 years")
```

> Since this doesn't give us much details, we need to scale down and focus on the recent years where there are most data


##### Focusing on subset

We use the `subset` function to manipulate data to store only instances that happened after a certain date. After this we apply the `nrow` function to get the number of instances.

```r
new.hist <- ggplot(ufo.us, aes(x = DateOccurred)) +
  geom_histogram(aes(fill='white', color='red')) +
  scale_fill_manual(values=c('white'='white'), guide="none") +
  scale_color_manual(values=c('red'='red'), guide="none") +
  scale_x_date(major = "50 years")
```


#### Counting UFO sightings

We use `strftime` --> Convert date into "YYYY-MM" format and add a column.

```r
ufo.us$YearMonth<-strftime(ufo.us$DateOccurred, format="%Y-%m")
```

We use `ddply` --> Maps the selected data into appropriate rows and store as a new object (matrix)

```r
sightings.counts <- ddply(ufo.us, .(USState,YearMonth), nrow)
```

Since there are some months with 0 occurrences, we need to fill in those as well by creating a new data frame with all the dates and states just like the one we have, then merge it with the existing frame and let R do the filling in the blank. 

```r
date.range <- seq.Date(from = as.Date(min(ufo.us$DateOccurred)),
                       to = as.Date(max(ufo.us$DateOccurred)),
                       by = "month")

date.strings <- strftime(date.range, "%Y-%m")

# To fill in the missing dates from the 'sightings.counts' data frame we will need to create a separate data
# frame with a column of states and Year-Months.
states.dates <- lapply(state.abb, function(s) cbind(s, date.strings))
states.dates <- data.frame(do.call(rbind, states.dates),
                           stringsAsFactors = FALSE)

# We use 'merge' to take the counts we have and merge them with the missing dates.  Note, we have to specify

# the columns from each data frame we are using to do the merge, and set 'all' to TRUE, which will fill in 

# this missing dates from the original data frame with NA.
all.sightings <- merge(states.dates,
                       sightings.counts,
                       by.x = c("s", "date.strings"),
                       by.y = c("USState", "YearMonth"),
                       all = TRUE)
```

We clean up the data. Put 0 for all NA values and reset the YearMonth column to Date type as well as refactor the State column

```r
# Covert the NAs to 0's, what we really wanted
all.sightings$Sightings[is.na(all.sightings$Sightings)] <- 0

# Reset the character Year-Month to a Date objects
all.sightings$YearMonth <- as.Date(rep(date.range, length(state.abb)))

# Capitalize the State abbreviation and set as factor
all.sightings$State <- as.factor(all.sightings$State)
```

#### Plotting Data

After all is done, we build another `ggplot2` object

```r
# First we have to create a ggplot2 object and then create a geom layer, which in this case is a line.
# Additional points of note:
# (1) facet_wrap() will create separate plots for each state on a 10x5 grid.
# (2) theme_bw() changes the default ggplot2 style from grey to white (personal preference).
# (3) scale_color_manual() sets the line color to dark blue.
# (4) scale_x_date() scales the x-axis as a date, with major lines every 5 years.
# (5) xlab() and ylab() set axis labels.
# (6) opts() sets a title for the plot

state.plot <- ggplot(all.sightings, aes(x = YearMonth,y = Sightings)) +
  geom_line(aes(color = "darkblue")) +
  facet_wrap(~State, nrow = 10, ncol = 5) + 
  theme_bw() + 
  scale_color_manual(values = c("darkblue" = "darkblue"), guide = "none") +
  scale_x_date(date_breaks = "5 years", date_labels = '%Y') +
  xlab("Years") +
  ylab("Number of Sightings") +
  ggtitle("Number of UFO sightings by Month-Year and U.S. State (1990-2010)")
```

![](http://i.imgur.com/W7GKntm.png)

## Data Exploration

There are two parts to data exploration: 

| Name | Function | 
| ---- | :----: |
| Exploration | Explore and find hidden patterns in the data |
| Confirmation | Confirming the discovered patterns, remove fallacies |


> Data Set: A big table of numbers & strings in which the rows are descriptions & observations the collumns are attributes.

### Summary Methods

- Numerial
- Dimensionality 
- Correlation 
- Standard Varation & Variances











