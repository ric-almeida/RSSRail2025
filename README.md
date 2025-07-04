# RSSRail2025

We present three Bigraphical Reactive Systems (BRS) for railway models:
- `Rail_BRS.big`: a BRS that models a railway scenario with train segments (some of which electrified) and battery trains that complete routes
- `Rail_PBRS.big`: a Probabilistic Bigraphical Reactive System (PBRS) that extends the BRS with the probabilistic event of electrification failing in an electrified track segment
- `Rail_ABRS.big`: an Action Bigraphical Reactive System (ABRS) that extends the BRS with an initial non-deterministic choice that considers all possible eletrification configurations

## Dependencies 

Install `BigraphER` by cloning the repository https://bitbucket.org/uog-bigraph/bigraph-tools/src/master and following the instructions.

For the analysis, please install PRISM: https://www.prismmodelchecker.org/

## Generating transition systems for visualization purposes

The full transition systems can be generated for visualization purposes from each of the BRS files using `BigraphER`:
- For `Rail_BRS.big`, the system is generated running `bigrapher full --solver=GBS -f svg -t Rail_BRS Rail_BRS.big -s`
- For `Rail_PBRS.big`, the Discrete-Time Markov Chain (DTMC) is generated running `bigrapher full --solver=GBS -f svg -t Rail_PBRS Rail_PBRS.big -M 3000 -s`
- For `Rail_ABRS.big`, the Markov Decision Process (MDP) is generated with `bigrapher full --solver=GBS -f svg -t Rail_ABRS Rail_ABRS.big -s`
 
In all cases, the options do as follows:
- `full` indicates we want the full transition system (and not just a trace, as with `sim`) 
- `--solver=GBS` indicates the subgraph isomorphism solver should be used when looking for rewrite matches
- `-f svg` indicates the output file format should be .svg
- `-t` gives the name of the output file with the full transition system (e.g., `Rail_BRS.svg`)
- `-s` asks for an .svg to be generated for each bigraphical state (to allow visually inspecting the contents of individual states)
- `-M 3000` increases the state-space limit from 1,000 to 3,000

## Generating transition systems for analysis purposes

To prepare the inputs for the model analysis, we generate the transition systems in PRISM format as follows:

- For `Rail_BRS.big`: `bigrapher full --solver=GBS -p Rail_BRS.tra -l Rail_BRS.csl Rail_BRS.big` will generate `Rail_BRS.tra`
- For `Rail_PBRS.big`: `bigrapher full --solver=GBS -p Rail_PBRS.tra -l Rail_PBRS.csl Rail_PBRS.big -M 3000`
- For `Rail_ABRS.big`: `bigrapher full --solver=GBS -p Rail_ABRS.tra -l Rail_ABRS.csl Rail_ABRS.big` 
where:
- the `.tra` files will be the PRISM input files
- `-l` indicates the name of the output file where the states satisfying the predicates are stored.

## PBRS Analysis

The output files of this analysis can be found under `PBRS_Analysis`. Below we describe the process of generating them, in particular `Rail_PBRS_analysis.txt`, that populates `Table-1` in the paper.

We start by augmenting the `.csl` file with the PCTL queries that correspond to the headers of `Table-1` (eg, do `cat PBRS_Analysis/Queries.props >> Rail_PBRS.csl`). The modifed file can now serve as input to the PRISM model checker with the `.tra` file to model check and produce the results:

`prism -importtrans Rail_PBRS.tra Rail_PBRS.csl -dtmc -exportresults Rail_PBRS_analysis.txt`

This saves the output results in `Rail_PBRS_analysis.txt`, and these can be read as explained below. 
```
P=? [ F "charge_fail[n]_[l]" ]:
Result
[p]
```
means that the probability of a train reaching [b]% battery level given [n] track failures is [p].


## ABRS Analysis

### Track usage over time

Results on the total numbers of times a track has been used over time (shown on `Fig. 8`) must be generated per track. First, uncomment in `Rail_ABRS.big` the line with the predicate corresponding to the track to analyse (eg, `#, tr_use1("T1")[1],`).

Next, we generate the transition matrix and the predicate information as above using (eg for T1):

`bigrapher full --solver=GBS -p Rail_ABRS_1_T1.tra -l Rail_ABRS_1_T1.csl -r Rail_ABRS_1_T1.srew Rail_ABRS.big -M 3000`,
where `-r` stores the state rewards in `Rail_ABRS_1_T1.srew`.

Then `cat ABRS_Analysis/Track_Usage/Queries.props >> Rail_ABRS_1_T1.csl`

Next we run `prism -importtrans Rail_ABRS_1_T1.tra -importstaterewards Rail_ABRS_1_T1.srew Rail_ABRS_1_T1.csl -mdp -const j=0:1:45 -exportresults Rail_ABRS_1_T1.csv:csv`

This will store the results corresponding to the experiments run while varying the variable $t$ in the PCTL formula from $0$ to $45$ at an interval of $1$ in the `csv` format. Once results have been generated and stored for the 11 tracks, the Jupyter notebook can be used (to be updated) in the end to generate the plots presented in the paper.

We have already generated and stored all the relevant files in the subdirectories inside `ABRS_Analysis`. The plots are saved as well.

### Battery levels per track over time 

Here we describe how to generate results on the variation of trains' charge levels per track over time when tracks 1,2,6,9,10,11 are electrified (`Fig 9a`) and when none were electrified (`Fig 9b`).

The results need to be generated for each track individually, so we start my uncommenting the line in `Rail_ABRS.big` containing the predicates corresponding to a track to analyse (eg, `#, tr_use("T1", 5)[5], tr_use("T1", 10)[10], ..., tr_use("T1", 100)[100] `).

Next, we generate the transition matrix and the predicate information as above using (eg for T1):

`bigrapher full --solver=GBS -p Rail_ABRS_2_T1.tra -l Rail_ABRS_2_T1.csl -r Rail_ABRS_2_T1.srew Rail_ABRS.big -M 3000`.

Then the results must be generated separated for the two electrification scenarions, so we start by copying :
`mv Rail_ABRS_2_T1.csl Rail_ABRS_2a_T1.csl` and `cp Rail_ABRS_2b_T1.csl`.

Then `cat ABRS_Analysis/Track_Battery_Levels/Query_T1T2T6T9T10T11_electr.props >> Rail_ABRS_2a_T1.csl`
and `cat ABRS_Analysis/Track_Battery_Levels/Query_no_electr.props >> Rail_ABRS_2b_T1.csl`.

Next we run `prism -importtrans Rail_ABRS_2_T1.tra -importstaterewards Rail_ABRS_2_T1.srew Rail_ABRS_2a_T1.csl -mdp -const j=0:1:45 -exportresults Rail_ABRS_2a_T1.csv:csv`
and `prism -importtrans Rail_ABRS_2_T1.tra -importstaterewards Rail_ABRS_2_T1.srew Rail_ABRS_2b_T1.csl -mdp -const j=0:1:45 -exportresults Rail_ABRS_2b_T1.csv:csv`.