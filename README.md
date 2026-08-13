# Mage_Innervate_DPS

## Methods

### Data Cleaning (Valid Logs)
Warcraft Logs reports from Serpentshrine Cavern (SSC) and Tempest Keep (TK) were filtered to eliminate duplicate logs and invalid non-SSC/TK encounters. Wipes and mages that died during the encounter were excluded to isolate full-encounter performance, resulting in XX,XXX total Mage-encounter observations.

<img src="/owo.png" />

### Exposure Definition (Valid Innervates)
An Innervate was considered valid if it was cast before or during the encounter and at least 20 seconds before the encounter ended. Innervates were classified into three categories (0, 1, or 2+).

### Statistical & Causal Modeling
We estimated the causal effect of Innervate on Arcane Mage DPS using a Generalized Additive Model (GAM) fitted via mgcv::bam(). The model considered adjusted for variables that could act as confounders:

- Encounter & Raid Factors: Boss ID; fight duration (seconds); # of Druids in raid.
- Player Factors: Difference between Mage-ilvl) and Raid-ilvl; Individual Performance(modeled via player-level as random effects).
- External Buffs: Number of Power Infusions received and uptime on Moonkin Aura
- Exposure Dynamics: The effect of Innervate was modeled as a smooth interaction with fight duration to capture non-linear DPS returns across varying encounter lengths.

These covariates were selected according to their plausible causal role, rather than statistical significance. Nevertheless, all except XX were statistically significant. 

<img src="/CodeCogsEqn.png" />

- Berserking was evaluated but excluded from the final model as it did not meaningfully improve model fit or change the effect estimate.
- Destruction Potion use was excluded in the model because it may be a mechanism through which additional mana is converted into DPS (it is a result of). When tested as a sensitivity variable, it did not meaningfully improve model fit or change the effect estimate due to low number of observations.
- Similarly, major sources of mana gains such as Vampiric Touch and Judgment of Wisdom, were treated primarily as sensitivity variables because they are part of the causal pathway that generates additional DPS with Innervate (more mana gains means proportionally more AB casts, which in turns will lead to more hits on target and more DPS).

## Estimations
From our model estimation, counterfactual DPS outcomes were predicted for every encounter under two scenario conditions:

- (A) $Y^{(1)}$: Exactly one valid Innervate received.
- (B) $Y^{(0)}$: Zero Innervates received.

All other measured covariates were held constant. 

The individual treatment effect was calculated as $Y^{(1)} - Y^{(0)}$ or in other words "Scenario A minus Scenario B".

To account for within-player correlation across multiple boss encounters and raid weeks, 95% confidence intervals were constructed using 1,000 clustered bootstrap iterations resampled at the player level.

## Assumptions and limitations

Results generalize strictly to completed boss kills where the Mage survived the entire encounter. Hypotethically, if an Innervate makes a really greedy Made play more risky, and in turns leads to that mage dying, that isn't accounted for.

Innervates casts differ in timing and are not really assigned at random. So even though we tried to account for player random effects in our model, as well as introducing additional uncertainty propagation with clustered bootstraping, there will always be some unemasured coordination that conditions that Innervate assignment. 

Results could be a bit conservative, due to the assumption that fight duration does not "significantly decreases" when an Innervate is given to a Mage. In reality, the additional DPS will shorten the fight lenght, indirectly increasing DPS.  

Finally, the analysis estimates the DPS benefit to the mage receiving the Innervate. Innervate is a limited raid resource, and assigning it to one player may prevent it from being assigned to another Mage, DPS player, or healer. The estimated DPS gain therefore should not be interpreted directly as the whole raid DPS gain or as evidence that a particular Innervate allocation is optimal for every raid.
