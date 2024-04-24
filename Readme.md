# Experiments for Efficient Mixed Integer Linear Programming Approaches to Dynamic Path Restoration



In this repository we provide data and results for the experiments discussed in the paper Efficient Mixed Integer Linear Programming Approaches to Dynamic Path Restoration (which is currently under review). The detailed description of the implemented algorithm can be found in this paper. (Currently this repository seems to be useful for referee only.)



## Restoration Problem

We improved a known MILP approach for Routing and Spectrum Allocation Problem (RSA) that we apply to a special case of the generalized version of RSA problem. We consider an optical network with routed  traffic, thus there are paths in the network corresponding to services (*demands*). We consider the case of a link failure, so the paths routed through this link must be rerouted or it should be proved that a rerouting is not possible. This problem is called the Restoration Problem.



## General Description of the Approach

In classical MILP models variables are indexed via contiguous range. Our base model has variables indexed by a triple consisting of a demand, a link, and a color (frequency slot): $x^{d,l}_c = 1$ if the color $c$ of the link $l$ is occupied by the demand $d$; otherwise  $x^{d,l}_c = 0$.  Since our investigation is focused on congested networks, many colors of links are already occupied, so many variables $x^{d,l}_c$ are forced to be zero. Moreover, we can remove variables from the model, but we cannot do it directly since some constraints rely on the absence of gaps in variables range. So, we first modify the model to support gaps in the ranges of the variables.

If a color $c$ of an edge $l$ can never occur on a reachable path that connects endpoints of a demand $d$, then variable $x^{d,l}_c$ is forced to be zero as well. We call a triple $(d, l, c)$ *useless* if $x^{d,l}_c = 0$, because the color $c$ of the link $l$  is either already occupied, or there is no reachable path for $d$ that uses the color $c$ of $l$.  We developed an algorithm called *trimming* that finds all useless triples and constructs the improved model (that supports gaps). In this model, all variables represent triples that are useful (i.e., not useless).

So, in our tests we consider three models:

- **Base** — the initial model, where we set $x^{d,l}_c$ to $0$ if the color $c$ of the link $l$ is already occupied.
- **NoTrim** — the model where we got rid of variables $x^{d,l}_c$  for which the color $c$ of the link $l$ is already occupied.
- **Trimmed** — the model, where we got rid of all variables $x^{d,l}_c$ with useless triples $(d,l,c)$​.



## Description of Test Cases

Experiments are done in the context of optical networks. We use the public networks USNET and NSFNET that were also used in the paper [Lightpath Management in SDN-Based Elastic Optical Networks With Power Consumption Considerations](https://www.researchgate.net/publication/321952636_Lightpath_Management_in_SDN-Based_Elastic_Optical_Networks_With_Power_Consumption_Considerations?enrichId=rgreq-9930aaf6f78f60cc156a0f4c0091e69c-XXX&enrichSource=Y292ZXJQYWdlOzMyMTk1MjYzNjtBUzo1NzQ2MjQxMjA0Nzk3NDRAMTUxNDAxMjQ3NDM2Ng%3D%3D&el=1_x_3&_esc=publicationCoverPdf) by Yu Xiong et al.).

To obtain non-trivial test-cases we routed paths in a network using shared path protection as follows. We randomly order all pairs of nodes and for each pair $(s,t), s \neq t$ we try to find a path $P$ that connects $s$ and $t$ and another path $R$ that does not share links with $P$. We call $P$ a *main path* and $R$ a *recovery path*. If such paths exist, we route them in the network. If two main paths do not share a link, then their recovery paths are allowed to share links, (hence, the name `shared' path protection). We repeat the process of path routing until both $P$ and $R$ exist for some pair of nodes $(s,t)$. When the process is finished, we remove all recovery paths from the network. This is how we generate the first kind of test cases.

It is easy to see that in the test cases of the first kind the recovery problem is solvable, since each broken demand has a corresponding recovery path that do not intersect with the main path and no two recovery paths  share the same link as well. So, to obtain test cases of the second kind, we break a link in the test cases of the first kind and route the paths corresponding to the broken demands. Such routing can have links with corresponding feasible and unfeasible test cases. 

For each routed network, we obtain test sets of the first kind by breaking each link. For the second kind, we need to break 2 links. The first link is the same for all test cases, each other link is chosen as a second link. Hence, the number of test sets of the second kind is exactly 1 more than the number of test sets of the first kind. 

The length of a path is bounded; we denote the upper bound of a demand $d$ by $r_d$ and call it the *reachability  threshold*. The value reachability  threshold depends on a *Modulation Scheme*: 

- 5000 km for BPSK
- 2500 km for QPSK
- 1250 km for 8-QAM 

We considered the modulation schemes from the paper [Lightpath Management in SDN-Based Elastic Optical Networks With Power Consumption Considerations](https://www.researchgate.net/publication/321952636_Lightpath_Management_in_SDN-Based_Elastic_Optical_Networks_With_Power_Consumption_Considerations?enrichId=rgreq-9930aaf6f78f60cc156a0f4c0091e69c-XXX&enrichSource=Y292ZXJQYWdlOzMyMTk1MjYzNjtBUzo1NzQ2MjQxMjA0Nzk3NDRAMTUxNDAxMjQ3NDM2Ng%3D%3D&el=1_x_3&_esc=publicationCoverPdf) by Yu Xiong et al, which also mentions the modulation scheme 16-QAM with the bound 625 km. We do not consider this modulation scheme since this bound is too short for our needs. 

So a test case of the first kind is described by the network and modulation scheme, and a test case of the second kind also depends on a broken link. In each test cases we use the same modulation scheme for each demand, but our method allows consideration of demands with different modulation schemes as well. Even for the chosen approach we have  10 routing schemes and 223 test cases, so it is enough for our analysis.  Our approach of test generation allowed us to consider the following test cases of the first kind:

- **USNET**

  - **BPSK**
  - **QPSK**
  - **8-QAM**

- **NSFNET**
  - **BPSK**
  - **QPSK**

To generate a congested routing with many demands we firstly routed demands of width 1 and then demands for width 4, 2, and 1 as well. So all colors of some links can be occupied by the paths of width 1, and we do not consider the test cases where such links are broken since it is important for our method to have demands of width greater than 1. So, we consider a link break scenario only for links that have at least one demand of width greater than 1. Because of that and the small reachability threshold for 8-QAM modulation, we have only 2 test cases for the routing scheme for USNET-8-QAM. The number of colors in all networks is the same for each link and equals 80.

  

## Overview of the Results

We performed the experiments via two noncommercial MILP solvers: CBC and SCIP. 
Detailed results are available in the Jupyter notebook and the file `Detailed results'. 

### Number of variables in models

#### Maximal

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |  228000 |   124000 |     18600 |
| USNET-BPSK 23.0	  |       40 |  276000 |   148000 |     14200 |
| USNET-QPSK		   |       37 |  175000 |   115000 |      7590 |
| USNET-QPSK 23.0	  |       36 |  171000 |   110000 |      7590 |
| USNET-8-QAM		  |        2 |  228000 |   218000 |     16300 |
| USNET-8-QAM 31.0	 |        3 |  223000 |   207000 |     16300 |
| NSFNET-BPSK		  |       21 |  111000 |    64500 |     14200 |
| NSFNET-BPSK 5.0	  |       20 |  118000 |    66500 |      9870 |
| NSFNET-QPSK		  |       12 |   63800 |    50100 |      5170 |
| NSFNET-QPSK 19.0	 |       11 |   76800 |    58800 |      5170 |


#### Mean

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |  146000 |    76600 |      9020 |
| USNET-BPSK 23.0	  |       40 |  148000 |    75700 |      7100 |
| USNET-QPSK		   |       37 |   99500 |    63800 |      2800 |
| USNET-QPSK 23.0	  |       36 |   97300 |    62400 |      2700 |
| USNET-8-QAM		  |        2 |  228000 |   218000 |     13600 |
| USNET-8-QAM 31.0	 |        3 |  214000 |   202000 |      5440 |
| NSFNET-BPSK		  |       21 |   64000 |    34900 |      6970 |
| NSFNET-BPSK 5.0	  |       20 |   67700 |    34400 |      4120 |
| NSFNET-QPSK		  |       12 |   42600 |    32500 |      2300 |
| NSFNET-QPSK 19.0	 |       11 |   46500 |    34700 |      1790 |


#### Minimal

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |   60500 |    30600 |      1740 |
| USNET-BPSK 23.0	  |       40 |   59000 |    28900 |      1020 |
| USNET-QPSK		   |       37 |    6720 |     4180 |        68 |
| USNET-QPSK 23.0	  |       36 |    6560 |     4080 |       124 |
| USNET-8-QAM		  |        2 |  228000 |   218000 |     10900 |
| USNET-8-QAM 31.0	 |        3 |  210000 |   200000 |        0* |
| NSFNET-BPSK		  |       21 |   37000 |    18400 |      2480 |
| NSFNET-BPSK 5.0	  |       20 |   35200 |    16400 |       948 |
| NSFNET-QPSK		  |       12 |   26900 |    20200 |       942 |
| NSFNET-QPSK 19.0	 |       11 |   25600 |    18600 |        0* |

\* Note that it during the trimming it can be proved that the model is infeasible (in the case where there is a demand for which there is no path of the bounded length).

### Time of computation (in seconds) for CBC

For a few edges, model generation for the Basic and NoTrim models can exceed more than 100 seconds, and these computations are terminated.  The solver's time limit is set to 500 seconds. Thus, if the computation exceeds 500 seconds, it is not completed. 

We also provide the aggregated time metrics above with excluded hard test cases (that take over 100 seconds) in the Jupyter notebook and the file `Detailed results'. Note that the termination of the solver sometimes happens slowly and its runtime can take up to 700 seconds despite the 500 second limit. 


#### Maximal Time

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |   37.57 |    21.87 |      3.38 |
| USNET-BPSK 23.0	  |       40 |  526.03 |   519.38 |      1.9  |
| USNET-QPSK		   |       37 |   49.49 |    35.57 |      2.5  |
| USNET-QPSK 23.0	  |       36 |  111.83 |    41.35 |      3.32 |
| USNET-8-QAM		  |        2 |  806.94 |   540.09 |      2.09 |
| USNET-8-QAM 31.0	 |        3 |  560.44 |   501.64 |      2.4  |
| NSFNET-BPSK		  |       21 |   17.97 |     8.68 |      1.95 |
| NSFNET-BPSK 5.0	  |       20 |  172.71 |   323.98 |      2.06 |
| NSFNET-QPSK		  |       12 |  110.09 |   644.92 |      1.05 |
| NSFNET-QPSK 19.0	 |       11 |  108.15 |   515.37 |      1    |


#### Mean

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |   15.6  |     9.92 |      1.69 |
| USNET-BPSK 23.0	  |       40 |   48.77 |    33.29 |      0.78 |
| USNET-QPSK		   |       37 |   14.48 |    11.07 |      0.83 |
| USNET-QPSK 23.0	  |       36 |   19.6  |    11.61 |      0.84 |
| USNET-8-QAM		  |        2 |  670.17 |   526.72 |      1.96 |
| USNET-8-QAM 31.0	 |        3 |  259.8  |   193.16 |      1.11 |
| NSFNET-BPSK		  |       21 |    6.95 |     4.81 |      1.07 |
| NSFNET-BPSK 5.0	  |       20 |   16.4  |    19.34 |      0.54 |
| NSFNET-QPSK		  |       12 |   25.54 |    61.85 |      0.61 |
| NSFNET-QPSK 19.0	 |       11 |   14.84 |    51.92 |      0.41 |


#### Minimal

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |    5.18 |     3.47 |      0.48 |
| USNET-BPSK 23.0	  |       40 |    4.16 |     2.22 |      0.23 |
| USNET-QPSK		   |       37 |    0.63 |     0.5  |      0.12 |
| USNET-QPSK 23.0	  |       36 |    0.68 |     0.54 |      0.14 |
| USNET-8-QAM		  |        2 |  533.41 |   513.35 |      1.84 |
| USNET-8-QAM 31.0	 |        3 |   85.76 |    29.52 |      0.46 |
| NSFNET-BPSK		  |       21 |    3.17 |     1.53 |      0.39 |
| NSFNET-BPSK 5.0	  |       20 |    2.84 |     1.09 |      0.13 |
| NSFNET-QPSK		  |       12 |    4.24 |     2.7  |      0.34 |
| NSFNET-QPSK 19.0	 |       11 |    2.77 |     2.62 |      0.16 |

### Time of computation (in seconds) for SCIP

#### Maximal

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |  114.9  |   100.99 |      8.77 |
| USNET-BPSK 23.0	  |       40 |  485.98 |   505.77 |      5.02 |
| USNET-QPSK		   |       37 |   74.61 |    78.35 |      2.49 |
| USNET-QPSK 23.0	  |       36 |  122.59 |   116.76 |      2.28 |
| USNET-8-QAM		  |        2 |  657.28 |   780.99 |     19.17 |
| USNET-8-QAM 31.0	 |        3 |  748.57 |   745.98 |     24.68 |
| NSFNET-BPSK		  |       21 |  594.92 |    93.56 |      3.96 |
| NSFNET-BPSK 5.0	  |       20 |   58.68 |    45.77 |      1.88 |
| NSFNET-QPSK		  |       12 |  106.07 |    74.52 |      0.77 |
| NSFNET-QPSK 19.0	 |       11 |   73.69 |    60.37 |      0.75 |


#### Mean

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |   39.23 |    34.43 |      2.72 |
| USNET-BPSK 23.0	  |       40 |   51.36 |    47.23 |      1.36 |
| USNET-QPSK		   |       37 |   28.23 |    26.27 |      0.75 |
| USNET-QPSK 23.0	  |       36 |   31.07 |    28.53 |      0.68 |
| USNET-8-QAM		  |        2 |  592.51 |   658.69 |     17.85 |
| USNET-8-QAM 31.0	 |        3 |  566.57 |   361.64 |      9.2  |
| NSFNET-BPSK		  |       21 |  116.06 |    32.29 |      1.77 |
| NSFNET-BPSK 5.0	  |       20 |   13.64 |    11.24 |      0.44 |
| NSFNET-QPSK		  |       12 |   47.47 |    26.49 |      0.44 |
| NSFNET-QPSK 19.0	 |       11 |   29.46 |    25.59 |      0.33 |


#### Minimal

|                          |   #tests |   Basic |   NoTrim |   Trimmed |
|:-------------------------|---------:|--------:|---------:|----------:|
| USNET-BPSK		   |       41 |   10.12 |     6.64 |      0.39 |
| USNET-BPSK 23.0	  |       40 |    8.22 |     6.26 |      0.23 |
| USNET-QPSK		   |       37 |    1.26 |     0.77 |      0.09 |
| USNET-QPSK 23.0	  |       36 |    0.81 |     0.64 |      0.1  |
| USNET-8-QAM		  |        2 |  527.74 |   536.38 |     16.54 |
| USNET-8-QAM 31.0	 |        3 |  410.39 |    45.84 |      1.46 |
| NSFNET-BPSK		  |       21 |    6.71 |     5.19 |      0.38 |
| NSFNET-BPSK 5.0	  |       20 |    1.82 |     0.87 |      0.09 |
| NSFNET-QPSK		  |       12 |   15.32 |     7.02 |      0.24 |
| NSFNET-QPSK 19.0	 |       11 |    9.58 |     4.12 |      0.13 |

### Conclusion

The results show that our trimming technique significantly improves the size of the model and the runtime of the solver. Most test cases are solved using the CBC-solver in less than one second, and the hardest CBC test case takes a few seconds. The SCIP solver is slower on average, but it solves some test cases that are hard for the CBC-solver better in case of the Basic and NoTrim models. All in all, the Trimmed model has significantly less variables and can be solved by noncommercial solvers a hundred times faster than models without trimming. The NoTrim model is faster than the Basic model in many cases, but there are also test cases in which solvers solve the Basic model is faster and some of them are hard. So by the analysis of the NoTrim model we proved that the main impact of our model is based on the variables excluded via the trimming procedure, but not excluded because the corresponding colors have been occupied. Note that in some cases the infeasibility of the model can be proven during the trimming process and we met such cases in our dataset.
