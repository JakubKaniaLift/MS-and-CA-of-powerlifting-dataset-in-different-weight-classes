# Multidimensional scaling and correspondence analysis of powerlifting dataset in different weight classes

## Libraries

```r
library("MASS")
library("ca")
library("RColorBrewer")
library("tidyverse")
library("dplyr")
library("ggplot2")
```
## Data

Dataset have:

* 16 columns
* 41,152 rows

Columns are:
1. name <- name of participants
2. sex 
3. event <- B (bench), SB (squat, bench), SBD (squat, bench, deadlift)
4. equipment <- used equipment in event
5. age 
6. age_class
7. division
8. bodyweight_kg
9. weight_class_kg
10. best3squat_kg <- best squat out of 3
11. best3bench_kg <- best bench out of 3
12. best3deadlift_kg <- best DL out of 3
13. place
14. date
15. federation
16. meet_name <- in what event the data was taken

### Loading data
```r
lifts <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-10-08/ipf_lifts.csv")
lifts <- lifts[lifts$place != 'DQ',]
lifts
```
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/e48ec4a2-08ba-4649-afcb-df11f14e6612)

### Cleaning data

We are going to compare only full SBD lifts, that ended qualified and took place at "World Masters Powerlifting Championships".
```r
lifts <- lifts[lifts$event == 'SBD',]
lifts <- lifts[lifts$place != 'DQ',]
lifts <- lifts[lifts$meet_name == 'World Masters Powerlifting Championships',]
```
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/f235759e-5d86-41d2-88d4-04d6b093e121)

This way we end up with 5,591 rows.

#### Searching and cleaning NAs

```r
#Checking NAs
sapply(lifts, function(x) sum(is.na(x)))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/b14b6d4f-348b-4bd5-94b3-b4674273f511)

```r
#Removing NAs
lifts <- lifts %>% drop_na(best3squat_kg)
lifts <- lifts %>% drop_na(best3bench_kg)
lifts <- lifts %>% drop_na(best3deadlift_kg)
lifts <- lifts %>% drop_na(bodyweight_kg)

sapply(lifts, function(x) sum(is.na(x)))
```
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/1b10b4ee-cb68-49fe-bee3-f886cb0ac284)

We cleaned data from NAs that appeared in our lifts and we cleane Nas in bodyweight for the future.

### Exploring and changing data

#### Data sets

We divide our data set by 'sex' parameter, because our analysis will be done around lifts done by female and men.

```r
lifts_m <- lifts[lifts$sex=='M',]
lifts_f <- lifts[lifts$sex=='F',]
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/058dcaf8-959b-4d53-be2e-a1761f819030)

#### Counting the amount of competitors in every weight class

##### Men:

```r
sort(table(lifts_m$weight_class_kg))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/c98cd6a9-a985-4f4c-9ac5-203fc7f23cb7)

##### Female:

```r
sort(table(lifts_f$weight_class_kg))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/af8f61e3-4ea3-4649-8754-654e25f215d2)

#### Columns

We are picking only columns that will be used in our analysis:
* name
* bodyweight_kg
* weight_class_kg
* best3squat_kg <- best squat out of 3
* best3bench_kg <- best bench out of 3
* best3deadlift_kg <- best DL out of 3
* date

##### Men:
```r
lifts_m <- lifts_m[,c(1,8,9,10,11,12,14)]
colnames(lifts_m) <- c('Name','Bodyweight','Weight_class','Squat','Bench','DL','Date')
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/84c309ad-8e2f-4e7b-b36a-2613234d67be)

##### Female:

```r
lifts_f <- lifts_f[,c(1,8,9,10,11,12,14)]
colnames(lifts_f) <- c('Name','Bodyweight','Weight_class','Squat','Bench','DL','Date')
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/0f4ee78e-e390-40d8-ab7d-1f01a64700f4)

## Wilks coefficient

The Wilks coefficient, also known as the Wilks score or Wilks formula, is a calculation used in powerlifting to compare the strength levels of lifters across different weight classes.

$Coeff = \frac{500}{a + bx + cx^2 + dx^3 + ex^4 + fx^5}$

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/82d89d6b-fb50-4808-b47f-ab89cff1b3e5)

We will calculate Wilks coefficient for all participants.

### Men - Wilks

```r
#Vector of coefficients
a_men <- c(47.46178854,
           8.472061379,
           0.07369410346,
           -0.001395833811,
           7.07665973070743e-6,
           -1.20804336482315e-8)

#Function
wilks_men <- function(total, bodyweight) {
  coefficient <- a_men[1]
  for (i in 2:length(a_men)) {
    coefficient <- coefficient + (a_men[i] * (bodyweight ^ (i-1)))
  }
  600 / coefficient
}
```

#### LOOP

```r
for (i in 1:nrow(lifts_m)){
  lifts_m$Wilks[i] <- wilks_men(sum(lifts_m[i,c(4,5,6)]),as.numeric(lifts_m$Bodyweight)) * sum(lifts_m[i,c(4,5,6)])
}

head(lifts_m,10)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/6c0bde64-0cef-4027-b9a8-bcb57f6a5033)

### Female - Wilks

```r
a_female <- c(-125.4255398,
              13.71219419,
              -0.03307250631,
              -0.001050400051,
              9.38773881462799e-6,
              -2.3334613884954e-8)

#Function
wilks_female <- function(total, bodyweight) {
  coefficient <- a_female[1]
  for (i in 2:length(a_female)) {
    coefficient <- coefficient + (a_female[i] * (bodyweight ^ (i-1)))
  }
  600 / coefficient
}
```

#### LOOP

```r
for (i in 1:nrow(lifts_f)){
  lifts_f$Wilks[i] <- wilks_female(sum(lifts_f[i,c(4,5,6)]),as.numeric(lifts_f$Bodyweight)) * sum(lifts_f[i,c(4,5,6)])
}

head(lifts_f,10)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/b404e000-9054-4e39-9a52-147e80ac0c9b)

## Multidimensional scaling and correspondence analysis between best participants over years in each weight class

* Multidimensional Scaling - It is a statistical technique used to analyze and visualize the similarities or dissimilarities between a set of objects or cases. It aims to represent the relationships between the objects in a lower-dimensional space while preserving their original distances as much as possible.

* Correspondence Analysis - It is a data exploration technique primarily used for categorical or discrete variables. It reveals the associations and dependencies between two or more categorical variables in a data set.

### Picking the best participants over years in each weight class

#### Men

```r
men_best <- lifts_m %>% 
  group_by(Weight_class) %>%
  filter(Wilks == max(Wilks))

men_best <- men_best %>% arrange(Name,Date)
print(men_best,n = nrow(men_best))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/0755d618-15e8-480d-97a1-6f2d48843dbd)

##### Removing duplicates

```r
men_best <- men_best[c(-3,-10),]
men_best
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/9542c9c1-7dc4-4572-96a7-99f4b59d8b3c)


#### Female

```r
female_best <- lifts_f %>% 
  group_by(Weight_class) %>%
  filter(Wilks == max(Wilks))

female_best <- female_best %>% arrange(Name,Date)
print(female_best,n = nrow(female_best))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/66992d60-f7f6-425b-a3bb-a526650d5770)

### Multidimensional scaling

#### Men

##### Distance matrix

```r
d <- dist(men_best)
d
```
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/10c85a3d-f58a-4db2-be3a-7c9547d21ffe)

##### Plot

```r
#Plot
cmds <- cmdscale(d, k=2, add=T, eig =T, x.ret=T)
x <- cmds$points [ ,1]
y <- cmds$points [ ,2]
plot(x, y, type = 'n')
text(x, y, labels=rownames(men_best))

#Groups
g1 <- text(x[3], y[3], col = 'red', labels = 3)
g2 <- text(x[c(5,16,7)], y[c(5,16,7)], col = 'blue', labels = c(5,16,7))
g3 <- text(x[c(11,13,1,6,8)], y[c(11,13,1,6,8)], col = 'green', labels = c(11,13,1,6,8))
g4 <- text(x[c(2,10,14,4,15,12)], y[c(2,10,14,4,15,12)], col = 'orange', labels = c(2,10,14,4,15,12))
g5 <- text(x[18], y[18], col = 'purple', labels = 18)
g6 <- text(x[17], y[17], col = 'black', labels = 17)
g7 <- text(x[9], y[9], col = 'brown', labels = 9)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/39c37e60-a314-43e0-8c16-4fffa2d9c02f)

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/f9274f3f-29d0-4117-ac8b-933f9031ac66)

#### Female

##### Distance matrix

```r
d <- dist(female_best)
d
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/8f719cd8-f7bf-45e0-b2fb-11d5d8979d00)

##### Plot

```r
cmds <- cmdscale(d, k=2, add=T, eig =T, x.ret=T)
x <- cmds$points [ ,1]
y <- cmds$points [ ,2]
plot(x, y, type = 'n')
text(x, y, labels=rownames(female_best))

#Groups
g1 <- text(x[c(9,13)], y[c(9,13)], col = 'red', labels = c(9,13))
g2 <- text(x[8], y[8], col = 'blue', labels = 8)
g3 <- text(x[c(1,6)], y[c(1,6)], col = 'green', labels = c(1,6))
g4 <- text(x[c(2,11)], y[c(2,11)], col = 'orange', labels = c(2,11))
g5 <- text(x[c(5,4,14,12,15,16,7)], y[c(5,4,14,12,15,16,7)], col = 'purple', labels = c(5,4,14,12,15,16,7))
g6 <- text(x[c(10,17)], y[c(10,17)], col = 'black', labels = c(10,17))
g7 <- text(x[3], y[3], col = 'brown', labels = 3)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/1ea0dfd2-9bb3-4f4d-b0b3-a0a94867a284)

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/598ca12e-a5a5-42c7-957c-ed0e73f5bf47)

### Correspondence analysis

#### Heatmaps

Heatmaps are a method of representing data graphically where values are depicted by color, making it easy to visualize complex data and understand it at a glance.

To make heatmaps we need our data to be a matrix.

```r
men_best_m <- as.matrix(men_best[1:nrow(men_best),as.numeric(c(4,5,6))])
female_best_m <- as.matrix(female_best[1:nrow(female_best),as.numeric(c(4,5,6))])
```

To make sure our values are independent, we are going to do $X^2$ test.

```r
chisq.test(men_best_m)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/c32e72ea-0d6f-467f-8788-84e0a1d5f018)

```r
chisq.test(female_best_m)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/8daa98d9-6d71-43b3-9afe-fcf44a0585a7)

Now we will count Pearson's residuals and make heatmaps from them.

```r
P = men_best_m/sum(men_best_m)
PP = outer(rowSums(P),colSums(P))
E = (P-PP)/sqrt(PP)
head(E)
```

```r
P = female_best_m/sum(female_best_m)
PP = outer(rowSums(P),colSums(P))
E = (P-PP)/sqrt(PP)
head(E)
```

Man             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/072dbade-6c8a-494f-bca0-22006df4d404) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/d541690f-54cd-4b41-9da3-19cf6d176804)

Finally we will make our heatmaps

```r
heatmap(E,scale="none",Colv=NA,col= brewer.pal(8,"Blues"))

```

Men             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/e16e97bb-cc50-41a3-ace9-d6acee227d0d) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/06854b5b-007b-457a-b3ee-0b09d24aae43)

Received heatmaps presents performance across different weight classes and lifts by using color intensity to indicate the relative strength levels achieved by lifters in each specific combination.

We can see that man have close Squats scores, while Bench and Deadlift are more scattered, same thing happens in Female category.

#### Plots

```r
par(bty = 'n',yaxt="n",xaxt="n")

##Men
plot(ca::ca(men_best_m),
     col = men_best$Weight_class,
     xlab = "",
     ylab = "")

##Female
plot(ca::ca(female_best_m),
     col = female_best$Weight_class,
     xlab = "",
     ylab = "")
```

Men             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/25dd6d65-87a5-42e7-8bb5-b6e4cdc8bfd7) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/74f75d4e-b7c5-4dce-8a72-82e4f59b1952)


We can see that while men appeared relatively equal in performance across all three lifts, women exhibited a distinct pattern. The plot demonstrated that women were positioned much closer to deadlift and squat in terms of their performance, while being comparatively further away from the bench press. This suggests a notable discrepancy in strength distribution among the lifts between the genders. Additional we can see that one participant in female class is very far from all three factors.

## Multidimensional scaling and correspondence analysis between all participants at 2007-06-10 powerlifting event, for men 100kg class and female 67.5kg class

### Preparing data

```r
sort(table(lifts$date)) #Picking most attendent date

nrow(lifts_m[lifts_m$Date == '2007-06-10',]) #Checking amount off male participants
nrow(lifts_f[lifts_f$Date == '2007-06-10',]) #Checking amount off female participants

lifts_m <- lifts_m[lifts_m$Date == '2007-06-10',]
lifts_f <- lifts_f[lifts_f$Date == '2007-06-10',]

#Finding most attendant weight class
sort(table(lifts_m$Weight_class))
sort(table(lifts_f$Weight_class))
```

#### Men

```r
men_100 <- lifts_m %>% 
  filter(Weight_class == '100') %>%
  arrange(desc(Wilks))

men_100
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/8d6a4d1d-ba80-4ab3-afbf-6f9d8f594eba)

#### Female

```r
female_100 <- lifts_f %>%
  filter(Weight_class == "67.5") %>%
  arrange(desc(Wilks))

female_100
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/4b87260f-b89a-4dd1-a226-bdc3569133df)

### Multidimensional scaling

#### Men

##### Distance matrix

```r
d <- dist(men_100)
d
```

##### Plot

```r
cmds <- cmdscale(d, k=2, add=T, eig =T, x.ret=T)
x <- cmds$points [ ,1]
y <- cmds$points [ ,2]
plot(x, y, type = 'n')
text(x, y, labels=rownames(men_100))

#Groups
g1 <- text(x[1], y[1], col = 'red', labels = 1)
g2 <- text(x[2], y[2], col = 'blue', labels = 2)
g3 <- text(x[19], y[19], col = 'green', labels = 19)
g4 <- text(x[27], y[27], col = 'orange', labels = 27)
g5 <- text(x[31], y[31], col = 'purple', labels = 31)
g6 <- text(x[34], y[34], col = 'black', labels = 34)
g7 <- text(x[c(3,7,10,9,8)], y[c(3,7,10,9,8)], col = 'brown', labels = c(3,7,10,9,8))
g8 <- text(x[c(26,30)], y[c(26,30)], col = 'pink', labels = c(26,30))
g9 <- text(x[c(14,15)], y[c(14,15)], col = 'grey', labels = c(14,15))
g10 <- text(x[c(29,32,33)], y[c(29,32,33)], col = 'lightblue', labels = c(29,32,33))
g11 <- text(x[c(4,5,6)], y[c(4,5,6)], col = 'lightgreen', labels = c(4,5,6))
g12 <- text(x[c(12,11,13)], y[c(12,11,13)], col = 'yellow', labels = c(12,11,13))
g13 <- text(x[c(23,22,25,28,21,20,18,16,17,24)], y[c(23,22,25,28,21,20,18,16,17,24)], col = 'darkred', labels = c(23,22,25,28,21,20,18,16,17,24))
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/20888258-898d-43c7-bfbb-777cecbf79eb)

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/08c3e6a7-840b-41d9-904d-7957ebb46780)

#### Female

##### Distance matrix

```r
d <- dist(female_100)
d
```

##### Plot

```r
cmds <- cmdscale(d, k=2, add=T, eig =T, x.ret=T)
x <- cmds$points [ ,1]
y <- cmds$points [ ,2]
plot(x, y, type = 'n')
text(x, y, labels=rownames(female_100))

#Groups
g1 <- text(x[1], y[1], col = 'red', labels = 1)
g2 <- text(x[c(2,3)], y[c(2,3)], col = 'blue', labels = c(2,3))
g3 <- text(x[6], y[6], col = 'green', labels = 6)
g4 <- text(x[11], y[11], col = 'orange', labels = 11)
g5 <- text(x[c(7,9,10)], y[c(7,9,10)], col = 'purple', labels = c(7,9,10))
g6 <- text(x[c(4,5)], y[c(4,5)], col = 'black', labels = c(4,5))
g7 <- text(x[8], y[8], col = 'brown', labels = 8)
```

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/8e0857ed-ad24-43ed-ac73-777e70ee63e3)

![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/9b1ac0bc-1aaa-4034-90e2-6cdee3c19d75)

### Correspondence analysis

#### Heatmaps

Matrix

```r
men_100_m <- as.matrix(men_100[1:nrow(men_100),as.numeric(c(4,5,6))])
female_100_m <- as.matrix(female_100[1:nrow(female_100),as.numeric(c(4,5,6))])

rownames(men_100_m) <- men_100$Name
rownames(female_100_m) <- female_100$Name
```

```r
chisq.test(men_100_m)
chisq.test(female_100_m)
```

Men             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/76b656f5-887a-4d36-b6dd-c51906ab846b) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/e65064b6-86c8-4c80-9c7c-f25f315fc840)

Pearson's residuals only for men matrix

```r
P = men_100_m/sum(men_100_m)
PP = outer(rowSums(P),colSums(P))
E = (P-PP)/sqrt(PP)
head(E)
```

Heatmaps

```r
heatmap(E,scale="none",Colv=NA,col= colors) #Men heatmap
heatmap(female_100_m,scale="none",Colv=NA,col= colors) #Female heatmap
```

Men             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/f6e7f6ad-35a2-4f0d-8564-78c21c582fc9) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/e4ec4591-f8bc-49fe-87f0-31bcd93afe21)

Female heatmap shows us what $X^2$ test did. There is no significant differences between lifts that all examined female performed.

### Plots

```r
par(bty = 'n',yaxt="n",xaxt="n")

##Men
plot(ca::ca(men_100_m),
     xlab = "",
     ylab = "")

##Female
plot(ca::ca(female_100_m),
     xlab = "",
     ylab = "")
```

Men             |  Female
:-------------------------:|:-------------------------:
![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/fadcaa54-468a-4b01-b33b-c98fd521c4c1) | ![image](https://github.com/JakubKaniaLift/Correspondence-analysis-of-powerlifting-dataset-in-different-weight-classes/assets/138041287/7d0f9a25-2da0-4ad1-b272-a04531f27c2b)












