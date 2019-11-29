ifdef LONG
INTT = -DLONG
endif

ifdef EDGELONG
INTE = -DEDGELONG
endif

ifdef PD
PD = -DPD
endif

ifdef BYTE
CODE = -DBYTE
else ifdef NIBBLE
CODE = -DNIBBLE
else
CODE = -DBYTERLE
endif

ifdef LOWMEM
MEM = -DLOWMEM
endif

#compilers
ifdef CILK
PCC = g++
PCFLAGS = -std=c++14 -fcilkplus -lcilkrts -O3 -DCILK $(INTT) $(INTE) $(CODE) $(PD) $(MEM)
PLFLAGS = -fcilkplus -lcilkrts

else ifdef MKLROOT
PCC = icpc
PCFLAGS = -std=c++14 -O3 -DCILKP $(INTT) $(INTE) $(CODE) $(PD) $(MEM)

else ifdef OPENMP
PCC = g++
PCFLAGS = -std=c++14 -fopenmp -march=native -O3 -DOPENMP $(INTT) $(INTE) $(CODE) $(PD) $(MEM)

else
PCC = g++
PCFLAGS = -std=c++14 -O3 $(INTT) $(INTE) $(CODE) $(PD) $(MEM)
endif

COMMON= ligra.h graph.h compressedVertex.h vertex.h utils.h IO.h parallel.h gettime.h index_map.h maybe.h sequence.h edgeMap_utils.h binary_search.h quickSort.h blockRadixSort.h transpose.h parseCommandLine.h byte.h byteRLE.h nibble.h byte-pd.h byteRLE-pd.h nibble-pd.h vertexSubset.h decoder.C hypergraphIO.h ligra-h.h bucket.h counting_sort.h dyn_arr.h edgeMapReduce.h histogram.h sequentialHT.h hypergraphEncoder.C hypergraphDecoder.C

ALL= hypergraphEncoder hypergraphDecoder HyperReader WeightedHyperReader HyperBFS HyperBPath HyperCC HyperSSSP HyperPageRank HyperBC HyperMIS HyperKCore HyperKCore-Efficient HyperBFS-EdgeAware HyperCC-EdgeAware HyperSSSP-EdgeAware HyperPageRank-EdgeAware HyperBC-EdgeAware HyperMIS-EdgeAware HyperKCore-EdgeAware HyperKCore-Efficient-EdgeAware

DIRECTION= HyperBFS-Sparse HyperBFS-Dense HyperCC-Sparse HyperCC-Dense HyperSSSP-Sparse HyperSSSP-Dense HyperPageRank-Sparse HyperPageRank-Dense HyperBC-Sparse HyperBC-Dense HyperMIS-Sparse HyperMIS-Dense HyperKCore-Efficient-Sparse HyperKCore-Efficient-Dense HyperBFS-Sparse-EdgeAware HyperBFS-Dense-EdgeAware HyperCC-Sparse-EdgeAware HyperCC-Dense-EdgeAware HyperSSSP-Sparse-EdgeAware HyperSSSP-Dense-EdgeAware HyperPageRank-Sparse-EdgeAware HyperPageRank-Dense-EdgeAware HyperBC-Sparse-EdgeAware HyperBC-Dense-EdgeAware HyperMIS-Sparse-EdgeAware HyperMIS-Dense-EdgeAware HyperKCore-Efficient-Sparse-EdgeAware HyperKCore-Efficient-Dense-EdgeAware

THRESHOLD= HyperBFS-Threshold HyperBFS-Threshold-EdgeAware HyperCC-Threshold HyperCC-Threshold-EdgeAware HyperBC-Threshold HyperBC-Threshold-EdgeAware HyperSSSP-Threshold HyperSSSP-Threshold-EdgeAware

all: $(ALL)

direction: $(DIRECTION)

threshold: $(THRESHOLD)

% : %.C $(COMMON)
	$(PCC) $(PCFLAGS) -o $@ $<

$(COMMON):
	ln -s ../../ligra/$@ .

.PHONY : clean

clean :
	rm -f *.o $(ALL) $(DIRECTION) $(THRESHOLD) 

cleansrc :
	rm -f *.o $(ALL) $(DIRECTION) $(THRESHOLD)
	rm $(COMMON)
