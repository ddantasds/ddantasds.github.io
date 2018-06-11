---
permalink: /fipe/
header:
  image: "/images/IMG_2046_1.JPG"
---

EDA: Brazilian Car Prices
================

### Read Data

You can include R code in the document as follows:

``` r
fipe <- fread("/home/ddantas/project/pre_git/FIPE/data/veiculos_preco_junho_de_2018.csv")
```

    ##    brand        vehicle year_model     fuel price_reference        price
    ## 1: Acura Legend 3.2/3.5       1998 Gasoline       June 2018 R$ 27.942,00
    ## 2: Acura Legend 3.2/3.5       1997 Gasoline       June 2018 R$ 23.392,00
    ## 3: Acura Legend 3.2/3.5       1996 Gasoline       June 2018 R$ 22.682,00
    ## 4: Acura Legend 3.2/3.5       1995 Gasoline       June 2018 R$ 20.648,00
    ## 5: Acura Legend 3.2/3.5       1994 Gasoline       June 2018 R$ 18.343,00
    ## 6: Acura Legend 3.2/3.5       1993 Gasoline       June 2018 R$ 16.563,00

I believe the data is pretty simple and intuitive. Two important points here are: (1) prices are in real (Brazilian currency); (2) price\_reference is the moment the price was \*\*calculated, in this case it was June 2018.

This information is monthly released by FIPE, a Brazilian instute.

### Vehicles by brand

You can also embed plots, for example:

``` r
options(repr.plot.width=10, repr.plot.height=4)
fipe%>%
group_by(brand)%>%
summarise(count=n())%>%
mutate(flag=ifelse(count>2000,1,0))%>%
ggplot(aes(x=reorder(brand,-count),y=count))+
geom_col(aes(fill=as.factor(flag)))+xlab("")+ylab("")+guides(fill=FALSE)+
theme_minimal()+theme(axis.text.x = element_text(angle=70,vjust = 0.7))
```

![](/_pages/fipe_files/figure-markdown_github/pressure-1.png)

**VW**, **GM - Chevrolet**, **Fiat** and **Ford** are the top 4 brands of with more than 2,000 vehicles each.

This number takes into account every year model available.

### Only Brand New Cars

``` r
fipe%>%
group_by(brand)%>%
filter(year_model=="32000")%>%
summarise(count=n())%>%
mutate(flag=ifelse(count>2000,1,0))%>%
ggplot(aes(x=reorder(brand,-count),y=count))+
geom_col(aes(fill=as.factor(flag)))+xlab("")+ylab("")+guides(fill=FALSE)+
theme_minimal()+theme(axis.text.x = element_text(angle=70,vjust = 0.7))
```

![](/figure-markdown_github/unnamed-chunk-3-1.png)
