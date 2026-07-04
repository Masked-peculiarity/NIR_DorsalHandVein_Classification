# Biometric classification of DHV - models - Training, Testing and Results

This Repository is a collaborative work of three members about the comphrehensive comparative model analysis of the dorsal hand vein dataset : https://github.com/wilchesf/dorsalhandveins
### Github member IDs : 
1) Masked-peculiarity
2) thenameisyashwanth-sudo
3) Sirthebeast

This README is a high level architectural / pipeline summary of this repository 
All the technical implementations and explanations will be uploaded further.
# Dataset Clarification

The dataset contains two databases DB-1 & DB-2 
 - Database 1 -- 138 people, 4 images for each hand for a total of 1104 pictures
 - Database 2 -- 113 people, 3 images for each hand for a total of 678 pictures --- taken by a NIR camera
 - The persons labeled from 114 to 138 were not available for the recapture of vein images for DB-2
 - The time gap between the images from DB-1 and DB-2 is 2 months
 - The naming convention for the left hand images in both the DBs follows : person_xxx_dbx_Lx.tif / png
 - The naming convention for the right hand images in both the DBs follows : person_xxx_dbx_Rx.tif / png
 - All the images in the both the DBs follow 752×560 pixels, 16-bit quantization, TIFF format and PNG format

Since the Databse has a 2 month gap, We decided to use the entire DB-1 images for training and DB-2 images for testing 
This leaves with the problem of unavailability of images for testing from persons 114 to 138 in DB-2, Hence the persons (114 - 138) were dropped 
during testing, So a total of 904 images was used for training and 678 images was used for testing.
This decision was made to improve the robustness of the models which we were to train.

The above is the convention for all the experiments that were done.
Further data splitting strategies were made in the experiments.

# Region of Interest Extraction and Image Enhacement 

The following pipeline applied to the dorsal hand vein dataset prior to classification. <br>
The pipeline converts raw NIR hand images into standardised 128×128 grayscale patches containing only the central palm vein region, free of 
background, finger edges, and wrist artifacts.

***
**→ Raw NIR Image (752×560) → Gaussian Blur (5×5) → Otsu Thresholding → Binary Mask → Morphological Closing (15×15 ellipse, 3 iterations) → Largest Contour Extraction → Solid Hand Mask → Distance Transform → Largest Inscribed Circle (cx, cy, r) → Largest Inscribed Square (half_s = r × 0.7071) → Square Crop from Raw Grayscale (≈154×154 px) → Lanczos4 Resize → 128×128 → Standardised ROI**
***
Further image processing algorithms were used to capture the morphological and structure of the images of Region of Interest (ROI)

1. **Black Hat Transform** was used to highlight small, dark regions or details that are darker than their immediate surroundings and smaller than the chosen structuring element, This is mainly used for Feature collection and Background correction.

2. **Frangi Vesselness Filter** was used to act as a shape detector that selectively enhances thin, elongated, and tubular structures while suppressing background noise or spherical objects (like dust). The primary use case of this was Vessel Extraction.

Separate images for both the image processing algorithms was processed and stored in the Database for training the models not only on ROI, but to act as a comparision..

***
Dependencies that was used <br>
- opencv-python    >= 4.5  
- numpy            >= 1.21  
- scikit-image     >= 0.19  
***

Full technical summary is to be uploaded...


The below is the high level explanation of the experiments ( based on the literature survey which was done prior ) performed and their results...



# Experiment - 01
# Classical Machine Learning pipeline (SVM-rbf, Random Forest, LDA) with Handcrafted Feature Extraction

Classical machine learning algorithms can achieve strong performance on small and medium-sized biometric datasets when paired with carefully designed feature extraction techniques.
<br>
A major objective of this work was to understand which vein characteristics contribute to recognition performance.
The handcrafted feature pipeline provides interpretable descriptors:

- HOG captures vein orientation and edge structure.
- LBP captures local vein texture patterns.
- Grid Statistics capture spatial intensity distributions.
Unlike deep neural networks, where learned representations are often difficult to interpret, these features have clear physical meaning and can be directly related to vein morphology.

## System Architecture

**Raw NIR Image → ROI Extraction → Vein Enhancement (Blackhat / Frangi) → Feature Extraction (HOG + LBP + Grid Statistics) → Feature Fusion → Standardization → PCA Dimensionality Reduction →  Machine Learning Classifiers → Identification & Verification**

## Methodology
## Feature Extraction
Instead of deep feature learning, discriminative handcrafted vein descriptors were extracted and fused into a single feature representation.

**HOG Features (Histogram of Oriented Gradients)**

HOG captures:
-  Vein edge orientations
- Local structural information
- Directional vein flow patterns

These descriptors encode the geometric arrangement of vein structures.

**LBP Features (Local Binary Patterns)**

LBP captures:
- Local texture information
- Micro-pattern variations
- Fine vessel textures

Uniform LBP encoding was used to improve robustness.

**Grid-Based Statistical Features**

The ROI is divided into a 4×4 grid.

For each cell: Mean intensity & Standard deviation are computed.

These features capture global vein distribution characteristics across the hand.

**Feature Fusion**

The final biometric representation is obtained by concatenating:  HOG Features + LBP Histogram + Grid Statistics
This creates a comprehensive feature vector containing:

- Structural information
- Texture information
- Spatial distribution information
- Dimensionality Reduction

Feature vectors are standardized using: StandardScaler

Followed by:

Principal Component Analysis (PCA)
PCA Configuration
Reduced to 30 principal components
Trained exclusively on DB1
Applied to DB2 without refitting

This prevents data leakage and preserves experimental integrity.

## Classification Models

Three classifiers were evaluated.

1) **Support Vector Machine (RBF Kernel)**

Characteristics:

-Non-linear decision boundaries
-Strong generalization capability
-Effective in high-dimensional spaces

2) **Random Forest**

Characteristics:

- Ensemble-based learning
- Robust to feature noise
- Captures complex decision boundaries

3) **Linear Discriminant Analysis (LDA)**

Characteristics:

- Maximizes class separability
- Computationally efficient
- Well suited for biometric identification tasks
- Identification Evaluation

## Results
Identification was performed as a 226-class classification problem.

Metrics
Rank-1 Accuracy, Rank-5 Accuracy, Confusion Matrix Analysis, Precision, Recall, F1 Score

### **Identification Results**

| Model           | Rank-1 Accuracy | Rank-5 Accuracy |
|----------------|-----------------|-----------------|
| SVM (RBF)      | 80.09%          | 84.96%          |
| Random Forest  | 65.04%          | 83.19%          |
| LDA            | 80.38%          | 88.79%          |

### **Observation**

LDA achieved the highest identification performance, obtaining:

80.38% Rank-1 Accuracy
88.79% Rank-5 Accuracy

despite the challenging two-month acquisition gap.

**Verification Evaluation**

Verification was performed using balanced genuine/impostor comparisons.

For each selected identity:

Genuine samples were collected.
Equal numbers of impostor samples were randomly selected.
Binary classifiers were trained and evaluated independently.
Metrics-  Equal Error Rate (EER), Area Under ROC Curve (AUC)

### **Verification Results**

| Model          | EER ↓  | AUC ↑  |
|---------------|--------|--------|
| SVM (RBF)     | 15.56% | 0.8593 |
| Random Forest | 17.78% | 0.8111 |
| LDA           | 35.56% | 0.6889 |

### **Observation**

**SVM** demonstrated the strongest verification capability with: **15.56%- Equal Error Rate, 0.8593 - AUC**
indicating superior genuine/impostor discrimination.

### Advanced Performance Analysis

**For the best identification model (LDA):

Metric	Value
Macro Precision - 	77.07%
Macro Recall	- 80.38%
Macro F1 Score	- 77.06%**

Confusion analysis revealed that most errors occurred between identities exhibiting highly similar dorsal vein structures or left/right hand symmetry.

### Key Contributions
- Automated anatomical ROI extraction using the largest inscribed circle and square strategy.
- Robust vein enhancement using Blackhat and Frangi filtering.
- Hybrid handcrafted feature representation combining HOG, LBP, and statistical descriptors.
- Leakage-free PCA and chronological train-test separation.
- Comprehensive evaluation under a realistic two-month temporal gap.
- Simultaneous assessment of both identification and verification performance.



# Experiment - 02
# Deep learning architecture - CNN and Pretrained Model (ResNet-18)

Unlike the classical machine learning pipeline, which relies on handcrafted feature extraction, this experiment employs deep neural networks to automatically learn discriminative vein representations directly from ROI images.

Two complementary architectures are investigated:

- Custom Convolutional Neural Network (VeinCNN)
- Transfer Learning with ResNet-18

## System Architecture

**Raw NIR Image → ROI Extraction → Vein Enhancement → Image Preprocessing (Resize + Normalization) → Deep Neural Network (CNN & ResNet-18) → Deep Feature Learning → 226-Class Softmax Classifier → Prediction**

## Deep Learning Models

### Custom VeinCNN

A lightweight convolutional neural network was developed specifically for dorsal hand vein recognition.

The network progressively learns:

- Edge information
- Local vein textures
- Vascular structures
- Identity-specific biometric features

The architecture provides a computationally efficient solution while learning discriminative feature representations directly from the input images.

### ResNet-18

A transfer learning approach based on ResNet-18 was implemented to leverage deep residual feature learning.

The pretrained architecture was modified for dorsal hand vein recognition by:

- Adapting the first convolutional layer for grayscale input
- Replacing the final fully connected layer with a 226-class classifier
- Fine-tuning the network on dorsal hand vein images

Residual learning enables deeper feature extraction while reducing optimization difficulties associated with deeper networks.

<br>

## Deep Feature Learning

Unlike handcrafted descriptors such as HOG or LBP, the neural networks automatically learn hierarchical biometric representations during training.
The learned feature embeddings encode:

- Vein geometry
- Vessel connectivity
- Local vascular texture
- Global vein distribution
- Identity-specific anatomical characteristics
These learned representations replace manual feature engineering and are optimized directly through supervised learning.

## Training Strategy

Both networks were trained using supervised learning on DB1. The training pipeline includes:

- Cross-Entropy Loss
- Adam Optimizer
- Mini-batch training
- Validation monitoring
- Model checkpointing

The trained models are evaluated exclusively on DB2, ensuring that no information from the testing session influences training.

## Identification Evaluation

Identification is formulated as a 226-class closed-set classification problem. Each probe image is assigned to one of the enrolled left/right hand identities.

Evaluation Metrics : Rank-1 Accuracy, Rank-5 Accuracy, Precision, Recall, F1 Score, Confusion Matrix

## Identification Performance

| Model          | Rank-1 Accuracy | Rank-5 Accuracy |
|----------------|----------------:|----------------:|
| Custom VeinCNN | 51.18%          | 71.83%          |
| ResNet-18      | 73.16%          | 87.02%          |

### Observation

The ResNet-18 model substantially outperformed the custom CNN, achieving a 73.16% Rank-1 accuracy and 87.02% Rank-5 accuracy. The deeper residual architecture demonstrated a stronger ability to learn discriminative vein representations across the 226 biometric classes despite the temporal separation between enrollment and testing sessions.


## Verification Performance

Deep feature embeddings extracted from the trained models were further evaluated for biometric verification.

| Model          | Equal Error Rate (EER ↓) | AUC (↑) |
|----------------|-------------------------:|--------:|
| Custom VeinCNN | **9.99%**                | **0.9650** |
| ResNet-18      | **5.45%**                | **0.9878** |

### Observation

ResNet-18 also achieved the strongest verification performance, reducing the Equal Error Rate to 5.45% while obtaining an AUC of 0.9878, indicating highly discriminative feature embeddings for genuine and impostor pair separation.

<br>
<br>

This experiment investigates deep learning as an alternative to handcrafted feature engineering by enabling the model to learn biometric representations directly from image data.

The approach was selected because it offers:

- Automatic extraction of hierarchical vein representations
- Reduced dependence on manually designed descriptors
- Better scalability to larger biometric datasets
- Improved capability to model complex vascular patterns

By comparing a lightweight custom CNN with a deeper transfer learning architecture, the project explores the trade-off between computational efficiency and recognition performance.

## Key Contributions
- Adaptation of ResNet-18 for grayscale NIR dorsal hand vein images.
- 226-class biometric identification framework treating left and right hands as independent identities.
- Deep feature learning without handcrafted feature engineering.
- Comprehensive assessment using both identification and verification metrics.


# Experiment - 03
