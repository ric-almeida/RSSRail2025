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
- `-l` indicates the name of the output file where the CSL query results are stored

