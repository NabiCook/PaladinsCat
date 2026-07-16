<p align="center">
  <a href="https://paladinscat.com/">
    <img src="assets/paladinscat.png" alt="PaladinsCat logo" width="120">
  </a>
</p>

<p align="center">
  <strong>Paladins: Comp Analytics Tool</strong> — advanced statistics, or just meow.<br>
  <a href="https://paladinscat.com/"><strong>Visit PaladinsCat →</strong></a>
</p>

## Introduction

PaladinsCat is a community-focused intelligence and analytics platform for [Paladins](https://store.steampowered.com/app/757350/Paladins_Champions_Deep_Dive/). Its purpose is to track account and match patterns associated with suspected cheating, private profiles, and repeated disruptive behavior so players can understand threats to the competitive community through inspectable, data-driven evidence instead of rumors.

The platform brings player profiles, match histories, champion statistics, and account signals together in one place. Its findings are intended to inform community awareness and careful player judgment—not replace official moderation or treat an unverified suspicion as a final determination.

## Founding breakthrough: solving a long-standing data corruption issue

PaladinsCat began development in **May 2026** while investigating a skin ID corruption issue that had been known for more than five years but remained unaddressed. The investigation traced missing and incorrect skin information to a silent signed 16-bit integer overflow and identified **31 affected skins across 20 champions** whose IDs had crossed the `32,767` limit.

That finding became PaladinsCat's first major technical breakthrough and the starting point for the platform itself. PaladinsCat now detects affected matches and uses a multi-stage recovery pipeline to reconstruct player details, fill gaps, and label each record by its source and quality. Instead of silently accepting incomplete data—or discarding the match—the platform preserves what is valid and clearly identifies what was recovered.

[**Read the investigation, published July 2026 →**](docs/blog/skin-id-overflow.md)

## Key points

- **Account intelligence:** connects player and match activity to reveal suspicious or harmful patterns that individual matches may not show.
- **Private-profile visibility:** preserves relevant match activity for analysis even when an account's profile is private.
- **Evidence over accusation:** gives players documented activity they can inspect instead of relying on hearsay alone.
- **Resilient, transparent analytics:** detects and reconstructs broken matches while identifying whether records are direct, recovered, or minimal.
- **Data integrity first:** investigates and validates upstream game data rather than automatically treating every response as correct.
- **Open technical findings:** publishes investigations and production lessons as detailed blog posts.

## Public release repository

This repository contains PaladinsCat's **public release artifacts, technical documentation, and blog posts**. It does not contain the platform's proprietary source code.

## Blog posts

| Post | Description |
|:---|:---|
| [The Int16 Skin ID Overflow](docs/blog/skin-id-overflow.md) | How signed 16-bit integer limitations silently corrupt game statistics data—and how PaladinsCat recovered it |

## License

© 2026 PaladinsCat. All rights reserved.
