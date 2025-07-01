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
- For `Rail_BRS.big`, the system is generated running `bigrapher full --solver=GBS -f svg -t Rail_BRS -s Rail_BRS.big`
- For `Rail_PBRS.big`, the Discrete-Time Markov Chain (DTMC) is generated running `bigrapher full --solver=GBS -f svg -t Rail_PBRS -s Rail_PBRS.big -M 3000`
- For `Rail_ABRS.big`, the Markov Decision Process (MDP) is generated with `bigrapher full --solver=GBS -f svg -t Rail_ABRS -s Rail_ABRS.big`
 
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

For the analysis of the ABRS model, first we fix the query we wish to run. We then uncomment the predicates corresponding to the query in `Rail_ABRS.big`. We also uncomment the reward-based query for which the experiment is to be run. Next, we generate the transition matrix and the predicate information as above using:

`bigrapher full --solver=GBS -p Rail_ABRS.tra -l Rail_ABRS.csl -r Rail_ABRS.srew Rail_ABRS.big -M 3000`

Next, we augment the `.csl` file with the PCTL queries (`/ABRS_Analysis/Queries.props`) that correspond to the plots in the paper. The modifed `.csl` file can now serve as input to the PRISM model checker with the `.tra` file to model check and give the results:

`prism -importtrans Rail_ABRS.tra -importstaterewards Rail_ABRS.srew Rail_ABRS.csl -mdp -const j=0:1:45 -exportresults Rail_ABRS.txt:csv`

This will store the results corresponding to the experiments run while varying the variable $t$ in the PCTL formula from $0$ to $45$ at an interval of $1$ in the `csv` format. The Jupyter notebook can be used in the end to generate the plots presented in the paper when all the queries have been successfully run and stored.

We have already generated and stored all the relevant files in the subdirectories inside `ABRS_Analysis`. The plots are saved as well.


