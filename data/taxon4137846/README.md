# Raw data

Taxonomic recovery for proteins assigned to the AFESM composite taxon 4137846 ("unassigned Archaea").

## Final classification of all members of taxon 4137846

AFESM assigns 1,035,956 proteins to taxon 4137846 ("cellular organisms; Archaea").
Of these, 738,717 are UniProt accessions and are reclassified from their own lineage
(UniProtKB, or UniSave for deleted entries). The remaining 297,239 are MGnify (MGYP)
sequences. They have no UniProt record, so their lineage cannot be recovered, and they
are counted as unresolved Archaea.

| Source | Category | Criterion | Label | Proteins |
|---|---|---|---|---:|
| UniProtKB | Asgard archaea | Lineage contains "Promethearchaeati", "Asgard" or "Panguiarchaeales" | `As` | 18,659 |
| UniProtKB | Non-Asgard archaea | Resolved below the domain level, none of the above labels | `A` | 574,723 |
| UniSave (deleted from UniProtKB) | Non-Asgard archaea | Archived lineage at phylum level or below, none of the above labels | `A` | 50,213 |
| UniProtKB | Unresolved Archaea | Domain level only ("uncultured archaeon", NCBI taxid 115547) | `uA` | 95,122 |
| MGnify (MGYP) | Unresolved Archaea | No UniProt record | `uA` | 297,239 |
| **Total** | | | | **1,035,956** |

Counts are over all of AFESM, before any cluster-size or pLDDT filter.

## Files

### `memId_with_gtdbtaxid4137846.Uniprot.tsv.gz`

UniProt ID Mapping output (https://www.uniprot.org/id-mapping) for the 738,717 UniProt accessions assigned to taxon 4137846.

- 688,504 accessions still in UniProtKB: protein name (functional annotation), source organism, NCBI taxonomy identifier and full taxonomic lineage.
- 50,213 accessions deleted from UniProtKB: `deleted` in the *Protein names* field and no other annotation. Their taxonomy, recovered from UniSave, is in `deleted_memId_with_taxids.tsv`.

Columns (tab-separated, with header):

| Column | Description |
|---|---|
| From | Accession submitted to ID Mapping (AFESM member identifier) |
| Entry | UniProtKB accession (identical to *From* in all rows) |
| Protein names | Protein name(s) from UniProtKB as functional annotation, or `deleted` for accessions removed from UniProtKB |
| Taxonomic lineage | Full NCBI lineage, from "cellular organisms" to the lowest assigned taxon, comma-separated, with each rank in parentheses |
| Organism | Source organism name |
| Organism (ID) | NCBI taxonomy identifier |

### `deleted_memId_with_taxids.tsv`

Taxonomy of the 50,213 UniProt accessions on taxon 4137846 that have been deleted from UniProtKB. NCBI taxonomy identifiers and lineages were taken from the first archived version of each entry in UniSave (https://rest.uniprot.org/unisave).

Lineages use the NCBI taxonomy in force when each entry was archived (e.g. Euryarchaeota, Crenarchaeota, Thaumarchaeota). All resolve to at least phylum level, and none contains "Promethearchaeati", "Asgard" or "Panguiarchaeales", so all are classified as non-Asgard archaea.

Columns (tab-separated, with header):

| Column | Description |
|---|---|
| UniProtID | Deleted UniProt accession (AFESM member identifier) |
| NCBI_TaxID | NCBI taxonomy identifier from the archived UniSave entry |
| Lineage | Taxonomic lineage from the archived entry, from domain to the lowest assigned taxon, semicolon-separated, without rank labels |

### `memId_with_gtdbtaxid4137846_Uniprot_Unisave.ipynb`

Pipeline that produced both data files: extraction of the members from the AFESM cluster mapping file, UniProt ID Mapping, classification, and UniSave retrieval for the deleted accessions.
