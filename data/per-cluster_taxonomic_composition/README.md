# AFESM per-cluster taxonomic composition

A Google Colab notebook that counts, for every cluster of the **AFESM** structural database (*Metagenomic-scale analysis of the predicted protein structure universe*, [bioRxiv 2025.04.23.650224](https://doi.org/10.1101/2025.04.23.650224)), how many member proteins belong to each taxonomic category. Eukaryotes are split by supergroup and Asgard archaea by GTDB class.

The output is one row per cluster with integer protein counts. Use it to find clusters shared between Asgard archaea and eukaryotes, or to filter clusters by taxonomic make-up.

## Repository contents

| File | Description |
|---|---|
| `AFESM_cluster_composition.ipynb` | The Colab notebook that builds the table. |
| `data/taxon4137846/memId_with_gtdbtaxid4137846.Uniprot.tsv.gz` | UniProt / UniSave annotation of the taxID-4137846 members, read by cell 2b. |
| `afesm_cluster_composition.geq20.noFlag3.slim.tsv.gz` | The resulting table (22 MB): 1,894,714 clusters × 41 columns, built with `MIN_TOTAL=20`, `DROP_FLAG3=True`, `APPLY_ASGARD_FIX=True`. |

You only need to run the notebook to rebuild the table or to use different settings.

```python
import pandas as pd
cc = pd.read_csv("afesm_cluster_composition.geq20.noFlag3.slim.tsv.gz", sep="\t")
shared = cc[(cc.As > 0) & (cc.E_total > 0)]   # clusters with both Asgard and eukaryotic members
```

## Quick start

1. Open `AFESM_cluster_composition.ipynb` in Google Colab.
2. Set the parameters in the **Setup** cell (defaults below).
3. *Runtime → Run all*.

| Parameter | Default | Meaning |
|---|---|---|
| `MIN_TOTAL` | `20` | Minimum number of proteins per cluster. |
| `DROP_FLAG3` | `True` | Exclude `cluFlag == 3` members (sequence clusters whose representative has pLDDT < 60). |
| `APPLY_ASGARD_FIX` | `True` | Reclassify members of the composite taxID 4137846 from their own UniProt / UniSave lineage (see below). |

**Resources:** ~7.5 GB download, ~8 GB disk, constant memory. The per-cluster pass streams 820 M rows with `awk` and takes about 60–90 min.

## Inputs

All inputs are downloaded by the notebook.

| File | Source |
|---|---|
| `7-taxonomy.tar.gz` | AFESM NCBI/GTDB-style taxonomy dump (`afesm_nodes.dmp`, `afesm_names.dmp`, `afesm_merged.dmp`), from `https://afesm.steineggerlab.workers.dev/v1/` |
| `3-repId_memId_cluFlag_taxId_biomeId.tsv.gz` | AFESM cluster membership, 820,460,384 rows / 157,068,338 clusters, 7.2 GB, same server |
| `memId_with_gtdbtaxid4137846.Uniprot.tsv.gz` | UniProt (and UniSave, for deleted entries) annotation of every member filed under taxID 4137846; included in this repository (`data/taxon4137846/`) |

## Classification

Each member's taxID is turned into a lineage from the dump and assigned one category:

| Code | Category | Rule |
|---|---|---|
| `B` | Bacteria | top rank Bacteria |
| `A` | Archaea (non-Asgard) | top rank Archaea, resolved below domain |
| `uA` | Unresolved archaea | lineage stops at the domain (`environmental samples`, `unclassified Archaea`, `Archaea incertae sedis`), MGnify (MGYP) proteins on taxID 4137846, and UniProt "uncultured archaeon" (NCBI 115547) |
| `As_<class>` | Asgard archaea, by GTDB class | lineage contains *Asgard* / *Promethearchaeati* / *Panguiarchaeales* |
| `E_<group>` | Eukaryotes, by supergroup | first matching rank in `EUK_GROUPS` (21 groups) |
| `V` | Viruses | top rank Viruses |
| `U` | Unassigned | root, other entries, unclassified sequences |

### Asgard classes

The dump files most Asgard taxa under GTDB (`Asgardarchaeota; <class>; …`) and some under NCBI (`Asgard group` / `Promethearchaeati; Candidatus <phylum>`). Everything is reported at GTDB class level:

- GTDB classes are used as they are, including placeholder classes such as `JAGGXT01`.
- NCBI phyla are mapped to the GTDB class:
  - Heimdall-, Hod-, Gerd- and Kariarchaeota → **Heimdallarchaeia**
  - Loki-/Promethe-, Hel- and Sigynarchaeota → **Lokiarchaeia**
  - Borrarchaeota → **Sifarchaeia**
  - any other phylum → `<stem>archaeia`
- **Panguiarchaeales** are filed in GTDB under *Thermoproteota; JANJXX01*; they are counted as Asgard and placed in **Njordarchaeia** ([MBE 2025](https://doi.org/10.1093/molbev/msaf201)). 
- `incertae sedis` ranks are skipped and the next rank down is used. Taxa resolved only to phylum (`Asgardarchaeota`, *Asgard group archaeon*) go to `As_unresolved`.

### Eukaryotic supergroups

Opisthokonta are split into Metazoa, Fungi and other. The other main groups are Archaeplastida, SAR (Stramenopiles, Alveolata, Rhizaria, unknown), Haptista, Cryptophyceae and Amoebozoa. In addition, the table separates the 'excavate' lineages Discoba, Metamonada and Malawimonadida (`E_basal_*` columns), which fall on both sides of the inferred eukaryotic root [Nature 640, 974–981 (2025)](https://doi.org/10.1038/s41586-025-08709-5). The orphan lineages are Hemimastigophora, Apusozoa, Breviatea, Provora, Ancyromonadida and CRuMs.

`E_Unassigned` pools eukaryotes that cannot be placed in a supergroup: domain-only lineages, `environmental samples`, `unclassified eukaryotes` and `Eukaryota incertae sedis`.

### Taxon 4137846 fix (`APPLY_ASGARD_FIX`)

AFESM files 1,035,956 proteins under the composite taxID 4137846 (`cellular organisms; Archaea`). UniProt members are reclassified from their own record:

| Source | Criterion | Label | Proteins (all AFESM) |
|---|---|---|---|
| UniProtKB | lineage contains Promethearchaeati / Asgard / Panguiarchaeales | `As_<class>` | 18,659 |
| UniProtKB | resolved below domain | `A` | 574,723 |
| UniSave (deleted entries) | archived lineage at phylum level or below | `A` | 50,213 |
| UniProtKB | "uncultured archaeon" (NCBI 115547) | `uA` | 95,122 |
| MGnify (MGYP) | no UniProt record | `uA` | 297,239 |

## Output

`afesm_cluster_composition.geq{MIN_TOTAL}[.noFlag3].tsv.gz`: one row per cluster, tab-separated, integer counts.

| Column | Meaning |
|---|---|
| `repId` | cluster representative |
| `total` | proteins counted (flag-3 excluded when `DROP_FLAG3`); all category columns sum to it |
| `total_all` | all members, including flag-3 |
| `n_flag3` | flag-3 members |
| `cluFlag12` | 1 if the cluster has at least one flag-1/2 member |
| `E_total` | sum of the `E_` columns |
| `As` | sum of the `As_` columns |
| `B`, `A`, `uA`, `V`, `U` | see *Classification* |
| `As_<class>` | Asgard proteins per class |
| `E_<group>` | eukaryotic proteins per supergroup |

Cell 6 writes a **slim** version (`….slim.tsv.gz`). It drops `total_all`, `n_flag3` and `cluFlag12`, which are constant when `MIN_TOTAL=20, DROP_FLAG3=True`, plus every all-zero column. It checks that all sums still hold before writing.

### Columns of the provided slim table

| Column(s) | Meaning |
|---|---|
| `repId` | cluster representative (UniProt accession or MGnify `MGYP…` ID) |
| `total` | proteins in the cluster (flag-3 members excluded); equals the sum of `B A uA V U`, all `As_` and all `E_` columns |
| `E_total` | eukaryotic proteins (sum of `E_` columns) |
| `As` | Asgard proteins (sum of `As_` columns) |
| `B`, `A`, `uA`, `V`, `U` | Bacteria, non-Asgard Archaea, unresolved Archaea, Viruses, Unassigned |
| `As_Baldrarchaeia`, `As_Heimdallarchaeia`, `As_Hermodarchaeia`, `As_JAGGXT01`, `As_Jordarchaeia`, `As_Lokiarchaeia`, `As_Njordarchaeia`, `As_Odinarchaeia`, `As_Sifarchaeia`, `As_Thorarchaeia`, `As_Wukongarchaeia`, `As_unresolved` | Asgard proteins by GTDB class (12 columns) |
| `E_O_Metazoa`, `E_O_Fungi`, `E_O_other`, `E_Sar_Alveolata`, `E_Sar_Stramenopiles`, `E_Sar_Rhizaria`, `E_Sar_unknown`, `E_Archaeplastida`, `E_Amoebozoa`, `E_Haptista`, `E_Cryptophyceae`, `E_Apusozoa`, `E_basal_Discoba`, `E_basal_Metamonada`, `E_basal_Malawimonadida`, `E_CRuMs`, `E_Ancyromonadida`, `E_Breviatea`, `E_Provora`, `E_Unassigned` | eukaryotic proteins by supergroup (20 columns) |

Columns dropped because they are all zero in this scope: `As_Asgardarchaeia`, `As_Freyarchaeia`, `As_Friggarchaeia`, `As_Gefionarchaeia`, `As_Idunnarchaeia`, and `E_Hemimastigophora` (no Hemimastigophora protein falls in a cluster of ≥20 that passes the pLDDT filter).

### Reference result (`MIN_TOTAL=20`, `DROP_FLAG3=True`, `APPLY_ASGARD_FIX=True`)

| | Value |
|---|---|
| Clusters | 1,894,714 |
| Proteins | 555,961,987 |
| B | 448,066,625 |
| U | 46,887,887 |
| E (all) | 44,319,915 |
| A | 12,369,794 |
| V | 3,567,776 |
| As | 442,684 |
| uA | 307,306 |

Asgard proteins by class:

| Class | Proteins | % of As | Clusters |
|---|---|---|---|
| Lokiarchaeia | 118,024 | 26.7 | 23,994 |
| unresolved | 94,559 | 21.4 | 18,263 |
| Thorarchaeia | 83,349 | 18.8 | 12,858 |
| Heimdallarchaeia | 52,508 | 11.9 | 17,602 |
| Njordarchaeia | 38,024 | 8.6 | 7,926 |
| Sifarchaeia | 18,103 | 4.1 | 3,734 |
| Baldrarchaeia | 13,891 | 3.1 | 4,741 |
| Jordarchaeia | 13,863 | 3.1 | 4,931 |
| JAGGXT01 | 4,462 | 1.0 | 2,151 |
| Odinarchaeia | 2,945 | 0.7 | 1,637 |
| Hermodarchaeia | 2,626 | 0.6 | 1,461 |
| Wukongarchaeia | 330 | 0.07 | 221 |

