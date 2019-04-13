# SL-6 SPPU Cheatsheet


## Removing Values:


### Removing negative values:

```
dataset$X=ifelse(is.na(dataset$X),ave(dataset$X,FUN=function(x) mean(x,na.rm=TRUE)),dataset$X)
```

OR

Instead of eliminating null values, you can ignore it while calculating menas for each row:

```
rowMeans(dframe, na.rm=TRUE) 
```

OR

```
dframe[dframe < 0] <- NA
```


To consider only non-NaN values, you can subset for not NaN using !is.nan, see this example:

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


## Handle categorical data:

- Categorical data is the data with multiple category variables like (Male, Female), (Yes, No).
- This data cannot be processed by machine learning models, so we need to convert this to it's numerical equivalent. 

Example:
Here we are replacing yes and no labels in dataset for the cd column with 0 or 1

```
# handling categorical data ...'no' is 0 and 'yes' is 1
dataset$cd=factor(dataset$cd,levels = c('no','yes'),labels = c(0,1))
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



Disclamer: The content provided here is not entirely mine and is gathered from various sources