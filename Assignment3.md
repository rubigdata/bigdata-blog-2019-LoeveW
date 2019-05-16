# Assignment 3A - part 1
## Questions from the notebook
1.  Q: Explain why there are multiple result files.
    A: Because there are multiple partitions.

2.  Q: why are the counts different?
    A: Because of the mapping: every letter is converted to lowercase and the removal of non-letters. Before, it was case sensitive. And I think that when a dot, comma, or other punctuation marks prevented the word "Macbeth" from being recognized (it should be precisely Macbeth, and the split is on spaces, so a comma or dot will be part of the word), wheras in this new mapping, these characters are removed.

# Assignment 3A - part 2

1.  Q: Spot the differences between the results, and try to map what you see on Chapter 2 that we read for the course.
    A: The `rddPartsGroupPart2` had a `Hashpartitioner(2)` assigned to it, where the 2 indicates the number of partitions that we will end up with, while the `rddPaursGroup` had no partitioner. The default number of partition is the amount of CPU cores that we run on, so that is why there are 8 partitions created in the first one.
    
2.  Q: do you understand why the partitioner is none? (No worries if not, you will find a clue below.)
    A: Because we perform a `map()` on `rddPairsGroupPart2`, which means the 'new' value does not have a partitioner anymore, which is why it reports `none`. Basically, map performs on the keys AND the values of the key,value pairs (and thus forgets our partitioning).
    
3.  Q: Why are the results different for `rddA` and `rddB`? How is query processing affected by the partitioners?
    A: Because on `rddA` we call a `map()` operation, as we said above, this will omit the partitioner that was on there. However, `rddB` calls `mapValues()`, which only operates on the value of the key,value pair. Therefore, partioning is still intact.
    
4.  Q: Compare the two query plans for `rddC` and `rddD`. Can you explain why the second query plan has on less shuffle phase?
    A: The `coalesce()` function, performed on `rddB`, which became `rddD`, avoids a full shuffle. It uses the existing partitions of the dataframe, whereas `repartition()`, performed on `rddA`, which became `rddC`, does a full shuffle, and creates partitions based in the user input (in our case, that is 2). `coalesce()` moves data from partition to other existing partitions. The downside with coalesce is that the output data is not equaly partitioned, which it will be when using the `repartition()` function.