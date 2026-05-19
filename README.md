# AAI-540 Final Project – Group 5  
## Water Quality Risk Classification by ZIP Code Using Machine Learning  
Shiley-Marcos School of Engineering – University of San Diego  
Master of Applied Artificial Intelligence

---

## Team Members
- Puja Nandini
- Gregory Bauer
- Matt Hashemi

---

## Project Overview

This project uses ZIP-code-level water quality, violation, enforcement, and environmental data to classify U.S. ZIP codes as either **High Water Quality Risk** or **Normal/Lower Risk**.

The goal is to build an interpretable machine learning model that helps identify areas that may need further public health or infrastructure review.

---

## Objectives

- Clean and prepare the water quality dataset.
- Engineer risk-related features such as violation count, lead risk, and unresolved issues.
- Train and compare models such as Random Forest, XGBoost, or LightGBM.
- Evaluate the model using precision, recall, F1-score, and confusion matrix.
- Identify the most important factors driving high-risk predictions.

---

## Dataset

The project uses the **US Water Quality Data by ZIP Code** dataset, which includes water quality indicators for more than 41,000 U.S. ZIP codes.

Example features include violations, contaminant count, lead/copper results, enforcement actions, boil water advisories, population served, and Home Safety Score.

---

## Expected Outcome

The final model will classify ZIP codes by water quality risk and provide insight into the main factors contributing to high-risk predictions.

The model is intended as a decision-support tool and does not replace official EPA, state, or local water authority reporting.
