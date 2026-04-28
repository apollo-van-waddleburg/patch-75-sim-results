# Patch 7.5 Simulation Results (BIS)

This repo contains generated results for target dummy rotations. We used 12:30 rotations in a full party, single-target
scenario
to generate expected values for every hit and derive a true expected dps for each comp. This value is not impacted
by crit-rng (like logs are), so it represents a more precise measurement of the rotation.

Additionally, for each comp we ran 10000 simulations to build up statistical metrics like variance to help visualize
the lower and upper bounds of each comp. These 10000 simulations are also used to handle jobs with random rotations;
we have solvers for the MNK/BRD/DNC random elements (RDM we treated as static), which should normalize the effects of
rng on these jobs as well.

### Disclaimer

**You should not use the results of these tools to discriminate against any jobs in a public setting.**

## Contents and Methodology

We experimented with simulating jobs in 8-player comp situations to assess the strength of particular jobs in a target
dummy situation against their peers. To limit variables, we started with three baseline comps:

* `DRK-GNB-AST-SCH-MNK-NIN-BRD-PCT` (Cards on NIN, PCT)
    * This is intended to be the "meta" raid-buff comp.
* `DRK-GNB-WHM-SCH-MNK-NIN-BRD-PCT`
    * This is a raid buff comp without cards. I use this when the targets for cards might vary to isolate card impact.
* `PLD-WAR-SGE-WHM-SAM-VPR-MCH-BLM`
    * This comp has tests for performance under a low-to-no raid buff situation.

From these comps, we focus on specific roles by testing all **unique permutations without duplicates**. Each comp is
labeled with the 1-2 jobs it is specifically testing. All permutations are simulated against each other and plotted in
the same graph for easy side-by-side comparison.

Below are the links to all generated results, but I will highlight important ones later in the doc.

| Comp                                   | Link                                                                                                                       | Comment                                |
|:---------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:---------------------------------------|
| `(Tanks)-AST-SCH-MNK-NIN-BRD-PCT`      | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/TankBuffCompCompare)      |                                        |
| `(Tanks)-SGE-WHM-SAM-VPR-MCH-BLM`      | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/TankNoBuffCompCompare)    |                                        |
| `DRK-GNB-(Healers)-MNK-NIN-BRD-PCT`    | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/HealerBuffCompCompare)    |                                        |
| `PLD-WAR-(Healers)-SAM-VPR-MCH-BLM`    | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/HealerNoBuffCompCompare)  |                                        |
| `DRK-GNB-WHM-SCH-(Melees)-BRD-PCT`     | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/MeleeBuffCompCompare)     | AST excluded to remove carding impact. |
| `PLD-WAR-SGE-WHM-(Melees)-MCH-BLM`     | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/MeleeNoBuffCompCompare)   |                                        |
| `DRK-GNB-AST-SCH-SAM-NIN-(Ranged)-PCT` | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/RangedBuffCompCompare)    |                                        |
| `PLD-WAR-SGE-WHM-SAM-VPR-(Ranged)-BLM` | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/RangedNoBuffCompCompare)  |                                        |
| `DRK-GNB-WHM-SCH-MNK-NIN-BRD-(Caster)` | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterBuffCompCompare)    |                                        |
| `DRK-GNB-AST-SCH-MNK-NIN-BRD-(Caster)` | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterAstBuffCompCompare) | Ranged cards were used on the caster.  |
| `PLD-WAR-SGE-WHM-SAM-VPR-MCH-(Caster)` | [results folder](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterNoBuffCompCompare)  |                                        |

### Rotations

All rotations are hand-sheeted `12:30` rotations, using pots in the opener, 6 minute, and 12 minute windows.
All rotations attempted to align with a ~6s buff opener. Regarding GCD tiers, we picked a commonly recommended
tier for each job.

As mentioned before, MNK/BRD/DNC have some random elements
in the simulation:

* MNK: Chakra is randomly simulated based on game values. TFC's are used whenever possible with no holding.
* BRD:
    * The sim uses Heartbreak Shot and Pitch Perfect in open weaves based on randomized procs.
        * Raging windows are used to determine when to hold HB shot: The sim will start holding them if it would not
          lose a usage before the next window.
    * Burst Shot and Refulgent Arrow are used interchangeably based on procs.
    * Apex Arrow has its damage simulated based on simulated gauge procs. Blast Arrow is skipped for a Burst/Refulgent
      if requirements are not met.
        * In practice, a player would work around this. We allow the sim to look at the sum of gauge generated instead
          of hard resets to avoid having to move Apex from intended usage times.
    * Dot refreshes are not moved: if this leads to an overcap for Refulgent Arrow, the sim makes an exception and
      allows 2 usages back to back.
        * This is to simplify how a real player would refresh early to avoid this situation.
    * Army's Paeon GCD effects are ignored: we sheeted the gcd increase manually and consistently.
    * All other abilities are not moved or overwritten by the solver.
* DNC:
    * The sim replaces almost all gcds and ogcds following the generally recommended priority system.
    * Dance timings, Standard refreshes, and Flourish/Devilment usages are never moved.

<figure style="flex: 1; text-align: center;">
    <img src="https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/figures/dnc-rng.png?raw=true" style="width: 100%;">
    <figcaption>DNC Solver Output</figcaption>
  </figure>

MNK and DNC also scan other rotations in the comp to simulate the chakra/esprit generation from the party.

We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out
blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific
optimizations
in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us
to review and resimulate. If the outcome is significantly different (100s of dps), we will post revised results.

### Important Simulations

There are many generated outputs and plenty of them might not be very insightful, so I'll outline the most important
ones.

* `rotations`: These are generated damage outputs from the actual rotations and are the base data for the visualizations
  here.
    * These are included specifically to help validation of this work over time.
    * This is also where you will notice the simulated probabilities of random events for MNK/BRD/DNC.
* `comp_dps`: This tracks simulated percentiles for all the comps in terms of DPS @ `12:27.500`
    * By percentiles, it's the statistics version: `x%` means "`x%` of data is less than or equal to this value."
    * This is computed via two different methods
        * `expected +- standard deviation` (`comp_dps_standard_percentiles`)
        * Sampled data percentiles (`comp_dps_sampled_percentiles`)
    * `comp_dps_sampled_percentiles` is what we will use for analysis. It will deviate from pure expected value due to
      random simulations, but it makes no assumptions on the distribution of the underlying data.
* `comp_dps_over_time`: Tracks expected full comp dps for every damage application, which can be used to see specific
  spikes in damage by comp.

### Files and Outputs

* [configs](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/configs) contains the original csvs
  and 8-player comp configurations for generating these outputs.
* [outputs](https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs) contains simulated results
  for all of these comps in `csv` and `html` formats.
    * `https://htmlpreview.github.io/?` can be used to view `html` files without any other tools by appending the full
      url.
    *
    Example: [https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps_over_time/comp_dps_over_time.html](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps_over_time/comp_dps_over_time.html)

## Outcomes

These are initial reactions based on the `comp_dps` graphs generated by our sim. You can also draw similar conclusions
by looking at the `comp_dps_over_time` version if you only care about expected value.

| Name                       | Comment                                                                                                     | Link                                                                                                                                                                                       |
|:---------------------------|:------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Buff Comp Dps (Tanks)      | DRK comps are the strongest, but now WAR is the best partner for it, overtaking GNB.                        | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/TankBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)      |
| No Buff Comp Dps (Tanks)   | WAR and GNB seem relatively good here, with DRK suffering in a low buff environment.                        | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/TankNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| Buff Comp Dps (Healers)    | Buff-using healers seem very strong in this experiment, AST/SCH gaining over `2000 DPS` against WHM/SGE.    | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/HealerBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Healers) | SGE/SCH seems to actually be a narrow winner here. The SGE buffs are massive, so this makes sense.          | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/HealerNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |
| Buff Comp Dps (Melees)     | The new buffs put RPR/NIN as the new top duo, while also reducing the overall spread.                       | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/MeleeBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)     |
| No Buff Comp Dps (Melees)  | RPR/MNK is the winner, with RPR still being the strongest here.                                             | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/MeleeNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)   |
| Buff Comp Dps (Ranged)     | BRD is the strongest in this experiment, even with using SAM for DNC Partner. MCH is down about `3000 DPS`. | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/RangedBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Ranged)  | BRD is still a clear winner, but by a smaller margin (`~1200 DPS`). DNC is very slightly above MCH.         | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/RangedNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |
| AST Buff Comp Dps (Caster) | PCT seems to win by a narrow margin. All casters (including SMN!!!) are within a ~1000 DPS spread.          | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html) |
| Buff Comp Dps (Caster)     | All casters are in a very tight spread, with RDM being a slight favorite.                                   | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Caster)  | BLM takes a dominant lead in this scenario, cementing a niche in low-buff comps.                            | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-75-sim-results/blob/main/outputs/CasterNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |

### 7.4 Retrospective & 7.5 Predictions

For 7.4 ([link](https://apollo-van-waddleburg.github.io/patch-74-sim-results/)), I focused more on general strength of jobs
relative to their roles. This time around, I would like to review some of the 7.4 outcomes and speculate a little 
on what might be the most interesting jobs in the new ultimate.

The jobs I would like to talk about the most are RPR and RDM. These jobs have both been overshadowed in peak output
by other jobs like MNK/NIN/PCT, but the buffs to these jobs have put them in a comparable spot while also giving both of
these jobs a niche: resource pooling and multi-target cleave.

Top speedkill logs this tier utilized both: M9S was a fight defined by how teams navigated the tower add phase. 
The top 2 times took a "non-meta" job to solve the adds phase with on-demand cleave.
* [6:58](https://www.fflogs.com/reports/AqHm26khrWJRCvzx?fight=1&type=damage-done) w/ RDM ([VOD](https://www.fflogs.com/video/view/73/fight/147666431))
* [7:05](https://www.fflogs.com/reports/6pTWDkfYgPKQ4tBq?fight=20&type=damage-done) w/ RPR ([VODS](https://www.fflogs.com/video/view/73/fight/147342228))

The idea here is simple: both RDM and RPR supply on-demand burst and cleave, which can be very helpful when every 
other job might be lacking free cleave from their CDs. This lets everyone else stay focused on the boss and minimizes
boss damage losses. 

M10S saw the top team bring out a double caster comp, replacing a melee with RDM. RDM is shocking powerful with cleave, 
enough so that it was worth dropping a whole other melee for. 
* [7:07](https://www.fflogs.com/reports/28kP43gqW9RaLCpT?fight=54&type=damage-done) w/ RDM/PCT Double caster ([VODs](https://www.fflogs.com/video/view/73/fight/147613054))

Regarding the other melees, I can also provide some context here since I was working with parryprog for their theorycraft: 
they checked the other melees and found them all to be roughly comparable (NIN/MNK/DRG/RPR) in net outcomes for their goal. 
So, NIN was taken due to time constraints since it was a comfort pick. 

M12SP2 had a korean team do some work with carryover RPR, using the P1 fight to bring a full gauge. This is a known 
strength of RPR, so it's nice to see a team invest the effort in executing runs like this.
* [7:40](https://www.fflogs.com/reports/KjDrwdWLPGxHCv1q?fight=2&type=damage-done) w/ Carryover RPR ([VOD](https://www.fflogs.com/video/view/73/fight/147655352))

Moving onto 7.5, we see some well reasoned buffs to some underperformers (idk why RPR was buffed and not MCH though). 
However, the new ultimate can always pose a challenge: RPR/RDM can struggle when it can't generate gauge, and ultimates
always mix in downtimes that allow other jobs like NIN to wait out their CDs while losing less relative to more 
gauge dependent jobs. PCT is the most extreme example of a job that can get a lot of work down during downtime (via painting), 
so our new standouts have the odds stacked against them. 

Gauge-dependence can be an asset though: if your group can find ways to allow these jobs to pool gauge, you can find a 
massive amount of extra damage. Not every phase in ultimate is necessarily a tight check, so a clever group might be 
able to make this work. We also might see two-target or add phases as well, which can allow RDM/RPR to shine. RPR in 
particular is unique because it generates extra gauge when something dies under its Death's Design, allowing you to cheat 
out a lot of extra damage.

## Disclaimers

### Bias

These simulations bias towards buff-stacking comps. While I do believe the strongest comp will heavily utilize
raid-buffs, the reality for most people in most situations is that people will fail to play around raid-buffs well.
This failure to perform optimally is not something we try to emulate with these simulations, so it's hard to know
exactly
how these results translate to a real world scenario. As such, I wouldn't overthink the results here regarding specific
underperformers and I encourage you to consider the context of your own situation when evaluating this data.

### Mistakes

To reiterate:
> We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out
> blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific
> optimizations
> in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us
> to review and resimulate. If the outcome is significantly different (100s of dps), we will post revised results.

Additionally, we accept there might be bugs in the underlying sim code and will be on constant lookout for them.
We do verify our expected damage against gearset-annotated logs to make sure we accurately simulate the correct ranges.
We also verified against the generated rotations in xivgear.app for line-by-line expected value
(we consistently were within 1-2 damage per line).

All that said, I personally am not an expert on every job and there are certainly... decisions... you might find in some
sheets.
I also coded the solvers for RNG jobs, so there's definitely the possibility of mistakes.

### Discriminatory Use

> **You should not use the results of these tools to discriminate against any jobs in a public setting.**

These results come from an idealistic scenario and are biased towards jobs that play into a raid-buff meta.
Some of the jobs that are under-performing in these simulations fare better in more real-world situations since they
are less dependent on their team and can have more consistent output. The goal of this assessment is to provide a
new perspective on job performance that is not feasible to see via submitted logs.

## Credits

* **Ama** for providing an open-source simulation tool in python, which was used extensively for this project
    * [Pip](https://pypi.org/project/ama-xiv-combat-sim/#description)
    * [xivraider](https://xivraider.com/)
* **Shanzhe** and the supporting team for [xivintheshell](https://xivintheshell.com/) for a public rotation tool.
* [xivgear.app](https://xivgear.app/) for their gearset support and damage validation.
* [Caro](https://www.youtube.com/@CaroKannffxiv) and [Parryprog](https://www.fflogs.com/guild/id/135994/) for writing the vast majority of the rotations and
  reviewing work.
    * Checkout their speedkills in 7.4! They did a great job.
* **Sleepocat** for being the original testers for many of these simulations and visualizations.
    * For context, I played as a Tank for the team and managed a lot of the simulations, tools, and planning around
      their runs.
* Revisions
    * **Siora** (2025/01/05): Corrected DRG sheet.

### P.S

If you are interested in speeds or spreadsheet theorycraft, please reach out to me on discord (apollo.van.waddleburg).
We are slowly rolling out many of the tools we used to generate this report to interested parties and would love
to develop a community around damage simulation and speed kills.