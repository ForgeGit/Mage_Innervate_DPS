# **Innervate Returns on Arcane Mage DPS**
### _A Real-World Analysis of 833 Logs and 4,835 Encounters in TBC Classic Anniversary_

Simulations (_sims_) are useful but sometimes questioned [[1](https://discord.com/channels/253212375790911489/817495452328001536/1534925337581064302)] because the simplified scenarios they simulate (‘_Patchwerk fights_’) [[2](<https://github.com/ForgeGit/naxx_mage_movement>)] can differ substantially from real raid encounters, where mechanics (‘_don’t stand on fire_’) and player execution carry a significant role [[3](<https://github.com/ForgeGit/class_cope>)].

Innervate (spell ID: `29166`) is an instant-cast Druid skill with a 6-minute cooldown used to regenerate mana. When assigned to a DPS, Arcane Mages are commonly considered the best recipients [[4](https://www.wowhead.com/tbc/guide/classes/druid/feral/dps-rotation-cooldowns-abilities-pve),[5](https://www.wowhead.com/tbc/guide/classes/druid/feral/tank-rotation-cooldowns-abilities-pve)].  _Sims_ from both current and previous versions of _The Burning Crusade_ (TBC) suggest that giving Innervate to an Arcane Mage can increase their damage output by at least ~100 DPS [[6](https://discord.com/channels/253212375790911489/817495452328001536/837273535356862465),[7](https://discord.com/channels/253212375790911489/817495452328001536/884173843248316486),[8](https://discord.com/channels/253212375790911489/817495452328001536/1535312592111927396),[9](https://discord.com/channels/253212375790911489/812641242222821376/972068177964068875),[10](https://discord.com/channels/253212375790911489/812641242222821376/820088886171533370)].

Using real-world data (logs from _Warcraft Logs_), I estimated the effect of Innervate on Arcane Mage DPS across all bosses in Serpentshrine Cavern (SSC) and Tempest Keep (TK) during raid weeks 24-25 of TBC _Classic Anniversary - Phase 2_ (TBC Fresh). 

The analysis adjusts for player, raid, and encounter characteristics, and is designed to answer the question: 
> "For fights like those observed in SSC/TK, how much additional DPS would we expect if an Arcane Mage received exactly one Innervate instead of none?"

<img src="/Figure1.png" />

Between Jul 31, 2026 and Aug 07, 2026, the data included a total of 1,949 unique Arcane Mage players. On average, a 25-player raid contained 2.68 mages and 3.05 druids. Of 52,814 Innervate casts, 88.94%  (46,973) were used on Arcane Mages. 39.1% of all Mages received at least one Innervate.

Receiving exactly one Innervate instead of none was associated with an estimated average gain of +112.6 DPS (95% CI: 111.9–113.2). Importantly, the estimated benefit varied substantially with fight duration.

The largest predicted gains occurred in very short encounters (<120 seconds), reaching approximately +266 DPS at the peak (95% CI: 231.9–300.7). Across many mid-length fights, the estimated gain waved between +70 and +125 DPS.

The larger predicted DPS gains in very short encounters likely reflect not only the mana restored by Innervate (allowing for more Arcane Blasts), but also the choices that additional mana enables. In addition to Arcane Blast spam casting, additional mana allows Mages to use Destruction Potions, Molten Armor, and other raw DPS consumables instead of more mana-conservative alternatives.

<img src="/Figure2.png" />

Raids with shorter kill times were also more likely to give at least one innervate to an Arcane Mage. This relationship is important because Innervate assignment is not random. Better-geared or better-performing raids tend to kill bosses faster and may also use available raid resources more deliberately. As raids gain gear and become more comfortable with encounter mechanics, they may also have less reason to reserve Innervate for healers or other defensive purposes (e.g. battle rez'd teammate)

These results should also not be interpreted in a vacuum. Giving an Innervate to an AFK Mage will effectively provide [0 DPS](https://fresh.warcraftlogs.com/reports/a:aYCcnqmZJD674vjK?fight=25&pins=0%24Main%24%23244F4B%24auras-cast%24-4%240.0.0.Any%240.0.0.Any%24true%2446980252.0.0.Mage%24true%2429166%2432&source=40). Likewise, a high-performing Arcane Mage may not be able to convert the full theoretical value of an Innervate into damage in every encounter. Fight duration, mechanics, movement, player execution, and the performance of the rest of the raid all influence how useful that additional mana will be in practice.

## Methods

### - Data Cleaning (Valid Logs)
Warcraft Logs reports from Serpentshrine Cavern (SSC) and Tempest Keep (TK) were filtered to eliminate duplicate logs and invalid non-SSC/TK encounters. Wipes and mages that died during the encounter were excluded to isolate full-encounter performance, resulting in XX,XXX total Mage-encounter observations.

<img src="/owo.png" />

### - Exposure Definition (Valid Innervates)
An Innervate was considered valid if it was cast before or during the encounter and at least 20 seconds before the encounter ended. Innervates were classified into three categories (0, 1, or 2+).

### - Statistical & Causal Modeling
We estimated the causal effect of Innervate on Arcane Mage DPS using a Generalized Additive Model (GAM) fitted via mgcv::bam(). The model adjusted for variables that could act as confounders:

- <ins>**Encounter & Raid Factors**</ins>: Boss ID; fight duration (seconds); # of Druids in raid.
- <ins>**Player Factors**</ins>: Difference between Mage-ilvl) and Raid-ilvl; Individual Performance(modeled via player-level as random effects).
- <ins>**External Buffs**</ins>: Number of `Power Infusions` received and uptime on `Moonkin Aura`
- <ins>**Exposure Dynamics**</ins>: The effect of Innervate was modeled as a smooth interaction with fight duration to capture non-linear DPS returns across varying encounter lengths.

These covariates were selected according to their plausible causal role, rather than statistical significance. Nevertheless, all except XX were statistically significant. 

$$
\mathrm{DPS}_i = \beta_0 + \beta_1 \mathrm{Innervate}_i + \boldsymbol{\beta X}_i + u_{\mathrm{Mage}[i]} + f_{\mathrm{Boss}[i]}(\mathrm{Duration}_i) + g(\mathrm{Duration}_i)\,\mathrm{Innervate}_i + \varepsilon_i
$$

Here, *i* represents one Mage–encounter observation. **β₀** is the baseline DPS; **β₁** is the average shift associated with receiving one Innervate; and **βXᵢ** represents the measured adjustment variables (gear difference, number of Druids, Power Infusion, uptime, and boss).

**u<sub>Mage[i]</sub>** accounts for persistent differences between individual Mages, **f<sub>Boss[i]</sub>(Durationᵢ)** allows the relationship between fight duration and DPS to differ by boss, and **g(Durationᵢ) × Innervateᵢ** allows the additional DPS from Innervate to change with fight duration. **εᵢ** is the remaining unexplained variation in DPS.

### - Additional Variable Exploration
 
- `Berserking` was evaluated but excluded from the final model as it did not meaningfully improve model fit or change the effect estimate.
- `Destruction Potion` use was excluded in the model because it may be a mechanism through which additional mana is converted into DPS (it is a result of). When tested as a sensitivity variable, it did not meaningfully improve model fit or change the effect estimate due to low number of observations.
- Similarly, major sources of mana gains such as `Vampiric Touch` and `Judgment of Wisdom`, were treated primarily as sensitivity variables because they are part of the causal pathway that generates additional DPS with Innervate (more mana gains means proportionally more AB casts, which in turns will lead to more hits on target and more DPS).

The model here explains ~93.6% of deviance in the observed data.

### - Estimation
From our model estimation counterfactual DPS outcomes were predicted for every encounter under two scenario conditions:

- (A) $Y^{(1)}$: Exactly one valid Innervate received.
- (B) $Y^{(0)}$: Zero Innervates received.

All other measured covariates were held constant. 

The individual "exposure to Innervate" effect was calculated as $Y^{(1)} - Y^{(0)}$ or in other words "Scenario A minus Scenario B".

To account for within-player correlation across multiple boss encounters and raid weeks, 95% confidence intervals were constructed using 1,000 clustered bootstrap iterations resampled at the player level.

## Assumptions and limitations

Results generalize strictly to completed boss kills where the Mage survived the entire encounter. Hypotethically, if an Innervate makes a really greedy Made play more risky, and in turns leads to that mage dying, that isn't accounted for.

Innervates casts differ in timing and are not really assigned at random. So even though we tried to account for player random effects in our model, as well as introducing additional uncertainty propagation with clustered bootstraping, there will always be some unemasured coordination that conditions that Innervate assignment. 

Results could be a bit conservative, due to the assumption that fight duration does not "significantly decreases" when an Innervate is given to a Mage. In reality, the additional DPS will shorten the fight lenght, indirectly increasing DPS.  

Finally, the analysis estimates the DPS benefit to the mage receiving the Innervate. Innervate is a limited raid resource, and assigning it to one player may prevent it from being assigned to another Mage, DPS player, or healer. The estimated DPS gain therefore should not be interpreted directly as the whole raid DPS gain or as evidence that a particular Innervate allocation is optimal for every raid.
