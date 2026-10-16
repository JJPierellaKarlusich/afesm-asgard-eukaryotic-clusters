# Raw data

Taxonomic recovery for proteins assigned to AFESM composite taxon **4137846** ("unassigned Archaea").

## `memId_with_gtdbtaxid4137846.Uniprot.tsv.gz`

UniProt ID Mapping output ([https://www.uniprot.org/id-mapping](https://www.uniprot.org/id-mapping)) for all **738,717** UniProt accessions assigned to AFESM composite taxon 4137846.

- **688,504** accessions still present in UniProtKB: protein name (functional annotation), source organism, NCBI taxonomy identifier and full taxonomic lineage.
- **50,213** accessions deleted from UniProtKB: `deleted` in the `Protein names` field and no other annotation. Their taxonomy, recovered from UniSave, is in [`deleted_memId_with_taxids.tsv`](deleted_memId_with_taxids.tsv).

Classification of the current accessions:

| Category | Criterion | n |
|---|---|---:|
| Asgard archaea | Lineage contains "Promethearchaeati", "Asgard" or "Panguiarchaeales" | 18,659 |
| Non-Asgard archaea | Resolved below the domain level, none of the above labels | 574,723 |
| Unresolved Archaea | Domain level only ("uncultured archaeon", NCBI taxid 115547) | 95,122 |

**Columns** (tab-separated, with header):

| Column | Description |
|---|---|
| `From` | Accession submitted to ID Mapping (AFESM member identifier) |
| `Entry` | UniProtKB accession (identical to `From` in all rows) |
| `Protein names` | Protein name(s) from UniProtKB as functional annotation, or `deleted` for accessions removed from UniProtKB |
| `Taxonomic lineage` | Full NCBI lineage, from "cellular organisms" to the lowest assigned taxon, comma-separated, with each rank in parentheses |
| `Organism` | Source organism name |
| `Organism (ID)` | NCBI taxonomy identifier |

## `deleted_memId_with_taxids.tsv`

Taxonomic information for the **50,213** UniProt accessions assigned to AFESM composite taxon 4137846 that have been deleted from UniProtKB. NCBI taxonomy identifiers and lineages were taken from the last archived entry version in UniSave ([https://rest.uniprot.org/unisave](https://rest.uniprot.org/unisave)).

Lineages use the NCBI taxonomy in force when each entry was archived (e.g., Euryarchaeota, Crenarchaeota, Thaumarchaeota). All resolve to at least phylum level, and none contains "Promethearchaeati", "Asgard" or "Panguiarchaeales". All accessions were therefore classified as **non-Asgard archaea**.

**Columns** (tab-separated, with header):

| Column | Description |
|---|---|
| `UniProtID` | Deleted UniProt accession (AFESM member identifier) |
| `NCBI_TaxID` | NCBI taxonomy identifier from the archived UniSave entry |
| `Lineage` | Taxonomic lineage from the archived entry, from domain to the lowest assigned taxon, semicolon-separated, without rank labels |
