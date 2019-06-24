# Final Assignment
## WARC for Spark
For this assignment, I pulled a WARC file from the ComonCrawl using the query "BBC". I plugged this into the notebook and see what would happen.
As I wanted to start working on the actual assignment, I did no fancy analysis on this file, I continued creating a Self-contained Spark application using the available instructions

## Self-contained Spark applications

#### That stuff that makes you pull your hair out
The set-up of the Spark-workers and the Spark-master was painless. Docker ran them without issues and everything seemed to go smooth.
As we had worked inside the containers for the other parts of the course, I had already executed 

```bash
docker run -it bde2020/spark-master:2.4.1-hadoop2.7 /bin/bash```

Since nothing was working, I couldn't run anything, Googles solutions didn't work and my patience was depleted, I asked the teacher what I was doing wrong. Yap. I should not have opened the container.
So, I removed the downloaded files that I obtained using `wget` from http://rubigdata.github.io/course/background/rubigdata.tgz from the container.
The command `exit` got me out of the docker container and this solved most of the problem.

However..
As I run MacOS (yes... I know) there is no such thing as `wget`. This seemed to be a slight issue since `curl` didn't download the files as I wanted. 
The obvious workaround was to download it via a regular browser, but to be honest, the satisfaction factor goes down by doing this.
Then, I remembered that there existed a day in the domain of days where it was the case that I installed a program that worked like the `apt-get` commands that one would use in Linux.
It is called Homebrew (and for every Mac-user that ever uses the terminal, this is a great tool -- dear TA, be a hero for next-years students and let them install Homebrew).

So, for maximum satisfaction, we enter:

```bash
brew install wget```

and execute
```bash
wget http://rubigdata.github.io/course/background/rubigdata.tgz
tar xzvfp rubigdata.tgz
cd rubigdata

```
And we were all set :).


#### Of course, I did not actually pull my hair out
The spark-app was built, smashed together in a JAR file and submitted to the cluster. Judging from my web-interface, it ran for 16 seconds on all 4 cores. During this time, it was counting the number of lines where "a" occurred in and the number of lines where "e" occurred in.
Also, it collected the indices of all values where "i" occurred, then multiply those indices by 2, and filtering out all even numbers, by issueing

```bash
val numAs = data.indices("i").map(a => a * 2).filter(b => b % 2 == 0)
```
Of course, you can do a lot more with this data, you just use Scala operations to an add/replace this program.
For my purposes, this is working, after quite some trouble, so I will happily finish this assignment, being able to say that I wrote some program that executes on a cluster :)

Happy summer holidays!
