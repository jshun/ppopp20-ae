Practical Parallel Hypergraph Algorithms (PPoPP 2020 Artifact Evaluation)
======================

Getting Started Guide
--------

The framework code for Ligra-H is located in the ligra/ directory.
The code for the hypergraph algorithms is in the apps/hyper/
directory, which is where compilation should be performed for the
artifact evaluation.  Hypergraph generators and converters are
provided in the utils/ directory.

Compilation is done from within the apps/hyper/ directory. Experiments
in the paper were performed using the g++ version 7.4.0 compiler with
support for Cilk Plus. The operating system was Ubuntu 7.4.0 and the
machine was a 72-core machine with 2-way hyper-threading (Dell
PowerEdge R930 with four 2.4GHz 18-core E7-8867 v4 Xeon processors)
with 1TB of RAM. We use 'numactl -i all' for parallel experiments for
better performance.

First, install a version of g++ with support for Cilk Plus (g++
version 5.4.0 or later) on a Linux machine.  If using an AWS EC2 instance with
RedHat OS, the following instructions can be used to easily install g++ with
Cilk Plus:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/compile-software.html.

Install [numactl](https://linux.die.net/man/8/numactl), which is
used in the parallel experiments.

To compile with g++ using Cilk Plus, define the environment variable
CILK. (Note: OpenMP compilation is also supported, but we found that
Cilk Plus gives the best performance and thus should be used for the
artifact evaluation.)  Resuls will vary depending on the machine
(number of cores and sockets, cache sizes, memory bandwidth, clock
speed, etc.) used for experiments.

The commands for compilation are as follows:

```
$ export CILK=1 #sets environment variable
$ make -j  #compiles with all threads
```

The following commands cleans the directory:
```
$ make clean #removes all executables
$ make cleansrc #removes all executables and linked files from the ligra/ directory
```

Python 2.7 is used for running the scripts, so make sure it is
installed
(https://docs.datastax.com/en/install/6.7/install/installPython27RHEL.html).


Step-by-Step Instructions
--------

All of the scripts with default configurations (without large test
files) can be run with the following command. The expected running
time of this script is several days on a multicore machine with tens
of cores.

```
$ cd ppopp20-ae/
$ ./runall
```

The following script will skip the scalability tests for most of the
inputs, and will finish in several hours.

```
$ cd ppopp20-ae/
$ ./runall-quick
```

This section describes each of the commands in more detail, including
different possible configurations.

The following command downloads all of the datasets used in the
paper. By default, the large Rand2 hypergraph and the clique-expanded
graph for Friendster are not downloaded. However, the user may
download the large datasets by passing "LARGE" as an argument to the
script.  The total storage required without the large datasets is 313 GB
and with the large datasets is xxx GB.

For a quick test, the user may download and run the scalability
experiments on just two datasets (com-orkut and Rand1) by passing
"QUICK" as an argument to the scripts. The total storage required for
these two datasets is 49 GB.  (The download script will only download
the two datasets if "QUICK" is a parameter, and will need to be rerun
without "QUICK" if the user later wants to download other datasets.)

```
$ cd ppopp20-ae/inputs/
$ ./download_datasets #downloads all datasets except for the large ones; this will take a few hours
$ cd ..
```

Here are other possible ways to run the download_datasets script:

```
$ ./download_datasets LARGE #downloads the large datasets; this will take a few hours
$ ./download_datasets QUICK #downloads the com-orkut and Rand1 datasets for quick testing
$ ./download_datasets SIZES #downloads random hypergraphs of varying sizes
$ ./download_datasets DIRECTION #downloads the com-orkut and livejournal datasets 
```

Then, navigate to the directory with the hypergraph applications:

```
$ cd apps/hyper/
```

The command below runs all of the scalability experiments in the paper
as reported in Table 2, and outputs the numbers to a file. The
parallel times in the paper use all hyper-threads, and our script
prints the times using all hyper-threads as well as times using half
of the hyper-threads.  Experiments on more than 4 threads are run
three times, except for MIS, which is only run once since the program
modifies the input.

Edge-aware parallelization is used for the Orkut-group, Web, and
LiveJournal hypergraphs due to their highly-skewed degree
distributions. By default, the large Rand2 hypergraph is not included
in the experiments, however it will be included if "LARGE" is passed
as an argument to the script. If "QUICK" is passed as an argument to
the script, the experiments will only be run on com-orkut and Rand1.

Note: "HyperBFS" corresponds to "Hypertree" in the paper and
"HyperKCore-Efficient" corresponds to "WE k-core" in the paper.


```
$ ./run_scalability | tee scalability_results.txt #default: runs experiments on all datasets except Rand2; this will take several days to run
```

Here are other ways to run the run_scalability script:
```
$ ./run_scalability QUICK | tee scalability_results.txt #runs experiments on only com-orkut and Rand1
$ ./run_scalability LARGE | tee scalability_results.txt #runs experiments on all datasets; this will take at least several days to run
```

The following command runs all of algorithms on a varying number of
threads on Rand1 (Figure 2 in the paper), and outputs the numbers to a
file:

```
$ ./run_varying_threads | tee varying_threads_results.txt
```

The following command runs all of algorithms on a varying number of
hyperedges (Figure 3 in the paper), and outputs the numbers to a
file. The files for this experiment can be downloaded by going to the
inputs/ directory, and running the download_datasets script described
above with the "SIZES" argument.

```
$ ./run_varying_hyperedges | tee varying_hyperedges_results.txt
```

The following command computes the running time of sparse, dense, and
hybrid traversals on com-Orkut and LiveJournal (Figures 4 and 5 in the
paper), and outputs the numbers to a file. The files for this
experiment can be downloaded by going to the inputs/ directory, and
running the download_datasets script described above with the
"DIRECTION" argument.

```
$ ./run_directions | tee directions_results.txt
```

The following command computes the running time using different
thresholds for direction-optimization on com-Orkut and LiveJournal
(Figures 6 and 7 in the paper), and outputs the numbers to a file.
The files for this experiment can be downloaded by going to the
inputs/ directory, and running the download_datasets script described
above with the "DIRECTION" argument.

```
$ ./run_thresholds | tee thresholds_results.txt
```

The following command computes the running time on the clique-expanded
graph for Friendster using breadth-first search, connected components,
and SSSP in Ligra. This script assumes that the clique-expanded graph
for Friendster has been downloaded (see instructions above for
downloading the large datasets).

```
$ cd ..;
$ ./run_clique | tee clique_results.txt
```

The paper compares to the MESH hypergraph processing system, which can
be downloaded from https://github.com/mesh-umn/MESH. If the user
wishes to test the performance of MESH, please follow the instructions
on their GitHub page.  The com-orkut hypergraph in MESH format can be
downloaded from https://ppopp20-ae.s3.amazonaws.com/com-orkut-MESH.


List of claims from the paper supported by the artifact
--------

* The parallel hypergraph algorithms achieve good performance and
  parallel speedup (Table 2 and Figure 2 of the paper).

* The parallel hypergraph algorithms achieve good scalability with
  respect to input size (Figure 3).

* The hybrid traversal strategy usually improves performance over
  using only sparse or only dense traversals (Figures 4 and 5).

* The performance is similar across a wide range of thresholds in the
  hybrid traversal strategy (Figures 6 and 7).

* Using the hypergraph algorithms is faster than using the
  corresponding graph algorithms on the clique-expanded representation
  for BFS, CC, and SSSP, which are the only three algorithms in the
  paper for which the graph algorithm would produce the correct result
  on the clique-expanded representation.

* Our hypergraph algorithms outperform the corresponding algorithms in
  MESH.

List of claims from the paper not supported by the artifact
--------

* The raw running times obtained by the user may differ from the
  numbers reported in the paper due to the use of a different machine
  and/or compiler.

* The performance counters in Table 3 are obtained using the
  [perf](https://perf.wiki.kernel.org/index.php/Tutorial)
  tool. Obtaining these counters requires root access to the machine
  as well as modifying the code to exclude the loading of the
  dataset in the performance measurements. Not all performance
  counters are supported by all machines. We have not included
  this in the artifact evaluation due to the significant effort that
  would be required by the user to get the performance counters to
  work properly.


The remaining sections contain instructions to run any individual
experiments that the user may want to run.


Running code in Ligra-H
-------
The hypergraph applications are located in the apps/hyper/
directory. The applications take the input hypergraph as input as well
as an optional flag "-s" to indicate a symmetric hypergraph.
Symmetric hypergraphs should be called with the "-s" flag for better
performance. For example:

```
$ ./HyperBFS -s ../inputs/test
$ ./HyperSSSP -s ../inputs/test-wgh
``` 

On multi-socket machines, adding the command "numactl -i all " when
running the program may improve performance for large inputs. For
example:

```
$ numactl -i all ./HyperBFS -s <input file>
```

For traversal algorithms, one can also pass the "-r" flag followed by
an integer to indicate the source vertex.  Random hypergraphs can be
generated with the hypergraph generator in the utils/ directory.



Input Format for Ligra-H applications
-----------
The input can be in either adjacency hypergraph format or binary format.

1) The adjacency hypergraph format starts with a sequence of offsets
 one for each vertex, followed by a sequence of incident hyperedges
 (the vertex is an incoming member of the hyperedge) ordered by
 vertex, followed by a sequence of offsets one for each hyperedge, and
 finally a sequence of incident vertices (the vertex is an outgoing
 member of the hyperedge) ordered by hyperedge.  All vertices,
 hyperedges, and offsets are 0 based and represented in decimal. For a
 graph with *nv* vertices, *mv* incident hyperedges for the vertices,
 *nh* hyperedges, and *mh* incident vertices for the hyperedges, the
 specific format is as follows:

AdjacencyHypergraph  
&lt;nv>
&lt;mv>
&lt;nh>
&lt;mh>
&lt;ov0>  
&lt;ov1>  
...  
&lt;ov(nv-1)>  
&lt;ev0>  
&lt;ev1>  
...  
&lt;ev(mv-1)>  
&lt;oh0>  
&lt;oh1>  
...  
&lt;oh(nh-1)>  
&lt;eh0>  
&lt;eh1>  
...  
&lt;eh(mh-1)>  

This file is represented as plain text.

2) In binary format. This requires five files NAME.config, NAME.vadj,
NAME.vidx, NAME.hadj, NAME.hidx, where NAME is chosen by the user. The
.config file stores *nv*, *mv*, *nh*, and *mh* in text format. The
.vidx and .hidx files store in binary the offsets for the vertices and
hyperedges (the &lt;ov>'s and &lt;oh>'s above). The .vadj and .hadj
files stores in binary the neighbors (the &lt;ev>'s and &lt;eh>'s
above).

Weighted hypergraphs: For format (1), the weights are listed as
another sequence following the sequence of neighbors for vertices or
hyperedges file (i.e., after &lt;ev(mv-1)> and &lt;eh(mh-1)>), and the
first line of the file should store the string
"WeightedAdjacencyHypergraph". For format (2), the weights are stored
after all of the edge targets in the .vadj and .hadj files.



Utilities
---------

Several utilities are provided in the utils/ directory and can
be compiled using "make".


### Random Hypergraph Generator

The random hypergraph generator **randHypergraph** takes as input the
number of vertices ('-nv'), number of hyperdges ('-nh') and
cardinality of each hyperedge ('-c'). It generates a symmetric
hypergraph where each hyperedge has the specified cardinality and
where member vertices uniformly at random.

Examples:
```
$ ./randHypergraph -nv 100000000 -nh 100000000 -c 10 randHypergraph_output 
```

### Hypergraph Converters

**communityToHyperAdj** converts a communities network in [SNAP
format](http://snap.stanford.edu/data/index.html) and converts it to
symmetric adjacency hypergraph format. The first required parameter is
the input (SNAP) file name and second required parameter is the output
(Ligra) file name.

**KONECTtoHyperAdj** converts a bipartite graph in [KONECT
format](http://konect.uni-koblenz.de/) (the out.* file) and converts
it to symmetric adjacency hypergraph format. The first required
parameter is the input (KONECT) file name and second required
parameter is the output (Ligra) file name.

**adjHypergraphAddWeights** adds random integer weights in the range
[1,...,*log<sub>2</sub>*(max(number of vertices, number of
hyperedges))] to an unweighted hypergraph in adjacency hypergraph
format, and takes as input the input file name followed by the output
file name.

**hyperAdjToBinary** converts an input in adjacency hypergraph format
to binary format. The argument is the adjacency hypergraph file
name. For a weighted hypergraph, pass the "-w" flag before the file
name. The program will generate 5 output files with the input file
name followed by each of the prefixes .config, .vadj, .vidx, .hadj,
and .hidx.

Examples:
```
$ ./communityToHyperAdj SNAPfile LigraHFile
$ ./KONECTtoHyperAdj KONECTfile LigraHfile
$ ./adjHypergraphAddWeights unweightedLigraHFile weightedLigraHFile
$ ./hyperAdjToBinary test
$ ./hyperAdjToBinary -w test-wgh
```
