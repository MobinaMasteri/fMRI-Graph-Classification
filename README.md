# AD Graph-Based Classification

A three-stage graph-based pipeline for Alzheimer's Disease (AD) classification using resting-state fMRI data.

This project processes fMRI time-series data, converts each subject's brain activity into a functional connectivity graph, measures similarities between subjects, and finally uses a Graph Convolutional Network (GCN) for classification.

---

## Project Overview

The project consists of three main stages:

1. **Stage 1 — Brain Functional Connectivity Graph Construction**
2. **Stage 2 — Subject Similarity Graph Construction**
3. **Stage 3 — Alzheimer's Disease Classification Using GCN**

The overall workflow is:

```text
fMRI Data
    │
    ▼
Stage 1
Functional Connectivity Graph for Each Subject
    │
    ▼
Stage 2
Subject Similarity Graph
    │
    ▼
Stage 3
GCN Classification
    │
    ▼
Prediction Results
Accuracy
Confusion Matrix
```

---

# 1. Dataset

This project uses fMRI time-series data stored in the `rois_aal` directory.

The dataset is not included directly in this repository. It can be downloaded using the instructions provided in:

```text
DATASET.md
```

Download the `rois_aal` dataset from the following Google Drive link:

https://drive.google.com/file/d/19MvA3VDgXzv9KyyYDljTH1xss3vcmSyz/view?usp=drivesdk

After downloading the dataset, place the `rois_aal` folder directly inside the project root.

The project also requires:

```text
Phenotypic_V1_0b_preprocessed1.csv
```

This file must also be placed directly inside the project root.

---

# 2. Project Structure

The project has the following structure:

```text
AD-Graph-Based-Classification/
│
├── rois_aal/
│   ├── subject_1.1D
│   ├── subject_2.1D
│   ├── ...
│   └── ...
│
├── stage1.py
├── stage2.py
├── stage3.py
│
├── Phenotypic_V1_0b_preprocessed1.csv
│
├── stage1_outputs/
├── stage2_outputs/
├── stage3_outputs/
│
├── DATASET.md
├── requirements.txt
├── .gitignore
└── README.md
```

The output directories are generated automatically when the corresponding stages are executed.

---

# 3. Requirements

The project is implemented in Python.

The main libraries used are:

* NumPy
* Pandas
* NetworkX
* Matplotlib
* Scikit-learn
* PyTorch

All required packages are listed in:

```text
requirements.txt
```

---

# 4. Installation

## Step 1 — Install Python

Make sure Python is installed on your computer.

You can check the installed version using:

```bash
python --version
```

Python 3.10+ is recommended.

---

## Step 2 — Clone the Repository

Clone this repository:

```bash
git clone <YOUR_GITHUB_REPOSITORY_LINK>
```

Then enter the project directory:

```bash
cd AD-Graph-Based-Classification
```

---

## Step 3 — Install Requirements

Install all required Python packages:

```bash
pip install -r requirements.txt
```

---

# 5. Prepare the Dataset

Before running the project, make sure the required files are located in the project root.

The required structure is:

```text
AD-Graph-Based-Classification/
│
├── rois_aal/
│
├── Phenotypic_V1_0b_preprocessed1.csv
│
├── stage1.py
├── stage2.py
└── stage3.py
```

The Python scripts automatically locate these files relative to the project directory.

Therefore, no username-specific or computer-specific paths are required.

---

# 6. Stage 1 — Functional Connectivity Graph Construction

File:

```text
stage1.py
```

## Purpose

Stage 1 processes the fMRI data of each subject and constructs a functional connectivity graph representing relationships between brain regions.

Each `.1D` file represents the fMRI time-series data of one subject.

---

## Processing Steps

For each subject:

### 1. Read the fMRI data

The `.1D` file is loaded using NumPy.

### 2. Calculate the correlation matrix

Pearson correlation is calculated between the time series of different brain regions.

The result is a correlation matrix representing functional connectivity between regions.

### 3. Select a correlation threshold

Several threshold values are tested:

```text
0.3
0.4
0.5
0.6
0.7
```

The program selects a threshold according to the predefined graph-density criterion.

A random sample of 100 subjects is used during threshold selection.

The random seed is fixed to:

```text
42
```

to make the selection reproducible.

### 4. Construct the graph

Each brain region is represented as a node.

An edge is created between two regions when:

```text
|correlation| >= selected threshold
```

The edge contains:

* Correlation
* Distance

where distance is calculated as:

```text
distance = 1 - |correlation|
```

### 5. Calculate graph information

For each subject, the program records information such as:

* Number of nodes
* Number of edges
* Graph density
* Strongest connection
* Weakest connection
* Selected threshold

### 6. Save the results

The results are saved in:

```text
stage1_outputs/
```

Each subject has its own output directory.

---

## Run Stage 1

From the project root:

```bash
python stage1.py
```

After successful execution, the Stage 1 results will be stored in:

```text
stage1_outputs/
```

---

# 7. Stage 2 — Subject Similarity Graph

File:

```text
stage2.py
```

## Purpose

Stage 2 compares subjects with each other based on their functional connectivity patterns.

Instead of representing brain regions as nodes, Stage 2 represents the **subjects** as nodes.

---

## Processing Steps

### 1. Calculate the correlation matrix

The fMRI data of each subject is processed again to obtain its functional connectivity matrix.

### 2. Extract the upper triangle

Only the upper triangular part of the correlation matrix is used.

This avoids duplicate connections because the correlation matrix is symmetric.

The resulting values form a feature vector for each subject.

### 3. Calculate subject similarity

The feature vectors of subjects are compared using:

```text
Cosine Similarity
```

A higher similarity value means that the functional connectivity patterns of two subjects are more similar.

### 4. Select a similarity threshold

The program tests several similarity thresholds:

```text
0.10
0.20
0.30
0.40
0.50
0.60
```

A random sample of 100 subjects is used for threshold selection.

The random seed is:

```text
42
```

### 5. Construct the subject graph

Each subject becomes a node.

Two subjects are connected when their similarity is greater than or equal to the selected threshold.

The edge weight represents their similarity.

### 6. Save the results

The results are saved in:

```text
stage2_outputs/
```

An important output file is:

```text
person_graph_edges.csv
```

This file contains the connections between subjects and their similarity values.

---

## Run Stage 2

After Stage 1 has been completed, run:

```bash
python stage2.py
```

The results will be stored in:

```text
stage2_outputs/
```

---

# 8. Stage 3 — Alzheimer's Disease Classification

File:

```text
stage3.py
```

## Purpose

Stage 3 uses the subject similarity graph created in Stage 2 to classify subjects into Alzheimer's Disease-related diagnostic groups.

A:

```text
Graph Convolutional Network (GCN)
```

implemented using PyTorch is used for classification.

---

## Input Data

Stage 3 uses:

```text
stage2_outputs/person_graph_edges.csv
```

and:

```text
Phenotypic_V1_0b_preprocessed1.csv
```

The phenotypic file provides the diagnostic labels associated with subjects.

---

## Node Features

For each subject, two graph-based features are used:

```text
1. Degree
2. Clustering coefficient
```

These features describe the subject's position and connectivity within the subject similarity graph.

---

## Graph Convolutional Network

The GCN consists of two graph convolution layers.

The model uses:

```text
Input features: 2
Hidden layer: 16
Output classes: 2
```

ReLU activation is used between the layers.

The model is trained using:

```text
Adam optimizer
Learning rate: 0.01
Epochs: 200
```

Cross-entropy loss is used for classification.

---

## Training and Testing

The subjects are divided into training and testing sets.

The split uses:

```text
80% Training
20% Testing
```

A stratified split is used so that the class distribution is preserved as much as possible.

The random state is fixed to:

```text
42
```

---

## Evaluation

The model is evaluated using:

### Accuracy

Accuracy represents the proportion of correctly classified subjects among all tested subjects.

```text
Accuracy =
Correct Predictions / Total Predictions
```

### Confusion Matrix

The confusion matrix shows how many subjects from each actual class were classified into each predicted class.

It helps examine the classification results for both classes separately.

---

## Stage 3 Outputs

The results are saved in:

```text
stage3_outputs/
```

Important outputs include:

```text
stage3_report.txt
prediction_results.csv
```

The report contains information such as:

* Number of subjects
* Training set size
* Test set size
* Accuracy
* Confusion matrix

The prediction results contain the actual and predicted labels for the subjects in the test set.

---

## Run Stage 3

After completing Stage 2, run:

```bash
python stage3.py
```

The results will be stored in:

```text
stage3_outputs/
```

---

# 9. Complete Execution Order

The stages should be executed in the following order:

### Step 1

Prepare:

```text
rois_aal/
Phenotypic_V1_0b_preprocessed1.csv
```

inside the project root.

### Step 2

Run:

```bash
python stage1.py
```

### Step 3

Run:

```bash
python stage2.py
```

### Step 4

Run:

```bash
python stage3.py
```

The complete workflow is:

```text
Dataset
   │
   ▼
stage1.py
   │
   ▼
stage1_outputs/
   │
   ▼
stage2.py
   │
   ▼
stage2_outputs/
   │
   └── person_graph_edges.csv
              │
              ▼
         stage3.py
              │
              ▼
       stage3_outputs/
              │
              ├── stage3_report.txt
              └── prediction_results.csv
```

---

# 10. Reproducibility

Several random operations are used in the project.

To make the results more reproducible, the project uses:

```text
random seed = 42
```

This seed is used during:

* Random subject sampling
* Threshold selection
* Train/test splitting

The exact numerical results can still depend on the software environment and installed library versions.

---

# 11. Output Directories

The project generates three main output directories:

```text
stage1_outputs/
stage2_outputs/
stage3_outputs/
```

### Stage 1

Contains the functional connectivity graph results for individual subjects.

### Stage 2

Contains the subject similarity graph and related reports/visualizations.

### Stage 3

Contains classification results, including:

```text
stage3_report.txt
prediction_results.csv
```

These outputs can be inspected after running the corresponding stages.

---

# 12. Technologies Used

The project uses the following technologies and libraries:

| Technology   | Purpose                                   |
| ------------ | ----------------------------------------- |
| Python       | Main programming language                 |
| NumPy        | Numerical and matrix operations           |
| Pandas       | Tabular and phenotypic data processing    |
| NetworkX     | Graph construction and analysis           |
| Matplotlib   | Graph visualization                       |
| Scikit-learn | Similarity, data splitting and evaluation |
| PyTorch      | GCN implementation and model training     |

---

# 13. Project Workflow Summary

The project transforms raw fMRI data into a classification result through three graph-based stages.

### Stage 1

```text
fMRI Time Series
       ↓
Correlation Matrix
       ↓
Thresholding
       ↓
Brain Functional Connectivity Graph
```

### Stage 2

```text
Subject fMRI Features
       ↓
Cosine Similarity
       ↓
Subject Similarity Graph
```

### Stage 3

```text
Subject Similarity Graph
       +
Phenotypic Labels
       ↓
Graph Features
       ↓
GCN
       ↓
Classification
       ↓
Accuracy + Confusion Matrix + Predictions
```

---

# 14. Notes

* The dataset must be placed in the correct location before running the scripts.
* The three stages should be executed in order.
* `stage2.py` depends on the data used by Stage 1.
* `stage3.py` depends on the output generated by Stage 2.
* Do not rename the required dataset files unless the corresponding Python code is also updated.
* The scripts use paths relative to the project directory, so the project can be placed on another computer without changing the paths manually.

---

# 15. Author

This project was developed as an academic project in Computer Engineering.

**Project:** AD Graph-Based Classification

**Main approach:** fMRI Functional Connectivity + Graph Analysis + Graph Convolutional Network (GCN)
