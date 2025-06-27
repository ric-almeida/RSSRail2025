# RSSRail2025

The full transition systems can be generated for visualization purposes from each of the BRS files with the following commands:

bigrapher full --solver=GBS -f svg -t Rail_BRS -s -l Rail_BRS.csl Rail_BRS.big

bigrapher full --solver=GBS -f svg -t Rail_PBRS -s -l Rail_PBRS.csl Rail_PBRS.big  -M 3000

bigrapher full --solver=GBS -f svg -t Rail_ABRS -s -l Rail_ABRS.csl Rail_ABRS.big
 
The options used do as follows:
- `full` indicates we want the full transition system (and not just a trace, as with `sim`) 
- `--solver=GBS` indicates the subgraph isomorphism solver should be used when looking for rewrite matches
- `-f svg` indicates the output file format should be .svg
- `-t` gives the name of the output file with the full transition system (e.g., Rail_BRS.svg)
- `-s` asks for an .svg to be generated for each bigraphical state (to allow visually inspecting the contents of individual states)
- `-l` indicates the name of the output file where the CSL query results are stored
- `-M 3000` increases the state-space limit from 1,000 to 3,000
