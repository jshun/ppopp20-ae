Practical Parallel Hypergraph Algorithms (PPoPP 2020 Artifact Evaluation)
======================

The latest code is in the [Ligra GitHub repository](https://github.com/jshun/ligra/).

Getting Started Guide
--------

The framework code for Hygra is located in the ligra/ directory.
The code for the hypergraph algorithms is in the apps/hyper/
directory, which is where compilation should be performed for the
artifact evaluation.  Hypergraph generators and converters are
provided in the utils/ directory.

Compilation is done from within the apps/hyper/ directory. Experiments
in the paper were performed using the g++ version 5.5.0 compiler with
support for Cilk Plus. The operating system was Ubuntu 16.04 and the
machine was a 72-core machine with 2-way hyper-threading (Dell
PowerEdge R930 with four 2.4GHz 18-core E7-8867 v4 Xeon processors)
with 1TB of RAM. We use 'numactl -i all' for parallel experiments for
better performance.

First, install a compiler with support for Cilk Plus (g++
version 5.4.0 or later or Tapir/LLVM) on a Linux machine.  If using an AWS EC2 instance with
RedHat OS, the following instructions can be used to easily install g++ with
Cilk Plus:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/compile-software.html.
A docker image containing the g++ cilk compiler can be found here: https://hub.docker.com/r/jonniesweb/cilkplus. Instructions for installing the Tapir/LLVM compiler are available here: http://cilk.mit.edu/.

Install [numactl](https://linux.die.net/man/8/numactl), which is
used in the parallel experiments.

To compile with g++ using Cilk Plus, define the environment variable
CILK. (Note: OpenMP compilation is also supported, but we found that
Cilk Plus gives the best performance and thus should be used for the
artifact evaluation.)  Results will vary depending on the machine
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

For a quick test, the user may run the scalability experiments on a
small dataset provided in the inputs/ directory by passing "QUICK" as
an argument to the script. This will not require any generation or
download of files.

```
$ cd ppopp20-ae/apps/hyper/
$ ./run_scalability QUICK | tee scalability_results.txt #default: runs experiments on a small dataset
```

All of the scripts with default configurations (without large test
files and the experiment in Ligra with the clique-expanded graph) can
be run with the following command. The expected running time of this
script is several days on a multicore machine with tens of cores.

```
$ cd ppopp20-ae/
$ ./runall
```

The following script will skip the scalability tests for all of the
inputs except a small dataset, and also skip the test on varying thread
counts on Rand1. This script should finish in several hours on a
multicore machine with tens of cores.

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
and with the large datasets is 1 TB.


```
$ cd ppopp20-ae/inputs/
$ ./download_datasets #downloads all datasets except for the large ones; this will take a few hours
$ cd ..
```

Here are other possible ways to run the download_datasets script:

```
$ ./download_datasets LARGE #downloads the large datasets; this will take a few hours
$ ./download_datasets RAND1 #downloads the Rand1 dataset for testing varying thread counts
$ ./download_datasets SIZES #downloads random hypergraphs of varying sizes
$ ./download_datasets DIRECTION #downloads the com-orkut and livejournal datasets 
```

The datasets come from the [Stanford Large Network Dataset Collection
(SNAP)](http://snap.stanford.edu/data/index.html), the [Koblenz
Network Collection (KONECT)](http://konect.uni-koblenz.de/), and
synthetic random hypergraph generator provided in the utils/
directory. These datasets can be downloaded individually, and the
links are provided in the **Datasets** section below.

To run the timing experiments, navigate to the directory with the hypergraph applications:

```
$ cd apps/hyper/
```

The command below runs all of the scalability experiments in the paper
as reported in Table 2, and outputs the numbers to a file. The
parallel times in the paper use all hyper-threads, and our script
prints the times using all hyper-threads.  Experiments are run three
times, except for MIS, which is only run once since the program
modifies the input (numbers reported in the paper take the minimum of
three trials).

Edge-aware parallelization is used for the Orkut-group, Web, and
LiveJournal hypergraphs due to their highly-skewed degree
distributions. By default, the large Rand2 hypergraph is not included
in the experiments, however it will be included if "LARGE" is passed
as an argument to the script. If "QUICK" is passed as an argument to
the script, the experiments will only be run on com-orkut.

**Note: "HyperBFS" corresponds to "Hypertree" in the paper, "HyperKCore"
corresponds to "WI k-core" in the paper, and "HyperKCore-Efficient"
corresponds to "WE k-core" in the paper.**


```
$ ./run_scalability | tee scalability_results.txt #default: runs experiments on all datasets except Rand2; this will take several days to run
```

Here are other ways to run the run_scalability script:
```
$ ./run_scalability QUICK | tee scalability_results.txt #runs experiments on a small dataset
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

Expected output format
--------

The expected output format of the Hygra code is one or more lines with
the running time of each trial. For example:

```
$ numactl -i all ./HyperBFS -s ../../inputs/com-orkut-hygra #running 3 trials on all threads
Running time : 0.031
Running time : 0.031
Running time : 0.033

$ numactl -i all ./HyperBFS -s -rounds 1 ../../inputs/com-orkut-hygra #running 1 trial on all threads
Running time : 0.031

$ CILK_NWORKERS=1 ./HyperBFS -s -rounds 1 ../../inputs/com-orkut-hygra #running 1 trial on one thread
Running time : 1.04

```

The provided scripts will run multiple experiments, and for each
experiment it will output a line containing the name of the algorithm,
number of threads used, and the name of the dataset (with a suffix of
-hygra), followed by one or more lines with the running time of each
trial. For example:

```
HyperBFS 1 thread(s) on com-orkut-hygra
Running time : 1.04
Running time : 1.04
Running time : 1.04

HyperBFS 144 thread(s) on com-orkut-hygra
Running time : 0.031
Running time : 0.031
Running time : 0.033

```

Datasets
--------


The datasets come from the following sources:

* [Stanford Large Network Dataset Collection (SNAP)](http://snap.stanford.edu/data/index.html)

* [Koblenz Network Collection (KONECT)](http://konect.uni-koblenz.de/)

* The synthetic random hypergraph generator provided in the utils/ directory.

Files can be downloaded directly from
[SNAP](http://snap.stanford.edu/data/index.html) and
[KONECT](http://konect.uni-koblenz.de/) and converted to Hygra format
using the converters described below in the **Utilities** section of
this README file. Random hypergraphs can be generated directly using
the random hypergraph generator described in the **Utilities**
section. Weighted hypergraphs can be generated from the unweighted
hypergraphs using the **adjHypergraphAddWeights** script described in
the **Utilities** section.

Here are the links to specific files in Hygra format (which can be
downloaded individually) and the corresponding files in SNAP or KONECT
format:

* com-orkut: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/com-orkut-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/com-orkut-wgh-hygra), [\[SNAP\]](http://snap.stanford.edu/data/com-Orkut.html), [\[MESH\]](https://ppopp20-ae.s3.amazonaws.com/com-orkut-MESH)
* friendster: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/friendster-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/friendster-wgh-hygra), [\[SNAP\]](http://snap.stanford.edu/data/com-Friendster.html), [\[Ligra clique-expanded graph (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/friendster-clique), [\[Ligra clique-expanded graph (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/friendster-wgh-clique)
* orkut-group: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/orkut-group-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/orkut-group-wgh-hygra), [\[KONECT\]](http://konect.uni-koblenz.de/networks/orkut-groupmemberships) 

* web: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/web-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/web-wgh-hygra), [\[KONECT\]](http://konect.uni-koblenz.de/networks/trackers-trackers)

* livejournal: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/livejournal-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/livejournal-wgh-hygra), [\[KONECT\]](http://konect.uni-koblenz.de/networks/livejournal-groupmemberships)

* rand1: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand1-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand1-wgh-hygra)

* rand2: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand2-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand2-wgh-hygra)

* rand3: [\[Hygra (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand3-hygra), [\[Hygra (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand3-wgh-hygra)

* random hypergraphs (X, Y) in Hygra format with X vertices and Y hyperedges, each with cardinality 10: [\[(10M, 10M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-10M-10-hygra), [\[(10M, 10M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-10M-10-wgh-hygra),
[\[(10M, 20M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-20M-10-hygra), [\[(10M, 20M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-20M-10-wgh-hygra),
[\[(10M, 30M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-30M-10-hygra), [\[(10M, 30M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-30M-10-wgh-hygra),
[\[(10M, 40M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-40M-10-hygra), [\[(10M, 40M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-40M-10-wgh-hygra),
[\[(10M, 50M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-50M-10-hygra), [\[(10M, 50M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-50M-10-wgh-hygra),
[\[(10M, 60M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-60M-10-hygra), [\[(10M, 60M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-60M-10-wgh-hygra),
[\[(10M, 70M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-70M-10-hygra), [\[(10M, 70M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-70M-10-wgh-hygra),
[\[(10M, 80M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-80M-10-hygra), [\[(10M, 80M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-80M-10-wgh-hygra),
[\[(10M, 90M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-90M-10-hygra), [\[(10M, 90M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-90M-10-wgh-hygra),
[\[(10M, 100M) (unweighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-100M-10-hygra), [\[(10M, 100M) (weighted)\]](https://ppopp20-ae.s3.amazonaws.com/rand-10M-100M-10-wgh-hygra). These can be generated by running the **randHypergraph** generator described in the **Utilities** section below.

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


Running code in Hygra
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
an integer to indicate the source vertex.  By default, the programs
are run for three trials, but one can change the number of trials by
passing the "-rounds" flag followed by an integer indicating the
number of trials.



Input Format for Hygra applications
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
(Hygra) file name.

**KONECTtoHyperAdj** converts a bipartite graph in [KONECT
format](http://konect.uni-koblenz.de/) (the out.* file) and converts
it to symmetric adjacency hypergraph format. The first required
parameter is the input (KONECT) file name and second required
parameter is the output (Hygra) file name.

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
$ ./communityToHyperAdj SNAPfile HygraFile
$ ./KONECTtoHyperAdj KONECTfile Hygrafile
$ ./adjHypergraphAddWeights unweightedHygraFile weightedHygraFile
$ ./hyperAdjToBinary test
$ ./hyperAdjToBinary -w test-wgh
```
