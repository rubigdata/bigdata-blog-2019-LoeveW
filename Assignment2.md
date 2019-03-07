# Setup: Hadoop Distributed File System

I have finished the assignment on the Hadoop Distributed File System (HDFS).
The HDFS was run in a docker container. We used hadoop version 2.7.3.
Of course, it was only pseudo-distributed, as we ran in inside of one docker container, instead of over multiple clusters.

Before we start working on our hdfs, we need to format it. Then, we can start it using `sbin/start-dfs.sh`. After that, we can make directories.
To run filesystem commands, we use `dfs`, so creating a new directory in this filesystem, we should enter `bin/hdfs dfs -mkdir /some-directory`. We make a user directory, with a root folder in it.

We want to run map-reduce jobs, but for that we need data. We can copy the data from our docker container to the file system using `bin/hdfs dfs -put etc/hadoop input`, to start the secondary namenodes, followed by `bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'`
This shows every match of the regular expression `'dfs[a-z.]+'` we gave it. The output is written to the specified output directory `output`. You can use the `bin/hdfs dfs -get output output` command to get your output. 
As specified in the `core-site.xml` file, we have the node running on our localhost, on port 9000. We can view its contents via `bin/hdfs dfs -ls hdfs://localhost:9000/user/root/input`

HDFS is now set up.

# WordCount
We're running WordCount.java 100.txt, which we have set as the input.
For this, we have made a new WordCount.java file with nano, by copying the contents from the Hadoop documentation, and compiled it as wc.jar.
We can now use this file, to run WordCount on some input, writing the result to some output: ```bin/hadoop jar wc.jar WordCount input output```

We then `-get` this output to our normal system and inspect the output of wordCount.

This WordCount.java works as follows: the mapper gets all words from the documents, which is going the be the input for the reducer. The reducer then reads this input, and starts counting the words, thus making the list smaller, by using word counts.
This will then be written to an output file, which we can inspect.

To inspect who appears more often (Romeo or Juliet), I have altered the mapper, now only delivering 'romeo' and 'juliet' strings to the reducer:

```java
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        String w = word.toString();
        if(w.toLowerCase().contains("romeo") || w.toLowerCase().contains("juliet")){
                context.write(new Text(w), one);
        }
      }
    }
  }
  ```
So, by altering the mapper as shown above, we can see who appears more often, by only going over the whole dataset once.
  
This results in a more comprehensible file, which shows us that "Romeo" occurs 313 times and "Juliet" occurs 204 times. Thus, Romeo makes more occurences. 

