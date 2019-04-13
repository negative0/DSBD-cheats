# DSBDA SPPU Cheatsheet

## First things first

- R seems to work like a procedural language, but it is interally a Object Oriented Language.

- The assignment operator ``` <- ``` is just a fancy way for writing ``` = ```, both can be used interchangeably.

- If the dataset contains very high number of values, consider to use a subset of these values for exploratory analysis and later take the entire set.

- Never train any model on all values.

- Always remove outliers before feeding to a statistical model.

- Always remove null values before taking mean and medians.

- Always read dataset description before doing anything on it.

- Always summarize data before modifications.

## Outlier/ missing values removal:

Note: Outliers do not equal errors. They should be detected, but not necessarily
removed. Their inclusion in the analysis is a statistical decision.

### Omit all na values:

- This deletes all rows with missing values:
```
df <- na.omit(df)
```

```
dataset$X=ifelse(is.na(dataset$X),ave(dataset$X,FUN=function(x) mean(x,na.rm=TRUE)),dataset$X)
```

OR

Instead of eliminating null values, you can ignore it while calculating menas for each row:

```
rowMeans(dframe, na.rm=TRUE) 
```

### Removing negative values:

```
dframe[dframe < 0] <- NA
```

To consider only non-NaN values, you can subset for not NaN using ```!is.nan```, see this example:

```
mea <- c(2, 4, NaN, 6)
mea
# [1]   2   4 NaN   6
!is.nan(mea) # not NaN, output logical
# [1]  TRUE  TRUE FALSE  TRUE 
mea <- mea[!is.nan(mea)]
# [1] 2 4 6
```

Or you can replace NaN values with some desired value by setting ``` mea[is.nan(mea)] <- ?? ```

#### Points to remember

- NA is not same as NaN

- NA indicates "Not Available" and NaN indicates "Not a Number", both have to be handled seperately.

### Remove other values

```
df$data_col[df$data_col == '?'] <- '0' 
```
```
marx_m <- marx
I <- marx$unit == "cm"
marx_m[I, "height"] <- marx$height[I]/100
I <- marx$unit == "inch"
marx_m[I, "inch"] <- marx$height[I]/39.37
I <- marx$unit == "ft"
marx_m[I, "ft"] <- marx$height[I]/3.28
marx_m$unit <- "m"
```

### C



## Handle categorical data:

- Categorical data is the data with multiple category variables like (Male, Female), (Yes, No).
- This data cannot be processed by machine learning models, so we need to convert this to it's numerical equivalent. 

Example:
Here we are replacing yes and no labels in dataset for the cd column with 0 or 1

```
# handling categorical data ...'no' is 0 and 'yes' is 1
dataset$cd=factor(dataset$cd,levels = c('no','yes'),labels = c(0,1))
```

### Convert continuous data to categorical data

```
db.adult$hours_w[db.adult$hours_per_week < 40] <- " less_than_40"

db.adult$hours_w[db.adult$hours_per_week >= 40 & 
                 db.adult$hours_per_week <= 45] <- " between_40_and_45"

db.adult$hours_w[db.adult$hours_per_week > 45 &
                 db.adult$hours_per_week <= 60  ] <- " between_45_and_60"

db.adult$hours_w[db.adult$hours_per_week > 60 &
                 db.adult$hours_per_week <= 80  ] <- " between_60_and_80"

db.adult$hours_w[db.adult$hours_per_week > 80] <- " more_than_80"

```

- You should prefer creating a new variable rather than replacing the current variable as follows:
```
db.adult$hours_w <- factor(db.adult$hours_w,
                           ordered = FALSE,
                           levels = c(" less_than_40", 
                                      " between_40_and_45", 
                                      " between_45_and_60",
                                      " between_60_and_80",
                                      " more_than_80"))

```
From the summary below we can see how many people belong to each category of the factor variable “hours_w”:
```
summary(db.adult$hours_w)
```

```
      less_than_40  between_40_and_45  between_45_and_60 
              6714              16606               5790 
 between_60_and_80       more_than_80 
               857                195 
```

As percentages we have the following:
```
for(i in 1:length(summary(db.adult$hours_w))){
    
   print(round(100*summary(db.adult$hours_w)[i]/sum(!is.na(db.adult$hours_w)), 2)) 
}
```
- Output - 

```
 less_than_40 		22.26 
 between_40_and_45  55.06 
 between_45_and_60  19.2 
 between_60_and_80  2.84 
 more_than_80  		0.65 
```

With the help of the ```“levels()”``` function, we can see that the factor variable “native_country” has 41 levels. If we build a (logistic regression) predictive model with “native_country” as a covariate, we will end up with 41 additional degrees of freedom due to this categorical variable. This will complicate unnecessarily the analysis and might lead to overfitting. Hence, it is better to group the native countries into several global regions. This coarsening of the data also makes the interpretation of the results easier to comprehend.
```
levels(db.adult$native_country)
```
```
 [1] " Cambodia"                   " Canada"                    
 [3] " China"                      " Columbia"                  
 [5] " Cuba"                       " Dominican-Republic"        
 [7] " Ecuador"                    " El-Salvador"               
 [9] " England"                    " France"                    
[11] " Germany"                    " Greece"                    
[13] " Guatemala"                  " Haiti"                     
[15] " Holand-Netherlands"         " Honduras"                  
[17] " Hong"                       " Hungary"                   
[19] " India"                      " Iran"                      
[21] " Ireland"                    " Italy"                     
[23] " Jamaica"                    " Japan"                     
[25] " Laos"                       " Mexico"                    
[27] " Nicaragua"                  " Outlying-US(Guam-USVI-etc)"
[29] " Peru"                       " Philippines"               
[31] " Poland"                     " Portugal"                  
[33] " Puerto-Rico"                " Scotland"                  
[35] " South"                      " Taiwan"                    
[37] " Thailand"                   " Trinadad&Tobago"           
[39] " United-States"              " Vietnam"                   
[41] " Yugoslavia"                
```
Below we create the new variable “native_region”, where we group the countries by global regions. We first define the regions:
```
Asia_East <- c(" Cambodia", " China", " Hong", " Laos", " Thailand",
               " Japan", " Taiwan", " Vietnam")

Asia_Central <- c(" India", " Iran")

Central_America <- c(" Cuba", " Guatemala", " Jamaica", " Nicaragua", 
                     " Puerto-Rico",  " Dominican-Republic", " El-Salvador", 
                     " Haiti", " Honduras", " Mexico", " Trinadad&Tobago")

South_America <- c(" Ecuador", " Peru", " Columbia")


Europe_West <- c(" England", " Germany", " Holand-Netherlands", " Ireland", 
                 " France", " Greece", " Italy", " Portugal", " Scotland")

Europe_East <- c(" Poland", " Yugoslavia", " Hungary")
```
Then we modify the data frame by adding an additional column named “native_region”. We do this with the help of the “mutate” function form the “plyr” package:
```
db.adult <- mutate(db.adult, 
       native_region = ifelse(native_country %in% Asia_East, " East-Asia",
                ifelse(native_country %in% Asia_Central, " Central-Asia",
                ifelse(native_country %in% Central_America, " Central-America",
                ifelse(native_country %in% South_America, " South-America",
                ifelse(native_country %in% Europe_West, " Europe-West",
                ifelse(native_country %in% Europe_East, " Europe-East",
                ifelse(native_country == " United-States", " United-States", 
                       " Outlying-US" ))))))))
```
Next we transform the new variable into a factor:
```
db.adult$native_region <- factor(db.adult$native_region, ordered = FALSE)
```

## Split datasets in training and test sets

- We have to split the data into two/ three subsets to avoid overfitting or underfitting to the training data. 

A seed is used to ensure that the sets do not change every time you run the code

```
set.seed(123)
```
Split the dataset in the ratio of 8:2 for training:testing. 
We can also use Validation set if needed.
```
split=sample.split(dataset$price,SplitRatio = 0.8)
```

```
tariningset=subset(dataset,split==TRUE)
nrow(tariningset)
```

## Benchmarking: 

- Using interquartile range (IQR) to calculate the benchmark
- You can choose any other method like mean median also

```
bhd<-528+1.5*IQR(dset$hd) #calculating benchmark of hd
```

- Replace the values greater than the benchmark with the benchmark
```
dset$hd[dset$hd>bhd]<-bhd #replacing outliers by benchmark
```


## Summarization

The column in a dataset can be summarized using the following command:

```
summary(dset$ram)
```

## Linear Regression
```
regressorfinal3=lm(formula = price ~ speed + hd + ram + screen + cd + premium + ads + trend + multi ,data=dset)
```
- Here price is the dependent variable (the one we want to predict)
- Others specified after tidle (~) are independent variables
- The formula given to the regressor is a of the following format (dependent variables ~ independent variables)

This will give the summary of the regressor and accuracy:

```
summary(regressorfinal3)
```

## Ordering the dataset

You can use the order() function directly without resorting to add-on tools 

```
R> dd[with(dd, order(-z, b)), ]
    b x y z
4 Low C 9 2
2 Med D 3 1
1  Hi A 8 1
3  Hi A 9 1
```
You can do this using column index. The answer is to simply pass the desired sorting column(s) to the order() function:

```
R> dd[order(-dd[,4], dd[,1]), ]
    b x y z
4 Low C 9 2
2 Med D 3 1
1  Hi A 8 1
3  Hi A 9 1
R> 
```
rather than using the name of the column (and with() for easier/more direct access).

## Setting new column names to the data frame

```
colnames(df) <- c('a', 'b', 'c')
```
- The the number of values in the c vector should be exactly equal to the number of columns in dataframe df.


## Create dataset from independent vectors

- This will create a new data frame with columns as Name, Sex and Age
```
Name <- c("John", "Tim", "Ami")
Sex <- c("men", "men", "women")
Age <- c(45, 53, 35)
dt <- data.frame(Name, Sex, Age)
dt
```
- Output

```
  Name   Sex Age
1 John   men  45
2  Tim   men  53
3  Ami women  35
```

## Subset datasets

- The subset function takes the dataframe to be processed, followed by condition on which the subset has to be done. 

```
dt2 <- subset(dt, Age>40&Sex==men)
dt2
```
- Here, we will get the new subset where Age is greater than 40 and sex is men

```
# Other example
(nrow(subset(db.adult, db.adult$capital_loss == 0))/nrow(db.adult))*100
```

Output - 

```
Name  Sex Age
John  men  45
Tim   men  53
```

## Reading files 

### Reading CSV

```
data = read.csv("/path/to/file", header= TRUE)
```

#### Header Attribute
- If the the file being read already contains a header, then specify the header as TRUE, so that it can automatically be read. header = TRUE by default. 
- If FALSE is specified, it names the colums as V1, V2 ... Vn. You can later give custom names for these columns


#### Specifing the NA strings:

- In many situations, the NA values are represented using some other character for ex: (?)
- We can replace these with NA while reading itself. 

```
db.adult <- read.table("adult.data",
                       sep = ",", 
                       header = FALSE, 
                       na.strings = " ?")
```
- Don't forget the space before the ? in the above code, it should match the dataset attribute exactly. "?" is not same as " ?".

### Export Dataset back to file:

```
write.csv(df, "file_name1.csv", row.names = FALSE)

write.csv(df, "file_name2.csv", row.names = FALSE)
```

### Getting the number of rows and columns

The following function will give the number of columns in the dataframe (df).
```
nrow(df)
```


## Merge datasets

TODO:

## Melting and casting data 

TODO:

### Looking at the top rows of the dataset

```
head(df, 10)
# where 10 is the number of rows to display
```
### Checking the datatype of variables

```
str(df, vec.len = 2, strict.width = "no", width = 30)
```
- Output
```
'data.frame':	32561 obs. of  15 variables:
 $ age           : int  39 50 38 53 28 ...
 $ workclass     : Factor w/ 9 levels " ?"," Federal-gov",..: 8 7 5 5 5 ...
 $ fnlwgt        : int  77516 83311 215646 234721 338409 ...
 $ education     : Factor w/ 16 levels " 10th"," 11th",..: 10 10 12 2 10 ...
 $ education-num : int  13 13 9 7 13 ...
 $ marital       : Factor w/ 7 levels " Divorced"," Married-AF-spouse",..: 5 3 1 3 3 ...
 $ occupation    : Factor w/ 15 levels " ?"," Adm-clerical",..: 2 5 7 7 11 ...
 $ relationship  : Factor w/ 6 levels " Husband"," Not-in-family",..: 2 1 2 1 6 ...
 $ race          : Factor w/ 5 levels " Amer-Indian-Eskimo",..: 5 5 5 3 3 ...
 $ sex           : Factor w/ 2 levels " Female"," Male": 2 2 2 2 1 ...
 $ capital-gain  : int  2174 0 0 0 0 ...
 $ capital-loss  : int  0 0 0 0 0 ...
 $ hours-per-week: int  40 13 40 40 40 ...
```

- As we can see from the output above, the variables ```“age”, “fnlwgt”, “education_num”, “capital_gain”, “capital_loss”``` and ``` “hours_per_week” ``` are of type integer, whereas all the other variables are factor variables with different number of levels. 

#### Other method

- You can check the data type of variable by using the ```is.data_type``` command: 

Example
- ``` is.factor ``` checks whether the variable is factor
- ``` is.int ``` checks whether the variable is int


### View Categorical variables

- In order to see what the levels of each factor variable are, we write the function “levels_factors()”, which takes as an argument a data frame, identifies the factor variables and prints the levels of each categorical variable:

```
levels_factors <- function(mydata) {
    col_names <- names(mydata)
    for (i in 1:length(col_names)) {
        if (is.factor(mydata[, col_names[i]])) {
            print(levels(mydata[, col_names[i]]))
        }
    }
}

levels_factors(df)
```


### Getting the Mean

```
age <- c(23, 16, NA)
mean(age)
## [1] NA
```

Removing na values and try again
```
mean(age, na.rm = TRUE)
## [1] 19.5
```


## KNN model
- Since the initial cluster assignments are random, let us set the seed to ensure reproducibility.
```
set.seed(20)
irisCluster <- kmeans(iris[, 3:4], 3, nstart = 20)
irisCluster
K-means clustering with 3 clusters of sizes 46, 54, 
```

## Naive Bayes
TODO


## Transposing dataframe

Use the t() function to transpose a matrix or a data frame. In the latter case, row names become variable (column) names.
```
    df > data[1:10, 1:10]
```
```
    t(df)
```
Source: https://www.r-statistics.com/tag/transpose/


#### Ensuring that numerical values are not converted to strings
```
    data<- as.data.frame(t(data))
```
This ensures that the numeric values are not converted to string values.




## Plotting graphs in R

- The plotting functionality is provided by the ggplot2 library in R.
- It can be installed and loaded using the following functions.
- You can directly use ggplot function which also return a object which can later be modified, or other functions like plot, boxplot etc.
- ```qplot``` is a shortcut if you're familier with ```plot()```. Ref: https://ggplot2.tidyverse.org/reference/qplot.html


### Quick plot
Source: R/quick-plot.r

qplot is a shortcut designed to be familiar if you're used to base plot(). It's a convenient wrapper for creating a number of different types of plots using a consistent calling scheme. It's great for allowing you to produce plots quickly, but I highly recommend learning ggplot() as it makes it easier to create complex graphics.
```
qplot(x, y, ..., data, facets = NULL, margins = FALSE, geom = "auto",
  xlim = c(NA, NA), ylim = c(NA, NA), log = "", main = NULL,
  xlab = NULL, ylab = NULL, asp = NA, stat = NULL,
  position = NULL)
```
```
quickplot(x, y, ..., data, facets = NULL, margins = FALSE,
  geom = "auto", xlim = c(NA, NA), ylim = c(NA, NA), log = "",
  main = NULL, xlab = NULL, ylab = NULL, asp = NA, stat = NULL,
  position = NULL)
```

### Plot Function

Generic X-Y Plotting
Description

Generic function for plotting of R objects. For more details about the graphical parameter arguments, see par.

For simple scatter plots, plot.default will be used. However, there are plot methods for many R objects, including functions, data.frames, density objects, etc. Use methods(plot) and the documentation for these.
Usage

	plot(x, y, ...)
```
Arguments
x 	 the x coordinates of points in the plot. Alternatively, a single plotting structure, function or any R object with a plot method can be provided.
y 	 the y coordinates of points in the plot, optional if x is an appropriate structure.
...  Arguments to be passed to methods, such as graphical parameters (see par). Many methods will accept the following arguments:



what type of plot should be drawn. Possible types are

"p" for points,

"l" for lines,

"b" for both,

"c" for the lines part alone of "b",

"o" for both ‘overplotted’,

"h" for ‘histogram’ like (or ‘high-density’) vertical lines,

"s" for stair steps,

"S" for other steps, see ‘Details’ below,

"n" for no plotting.

main: an overall title for the plot: see title.
sub: a sub title for the plot: see title.
xlab: a title for the x axis: see title.
ylab: a title for the y axis: see title.
asp: the y/x aspect ratio, see plot.window.
```

### Box Plots
```
ggplot(aes(x = factor(0), y = hours_per_week),
       data = db.adult) + 
  geom_boxplot() +
  stat_summary(fun.y = mean, 
               geom = 'point', 
               shape = 19,
               color = "red",
               cex = 2) +
  scale_x_discrete(breaks = NULL) +
  scale_y_continuous(breaks = seq(0, 100, 5)) + 
  xlab(label = "") +
  ylab(label = "Working hours per week") +
  ggtitle("Box Plot of Working Hours per Week") 
```

### Histograms
```
ggplot(data = df, 
       aes(x = df$capital_gain)) +
  geom_histogram(binwidth = 5000,
                 colour = "black",
                 fill = "lightblue",
                 alpha = 0.4) +
  scale_y_continuous(breaks = seq(0, 4000, 100)) +
  labs(x = "Capital gain", y = "Count") +
  ggtitle("Histogram of Nonzero Capital Gain") 
```
- Varying alpha is useful for large datasets
```
d <- ggplot(diamonds, aes(carat, price))
d + geom_point(alpha = 1/10)
```

- For shapes that have a border (like 21), you can colour the inside and outside separately. Use the stroke aesthetic to modify the width of the border
```
	ggplot(mtcars, aes(wt, mpg)) + geom_point(shape = 21, colour = "black",  fill = "white", size = 5, stroke = 5)
```

### Dot Plots

### Bar plots

### Line Charts


### Merging Plots


### String normalization

- String normalization techniques are aimed at transforming a variety of strings to a smaller set of string values which are more easily processed. By default, R comes with extensive string
manipulation functionality that is based on the two basic string operations: 
- finding a pattern in a string and replacing one pattern with another. We will deal with R 's generic functions below but start by pointing out some common string cleaning operations. 
- The stringr package 36 offers a number of functions that make some some string manipulation tasks a lot easier than they would be with R 's base functions. 
- For example, extra white spaces at the beginning or end of a string can be removed using str_trim.
	
	library(stringr)

	str_trim("  hello world ")
	## [1] "hello world"
	str_trim("  hello world ", side = "left")
	## [1] "hello world "
	str_trim("  hello world ", side = "right")
	## [1] "  hello world"

Conversely, strings can be padded with spaces or other characters with ```str_pad``` to a certain width. For example, numerical codes are often represented with prepending zeros. An introduction to data cleaning with R ``` str_pad(112, width = 6, side = "left", pad = 0) ```

	## [1] "000112"

Both str_trim and ```str_pad``` accept a side argument to indicate whether trimming or padding should occur at the beginning ( left ), end ( ) or both sides of the string. Converting strings to complete upper or lower case can be done with R's built-in ```toupper``` and ```tolower``` functions.

	toupper("Hello world")
	## [1] "HELLO WORLD"
	
	tolower("Hello World")
	## [1] "hello worl



Disclamer: The content provided here is not entirely mine and is gathered from various sources