# **Innervate Returns on Arcane Mage DPS**
### _A Real-World Analysis of 1,650 Logs containing 9,501 Encounters in TBC Classic Anniversary_

_Vivax (Pagle-US) -_ `Discfordge` _(Discord)_

Simulations (_sims_) are useful but sometimes questioned [[1](https://discord.com/channels/253212375790911489/817495452328001536/1534925337581064302)] because the simplified scenarios they simulate (‘_Patchwerk fights_’) [[2](<https://github.com/ForgeGit/naxx_mage_movement>)] can differ substantially from real raid encounters, where mechanics (‘"_don’t stand on fire_"’) and player execution carry a significant role [[3](<https://github.com/ForgeGit/class_cope>)].

Innervate (spell ID: `29166`) is an instant-cast Druid skill with a 6-minute cooldown used to regenerate mana. When assigned to a DPS, Arcane Mages are commonly considered the best recipients [[4](https://www.wowhead.com/tbc/guide/classes/druid/feral/dps-rotation-cooldowns-abilities-pve),[5](https://www.wowhead.com/tbc/guide/classes/druid/feral/tank-rotation-cooldowns-abilities-pve)].  _Sims_ from both current and previous versions of _The Burning Crusade_ (TBC) suggest that giving Innervate to an Arcane Mage can increase their damage output by at least ~100 DPS [[6](https://discord.com/channels/253212375790911489/817495452328001536/837273535356862465),[7](https://discord.com/channels/253212375790911489/817495452328001536/884173843248316486),[8](https://discord.com/channels/253212375790911489/817495452328001536/1535312592111927396),[9](https://discord.com/channels/253212375790911489/812641242222821376/972068177964068875),[10](https://discord.com/channels/253212375790911489/812641242222821376/820088886171533370)].

Using real-world data (logs from _Warcraft Logs_), I estimated the effect of Innervate on Arcane Mage DPS across all bosses in Serpentshrine Cavern (SSC) and Tempest Keep (TK) during raid weeks 24-25 of TBC _Classic Anniversary - Phase 2_ (TBC Fresh). 

The analysis adjusts for player, raid, and encounter characteristics, and is designed to answer the question: 
> "For fights like those observed in SSC/TK, how much additional DPS would we expect if an Arcane Mage received exactly one Innervate instead of none?"

<img src="/figure1_v3.png" />

Between Jul 31, 2026 and Aug 07, 2026, data collected included a total of 3,547 unique Arcane Mage players. On average, a 25-player raid contained 2.57 mages and 3.15 druids. Of 60,013 Innervate casts, 80.33%  (48,214) were used on Arcane Mages. 54.01% of all Mages received at least one Innervate.

Receiving exactly one Innervate instead of none was associated with an estimated average gain of +115.6 DPS (95% CI: 108.5–123.0). Importantly, the estimated benefit varied substantially with fight duration. The largest predicted gains occurred in very short encounters (<130 seconds), reaching approximately +253 DPS for encounters lasting 60 seconds (95% CI: 190.6–324.7).

The larger predicted DPS gains in very short encounters might not only reflect the change in rotation (more Arcane Blasts) from mana restored by Innervate, but also the choices that additional mana enables. In addition to Arcane Blast spam casting, additional mana allows Mages to use Destruction Potions, Molten Armor, and other raw DPS consumables instead of more mana-conservative alternatives.

<img src="/figures/figure2.png" />

Raids with shorter kill times were also more likely to give at least one Innervate to an Arcane Mage. This relationship is important because Innervate assignment is not random. Better-geared or better-performing raids tend to kill bosses faster and may also use available raid resources more deliberately. As raids gain gear and become more comfortable with encounter mechanics, they may also have less reason to reserve Innervate for healers or other defensive purposes (e.g. battle rez'd teammate)

These results should also not be interpreted in a vacuum. Giving an Innervate to an AFK Mage will effectively provide [0 DPS](https://fresh.warcraftlogs.com/reports/a:aYCcnqmZJD674vjK?fight=25&pins=0%24Main%24%23244F4B%24auras-cast%24-4%240.0.0.Any%240.0.0.Any%24true%2446980252.0.0.Mage%24true%2429166%2432&source=40). Likewise, a high-performing Arcane Mage may not be able to convert the full theoretical value of an Innervate into damage in every encounter. Fight duration, mechanics, movement, player execution, and the performance of the rest of the raid all influence how useful that additional mana will be in practice.

## Methods

### - Data Cleaning (Valid Logs)
2,500 Warcraft Logs reports from Serpentshrine Cavern (SSC) and Tempest Keep (TK) were filtered to eliminate duplicate logs and invalid non-SSC/TK encounters. Wipes and mages that died during the encounter were excluded to isolate full-encounter performance, alongside mages that did exactly 0 DPS. This resulted in 21,034 total Mage-encounter observations.

### - Exposure Definition (Valid Innervates)
An Innervate was considered valid if it was cast before or during the encounter and at least 20 seconds before the encounter ended. Innervates were classified into three categories (0, 1, or 2+).

### - Statistical & Causal Modeling
We estimated the causal effect of Innervate on Arcane Mage DPS using a Gaussian Generalized Additive Model (GAM) with an identity link using mgcv::bam() and fast restricted maximum likelihood (fREML). The model adjusted for variables that could act as confounders:

- <ins>**Encounter & Raid Factors**</ins>: Boss ID; fight duration (seconds); # of Druids in raid; `Vampiric Touch` mana gains.
- <ins>**Player Factors**</ins>: Difference between Mage-ilvl and Raid-ilvl; Individual Performance (modeled via player-level as random effects); `Berserking` use.
- <ins>**External Buffs**</ins>: Number of `Power Infusions` received; Uptime on `Moonkin Aura`
- <ins>**Exposure Dynamics**</ins>: The effect of Innervate was modeled as a smooth interaction with fight duration to capture non-linear DPS returns across varying encounter lengths.

The fitted model explained 93% of observed deviance ($R^2_{\mathrm{adj}} = 0.918$). These covariates were selected according to their plausible causal role, rather than statistical significance. All the linear adjustment variables were statistically associated with DPS.

$$
\mathrm{DPS}_i = \beta_0 + \beta_1 \mathrm{Innervate}_i + \boldsymbol{\beta X}_i + u_{\mathrm{Mage}[i]} + f_{\mathrm{Boss}[i]}(\mathrm{Duration}_i) + g(\mathrm{Duration}_i)\,\mathrm{Innervate}_i + \varepsilon_i
$$

Here, *i* represents one Mage–encounter observation. **β₀** is the baseline DPS; **β₁** is the average shift associated with receiving one Innervate; and **βXᵢ** represents the measured adjustment variables (gear difference, number of Druids, `Power Infusion`, `Moonkin Aura` uptime, `Vampiric Touch` mana gains, `Berserking` use, and boss).

**u<sub>Mage[i]</sub>** accounts for persistent differences between individual Mages.

**f<sub>Boss[i]</sub>(Durationᵢ)** allows the relationship between fight duration and DPS to differ by boss.

**g(Durationᵢ) × Innervateᵢ** allows the additional DPS from Innervate to change with fight duration. 

**εᵢ** is the remaining unexplained variation in DPS.

### - Additional Variable Exploration

- `Destruction Potion` use was excluded in the model because it is a mechanism through which additional mana is converted into DPS (it is a result of).
- `Judgment of Wisdom` was treated primarily as part of the sensitivity analysis because it is part of the causal pathway that generates additional DPS with Innervate (more mana gains means proportionally more AB casts, which in turns will lead to more hits on target).

### - Estimation
From our model estimation counterfactual DPS outcomes were predicted for every encounter under two scenario conditions:

- (A) $Y^{(1)}$: Exactly one valid Innervate received.
- (B) $Y^{(0)}$: Zero Innervates received.

All other measured covariates were held at their observed values. 

The individual "exposure to Innervate" effect was calculated as $Y^{(1)} - Y^{(0)}$ or in other words "Scenario A minus Scenario B".

To account for within-player correlation across multiple boss encounters and raid weeks, 95% confidence intervals were constructed using 999 clustered bootstrap iterations resampled at the player level.

## Assumptions and limitations

Results generalize strictly to completed boss kills where the Mage survived the entire encounter. Hypotethically, if an Innervate makes a really greedy Mage play more risky, and this in turn leads to that Mage death, that isn't something we can account for.

Innervates casts differ in timing and are not really assigned at random. So even though we tried to account for player random effects in our model, as well as introducing additional uncertainty propagation with clustered bootstraping, there will always be some unemasured coordination that conditions that Innervate assignment. 

Results could be conservative, due to the assumption that fight duration does not "significantly changes" when an Innervate is given to a Mage. In reality, the additional DPS will shorten the fight length, indirectly increasing DPS.

Finally, the analysis estimates the DPS benefit to the Mage receiving the Innervate. Innervate is a limited raid resource, and assigning it to one player may prevent it from being assigned to another Mage, DPS player, or healer. The estimated DPS gain therefore should not be interpreted directly as the whole raid DPS gain or as evidence that a particular Innervate allocation is optimal for every raid.
