---
title: "Spectroscopy-based plastic classfication"
date: 2025-02-24T10:00:00+01:00
---

<a href="https://github.com/AkkuRam/df-project">
  <i class="fab fa-fw fa-github"></i> Spectroscopy Prediction
</a>

## Dataset description
- Size: 373 rows, 998 columns
- Range of signals: ~700nm to ~1000nm
- Numerical columns: "spectrum_k", "sample_raw_k", "wr_raw_k" are 331 columns each
- Categorical columns: Color & Transparency
- Target variable: 1: PET, 2: HDPE, 3: PVC, 4: LDPE, 5: PP, 6: PS, 7: Other, 8: Unknown

The column "spectrum_k" are the columns of interest for us, since it is obtained by (spectrum_k = sample_raw_k / wr_raw_k), 
essentially dividing the raw signal by the white reference. Hence our total columns are 331 + 2 categorical columns to predict the target 
variable consisting of 7 different plastic types. We discarded the 8th plastic type, since it was unknown and it was only 10 samples.
            
Contributions:
- Preprocessing: Oversampling -> Baseline Correction -> Normalization -> Savitzy Golay
- High-level fusion: Bayesian Consensus
- Mid-level fusion: PCA + Models (XGBoost, AdaBoost, Multilayer Perceptron (MLP), Random Forest (RF)) 

## Preprocessing

Oversampling was perfoming, this there was a class imbalance in the target variable. BorderlineSMOTE was used to balance 
the class imbalance. It uses the basic implementation of SMOTE and improves the problem, where the synthetic samples from
SMOTE are too similar to the existing minority samples. Hence, BorderlineSMOTE generates new samples that are near the borderline between the minority and majority class majority classes. It is easier view a visualization from the imbalanced-learn docs, where the 3 different colors/classes are better oversampled with BorderlineSMOTE.

Now with BorderlineSMOTE, the first image below represents the imbalanced classes, where the second image represents the oversampled classes.

<p align="center">
  <img src="/images/class_imb.png" width="45%" />
  <img src="/images/class_bal.png" width="45%" />
</p>

These images represent the preprocessing steps of the signal. For 7 plastic types there are many samples, hence there are multiple signal lines. In addition, once the signals are preprocessed, if for each plastic type the samples are averaged, it should represent a distinct signal for each plastic type, which makes it identifiable which plastic type is which signal.

Referring to the images below, the first image represents the signals after being oversampled. There is clear background noise, therefore baseline correction is applied to isolate the peaks and flatten out the noise by using a quadratic polynomial (image 2). Hereafter, normalization is applied to bring them in a standard scale (image 3). Then the final step is to apply a Savitzky-Golay filter, which mainly smoothens out the signal (image 4).

<p align="center">
  <img src="/images/signal_1.png" width="45%" />
  <img src="/images/signal_2.png" width="45%" />
</p>

<p align="center">
  <img src="/images/signal_3.png" width="45%" />
  <img src="/images/signal_4.png" width="45%" />
</p>

## Mid-level fusion

- Extracting features using PCA + Categorical features

From the previous preprocessing steps, now the data is ready to use for the models. PCA is applied to reduce the dimensionality, since we are dealing with ~300 columns, where the PCA plot of 2 components below depicts a reasonable separation of the 7 classes. Moreover, from literature a threshold of keeping 90% of the variance was used which selected top 5 components in our case. The elbow method was not used, since this chose < 5 components, and the results (i.e. accuracy, precision, recall, f1score) were suboptimal.

<p align="center">
  <img src="/images/pca.png" width="45%" />
  <img src="/images/pca_thresholded.png" width="45%" />
</p>

With the following selections for mid-level fusion, it lead to the following results for the below metrics. Overall, the best performing models in terms of accuracy was XGBoost (78%), but in terms of other metrics, but XGBoost and RF were stable.

|    | Models   |   Accuracy |   Precision |   Recall |   F1-score |
|---:|:---------|-----------:|------------:|---------:|-----------:|
|  0 | XGBoost  |       0.78 |        0.77 |     0.77 |       0.77 |
|  1 | AdaBoost |       0.51 |        0.55 |     0.53 |       0.49 |
|  2 | MLP      |       0.60  |        0.53 |     0.56 |       0.53 |
|  3 | RF       |       0.71 |        0.74 |     0.73 |       0.72 |    

## High-level fusion

Unlike mid-level fusion, high-level fusion focuses on combining the outputs of multiple models, there is no
feature extraction in this stage. Hence, for high-level, all the preprocessing steps are used on the raw data, but without applying PCA. The formula for bayesian consensus is shown below. 

$$
p(h_g | e) = \frac{p(e | h_g) p(h_g)}{ \sum_g p(e | h_g) p(h_g)}
$$

- $p(e|h_g)$ is the likelihood estimate of the conditional probability that evidence e is observed
given that hypothesis $h_g$ is true
- $p(h_g)$ is the prior probability that hypothesis $h_g$ is true
        
Example:

|    | Model C   |   Predicted A |   Predicted B |   Likelihood A |   Likelihood B |
|---:|:----------|--------------:|--------------:|---------------:|---------------:|
|  0 | True A    |           127 |            19 |           0.87 |           0.13 |
|  1 | True B    |            12 |            60 |           0.17 |           0.83 |

$$
p(h_A|A) = \frac{0.87 \cdot 0.50}{0.17 \cdot 0.5 + 0.97 \cdot 0.5} = 0.84   
$$

$$
p(h_B|A) = 1 - 0.84 = 0.16  
$$

For the above example, our prior probabilities are represented by 0.5, as seen with our confusion matrices which is $2 \\times 2$,
as in Predicted vs True labels, where with the likelhood probabilities. We can compute this formula for some Model C, then these outputs (0.84, 0.16) are provided as input to the next model.        

The models used for combining the outputs are RF, Decision Tree, XGBoost, Logistic Regression, SVM, KNN. The main crux of bayesian consensus is that, in each iteration
it gives out two probabilities, essentially for the label "yes" and "no". Since for 1 vs Rest, this becomes binary, so after iterating  for each class, we get as the final output (two probabilites), where we choose the higher probability and this is accuracy  of predicting that class. 

Key Takeaway: The decision of the yes/no labels are chosen by us. As a result this is a bias, since the chosen labels can vary the results. The predicted labels will be given by the model and the actual/true label are defined by us.

## Future Work 
- Could have tried out many more fusion techniques to compare (i.e. Low-level, Federating learning), as well as  a few more within Mid/Low-level fusion
- Try another technique, which is averaging out all samples for each plastic type to create 7 distinct signals, then using cosine similarity to compare the true signal of each plastic type with our 7 distinct signals. 