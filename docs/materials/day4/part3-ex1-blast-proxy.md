Thursday Exercise 3.1: Using a Web Proxy for Large Shared Input
===============================================================


Continuing the series of exercises blasting mouse genetic sequences, the objective of this exercise is to use a web proxy to stage the large database, which will be downloaded into each of many jobs that use the split input files from the last exercise (Exercise 2.3).

Setup
-----

-   Make sure you are logged into `user-training.osgconnect.net`
-   Make sure you are in the same directory as the previous exercise, [Exercise 2.3](Education.UserSchool17Thu23BlastSplit) directory named `thur-blast-data`.

Place the Large File on the Proxy
---------------------------------

First, you'll need to put the `pdbaa_files.tar.gz` file onto the Stash web directory. Use the following command:

``` console
%UCL_PROMPT_SHORT% cp pdbaa_files.tar.gz ~/stash/public/
```

### Test a download of the file

Once the file is placed in the ~/stash/public directory, it can be downloaded from a corresponding URL such as `http://stash.osgconnect.net/~%RED%username%ENDCOLOR%/pdbaa_files.tar.gz`, where %RED%username<span class="twiki-macro ENDCOLOR"></span> is your username on `user-training.osgconnect.net`.

Using the above convention (and from a different directory on `user-training.osgconnect.net`, any directory), you can test the download of your `pdbaa_files.tar.gz` file with a command like the following:

``` console
%UCL_PROMPT_SHORT% <strong>wget http://stash.osgconnect.net/~%RED%username%ENDCOLOR%/pdbaa_files.tar.gz</strong>
```

You may realize that you've been using `wget` to download files from a web proxy for many of the previous exercises at the school!

Run a New Test Job
------------------

Now, you'll repeat the last exercise (with a single input query file) but have HTCondor download the `pdbaa_files.tar.gz` file from the web proxy, instead of having the file transferred from the submit server.

### Modify the submit file and wrapper script

In the wrapper script, we have to add some special lines so that we can pull from Stash and use the HTTP proxy. In `blast_wrapper.sh`, we will have to add commands to pull the data file:

``` file
#!/bin/bash
%RED%
# Set an environment variable to point to the local Proxy location at the cluster
export http_proxy=$OSG_SQUID_LOCATION
# Copy the pdbaa_files.tar.gz to the worker node
# Add the -S argument, so we can see if it was a cache HIT or MISS
wget -S http://stash.osgconnect.net/~username/pdbaa_files.tar.gz
%ENDCOLOR%
tar xvzf pdbaa_files.tar.gz
tar xvzf blastx.tar.gz


./blastx -db pdbaa -query $1 -out $1.result

rm pdbaa.*
rm blastx
```

The first line will set a special environment variable that `wget` will look at. `$OSG_SQUID_LOCATION` is a special variable that is set on OSG sites to the hostname of the nearest squid cache. The second line will download the `pdbaa_files.tar.gz` from stash, using the closes cache (because `wget` will look at `http_proxy` for the newest cache=).

In your submit file, you will need to remove the `pdbaa_files.tar.gz` file from the `transfer_input_files`, because we are using HTTP proxies!

### Submit the test job

You may wish to first remove the log, result, output, and error files from the previous tests, which will be overwritten when the new test job completes.

``` console
rm *.err *.out *.result *.log
```

Submit the test job!

When the job starts, the wrapper will download the `pdbaa_files.tar.gz` file from the web proxy. If the jobs takes longer than two minutes, you can assume that it will complete successfully, and then continue with the rest of the exercise.

After the job completes examine the \*.err file generated by the submission. At the top of the file, you will find something like:

``` file
--2017-07-19 00:36:29--  http://stash.osgconnect.net/~osguser199/pdbaa_files.tar.gz
Resolving cmsprxy01.colorado.edu... 192.12.238.152
Connecting to cmsprxy01.colorado.edu|192.12.238.152|:3128... connected.
Proxy request sent, awaiting response... 
  HTTP/1.0 200 OK
  Content-Type: application/octet-stream
  Content-Length: 22105114
  Accept-Ranges: bytes
  Server: nginx/1.10.2
  Date: Wed, 19 Jul 2017 00:36:29 GMT
  Last-Modified: Wed, 19 Jul 2017 05:03:09 GMT
  ETag: "596ee80d-1514c1a"
  X-Cache: HIT from cmsprxy01.colorado.edu
  Via: 1.1 cmsprxy01.colorado.edu:3128 (squid/frontier-squid-2.7.STABLE9-14)
  Connection: close
Length: 22105114 (21M) [application/octet-stream]
Saving to: `pdbaa_files.tar.gz'
...
```

Notice the `X-Cache` line. It says it was a cache HIT from the proxy `cmsprxy01.colorado.edu`. Yay! You successfully used a proxy to cache data near your worker node! Notice, the name of the cache file may be different.

Run all 100 Jobs!
-----------------

If all of the previous tests have gone okay, you can prepare to run all 100 jobs that will use the split input files. To make sure you're not going to generate too much data, use the size of files from the previous test to calculate how much total data you're going to add to the `thur-blast-data` directory for 100 jobs.

In the submit file, you will need to change the queue line to scan all the rna files:

``` file
...
queue inputfile matching mouse_rna.fa.*
```

Submit all 100 jobs! They may take a while to all complete, but it will still be faster than the many hours it would have taken to blast the single, large `mouse_rna.fa` file without splitting it up. In the meantime, as long as the first several jobs are running for longer than two minutes, you can move on to the [next exercise](Education.UserSchool17Thurs32BlastStash).


