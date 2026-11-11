# Typowanie MLST genomów *Staphylococcus aureus* w Snakemake

Zautomatyzowany potok obliczeniowy, który dla złożeń genomów
*S. aureus* określa typ sekwencji (ST) metodą MLST, a następnie buduje
i wizualizuje drzewo filogenetyczne znalezionych ST.

## Czym jest MLST?

MLST (*multi locus sequence typing*) jest to metoda oceny pokrewieństwa
szczepów bakterii. Jest ona stosowana na przykład w epidemiologii szpitalnej. Opiera
się na sekwencjach fragmentów 7 genów (przedstawionych w tabelce poniżej) metabolizmu podstawowego:

| Gen    | Produkt                              |
|--------|--------------------------------------|
| *arcC* | kinaza karbaminianowa                |
| *aroE* | dehydrogenaza szikimianowa           |
| *glpF* | kinaza glicerolowa                   |
| *gmk*  | kinaza guanylanowa                   |
| *pta*  | acetylotransferaza fosforanowa       |
| *tpi*  | izomeraza triozofosforanowa          |
| *yqiL* | acetylotransferaza acetylokoenzymu A |

Każdemu wariantowi (allelowi) genu przypisany jest numer, a kombinacji
7 numerów alleli, typ sekwencji (ST).

## Etapy potoku

| Etap         | Opis                                                                 |
|--------------|----------------------------------------------------------------------|
| `blastn`     | wyszukiwanie każdego z 7 genów w każdym genomie                      |
| `filter`     | pozostawienie dopasowań o 100% pokryciu i pełnej długości            |
| `merge_flt`  | połączenie wyników wszystkich genów dla jednego genomu               |
| `mlst`       | przypisanie numerów alleli i ST (baza PubMLST); `-1` gdy brak ST     |
| `merge_mlst` | połączenie wyników wszystkich genomów w jedną tabelę                 |
| `unique`     | wybór jednej połączonej sekwencji dla każdego ST (plik MSA)          |
| `tree`       | drzewo neighbour joining i wykres liczby genomów dla każdego ST      |

## Struktura repozytorium

```
.
├── Snakefile              # definicja potoku (główny plik opisujący potok)
├── config.yml             # ustawienia
├── envs/                  # opisy środowisk Condy
│   ├── powb-blast.yml
│   └── powb-python.yml
├── scripts/               # skrypty wykonujące poszczególne etapy
│   ├── filter.py
│   ├── merge_flt.sh       #Skrypt .sh zostały napisane samemu (nie były ściągnięte z serweru)
│   ├── mlst.py
│   ├── merge_mlst.sh
│   ├── unique.py
│   └── tree.py
├── input/
│   ├── queries/           # sekwencje zapytań
│   ├── alleles.fna        # baza sekwencji alleli (PubMLST)
│   └── bigsdb.tsv         # profile alleli i odpowiadające im ST
└── results/
    └── tree.png           # przykładowa wizualizacja wyniku
```

## Wymagania

- [Snakemake](https://snakemake.readthedocs.io)
- [Conda](https://docs.conda.io) - środowiska są tworzone automatycznie
  przez Snakemake na podstawie plików z katalogu `envs/`
- opcjonalnie [Apptainer](https://apptainer.org) - do uruchomienia
  potoku w kontenerze (w tym projekcie nie jest używany)

## Dane wejściowe

- `input/queries/*.fna` - sekwencje genów MLST używane jako zapytania
- `input/alleles.fna`, `input/bigsdb.tsv` – pliki bazy danych
  [PubMLST](https://pubmlst.org)
- katalog z genomami (`*.fna.gz`) wskazany w `config.yml`
  (domyślnie `genomes_test`)

## Uruchomienie

Test bez uruchamiania zadań (sprawdzenie poprawności potoku):
Nie trzeba podawać nazwy pliku Snakefile, jeśli w folderze ten plik ma domyślną, niezmienioną nazwę.

```
snakemake -n --sdm conda
```

Uruchomienie na genomach testowych:

```
snakemake --cores 4 --sdm conda
```

Uruchomienie na innym zestawie genomów, bez edycji plików:

```
snakemake --cores 4 --sdm conda --config genomesdir=genomes_all
```

## Wyniki

| Plik                 | Zawartość                                          |
|----------------------|----------------------------------------------------|
| `output/merged.tsv`  | ST i połączona sekwencja alleli dla każdego genomu |
| `output/unique.fna`  | jedna sekwencja dla każdego ST (FASTA)             |
| `output/tree.newick` | drzewo filogenetyczne w formacie Newick            |
| `output/tree.png`    | wizualizacja drzewa z liczbą genomów na ST         |

Wyniki pośrednie zapisywane są w podkatalogach `output/`, a logi
poszczególnych zadań w katalogu `log/`.

![Drzewo filogenetyczne ST](results/tree.png)
