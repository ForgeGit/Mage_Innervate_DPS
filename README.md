# Mage_Innervate_DPS

## Methods

Warcraft Logs reports were filtered to remove duplicate logs and encounters outside Serpentshrine Cavern (SSC) and Tempest Keep (TK). 
Wipes were excluded, as were Mage encounters in which the player died before the end of the fight. 
The resulting estimand therefore applies to Arcane Mages surviving successful SSC/TK boss kills. 

Each Arcane Mage × boss encounter was treated as one observation, resulting in XX,XXX observations from 1,949 unique players.

An Innervate was considered valid if it was cast before or during the encounter and at least 20 seconds before the encounter ended.

<img src="/owo.png" />

Arcane Mage DPS was modeled using a Generalized Additive Model (GAM) fitted with bam(). 
The model included boss effects, boss-specific nonlinear relationships between fight duration and DPS, persistent differences between individual Mages, and measured player- and raid-level characteristics that could plausibly influence both Innervate allocation and DPS. 
The effect of one Innervate was allowed to vary nonlinearly with fight duration.

Covariates for the primary model were selected according to their plausible causal role rather than statistical significance. 
Variables measured before or independently of Innervate assignment, such as player gear, raid composition, number of Druids, and relevant external-buff availability, were considered potential confounders. 

Other major sources of mana gains, such as Vampiric Touch and Judgment of Wisdom, were treated primarily as sensitivity variables because they occur during the encounter and may partly reflect player behavior or other processes downstream of treatment. 
Similarly, variables such as Destruction Potion use were not included in the primary adjustment set because they may represent mechanisms through which additional mana is converted into DPS.

<img src="/CodeCogsEqn.png" />

After fitting this model, DPS was predicted for every encounter under two scenarios:

(A) With (one) Innervate
(B) Without Innervate

All other measured variables were held constant.
The difference between the predictions of the two scenarios ("With Innervate" - "Without Innervate") was used to estimate the DPS gain from Innervate across different fight lengths and boss encounters.

Statistical uncertainty was estimated using a clustered bootstrap in which players, rather than individual encounters, were repeatedly resampled and the complete GAM was refitted for each bootstrap sample. 
This preserves within-player dependence across repeated boss encounters. 
The primary analysis used 1,000 bootstrap samples. 
Because players from the same raid may also share strategies, kill times, and resource-allocation practices, alternative raid-level or multi-level clustering specifications can be considered as sensitivity analyses.


## Assumptions and limitations

owo
