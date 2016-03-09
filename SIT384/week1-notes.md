# SIT384 - Week 1 Notes 

This week we are introduced to the course and we took a look at the UFO case study to demonstrate how powerful r is when it comes to analysing data.

## UFO Sighting Case Study 

### Overview 

Loading the data set to study a few possible questions such as: 
    - Seasonal trends
    - Frequencies
    - Variation across states 

### Using R for analysis 

First we need look at how to import data and format them in a way that proper analysis is possible.

#### Example Code

##### Using read.delim AND extras

- `read.delim` -> function for reading csv data 
- "stringAsFactors=False" -> Prevent auto conversion from raw to data 
- Specify that "No" header is used
- Set "0" values to NA to prevent errors 

```r
ufo<-read.delim("data/ufo/ufo_awesome.tsv",sep="\t",stringAsFactors=FALSE,header=FALSE,na.strings="")

head(ufo) #Generating header by R
```

##### Setting custom header names

Instead of V1,V2,V3... We want some headers that would help us better understand the column data.

```r
names(ufo)<-c("DateOccured","DateReported","Location","ShortDescription","Duration","LongDesc")
```

[!] C means constructing a **vector**

##### Organising Dates: 

We can manually grab date sub-objects and convert them:

```r
ufo$DateOccurred<-as.Date(ufo$DateOccurred,format="%Y%m%d")
ufo$DateReported<-as.Date(ufo$DateReported, format="%Y%m%d")
```

The command above should produce an error. This is due to some entry being out of format and malformed. 

We then need to to create a "data vector" for good entries and bad entries, which we will use to reapply into the frame. 

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

##### Organising locational data: 

We use `lapply` function to format location data using R's regex.

![Organising Location Data](http://i.imgur.com/zKP0cxn.png)

- `gsub` --> Global Substitution: Used to replace trailing white space
- `tryCatch` --> Use a generic error function to catch errors 
- `strsplit` --> Used to catch errors in splitting the delimiter

**After defining this method, we use `lapply` to apply this sequentualy to all instances in the iterable of locations**

##### Organising location into states: 

1. We aply `rbind` and `city.state` into `do.call` to convert it into a matrix.

2. We then use `transform` to cocatenate the location.matrix into appropriate new rows in the `ufo` list (Converting the US states into lowercase)

3. After this, we construct the `match` function to construct a list of `us.state`

4. The final two lines specify the condition for matching states into the list. If the result is returned positive, the state is added. Otherwise, "NA" will be returned

![Processing Location into States](http://i.imgur.com/iPyGRGC.png)

#### Processing data

We use `summary` to get a sense of the chronological order for these UFO sightings 

We then apply `ggplot` to the UFO object to get a general trend

##### Focusing on subset

We use the `subset` function to manipulate data to store only instances that happened after a certain date. After this we apply the `nrow` function to get the number of instances.

##### Counting UFO sights

- `strftime` --> Convert date into "YYYY-MM" format
- `ddply` --> Maps the selected data into appropriate rows and store as a new object (matrix)

#### Plotting Data

![ggplot function](http://i.imgur.com/rksKCeF.png)

![plotted data](http://i.imgur.com/WJ4CunL.png)

![plotted graph](http://i.imgur.com/H8O2q4g.png)

## Data Set

**A big table of numbers & strings in which the rows are descriptions & observations the collumns are attributes.**

### Summary Methods

- Numerial
- Dimensionality 
- Correlation 
- StdVar & Variances











