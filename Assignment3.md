# Assignment 3A - part 1
## Questions from the notebook
1.  **Q**: Explain why there are multiple result files.  
    **A**: Because there are multiple partitions.

2.  **Q**: why are the counts different?  
    A: Because of the mapping: every letter is converted to lowercase and the removal of non-letters. Before, it was case sensitive. And I think that when a dot, comma, or other punctuation marks prevented the word "Macbeth" from being recognized (it should be precisely Macbeth, and the split is on spaces, so a comma or dot will be part of the word), wheras in this new mapping, these characters are removed.

# Assignment 3A - part 2
## Questions from the notebook

1.  **Q**: Spot the differences between the results, and try to map what you see on Chapter 2 that we read for the course.  
    **A**: The `rddPartsGroupPart2` had a `Hashpartitioner(2)` assigned to it, where the 2 indicates the number of partitions that we will end up with, while the `rddPaursGroup` had no partitioner. The default number of partition is the amount of CPU cores that we run on, so that is why there are 8 partitions created in the first one.  
      
2.  **Q**: do you understand why the partitioner is none? (No worries if not, you will find a clue below.)  
    **A**: Because we perform a `map()` on `rddPairsGroupPart2`, which means the 'new' value does not have a partitioner anymore, which is why it reports `none`. Basically, map performs on the keys AND the values of the key,value pairs (and thus forgets our partitioning).
      
3.  **Q**: Why are the results different for `rddA` and `rddB`? How is query processing affected by the partitioners?  
    **A**: Because on `rddA` we call a `map()` operation, as we said above, this will omit the partitioner that was on there. However, `rddB` calls `mapValues()`, which only operates on the value of the key,value pair. Therefore, partioning is still intact.  
      
4.  **Q**: Compare the two query plans for `rddC` and `rddD`. Can you explain why the second query plan has on less shuffle phase?  
    **A**: The `coalesce()` function, performed on `rddB`, which became `rddD`, avoids a full shuffle. It uses the existing partitions of the dataframe, whereas `repartition()`, performed on `rddA`, which became `rddC`, does a full shuffle, and creates partitions based in the user input (in our case, that is 2). `coalesce()` moves data from partition to other existing partitions. The downside with coalesce is that the output data is not equaly partitioned, which it will be when using the `repartition()` function.
      
# Assignment 3B
## Questions from the notebook
1.  **Q**: Can you write the small numbers of lines of code to inspect these records?  
    **A**: Well, it was mostly already given away. It is one line of code: `bagdata.filter( $"X_COORD".isNull || $"Y_COORD".isNull).show()`   
    It does not look nice (at least not on my laptop), since there are a lot of columns. Like what has been done before in the notebook, we could select the columns we're interested in by using `.select()`  
    For example, I chose the do the following: `val DF2 = bagdata.select('STRAAT, 'HUISNUMMER, 'HUISLETTER, 'HUISNUMMERTOEVOEGING, 'OPENBARERUIMTE_ID, 'STATUS, 'WIJK_OMS)`, which gave me most of the useful data. I would not need the coordinates for example, since we know that they are `null` anyway.

2.  **Q**: Why would you apply the transformation on the artworks dataset to join the result against the addresses, instead of vice versa?  
    **A**: Because this dataset is much smaller, and you will only transform the entries needed. Doing it viceversa results in a lot of extra work, on data that you were not going to use in the first place.
    
3.  **Q**: Can you produce the list of quarters that is missing because no artwork has been situated in the quarter?  
    **A**: Yes, we can use SQL for this:  

```scala  
val spatjoin = spark.sql("select distinct quarter, min(jaar) as jaar from kosquarter group by quarter order by jaar")
spatjoin.createOrReplaceTempView("spatjoin")

val filteredTable = spark.sql("from kosquarter select distinct quarter as quarter")
filteredTable.createOrReplaceTempView("filteredTable")

val fullTable = spark.sql("from addrdataframe select distinct quarter as quarter")
fullTable.createOrReplaceTempView("fullTable")

val resultTable = spark.sql("from fullTable select quarter where quarter not in (from filteredTable select quarter)")
resultTable.show(100,false)
```

The resulting list is:  
```scala
+--------------------------+
|quarter                   |
+--------------------------+
|Malvert                   |
|Kwakkenberg               |
|Aldenhof                  |
|Bijsterhuizen             |
|Oosterhout                |
|Grootstal                 |
|Ressen                    |
|Neerbosch-West            |
|Tolhuis                   |
|Zwanenveld                |
|Haven- en industrieterrein|
|Vogelzang                 |
|Kerkenbos                 |
|Brakkenstein              |
|'t Broek                  |
|St. Anna                  |
|Westkanaaldijk            |
|Staddijk                  |
|Hatertse Hei              |
|Ooyse Schependom          |
|Groenewoud                |
|Heseveld                  |
|'t Acker                  |
|Lankforst                 |
|Weezenhof                 |
|De Kamp                   |
+--------------------------+
```

    
4.  **Q**: Can you produce a longer list of quarters and their oldest artworks?  
    **A**: I don't see a way to make a more complete list. We already used the coordinates to match the artworks to the quarter they are in. The quarters that are listed above were left untouched by this join. You could manually do it, but that will be a lot of work. 

5.  **Q**: What are the years associated to artworks not yet matched up with the addresses database? What does this mean for our initial research question?
    **A**: I first tried filtering on null values in kosquarter, but apparently those were not in the data anymore. Therefore, I decided to compare the list with kos:
    
```scala
kos.createOrReplaceTempView("kos")
val emptyQuarters = spark.sql("from kos select naam, bouwjaar where naam not in (from kosquarter select naam) order by bouwjaar")
emptyQuarters.groupBy("bouwjaar").count.show(100, false)
```

Resulting in:
```scala
+--------+-----+
|bouwjaar|count|
+--------+-----+
|1645    |1    |
|1655    |1    |
|1860    |1    |
|1880    |1    |
|1881    |1    |
|1894    |1    |
|1912    |1    |
|1922    |1    |
|1925    |2    |
|1936    |1    |
|1945    |1    |
|1948    |1    |
|1949    |1    |
|1950    |1    |
|1951    |2    |
|1953    |5    |
|1954    |3    |
|1955    |1    |
|1956    |4    |
|1957    |4    |
|1958    |3    |
|1959    |3    |
|1960    |4    |
|1961    |4    |
|1962    |5    |
|1964    |5    |
|1965    |2    |
|1966    |8    |
|1967    |2    |
|1969    |4    |
|1970    |3    |
|1972    |1    |
|1974    |1    |
|1975    |2    |
|1976    |3    |
|1977    |1    |
|1978    |3    |
|1979    |2    |
|1980    |7    |
|1981    |3    |
|1982    |11   |
|1983    |6    |
|1984    |2    |
|1985    |2    |
|1986    |2    |
|1987    |5    |
|1988    |4    |
|1989    |5    |
|1990    |1    |
|1991    |2    |
|1992    |6    |
|1994    |2    |
|1996    |1    |
|1998    |1    |
|1999    |5    |
|2000    |5    |
|2003    |2    |
|2005    |1    |
|2006    |8    |
|2008    |1    |
|2010    |3    |
|2011    |4    |
|2012    |3    |
|2013    |1    |
|2014    |5    |
|2017    |1    |
+--------+-----+
```
Our initial question for this part was: 
"Do they [the quarters without artworks] really have no artworks, or did we not manage to find their corresponding quarter using the spatial join on coordinates?"

The answer is that there probably are artworks, but we indeed didn't find their corresponding quarter. I have no idea on how cities work, but I can imaging that artworks are mostly placed in newer areas, since they are not saturated with artworks already.
Creating a list like the one above, including the name of the artwork, might lead to clues on what quarter they should be in.