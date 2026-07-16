<!--
  PaladinsCat Blog — Skin ID Int16 Overflow Investigation
  Public-facing content. No source code or backend internals.
-->

# 🔴 The Int16 Skin ID Overflow

> A deep dive into how signed 16-bit integer limitations can silently corrupt game statistics data — and how we recovered it.

---

**Published:** July 2026 &nbsp;|&nbsp; **Topic:** Data integrity · Game analytics · Hidden bugs

---

## The Problem

The Hi-Rez API returns `skin_id` values that exceed the signed 16-bit integer maximum of **32,767**. When a system stores these values as `smallint` (Int16), the values silently overflow and wrap around to negative numbers, corrupting data without raising any errors.

### What does overflow look like?

| Real `skin_id` | Stored as Int16 | Result |
|---:|---:|---|
| 32,829 | -32,707 | ❌ Corrupted |
| 33,060 | -32,676 | ❌ Corrupted |
| 33,741 | -31,795 | ❌ Corrupted |

The values don't throw an error. They don't log a warning. They just become *wrong* — silently rewriting player data.

---

## Detection

We discovered this during routine match investigation. Players appeared in match data with missing or garbled skin information. A deeper query revealed the root cause.

> [!NOTE]  
> The investigation started with match data where skin names were missing or incorrect. Tracing back revealed Int16 overflow as the root cause for specific high-value skin IDs.

**Result: 31 skins above the Int16 boundary** (as of July 2026).

---

## Affected Skins

**31 confirmed broken skins** across 20 champions:

| Champion | Skin ID | Skin Name |
|:---|---:|---|
| Tyra | 32,829 | Get Served |
| Makoa | 32,833 | Cold-Blooded |
| Furia | 32,841 | Soul's Animus |
| Nyx | 32,843 | Scorched |
| Khan | 32,844 | Judge & Jury |
| Khan | 32,845 | Firing Squad |
| Tyra | 32,852 | Daisy Dukes |
| Rei | 32,942 | Delight |
| Nyx | 32,943 | Permafrost |
| Tyra | 32,963 | Pop Royalty |
| Fernando | 32,997 | Ultimatum |
| Grover | 33,056 | Grovos |
| Grover | 33,058 | Chaos Grovos |
| Omen | 33,060 | Flame-born Jace |
| Omen | 33,062 | Frost-born Jace |
| Strix | 33,065 | Bazaar |
| Bomb King | 33,106 | Explosive Primer |
| Furia | 33,200 | Shrine Guardian |
| Moji | 33,258 | Red Ridin' |
| Moji | 33,292 | Akazukin |
| Furia | 33,445 | Shrine Demon |
| Tyra | 33,520 | Street Style |
| Seris | 33,690 | Street Style |
| Grohk | 33,703 | Shaman |
| Makoa | 33,705 | Makilla |
| Fernando | 33,740 | Exhortation |
| Rei | 33,741 | Coastline Cony |
| Drogoz | 33,742 | Infernal Lord |
| Makoa | 33,744 | Shikoa |
| Ash | 33,767 | Loci-Blaster |
| Zhin | 33,769 | Subjugator |

---

## Near-Miss Skins

Five skins were flagged as false positives by other groups but confirmed **safe** — their IDs remain below 32,767:

| Skin | ID | Status |
|---|---:|:---|
| Cuddle King (Bomb King) | 32,595 | ✅ Safe |
| Admiration (Rei) | 32,597 | ✅ Safe |
| Frozen Over (Skye) | 32,645 | ✅ Safe |
| Hyperpop (Seris) | 32,648 | ✅ Safe |
| Soul Devourer (Jenos) | 32,716 | ✅ Safe |

---

## Recovery Pipeline

When a match contains players with broken skin IDs, the standard match data endpoint returns partial or corrupted results. PaladinsCat handles this through a multi-stage recovery process:

### Recovery Stages

```
┌─────────────────────────────────────────────┐
│ Stage 1: Detect broken skin_id              │
│ (Int16 overflow sentinel in match data)     │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ Stage 2: Fetch participant IDs              │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ Stage 3: Recover player facts               │
│ (match history lookup)                      │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│ Stage 4: Fill gaps                          │
│ (Match details: duration, replay, bans)     │
└─────────────────────────────────────────────┘
```

Recovery preserves whatever valid direct data still exists, then fills in missing player information through supplementary lookups. When a direct match response contains fewer than 10 usable player rows, the recovery pipeline triggers automatically.

---

## Impact on Data Quality

Each player row in PaladinsCat is tagged with its data origin:

| Source | How it was obtained | Quality |
|---|---|:---:|
| `direct` | Full match data endpoint | 🟢 High |
| `recovered` | Match history lookup | 🟢 High |
| `minimal` | Profile-only fallback | 🟡 Low |

Matches with broken skins are marked with `broken=true` and `recovered=true` flags so users can verify data provenance.

---

### Summary Statistics

```
Int16 max (signed):    32,767
Current highest ID:   33,769
```

---

*This post was written based on real production data from July 2026. All skin IDs and champion names are sourced from the Hi-Rez API response data.*