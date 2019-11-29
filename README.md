Practical Parallel Hypergraph Algorithms (PPoPP 2020 Artifact Evaluation)
======================

to do
--------
* script for scalability experiments
* script for input graph generators
* script to download datasets on Amazon S3
* sparse vs. dense experiments
* threshold experiments
* varying size experiments
* performance counters?
* script for converting to clique and running in Ligra

Organization
--------

The framework code for Ligra-H is located in the ligra/ directory.
The code for the hypergraph algorithms is in the apps/hyper/
directory, which is where compilation should be performed for the
artifact evaluation.  Hypergraph generators and converters are
provided in the utils/ directory.

Compilation
--------

Compilation is done from within the apps/hyper/ directory. Experiments
in the paper were performed using the g++ version 7.4.0 compiler with
support for Cilk Plus. The operating system was Ubuntu 7.4.0 and the
machine was a 72-core machine with 2-way hyper-threading (Dell
PowerEdge R930 with four 2.4GHz 18-core E7-8867 v4 Xeon processors)
with 1TB of RAM. We use 'numactl -i all' for parallel experiments for
better performance.

Any version of g++ with support for Cilk Plus on an Ubuntu OS can be
used for the artifact evaluation.  To compile with g++ using Cilk
Plus, define the environment variable CILK. (Note: OpenMP compilation
is also supported, but we found that Cilk Plus gives the best
performance and thus should be used for the artifact evaluation.)
Resuls will vary depending on the machine (number of cores and
sockets, cache sizes, memory bandwidth, clock speed, etc.) used for
experiments.

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



Running all experiments in the paper
--------

This section describes all of the commands needed to run the
experiments presented in the paper.

First, install the g++ compiler with Cilk Plus support. Python 2.7 is
used for running the scripts.



The following command downloads or generates all of the datasets used
in the paper:

```
$ cd ppopp20-ae/
$ mkdir inputs
$ cd inputs
$ ./download_datasets
$ cd ..
```

Then, navigate to the directory with the hypergraph applications:

```
$ cd apps/hyper/
```


The following command runs all of the scalability experiments in the
paper as reported in Table 2, and outputs the numbers to a file:

```
$ ./run_scalability | tee results.txt
$ ./get_speedup | tee speedups.txt
```

The following command runs all of algorithms on a varying number of
threads on Rand1 (Figure 2 in the paper):

```
$ ./run_varying_threads | tee varying_threads.txt
```

The following command runs all of algorithms on a varying number of
hyperedges (Figure 3 in the paper):

```
$ ./run_varying_hyperedges | tee varying_hyperedges.txt
```

The following computes the running time of sparse, dense, and hybrid
traversals on com-Orkut and LiveJournal (Figures 4 and 5 in the paper):

```
$ ./run_directions | tee directions.txt
```

The following computes the running time using different thresholds for
direction-optimization on com-Orkut and LiveJournal (Figures 6 and 7
in the paper):

```
$ ./run_thresholds | tee thresholds.txt
```

The following sections contains instructions to run any additional
experiments that the reviewer may want to run.

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


Running code in Ligra
-------
The applications take the input graph as input as well as an optional
flag "-s" to indicate a symmetric graph.  Symmetric graphs should be
called with the "-s" flag for better performance. For example:

```
$ ./BFS -s ../inputs/rMatGraph_J_5_100
$ ./BellmanFord -s ../inputs/rMatGraph_WJ_5_100
``` 

For BFS, BC and BellmanFord, one can also pass the "-r" flag followed
by an integer to indicate the source vertex.  rMat graphs along with
other graphs can be generated with the graph generators in the utils/
directory.  By default, the applications are run four times, with
times reported for the last three runs. This can be changed by passing
the flag "-rounds" followed by an integer indicating the number of
timed runs.

On NUMA machines, adding the command "numactl -i all " when running
the program may improve performance for large graphs. For example:

```
$ numactl -i all ./BFS -s <input file>
```

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
name. For a weighted graph, pass the "-w" flag before the file
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
