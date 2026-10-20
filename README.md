# Dry Bean Variety Prediction

A model comparison for a sorting line prototype. An agricultural sorting company wants to predict dry bean variety from camera measurements. This project compares four supervised approaches on the Dry Bean dataset, checks for overfitting, recommends one model for a production prototype, and explores whether the measurements reveal natural groupings when variety labels are hidden. Everything was actually run. Every number below comes from the executed notebook.

## The data

13,611 beans, 16 numeric shape measurements, and a `Class` target with seven varieties: DERMASON, SIRA, SEKER, HOROZ, CALI, BARBUNYA, and BOMBAY. No missing values. The classes are imbalanced, from 3,546 DERMASON down to 522 BOMBAY, so all splits are stratified.

Citation: Koklu, M., and Ozkan, I. A. (2020). Dry Bean [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C50S4B. The dataset is licensed under CC BY 4.0.

## A note on how the data was loaded

The notebook first tries the assignment starter loader, `fetch_ucirepo(id=602)`. If that call fails because the connection to the UCI server breaks off mid download, the notebook falls back to the same dataset downloaded directly from the UCI archive. The repository includes `Dry_Bean_Dataset.arff`, and the fallback reads it with `scipy.io.arff`. Both paths give identical data: 13,611 rows, 16 features, same Class target.

## Setup

    python -m pip install -r requirements.txt

## How to run

    jupyter notebook dry_bean_analysis.ipynb

Run the cells top to bottom. The notebook loads the data with the UCI loader when it can reach the server, and otherwise uses the data file `Dry_Bean_Dataset.arff` included in this folder. The notebook writes `results.csv` when the comparison cell runs.

## Comparison table

80/20 stratified split, plus 5-fold stratified cross-validation on the training portion for the CV column.

| Model | Train accuracy | CV accuracy | Test accuracy | Train time (s) | Predict time per bean |
|---|---|---|---|---|---|
| Logistic Regression (baseline) | 0.9262 | 0.9242 | 0.9214 | 0.64 | about 2.3 us |
| Decision Tree | 1.0000 | 0.8958 | 0.8917 | 0.40 | about 2.3 us |
| Random Forest (300 trees) | 1.0000 | 0.9240 | 0.9218 | 10.4 | about 45 us |
| HistGradientBoosting | 0.9869 | 0.9292 | 0.9221 | 1.46 | about 17 us |

Timings are wall clock numbers from the reference run and shift a little between machines.

## Overfitting and underfitting notes

- The decision tree overfits: train 1.0000 against test 0.8917. It grew to depth 29 with 698 leaves and memorized the training data. Its cross-validation score of 0.8958 agrees with the test score, so this is real, not a lucky split.
- A depth-5 tree underfits: train 0.8934 and test 0.8810. Both are low because depth 5 is too shallow for seven varieties.
- The random forest also memorizes the training set (train 1.0000) but its test score holds at 0.9218, because averaging many trees cancels out the individual memorization. It costs 10 seconds to train, the slowest of the four.
- The boosting model mildly overfits (train 0.9869, test 0.9221) and has the best cross-validation average at 0.9292.
- Logistic regression neither overfits nor underfits here: train 0.9262, CV 0.9242, test 0.9214 all agree, and it lands within a hair of the best ensemble.

## Clustering summary (labels hidden)

Features were scaled and KMeans was run for k from 2 to 10. The silhouette score peaked at k=3 (0.405) but those three clusters matched the hidden labels poorly (ARI 0.30). With k=7, the silhouette is lower (0.31) but the agreement with the true varieties is much better (ARI 0.67). Cluster 5 is pure BOMBAY, 520 of 520. Other clusters are 92 to 94 percent SEKER, HOROZ, or DERMASON, and 75 percent SIRA. So BOMBAY, SEKER, HOROZ, DERMASON, and SIRA form real natural groups in the measurement space. BARBUNYA and CALI blend together and cannot be cleanly separated, which is the honest limit of what the measurements show.

## Recommendation

Use logistic regression for the production prototype. Its test accuracy of 0.9214 is within a tenth of a point of the best model, while it trains in well under a second, predicts in a couple of microseconds per bean, is the simplest to maintain, and is the only one whose decisions can be read directly from its coefficients. Caveat: this is a classroom comparison, not a production sign-off. Real sorting equipment would need more data from the actual cameras, checks over time, and human review of errors before any model is used on the line.

## Files

- `dry_bean_analysis.ipynb`: the full executed notebook
- `Dry_Bean_Dataset.arff`: the dataset
- `results.csv`: the comparison table as data
- `requirements.txt`: Python dependencies
- `README.md`: this file
