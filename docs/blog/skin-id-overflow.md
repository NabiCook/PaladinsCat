<!--
  PaladinsCat Blog — Skin ID Int16 API Failure Analysis
  Public-facing content. No source code or backend internals.
-->

# 🔴 The Int16 Skin ID API Failure

> Why Paladins skin IDs above 32,767 break the Hi-Rez match API, with a database-sourced list of known affected skins.

---

**Published:** July 2026 &nbsp;|&nbsp; **Topic:** Data integrity · Game analytics · Hidden bugs

---

## The Problem

Some Paladins `skin_id` values now exceed the signed 16-bit integer maximum of **32,767**, but part of the Hi-Rez match-data path still attempts to deserialize the field as an `Int16`. The value cannot fit, so the API emits an explicit error message instead of returning the affected player row:

```text
Value was either too large or too small for an Int16. Failing Field = skin_id
```

### What happens above the boundary?

| Real `skin_id` | Above Int16 maximum | API result |
|---:|:---:|:---|
| 32,829 | Yes | Player row dropped; Int16 `ret_msg` returned |
| 33,060 | Yes | Player row dropped; Int16 `ret_msg` returned |
| 33,741 | Yes | Player row dropped; Int16 `ret_msg` returned |

The failure is **not silent**. When a positive skin ID exceeds the Int16 limit of **32,767**, the Hi-Rez API returns an Int16 error and drops the match data.

---

## Known Broken Skins

The list below was fetched from existing PaladinsCat database records. It contains observed skin IDs above the signed Int16 boundary.

**Database snapshot: 31 known broken skins across 20 champions** (July 2026).

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

## Boundary Summary

```
Int16 max (signed):    32,767
Current highest ID:   33,769
```

---

## Continue reading

[**Beyond the Int16 Overflow: Recovering the Match Hi-Rez Drops →**](beyond-int16-match-recovery.md) follows the failure into the raw Hi-Rez payload and shows how PaladinsCat reconstructs the players that disappear after the Int16 sentinel.

---

*The listed skin IDs and champion names were fetched from existing PaladinsCat database records and reflect data available in July 2026.*
