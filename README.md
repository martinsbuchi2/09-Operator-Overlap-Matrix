# Analysis 09 — Operator Geographic Overlap Matrix

*Generated: 2026-04-30*

## 1. Objective

For the 60 largest licensees in Alberta, compute every pairwise geographic
overlap between their operating territories (convex hulls of their wells)
to identify:

1. **Maximum-overlap pairs** — operators competing in the same backyard.
2. **Engulfed pairs** — operators whose footprint sits entirely within a
   larger operator's footprint, the textbook signature of a natural M&A
   target.
3. **A "potential acquirer" leaderboard** — which operators sit on the
   most engulfed targets.

This is a follow-on to Analysis 02 (which mapped each licensee's hull in
isolation) and Analysis 07 (which partitioned Alberta into non-overlapping
Voronoi territories). The overlap matrix asks the explicitly *relational*
question: who shares territory with whom?

## 2. Input Data
| Item | Value |
|------|------:|
| Source | `Abandoned_Suspended_raw.shp` |
| Working CRS | EPSG:3400 (NAD83 / Alberta 10-TM Forest, metres) |
| Operators analysed | **Top 60 licensees by well count** (smallest holds 276 wells) |
| Combined wells of top-60 | 77,588 (82.2% of total) |
| Persisted as | `Input/all_wells.gpkg` |

## 3. Methodology

### 3.1 Operator hulls
Each top-60 licensee's wells were extracted with
`native:extractbyexpression`, then aggregated into a convex hull via
`qgis:minimumboundinggeometry`. Each hull was enriched with `Wells` and
`AreaKm2` for downstream computation.

### 3.2 Pairwise overlap matrix
For all `60 × 59 / 2 = 1,770` operator pairs, the script:
- Tested for hull intersection.
- Computed the intersection area in km².
- Skipped trivial overlaps (< 1 km²).
- Recorded `pct_smaller` (overlap as % of the smaller hull),
  `pct_larger` (overlap as % of the larger hull), and `jaccard`
  (intersection / union).

### 3.3 Overlap polygons (top 100)
The actual intersection polygon of the top 100 pairs (by absolute area)
was saved to `Output/overlap_regions_top100.gpkg` for visual inspection.

### 3.4 M&A-candidate network
A directed line layer was built from each "smaller" operator's centroid
to its potential acquirer's centroid, for pairs where:
- `pct_smaller >= 80%` — the smaller's hull is largely contained in the larger's
- Smaller has ≥100 wells — to filter out trivial micro-cases

The strength of the signal is encoded in `PctEngulfed`:
- 99 %+ = "fully engulfed"
- 90–99 % = "strongly engulfed"
- 80–90 % = "moderately engulfed"

## 4. Outputs
| File | Type | Contents |
|------|------|----------|
| `Input/all_wells.gpkg` | Vector (Point) | All 94,428 wells, EPSG:3400 |
| `Output/operator_hulls.gpkg` | Vector (Polygon) | 60 operator hulls with Wells, AreaKm2 |
| `Output/overlap_regions_top100.gpkg` | Vector (Polygon) | Top-100 pairwise intersection polygons |
| `Output/mna_candidate_network.gpkg` | Vector (LineString) | 1,101 candidate-acquirer-target pairs |
| `Output/overlap_pairs.csv` | CSV | Full 1,575-pair overlap matrix |
| `09_Operator_Overlap_Matrix.qgz` | QGIS project | Pre-styled, ready to open |

## 5. Key Findings

### 5.1 Overlap is the rule, not the exception
- **1,575 of 1,770 possible pairs (89%)** have non-trivial geographic overlap.
- Of those, **1101** pairs see the smaller operator's hull
  ≥80% engulfed by a larger competitor.
- This high overlap density is itself diagnostic: it confirms that
  **convex hulls dramatically overstate true operating footprint** (since
  large operators' hulls span the entire province and engulf vast empty
  regions). The Voronoi tessellation in Analysis 07 gives the *non-overlapping*
  view of the same operators.

### 5.2 Top 15 pairs by absolute overlap area (km²)
| Smaller operator | Wells | Larger operator | Wells | Overlap (km²) | Jaccard |
|:-----------------|------:|:----------------|------:|--------------:|--------:|
| Cenovus Energy Inc. | 5,874 | Canadian Natural Resources Limited | 20,769 | 509,215 | 88.4% |
| Imperial Oil Resources Limited | 1,699 | Cenovus Energy Inc. | 5,874 | 476,290 | 89.3% |
| ConocoPhillips Canada Resources Corp. | 924 | Canadian Natural Resources Limited | 20,769 | 468,421 | 83.9% |
| Imperial Oil Resources Limited | 1,699 | Canadian Natural Resources Limited | 20,769 | 468,123 | 82.8% |
| ConocoPhillips Canada Resources Corp. | 924 | Cenovus Energy Inc. | 5,874 | 466,316 | 86.9% |
| ConocoPhillips Canada Resources Corp. | 924 | Imperial Oil Resources Limited | 1,699 | 446,041 | 88.3% |
| TAQA North Ltd. | 898 | Canadian Natural Resources Limited | 20,769 | 442,273 | 79.6% |
| Paramount Resources Ltd. | 1,961 | Canadian Natural Resources Limited | 20,769 | 420,095 | 75.8% |
| Paramount Resources Ltd. | 1,961 | Cenovus Energy Inc. | 5,874 | 419,855 | 79.1% |
| Paramount Resources Ltd. | 1,961 | Imperial Oil Resources Limited | 1,699 | 415,688 | 85.9% |
| Paramount Resources Ltd. | 1,961 | ConocoPhillips Canada Resources Corp. | 924 | 411,030 | 85.4% |
| TAQA North Ltd. | 898 | Cenovus Energy Inc. | 5,874 | 409,926 | 72.7% |
| Prairie Provident Resources Canada Ltd. | 977 | Canadian Natural Resources Limited | 20,769 | 403,172 | 72.7% |
| Harvest Operations Corp. | 2,067 | Canadian Natural Resources Limited | 20,769 | 394,308 | 71.1% |
| TAQA North Ltd. | 898 | Imperial Oil Resources Limited | 1,699 | 391,111 | 73.6% |

The pattern is clear: the overlap rankings are dominated by the
**Calgary mega-majors** (CNRL, Cenovus, Imperial Oil, ConocoPhillips,
Paramount, TAQA North) — all of whose hulls span the entire province
and therefore overlap each other almost completely.

### 5.3 Top 15 pairs by Jaccard coefficient
The Jaccard coefficient (intersection ÷ union) is a normalised measure
of similarity — high Jaccard means the two hulls cover *the same*
territory, regardless of absolute size.

| Operator A | Wells A | Operator B | Wells B | Jaccard | Overlap (km²) |
|:-----------|--------:|:-----------|--------:|--------:|--------------:|
| Cenovus Energy Inc. | 5,874 | Imperial Oil Resources Limited | 1,699 | 89.3% | 476,290 |
| Cenovus Energy Inc. | 5,874 | Canadian Natural Resources Limited | 20,769 | 88.4% | 509,215 |
| Imperial Oil Resources Limited | 1,699 | ConocoPhillips Canada Resources Corp. | 924 | 88.3% | 446,041 |
| Cenovus Energy Inc. | 5,874 | ConocoPhillips Canada Resources Corp. | 924 | 86.9% | 466,316 |
| Harvest Operations Corp. | 2,067 | Tourmaline Oil Corp. | 1,331 | 86.5% | 356,831 |
| Canlin Energy Corporation | 1,376 | Long Run Exploration Ltd. | 1,077 | 86.5% | 328,225 |
| Paramount Resources Ltd. | 1,961 | Imperial Oil Resources Limited | 1,699 | 85.9% | 415,688 |
| Paramount Resources Ltd. | 1,961 | ConocoPhillips Canada Resources Corp. | 924 | 85.4% | 411,030 |
| Long Run Exploration Ltd. | 1,077 | SanLing Energy Ltd. | 1,105 | 85.2% | 340,183 |
| TAQA North Ltd. | 898 | Harvest Operations Corp. | 2,067 | 85.1% | 385,072 |
| West Lake Energy Corp. | 1,130 | Canadian Oil & Gas International Inc. | 454 | 84.2% | 315,063 |
| Canlin Energy Corporation | 1,376 | SanLing Energy Ltd. | 1,105 | 84.1% | 340,146 |
| Canadian Natural Resources Limited | 20,769 | ConocoPhillips Canada Resources Corp. | 924 | 83.9% | 468,421 |
| CNOOC Petroleum North America ULC | 627 | AlphaBow Energy Ltd. | 1,490 | 83.8% | 255,120 |
| Blue Sky Resources Ltd. | 983 | Paramount Resources Ltd. | 1,961 | 83.7% | 363,017 |

The 89% Jaccard between Cenovus and Imperial Oil — meaning their hulls
overlap on ~89% of their combined area — confirms that the major
Calgary integrated oil companies operate in the same broad geography.

Outside the mega-majors, **Harvest / Tourmaline (86.5%)** and
**Canlin / Long Run (86.5%)** stand out as similarly-sized operators
whose territories nearly coincide — particularly natural M&A pairs.

### 5.4 The "fully engulfed" target list (top 20)
| Smaller (engulfed) | Wells | Larger (could acquire) | Wells | % engulfed | Overlap (km²) |
|:-------------------|------:|:-----------------------|------:|-----------:|--------------:|
| Syncrude Canada Ltd. | 919 | Cenovus Energy Inc. | 5,874 | 100.0% | 877 |
| Syncrude Canada Ltd. | 919 | Canadian Natural Resources Limited | 20,769 | 100.0% | 877 |
| Syncrude Canada Ltd. | 919 | Suncor Energy Inc. | 2,060 | 100.0% | 877 |
| Syncrude Canada Ltd. | 919 | Imperial Oil Resources Limited | 1,699 | 100.0% | 877 |
| Aspenleaf Energy Limited | 317 | Cenovus Energy Inc. | 5,874 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Canadian Natural Resources Limited | 20,769 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | TAQA North Ltd. | 898 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Lexin Resources Ltd. | 852 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Blue Sky Resources Ltd. | 983 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Vermilion Energy Inc. | 321 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Canlin Energy Corporation | 1,376 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Paramount Resources Ltd. | 1,961 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Imperial Oil Resources Limited | 1,699 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Harvest Operations Corp. | 2,067 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Long Run Exploration Ltd. | 1,077 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | SanLing Energy Ltd. | 1,105 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | ConocoPhillips Canada Resources Corp. | 924 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Tourmaline Oil Corp. | 1,331 | 100.0% | 80,822 |
| Aspenleaf Energy Limited | 317 | Manitok Energy Inc. | 482 | 100.0% | 80,822 |
| AlphaBow Energy Ltd. | 1,490 | Canadian Natural Resources Limited | 20,769 | 100.0% | 281,556 |

**Cenovus Energy alone has its hull engulfing the entire convex hulls of
14 mid-sized operators.** This is partly an artifact of Cenovus's
province-spanning footprint, but it does correctly identify the population
of operators whose entire well portfolio sits inside ground that Cenovus
already operates in — minimal new geographic learning curve to acquire.

### 5.5 Potential-acquirer leaderboard
*(Number of top-60 operators each major could absorb without expanding its
geographic footprint.)*

| Operator | # of engulfed targets among top-60 |
|:---------|----------------------------------:|
| Canadian Natural Resources Limited | 58 |
| Cenovus Energy Inc. | 58 |
| Imperial Oil Resources Limited | 55 |
| TAQA North Ltd. | 51 |
| ConocoPhillips Canada Resources Corp. | 51 |
| Paramount Resources Ltd. | 48 |
| Harvest Operations Corp. | 48 |
| Prairie Provident Resources Canada Ltd. | 48 |
| SanLing Energy Ltd. | 43 |
| Blue Sky Resources Ltd. | 42 |
| Tourmaline Oil Corp. | 42 |
| Canlin Energy Corporation | 38 |
| Long Run Exploration Ltd. | 38 |
| Manitok Energy Inc. | 37 |
| West Lake Energy Corp. | 34 |

CNRL and Cenovus tie at 58 engulfed targets each — they sit on the most
real estate in the province and therefore could swallow the most
peer-tier operators on a pure geographic-fit basis.

### 5.6 Standout findings
- **Genuinely M&A-natural pairs** (high Jaccard, similar size) include:
  *Harvest Operations & Tourmaline Oil* (86.5% Jaccard, similar
  west-central focus); *Canlin Energy & Long Run Exploration* (86.5%
  Jaccard, both ~1,000-1,400 wells); *West Lake & Canadian Oil & Gas
  International* (84.2% Jaccard).
- **Foreign-controlled operators in tight overlap with peers**:
  *CNOOC Petroleum North America & AlphaBow Energy* — 83.8% Jaccard.
  Sinopec Canada Energy is fully engulfed by Cenovus.

## 6. How to Reproduce
1. Open `09_Operator_Overlap_Matrix.qgz` in QGIS 3.x.
2. Four layers should load:
   - **Operator Hulls (top 60)** — semi-transparent fills, shaded by tier.
   - **Top-100 Overlap Regions** — the actual intersection polygons.
   - **M&A Candidate Network** — magenta lines connecting engulfed pairs.
     The "Moderately engulfed" rule is OFF by default to avoid hairball;
     turn it on to see the full 1,101-pair network.
   - **Wells** — toggled OFF by default; turn on for sanity check.
3. Open the `overlap_pairs.csv` for the full 1,575-pair matrix sortable
   by any column.

## 7. Notes & Caveats
- **Convex-hull caveat.** Convex hulls envelop the smallest convex region
  containing all an operator's wells, but engulf large empty zones in
  between (clearly visible in Analysis 07's Voronoi tessellation).
  Therefore a hull-engulfment claim like "Cenovus engulfs Bonterra
  100%" means Bonterra's hull is inside Cenovus's hull *as drawn*, not
  that Cenovus's wells are everywhere Bonterra's wells are. For a
  sharper M&A signal, well-level proximity (e.g., "% of target wells
  within 10 km of an acquirer well") would be a follow-on analysis.
- The top-60 cut excludes 892 smaller licensees. Their convex hulls
  could be added if extending the matrix is desired; the dominant
  engulfers (CNRL, Cenovus) would absorb most of them by the same
  geometric criterion.
- The Jaccard coefficient is a symmetric similarity metric; the
  "engulfment" framing requires the asymmetric `pct_smaller` view.
- Convex hulls do not consider working interest, basin, or play
  identity — two operators with overlapping hulls might be drilling
  totally different formations vertically. Subsurface stratigraphy is
  needed for a true play-level competition map.
