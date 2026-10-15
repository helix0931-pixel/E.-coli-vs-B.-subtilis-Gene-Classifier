# E. coli vs B. subtilis Gene Classifier

A machine learning project that predicts which bacterium a gene came from (*Escherichia coli* or *Bacillus subtilis*) using only its DNA sequence.

Built in Google Colab as my first computational biology + ML project.

## Results

| Model | Features | Test accuracy |
|---|---|---|
| Random forest | GC content only (1 feature) | **80%** |
| Random forest | 3-mer frequencies (64 features) | **96%** |

Random guessing on two equal classes would give 50%. The GC-only model is the baseline, and the gap to 96% shows that the pattern of short DNA "words" carries information beyond GC content alone.

### Most informative 3-mers

The five 3-mers the model relied on most, by random forest feature importance:

| 3-mer | Importance |
|---|---|
| CGC | 0.114 |
| GCG | 0.085 |
| TGG | 0.083 |
| CTG | 0.070 |
| ACC | 0.062 |

An even spread across all 64 3-mers would give about 0.016 each. Feature importance shows what the model *used*, not why it is biologically true. Linking these to codon usage is a hypothesis I have not tested.

## How it works

1. **Get data (Biopython).** Download both genomes from NCBI in GenBank format using `Entrez.efetch`.
   - *E. coli* K-12: `NC_000913.3`
   - *B. subtilis*: `NC_000964.3`
2. **Extract genes (Biopython).** Keep every `CDS` feature and pull out its DNA with `feature.extract(record.seq)`, which handles reverse-strand genes automatically.
   - *E. coli*: 4318 CDS
   - *B. subtilis*: 4237 CDS
3. **Clean.** Keep genes that are 300 to 3000 bases long and contain only A, C, G, T.
   - After cleaning: 3741 *E. coli* and 3568 *B. subtilis* genes
4. **Balance.** Randomly sample 1500 genes per organism (`random.seed(42)`), giving 3000 genes in total.
5. **Build features (NumPy).** Turn each gene into 64 numbers: the frequency of each possible 3-mer (AAA, AAC, ..., TTT). Frequencies are divided by the total count, so gene length does not matter.
   - `X` has shape `(3000, 64)`
   - `y` has shape `(3000,)`, with 0 = *E. coli* and 1 = *B. subtilis*
6. **Split (scikit-learn).** 80% train and 20% test, with `stratify=y` and `random_state=42`.
7. **Train and evaluate (scikit-learn).** `RandomForestClassifier(random_state=42)`, scored on the 600 held-out genes.
8. **Interpret (Matplotlib).** Plot the top 5 most important 3-mers.

## Libraries

- [Biopython](https://biopython.org/) for downloading and parsing GenBank files
- [NumPy](https://numpy.org/) for the feature matrix
- [scikit-learn](https://scikit-learn.org/) for the train/test split and random forest
- [Matplotlib](https://matplotlib.org/) for plots

## Run it

The easiest way is Google Colab, where everything except Biopython is already installed.

```python
!pip install biopython
```

Set your own email before downloading, since NCBI requires one:

```python
from Bio import Entrez
Entrez.email = "your_email@example.com"
```

Then run the notebook cells in order.

## Limitations

- Results come from a single train/test split, so the accuracy could vary a little with a different split. Cross-validation would give a steadier estimate.
- Only random forest was tried, with default settings and no tuning.
- Only 3-mers were used. Other values of k, or codon-based features, might do better or worse.
- *E. coli* and *B. subtilis* are quite distantly related. A closer pair, such as *E. coli* vs *Salmonella*, would be a much harder test.

## Next steps

- Add a confusion matrix to see which organism gets confused more often
- Try different k-mer sizes (2, 4, 5)
- Compare average 3-mer patterns with a Seaborn heatmap
- Test on a closely related species

## Author

SHASHANK.
