# Biometric classification of DHV - models - Training, Testing and Results

This Repository is a collaborative work of three members meddling with the dorsal hand vein dataset : https://github.com/wilchesf/dorsalhandveins
# Github member IDs : 
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
 - the naming convention for the left hand images in both the DBs follows : person_xxx_dbx_Lx.tif / png
 - the naming convention for the right hand images in both the DBs follows : person_xxx_dbx_Rx.tif / png
 - all the images in the both the DBs follow 752×560 pixels, 16-bit quantization, TIFF format and PNG format

Since the Databse has a 2 month gap, We decided to use the entire DB-1 images for training and DB-2 images for testing 
This leaves with the problem of unavailability of images for testing from persons 114 to 138 in DB-2, Hence the persons (114 - 138) were dropped 
during testing, So a total of 904 images was used for training and 678 images was used for testing.
This decision was made to improve the robustness of the models which we were to train.

The above is the convention for all the experiments that were done.
Further data splitting strategies were made in the experiments.

# Region of Interest Extraction and Image Enhacement 

The following pipeline applied to the dorsal hand vein dataset prior to classification. 
The pipeline converts raw NIR hand images into standardised 128×128 grayscale patches containing only the central palm vein region, free of 
background, finger edges, and wrist artifacts.

***
→ Raw NIR Image (752×560) → Gaussian Blur (5×5) → Otsu Thresholding → Binary Mask → Morphological Closing (15×15 ellipse, 3 iterations) → Largest Contour Extraction → Solid Hand Mask → Distance Transform → Largest Inscribed Circle (cx, cy, r) → Largest Inscribed Square (half_s = r × 0.7071) → Square Crop from Raw Grayscale (≈154×154 px) → Lanczos4 Resize → 128×128 → Standardised ROI 
***
Further image processing algorithms were used to capture the morphological and structure of the images of Region of Interest (ROI)

1. **Black Hat Transform** was used to highlight small, dark regions or details that are darker than their immediate surroundings and smaller than the chosen structuring element, This is mainly used for Feature collection and Background correction.

2. **Frangi Vesselness Filter** was used to act as a shape detector that selectively enhances thin, elongated, and tubular structures while suppressing background noise or spherical objects (like dust). The primary use case of this was Vessel Extraction.

Separate images for both the image processing algorithms was processed and stored in the Database for training the models not only on ROI, but to act as a comparision..

***
opencv-python    >= 4.5   -|
numpy            >= 1.21  -| ]→ Depndencies 
scikit-image     >= 0.19  -|
***

Full technical summary is to be uploaded...


The below is the high level explanation of the experiments performed and their results...


# Experiment - 01
