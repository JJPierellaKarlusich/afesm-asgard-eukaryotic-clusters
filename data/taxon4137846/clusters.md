# AFESM per-cluster taxonomic composition

A Google Colab notebook that counts, for every cluster of the **AFESM** structural database (AlphaFold DB + ESM Atlas; *Metagenomic-scale analysis of the predicted protein structure universe*, [bioRxiv 2025.04.23.650224](https://doi.org/10.1101/2025.04.23.650224)), how many member proteins belong to each taxonomic category. Eukaryotes are split by supergroup and Asgard archaea by GTDB class.

The output is one row per cluster with integer protein counts. Use it to find clusters shared between Asgard archaea and eukaryotes, or to filter clusters by taxonomic make-up.

## Quick start

1. Open `AFESM_cluster_composition_colab.ipynb` in Google Colab.
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
| `memId_with_gtdbtaxid4137846.Uniprot.tsv.gz` | UniProt (and UniSave, for deleted entries) annotation of every member filed under taxID 4137846; downloaded from Google Drive |

> **Note:** the AFESM server runs on a Cloudflare Workers plan and is sometimes rate-limited (HTTP 429, Cloudflare error 1027). Download the two AFESM files once, keep a copy on Google Drive, and copy them into `afesm/` before running. The download cells skip any file that is already present and valid.

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
- **Panguiarchaeales** are filed in GTDB under *Thermoproteota; JANJXX01*; they are counted as Asgard and placed in **Njordarchaeia** ([MBE 2025](https://doi.org/10.1093/molbev/msaf201)). Two GTDB taxIDs on this path are forced to Asgard: 4138202 and 4140603.
- `incertae sedis` ranks are skipped and the next rank down is used. Taxa resolved only to phylum (`Asgardarchaeota`, *Asgard group archaeon*) go to `As_unresolved`.

### Eukaryotic supergroups

Opisthokonta are split into Metazoa, Fungi and other. The other groups are Archaeplastida, SAR (Stramenopiles, Alveolata, Rhizaria, unknown), Haptista, Cryptophyceae and Amoebozoa. The basal excavate lineages Discoba, Metamonada and Malawimonadida follow [Nature 640, 974–981 (2025)](https://doi.org/10.1038/s41586-025-08709-5). The orphan lineages are Hemimastigophora, Apusozoa, Breviatea, Provora, Ancyromonadida and CRuMs.

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

The last cell writes a **slim** version (`….slim.tsv.gz`). It drops `total_all`, `n_flag3` and `cluFlag12`, which are constant when `MIN_TOTAL=20, DROP_FLAG3=True`, plus every all-zero column. It checks that all sums still hold before writing.

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

In AFESM every flag-3 member sits in a cluster with no flag-1/2 member. So `DROP_FLAG3=True` gives the same result as keeping only clusters with `cluFlag12 == 1`: 1,894,714 of the 2,355,093 clusters with ≥20 members.

## Validation

The notebook stops with an error if any of these checks fails:

- it sees exactly 157,068,338 clusters, and every taxID maps to a category;
- in every row, the category columns sum to `total`, `E_` columns sum to `E_total`, and `As_` columns sum to `As`;
- no category label contains whitespace;
- the 4137846 reclassification gives exactly the counts in the table above;
- the per-category totals (`As`, `A`, `uA`, `B`, `E`, `U`, `V`) reproduce an independent implementation (`1_clusters_with_20counts_treemap_for_taxonomy_4137846fix.ipynb`). The two outputs were also compared cluster by cluster: running with `MIN_TOTAL=1` and filtering to ≥20 gives a byte-identical table.

`AS_GROUP` in cell 5b pools `As_` columns, for example small classes into one bin, without re-running the pass.

## Citation

If you use this table, please cite the AFESM paper ([bioRxiv 2025.04.23.650224](https://doi.org/10.1101/2025.04.23.650224)) and this repository.

## License

<!-- e.g. MIT for code; AFESM data are subject to their original licenses -->
