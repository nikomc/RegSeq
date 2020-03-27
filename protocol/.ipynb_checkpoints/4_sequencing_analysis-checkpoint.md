# RegSeq Sequence Analysis

## Initial Tidying of Sequence Files and Counting of Barcode Frequencies

## Information Footprints
We use information footprints as a tool for hypothesis generation to identify regions which may
contain transcription factor binding sites. In general, a mutation within a transcription factor site
is likely to severely weaken that site. We look for groups of positions where mutation away from
wild type has a large effect on gene expression. Our data sets consist of nucleotide sequences, the
number of times we sequenced the construct in the plasmid library, and the number of times we
sequenced its corresponding mRNA. A simplified data set on a 4 nucleotide sequence then might
look like

| Sequence    | DNA Counts  | mRNA Counts |
| ----------- | ----------- | ----------- |
| ACTA        | 5           | 23          |
| ATTA        | 5           | 3           |
| CCTG        | 11          | 11          |
| TAGA        | 12          | 3           |
| GTGC        | 2           | 0           |
| CACA        | 8           | 7           |
| AGGC        | 7           | 3           |

One possible calculation to measure the impact of a given mutation on expression is to take
all sequences which have base b at position i and determine the number of mRNAs produced per
read in the sequencing library. By comparing the values for different bases we could determine
how large of an effect mutation has on gene expression. However, in this paper we will use mutual
information to quantify the effect of mutation, as [4] demonstrated could be done successfully.
In Table 1 the frequency of the different nucleotides in the library at position 2 is 40% A, 32% C, 14% G and 14% T. Cytosine is enriched in the mRNA transcripts over the original library, as it
now composes 68% of all mRNA sequencing reads while A, G, and T only compose only 20%,
6%, and 6% respectively. Large enrichment of some bases over others occurs when base identity
is important for gene expression. We can quantify how important using the mutual information
between base identity and gene expression level. Mutual information is given at position i by **Equation 1**:

![I_b =  \sum_{m=0}^1  \sum_{\mu=0}^1 p(m,\mu)\log_2\left(\frac{p(m,\mu)}{p_{mut}(m)p_{expr}(\mu)}\right)](https://render.githubusercontent.com/render/math?math=I_b%20%3D%20%20%5Csum_%7Bm%3D0%7D%5E1%20%20%5Csum_%7B%5Cmu%3D0%7D%5E1%20p(m%2C%5Cmu)%5Clog_2%5Cleft(%5Cfrac%7Bp(m%2C%5Cmu)%7D%7Bp_%7Bmut%7D(m)p_%7Bexpr%7D(%5Cmu)%7D%5Cright))

![p_{mut}(m)](https://render.githubusercontent.com/render/math?math=p_%7Bmut%7D(m)) in this equation refers to the probability that a given sequencing read will be from a mutated base, and   ![p_{expr}(\mu)](https://render.githubusercontent.com/render/math?math=p_%7Bexpr%7D(%5Cmu)) is a normalizing factor that gives the ratio of the number of DNA or
mRNA sequencing counts to total number of counts.
The mutual information quantifies how much a piece of knowledge reduces the entropy of a
distribution. At a position where base identity matters little for expression level, there would be
little difference in the frequency distributions for the library and mRNA transcripts. The entropy
of the distribution would decrease only by a small amount when considering the two types of
sequencing reads separately.

We are interested in quantifying the degree to which mutation away from a wild type sequence
affects expression. Although their are obviously 4 possible nucleotides, we can classify each base
as either wild-type or mutated so that b in equation 1 represents only these two possibilities.
If mutations at each position are not fully independent, then the information value calculated
in equation 1 will also encode the effect of mutation at correlated positions. If having a mutation at
position 1 is highly favorable for gene expression and is also correlated with having a mutation at
position 2, mutations at position 2 will also be enriched amongst the mRNA transcripts. Position
2 will appear to have high mutual information even if it has minimal effect on gene expression.
Due to the DNA synthesis process used in library construction, mutation in one position can
make mutation at other positions more likely by up to 10 percent. This is enough to cloud the
signature of most transcription factors in an information footprint calculated using equation 1.

We need to determine values for ![p_{i}(m | \mu)](https://render.githubusercontent.com/render/math?math=p_%7Bi%7D(m%20%7C%20%5Cmu))

when mutations are independent, and to do this we need to fit these quantities from our data. We assert **Equation 2**: ![\left\langle mRNA \right\rangle \propto e^{-\beta E_{eff}}](https://render.githubusercontent.com/render/math?math=%5Cleft%5Clangle%20mRNA%20%5Cright%5Crangle%20%5Cpropto%20e%5E%7B-%5Cbeta%20E_%7Beff%7D%7D)
is a reasonable approximation to make. ![\left\langle mRNA \right\rangle ](https://render.githubusercontent.com/render/math?math=%5Cleft%5Clangle%20mRNA%20%5Cright%5Crangle%20)
is the average number of mRNAs produced
by that sequence for every cell containing the construct and ![E_{eff}](https://render.githubusercontent.com/render/math?math=E_%7Beff%7D) is an effective energy for the
sequence that can be determined by summing contributions from each position in the sequence.
There are many possible underlying regulatory architectures, but to demonstrate that our approach is reasonable let us first consider the simple case where there is only a RNAP site in the
studied region. We can write down an expression for average gene expression per cell as **Equation 3**:

![\left\langle mRNA \right\rangle \propto p_{bound} \propto \frac{\frac{p}{N_{NS}}e^{-\beta E_P}}{1 + \frac{p}{N_{NS}}e^{- \beta E_P}}](https://render.githubusercontent.com/render/math?math=%5Cleft%5Clangle%20mRNA%20%5Cright%5Crangle%20%5Cpropto%20p_%7Bbound%7D%20%5Cpropto%20%5Cfrac%7B%5Cfrac%7Bp%7D%7BN_%7BNS%7D%7De%5E%7B-%5Cbeta%20E_P%7D%7D%7B1%20%2B%20%5Cfrac%7Bp%7D%7BN_%7BNS%7D%7De%5E%7B-%20%5Cbeta%20E_P%7D%7D)

Where ![p_{bound}](https://render.githubusercontent.com/render/math?math=p_%7Bbound%7D)
 is the probability that the RNAP is bound to DNA and is known to be proportional to gene expression in _E. coli_, ![E_p](https://render.githubusercontent.com/render/math?math=E_p)
 is the energy of RNAP binding, ![N_{NS}](https://render.githubusercontent.com/render/math?math=N_%7BNS%7D) is the number of nonspecific DNA binding sites, and ![p](https://render.githubusercontent.com/render/math?math=p)
is the number of RNAP. If RNAP binds weakly, then ![\frac{p}{N_{NS}}e^{-\beta E_p}](https://render.githubusercontent.com/render/math?math=%5Cfrac%7Bp%7D%7BN_%7BNS%7D%7De%5E%7B-%5Cbeta%20E_p%7D)
 << 1. We can simplify equation 3 to **Equation 4**:
 
 ![\left\langle mRNA \right\rangle \propto e^{- \beta E_p}](https://render.githubusercontent.com/render/math?math=%5Cleft%5Clangle%20mRNA%20%5Cright%5Crangle%20%5Cpropto%20e%5E%7B-%20%5Cbeta%20E_p%7D)

If we assume that the energy of RNAP binding will be a sum of contributions from each of the
positions within its binding site then we can calculate the difference in gene expression between
having a mutated base at position ![i](https://render.githubusercontent.com/render/math?math=i)
 and having a wild type base as **Equation 5**:
 
 ![\frac{\left\langle mRNA_{WT_i} \right\rangle}{\left\langle mRNA_{Mut_i} \right\rangle} =& \frac{e^{- \beta E_{P_{WT_i}}}}{e^{- \beta E_{P_{Mut_i}}}}](https://render.githubusercontent.com/render/math?math=%5Cfrac%7B%5Cleft%5Clangle%20mRNA_%7BWT_i%7D%20%5Cright%5Crangle%7D%7B%5Cleft%5Clangle%20mRNA_%7BMut_i%7D%20%5Cright%5Crangle%7D%20%3D%26%20%5Cfrac%7Be%5E%7B-%20%5Cbeta%20E_%7BP_%7BWT_i%7D%7D%7D%7D%7Be%5E%7B-%20%5Cbeta%20E_%7BP_%7BMut_i%7D%7D%7D%7D)

 
 **Equation 6**:































To determine putative transcription factor binding sites, we first compute the effect of mutations
on gene expression at a base pair-by-base pair level using information footprints. The information
footprints are a hypothesis generating tool and we choose which regions to further investigate
using techniques such as mass spectrometry by visually inspecting the data for regions of 10
to 20 base pairs that have high information content compared to background. Our technique
currently relies on using human intuition to determine binding sites, but to validate these choices
and to capture all regions important for gene expression we computationally identify regions
where gene expression is changed significantly up or down by mutation (p < 0.01), and discard
any potential sites which do not fit this criteria. We infer the effect of mutation using Markov
Chain Monte Carlo, and we use the distribution of parameters from the inference to form a 99 %
confidence interval for the average effect of mutation across a 15 base pair region. We include
binding sites that are statistically significant at the 0.01 level in any of the tested growth conditions.
Many false positives will be secondary RNAP sites and we remove from consideration any
sites that resemble RNAP sites. We fit energy matrices to each of the possible binding sites and
use the preferred DNA sequence for binding to identify the RNAP sites. We use both visual
inspection to compare the preferred sequence to known consensus sequences for each of the E.
coli sigma factor binding sites (for example, do the preferred bases in the energy matrix have
few mismatches to the TGNTATAAT extended minus 10 for σ
70 sites), and the TOMTOM tool
[47] to computationally compare the potential site to examples of σ
70
, σ
38, and σ
54 sites that we
determined in this experiment. For further details see Supplementary Figure S4. We discard any
sites that have a p-value of similarity with an RNAP site of less than 5x10−3
in the TOMTOM
analysis or are deemed to be too visually similar to RNAP sites. If a single site contains an RNAP
site along with a transcription factor site we remove only those bases containing the probable
24
RNAP site. This results in 95 identified transcription factor binding regions.
For primary RNAP sites, we include a list of probable sigma factor identities as Supplementary
Table 2. Sites are judged by visual similarity to consensus binding sites. Those sites where the true
sigma factor is unclear due to overlapping binding sites are omitted. Overlapping binding sites
(from multiple TFs or RNAP sites) in general can pose issues for this method. In many cases, looking at growth conditions where only one of the relevant transcription factors is present or active is
an effective way to establish site boundaries and infer correct energy matrices. For sites where no
adequate growth condition can be found, or when a TF overlaps with an RNAP site, the energy
matrix will not be reflective of the true DNA-protein interaction energies. If the TFs in overlapping
sites are composed of one activator and one repressor, then we use the point at which the effect
of mutation shifts from activator-like to repressor-like as a demarcation point between binding
sites. We see a case of a potentially overlooked repressor due to overlapping sites in Figure 4(B),
as there are several repressor like bases overlapping the RNAP -10 site and the effect weakens in
low oxygen growth. However, due to the effect of the RNAP site, when averaged over a potential 15 base pair region, the repressor-like bases do not have a significant effect on gene expression