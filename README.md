
# **ULTRA: A Query-Length-Adaptive Routing Framework for Dual-Embedding-Based Semantic News Recommendation in Urdu**

## **Overview**
This repository contains the implementation and supporting resources for ULTRA, a query-length-adaptive routing framework for semantic news recommendation in the Urdu language.

ULTRA employs a dual-embedding architecture that represents news articles using two complementary semantic representations: headline embeddings and full-content embeddings. A query-length-adaptive routing mechanism determines which representation should be used based on the length of the input query. Short queries are routed to the headline embedding pathway, whereas longer queries are routed to the full-content embedding pathway. This design aims to improve the suitability of semantic representations for queries of different lengths while maintaining computational efficiency.

The repository provides the preprocessing pipeline, embedding generation procedures (with pooling strategy analysis), dimensionality-reduction experiments (including comparisons of PCA, UMAP, and Autoencoders), and PCA-based dimension comparisons required to reproduce the reported experimental setup.

---

## **System Architecture**

The following diagram illustrates the overall architecture of the ULTRA framework, showing the dual-embedding pipeline and the query-length-adaptive routing mechanism:

![ULTRA System Architecture](ULTRA_Architecture (1).pdf)

*Figure 1: ULTRA architecture showing the dual pathways for headline (short query) and full-content (long query) embeddings, with PCA dimensionality reduction and ChromaDB vector storage.*

The architecture consists of two main processing pathways:

1. **Headline Pathway (Short Queries):** 
   - Processes queries with length < 150 characters
   - Uses CLS pooling with PCA 64D reduction
   - Optimized for concise semantic retrieval

2. **Content Pathway (Long Queries):**
   - Processes queries with length ≥ 150 characters
   - Uses chunking with 50-token overlap and Mean pooling
   - Applies PCA 128D reduction
   - Captures richer contextual information

The routing mechanism dynamically selects the appropriate pathway based on query length, ensuring optimal performance and efficiency.

## **Dataset**
The experiments use the Urdu News Dataset, originally obtained from Kaggle:

- **Dataset:** Urdu News Dataset
- **Original dataset source:** Mendeley Data
- **Kaggle distribution:** https://www.kaggle.com/datasets/saurabhshahane/urdu-news-dataset
- **Original license:** CC BY 4.0

The dataset contains Urdu news articles belonging to four major categories:

1. **Business & Economics**
2. **Science & Technology**
3. **Entertainment**
4. **Sports**

Following preprocessing, cleaning, duplicate removal, and filtering, approximately **111,853 articles** are retained for the experiments.

### **Dataset Citation**
The original dataset should be cited as follows:

Hussain, Khalid; Mughal, Nimra; Ali, Irfan; Hassan, Saif; Daudpota, Sher Muhammad (2021). Urdu News Dataset 1M. Mendeley Data, V3. https://doi.org/10.17632/834vsxnb99.3

---

## **Repository Structure and Experimental Notebooks**
The repository contains the following notebooks, which implement the main stages of the experimental pipeline.

### **1. Preprocessing.ipynb**
This notebook performs the preprocessing and normalization of the raw Urdu news dataset. The main operations include:

- Loading the original dataset.
- Correcting text encoding issues affecting Urdu characters.
- Removing irrelevant metadata columns, including Index, Date, URL, Source, and News Length.
- Combining the Headline and News Text fields into a unified content representation.
- Removing HTML tags, punctuation, special characters, and redundant whitespace.
- Removing a predefined set of 127 Urdu stop-words.
- Removing missing/null records.
- Removing exact duplicate articles.
- Filtering out entries containing fewer than 15 characters.

The stop-word removal procedure reduces the corpus size by approximately **33.5%**, thereby reducing the amount of textual data processed during subsequent stages.

---

### **2. Content_Col_Embeddings.ipynb**
This notebook generates transformer-based semantic embeddings for the full article content using the pre-trained Urdu model **urduhack/roberta-urdu-small** (768-dimensional contextualized embeddings).

#### **Pooling Strategy Analysis:**
Three pooling methods were tested and compared to aggregate token embeddings into a fixed-dimensional sentence embedding:

- **Mean Pooling** – averages all token embeddings (centroid of semantic information)
- **Max Pooling** – extracts the maximum value across each dimension (emphasizes salient features)
- **CLS Pooling** – uses the special classification token's embedding (captures sentence-level semantics)

Initial evaluation revealed that the optimal pooling strategy depends on the nature of the text. For full-content (long) embeddings, **Mean Pooling** was found to be most effective and is used in the final content pathway.

When article content exceeds the model's maximum sequence length (512 tokens), a chunking strategy with 50-token overlap is applied, and the final embedding is obtained by averaging the chunk embeddings.

---

### **3. Content_Col_Embeddings_DR.ipynb**
This notebook applies dimensionality-reduction techniques to the generated full-content embeddings in order to reduce their dimensionality while retaining relevant semantic information.

#### **Dimensionality Reduction Methods Compared:**
- **PCA** (Principal Component Analysis)
- **UMAP** (Uniform Manifold Approximation and Projection)
- **Autoencoders**

**PCA** was found to be more effective than the non-linear alternatives (UMAP, Autoencoders) across content embeddings, providing the highest overlap scores with the original 768-dimensional space while being deterministic and computationally efficient. PCA was therefore selected for the content retrieval pathway.

---

### **4. Content_Col_PCA_64D_128D_256D.ipynb**
This notebook evaluates PCA-based dimensionality reduction for the full-content embeddings at three target dimensions:

- **64 dimensions**
- **128 dimensions**
- **256 dimensions**

The comparison is used to investigate the trade-off between representation dimensionality, computational efficiency, and recommendation performance. For content embeddings (long-query pathway), **128 dimensions** provided the most favorable tradeoff, retaining high overlap with the original results.

---

### **5. Headline_Col_Embeddings.ipynb**
This notebook generates transformer-based semantic embeddings from news headlines using the pre-trained Urdu model **urduhack/roberta-urdu-small**.

#### **Pooling Strategy Analysis:**
The same three pooling methods were evaluated:

- **Mean Pooling**
- **Max Pooling**
- **CLS Pooling**

For headline (short) embeddings, **CLS Pooling** was found to be most effective and is used in the final headline pathway. These embeddings provide a compact representation particularly suitable for short user queries.

---

### **6. Headline_Col_Embeddings_DR.ipynb**
This notebook performs dimensionality reduction on the headline embeddings.

#### **Dimensionality Reduction Methods Compared (as detailed in the paper):**
- **PCA** (Principal Component Analysis)
- **UMAP** (Uniform Manifold Approximation and Projection)
- **Autoencoders**

**PCA** was found to be more effective than the non-linear alternatives across headline embeddings as well, and was selected for the headline retrieval pathway.

---

### **7. Headline_Col_PCA_64D_128D_256D.ipynb**
This notebook compares PCA-reduced headline embeddings at:

- **64 dimensions**
- **128 dimensions**
- **256 dimensions**

The objective is to identify an appropriate reduced representation while considering both semantic quality and computational efficiency. For headline embeddings (short-query pathway), **64 dimensions** was the optimal size that maintained the meaning of short headlines without a significant drop in retrieval quality.

---

## **Methodology**
The ULTRA preprocessing and representation pipeline consists of five primary stages.

### **Stage 1: Dataset Loading and Encoding Correction**
The raw dataset is loaded and examined for encoding inconsistencies. Encoding corrections are applied to ensure that Urdu characters are represented correctly throughout the processing pipeline.

### **Stage 2: Metadata Removal and Content Construction**
Non-essential metadata fields are removed. The headline and news text are subsequently combined to construct a unified article-content representation.

### **Stage 3: Text Normalization and Cleaning**
The textual content is normalized by removing HTML tags, unwanted characters, punctuation, and redundant whitespace. The resulting text is used as the input for subsequent semantic representation.

### **Stage 4: Urdu Stop-Word Removal**
A predefined list of 127 Urdu stop-words is applied to remove frequently occurring function words that provide limited discriminative information for semantic retrieval. This procedure substantially reduces the textual corpus while retaining the principal semantic content.

### **Stage 5: Data Filtering**
Records containing null values, exact duplicates, or fewer than 15 characters are removed. The resulting corpus contains approximately **111,853** news articles.

---

## **Dual-Embedding Representation**
ULTRA generates two independent semantic representations for each news article:

- **Headline representation:** generated from the article headline using CLS pooling, then reduced with PCA to **64D**. Optimized for short textual queries.
- **Full-content representation:** generated from the complete article content using Mean pooling, then reduced with PCA to **128D**. Intended to capture richer semantic information available in longer documents. Chunking with 50-token overlap is applied when content exceeds the model's maximum sequence length.

This dual-representation strategy enables the system to select an embedding space that is more appropriate for the characteristics of the incoming query.

---

## **Query-Length-Adaptive Routing**
The central component of ULTRA is a query-length-adaptive routing mechanism. The mechanism uses query length as a routing signal to select between the headline and full-content embedding pathways. A threshold **θ = 150 characters** (selected based on the distribution of headline lengths in the corpus) is used:

- **Short query (ℓ(q) < θ)** → Headline embedding pathway (CLS pooling + PCA 64D)
- **Long query (ℓ(q) ≥ θ)** → Full-content embedding pathway (Mean pooling + PCA 128D)

The underlying motivation is that short queries may be better aligned with concise headline-level representations, whereas longer queries generally provide additional contextual information that can benefit from representations derived from the complete article content.

---

## **Dimensionality Reduction**
To reduce the computational and storage requirements associated with high-dimensional transformer embeddings (768D), dimensionality-reduction experiments are conducted. Four methods were evaluated:

- **PCA** (Principal Component Analysis)
- **UMAP** (Uniform Manifold Approximation and Projection)
- **Autoencoders**

**PCA**, a linear approach that preserves maximum variance, outperformed the non-linear alternatives (UMAP and Autoencoders) in terms of overlap with the original ranking results for both content and headline embeddings. It is deterministic, fast, and was therefore selected for both retrieval pathways.

After selecting PCA, three target dimensionalities were evaluated:

- **64D**
- **128D**
- **256D**

### **Optimal Configurations Identified:**
- **Content (long-query) pathway:** 128 dimensions
- **Headline (short-query) pathway:** 64 dimensions

The reduced embeddings are stored in separate ChromaDB vector databases with HNSW indexing for efficient cosine-similarity search.

---

## **Reproducibility**
The complete implementation is available in the following repository:

**ULTRA Code Repository:** https://github.com/alishbahbashir2004-blip/ULTRA_Codes

To reproduce the preprocessing and embedding pipeline:

1. Download the original Urdu News Dataset from the Kaggle distribution.
2. Clone the ULTRA repository.
3. Install the required Python dependencies.
4. Execute **Preprocessing.ipynb** to generate the cleaned dataset.
5. Execute the headline and full-content embedding notebooks (which include pooling strategy comparison: Mean, Max, and CLS pooling).
6. Execute the corresponding dimensionality-reduction notebooks (which compare PCA, UMAP, and Autoencoders).
7. Run the PCA comparison notebooks to evaluate 64D, 128D, and 256D representations and select the optimal dimensions (128D for content, 64D for headlines).

---

## **Software Requirements**
The experiments require the following software environment:

- **Python 3.9** or later
- **Jupyter Notebook**
- **NumPy**
- **pandas**
- **scikit-learn**
- **Matplotlib**
- **Seaborn**
- **TensorFlow**
- **Keras**
- **Transformers**
- **Sentence-Transformers**
- **ChromaDB** (for vector storage)

The primary dependencies can be installed using:

```bash
pip install numpy pandas scikit-learn matplotlib seaborn tensorflow keras transformers sentence-transformers chromadb
```

---

## **Computational Environment**
The experiments were conducted on a workstation with the following approximate configuration:

- **Operating System:** Ubuntu 22.04
- **CPU:** Intel Core i7
- **RAM:** 64 GB
- **GPU:** NVIDIA RTX A6000

---

## **License**
The original Urdu News Dataset is distributed under the **CC BY 4.0** license. Users of the dataset should comply with the terms of the original dataset license and provide appropriate attribution.

The source code contained in this repository is released under the **MIT License**.

---

## **Contribution**
Contributions to the repository are welcome. Researchers and developers may submit issues or pull requests to report reproducibility problems, identify bugs, improve preprocessing or embedding procedures, or extend the ULTRA framework.

---

## **Repository**
**ULTRA Code Repository:**  
https://github.com/alishbahbashir2004-blip/ULTRA_Codes

**Dataset:**  
https://www.kaggle.com/datasets/saurabhshahane/urdu-news-dataset
