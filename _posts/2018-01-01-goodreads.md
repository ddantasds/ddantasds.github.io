---
title: "Good Reads: Recommender System"
date: 2018-01-01
tags: [.]
header:
  image: "/images/IMG_2046_1.JPG"
excerpt: "EDA, Recommender System, Goodreads, Data Science"
mathjax: "true"
---

![title](https://blog.tellwell.ca/wp-content/uploads/2016/12/goodreads.jpg)

Overview
--------

We cannot escape! If you use online services or buy anything from a e-commerce company, recommendation is part of your online routine. Online services are suggesting you products and services that you might like. It is everywhere. Netflix recommends shows/movies based on what you watched and recent demand. Amazon displays on your website products that you might be interested based on previous purchases, clicks, and user behavior. Youtube recommends videos and channels based on what you and others have watched. We could keep going and the idea would be the same: suggest something that you probably will enjoy.

Books are not different!! The website [goodreads.com](http://www.goodreads.com) could be defined as a social network dedicated for people interested on books. The idea is that you can interact with other readers, authors, and of course books. Between many features, the readers social network provides a members review database to give you more information for the books you are looking for. It works excatly as any other review system. You rate the book you have read (between 1 and 5 stars) and write down your opinion about it, explaining what you like or don't. I know, nothing is new here. The [goodreads.com](http://www.goodreads.com) also have its recommender system for books. It will look for the books you read and the rates you gave to suggest you new books.

In this notebook, I apply a very well known called **Item-Based Collaborative Filtering** (IBCF) technique to estimate the rate of books based on other similar books. The idea is simple, if I like a book (i.e., I rated it 5 stars) it is likely that I will also enjoy a very similar book to that one.

This technique is not new and also there are other methodologies to estimate the rates. Here we are going to focus on the IBCF and see how it works when predicting books rating!!!

Loading and Cleaning data
-------------------------

The data is available at [Kaggle](https://www.kaggle.com/zygmunt/goodbooks-10k/data). Here we find a dataset with millions of ratings for 10k books. In this page we can also find a very good kernel about **User-Based Collaborative Filtering** (UBCF). The objective is the same, to improve the recommendations system but **UBCF** approach looks for similar users instead of books.

``` r
ratings<-read.csv("~/project/Data-Science-Projects/goodreads/data/ratings.csv")
books<-read.csv("~/project/Data-Science-Projects/goodreads/data/books.csv")
```

For this application we are going to use 2 datasets:

-   **ratings**: Contains information about ratings. It is a simple dataset with only 3 variables: **book\_id**, **user\_id**, and **rating**. It is pretty straightforward. It represents the rate given from a user for a book.
-   **books**: This is dataset has more information. It contains different features for books such as **title**, **author name**, **year of publication**, **number of reviews**, and etc.

``` r
knitr::kable(
  head(books)
)
```

|   id|  book\_id|  best\_book\_id|  work\_id|  books\_count| isbn      |        isbn13| authors                     |  original\_publication\_year| original\_title                          | title                                                     | language\_code |  average\_rating|  ratings\_count|  work\_ratings\_count|  work\_text\_reviews\_count|  ratings\_1|  ratings\_2|  ratings\_3|  ratings\_4|  ratings\_5| image\_url                                                    | small\_image\_url                                             |
|----:|---------:|---------------:|---------:|-------------:|:----------|-------------:|:----------------------------|----------------------------:|:-----------------------------------------|:----------------------------------------------------------|:---------------|----------------:|---------------:|---------------------:|---------------------------:|-----------:|-----------:|-----------:|-----------:|-----------:|:--------------------------------------------------------------|:--------------------------------------------------------------|
|    1|   2767052|         2767052|   2792775|           272| 439023483 |  9.780439e+12| Suzanne Collins             |                         2008| The Hunger Games                         | The Hunger Games (The Hunger Games, \#1)                  | eng            |             4.34|         4780653|               4942365|                      155254|       66715|      127936|      560092|     1481305|     2706317| <https://images.gr-assets.com/books/1447303603m/2767052.jpg>  | <https://images.gr-assets.com/books/1447303603s/2767052.jpg>  |
|    2|         3|               3|   4640799|           491| 439554934 |  9.780440e+12| J.K. Rowling, Mary GrandPré |                         1997| Harry Potter and the Philosopher's Stone | Harry Potter and the Sorcerer's Stone (Harry Potter, \#1) | eng            |             4.44|         4602479|               4800065|                       75867|       75504|      101676|      455024|     1156318|     3011543| <https://images.gr-assets.com/books/1474154022m/3.jpg>        | <https://images.gr-assets.com/books/1474154022s/3.jpg>        |
|    3|     41865|           41865|   3212258|           226| 316015849 |  9.780316e+12| Stephenie Meyer             |                         2005| Twilight                                 | Twilight (Twilight, \#1)                                  | en-US          |             3.57|         3866839|               3916824|                       95009|      456191|      436802|      793319|      875073|     1355439| <https://images.gr-assets.com/books/1361039443m/41865.jpg>    | <https://images.gr-assets.com/books/1361039443s/41865.jpg>    |
|    4|      2657|            2657|   3275794|           487| 61120081  |  9.780061e+12| Harper Lee                  |                         1960| To Kill a Mockingbird                    | To Kill a Mockingbird                                     | eng            |             4.25|         3198671|               3340896|                       72586|       60427|      117415|      446835|     1001952|     1714267| <https://images.gr-assets.com/books/1361975680m/2657.jpg>     | <https://images.gr-assets.com/books/1361975680s/2657.jpg>     |
|    5|      4671|            4671|    245494|          1356| 743273567 |  9.780743e+12| F. Scott Fitzgerald         |                         1925| The Great Gatsby                         | The Great Gatsby                                          | eng            |             3.89|         2683664|               2773745|                       51992|       86236|      197621|      606158|      936012|      947718| <https://images.gr-assets.com/books/1490528560m/4671.jpg>     | <https://images.gr-assets.com/books/1490528560s/4671.jpg>     |
|    6|  11870085|        11870085|  16827462|           226| 525478817 |  9.780525e+12| John Green                  |                         2012| The Fault in Our Stars                   | The Fault in Our Stars                                    | eng            |             4.26|         2346404|               2478609|                      140739|       47994|       92723|      327550|      698471|     1311871| <https://images.gr-assets.com/books/1360206420m/11870085.jpg> | <https://images.gr-assets.com/books/1360206420s/11870085.jpg> |

``` r
knitr::kable(
  head(ratings)
)
```

|  book\_id|  user\_id|  rating|
|---------:|---------:|-------:|
|         1|       314|       5|
|         1|       439|       3|
|         1|       588|       5|
|         1|      1169|       4|
|         1|      1185|       4|
|         1|      2077|       4|

Let's start to clean our **ratings** dataset. It is possible for a user have more than one rating for the same book. We could assume that the user change its mind and reevalute the book rating. It definitely can happen, but for simplicity we remove these cases and assume that every combination of book and user has only one rating. In this case, we have to eliminate the cases where users gave more than on rating for the same book.

``` r
ratings<-ratings %>% group_by(user_id, book_id) %>% mutate(total=n())
cat('Number of duplicate ratings: ', format(nrow(ratings[ratings$total > 1,]),big.mark=",",scientific=FALSE))
```

    ## Number of duplicate ratings:  4,487

Example of duplicated rating: As you can see below, user **3204** rated book **8946** five times.

``` r
duplicated_ratings<-ratings%>%
    group_by(book_id,user_id)%>%
    summarise(total=n())%>%
    filter(total>1)%>%
    arrange(desc(total))

head(duplicated_ratings)
```

    ## # A tibble: 6 x 3
    ## # Groups:   book_id [5]
    ##   book_id user_id total
    ##     <int>   <int> <int>
    ## 1    8946    3204     5
    ## 2    2515    4359     4
    ## 3    3996   38259     4
    ## 4    6472     691     4
    ## 5    7420   34548     4
    ## 6    8946      42     4

The duplicated cases were removed.

``` r
ratings <- ratings[ratings$total == 1,]
cat(' Number of ratings: ',format(nrow(ratings),big.mark=",",scientific=FALSE),'\n',
    'Number of Books: ',format(length(unique(ratings$book_id)),big.mark=",",scientific=FALSE),'\n',
    'Number of Users: ',format(length(unique(ratings$user_id)),big.mark=",",scientific=FALSE))
```

    ##  Number of ratings:  977,269
    ##  Number of Books:  10,000
    ##  Number of Users:  53,380

For this problem, I decided to work with books that have atleast 100 reviews, and user who gave more than 20 reviews. Doing that I am significantly reducing reducing the number books and users. It helped me to reduce process time consumption.

``` r
ratings<-ratings%>%group_by(user_id)%>%mutate(total=n())%>%filter(total>20)
ratings<-ratings%>%group_by(book_id)%>%mutate(total=n())%>%filter(total==100)
cat(' Number of ratings: ',format(nrow(ratings),big.mark=",",scientific=FALSE),'\n',
    'Number of Books: ',format(length(unique(ratings$book_id)),big.mark=",",scientific=FALSE),'\n',
    'Number of Users: ',format(length(unique(ratings$user_id)),big.mark=",",scientific=FALSE))
```

    ##  Number of ratings:  145,600
    ##  Number of Books:  1,456
    ##  Number of Users:  6,735

Now, we have 9,806,160 combinatations of (**book**, **user**), and only 145,600 ratings. It means that less than 2% of these possible combination have been rated.

EDA: Understanding more about our data
--------------------------------------

### What's the most reviewed book?
