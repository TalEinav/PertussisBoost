# PertussisBoost
Predicting the response to the pertussis booster vaccine as part of the Computational Models of Immunity (CMI-PB) 2nd challenge.<br>
https://www.cmi-pb.org/

We develop algorithms that use a patient's pre-vaccination data to predict their post-vaccination response.
* *Training Data* contains responses from 2020 and 2021 used to train models.
* *Testing Data* contains responses from 2022 (baseline only) that need to be predicted for this challenge.
* *Code* contains the Mathematica notebook used for this analysis and the resulting prediction file. 
<br><br>
---
Lessons learned:
* Task 1 (predicting antibody titers) was  especially challenging because of a few quirks in the data.
    * Antibody titers were measured in units of UI/mL in 2020 but MFI in 2021 and 2022. It is not clear how to convert between both types of data (even a simple linear relation would be tricky to infer with no global reference, but there may also be a complex non-linear relation at play).
    * We considered ignoring the 2020 dataset entirely (which leaves very little data for training/validation), yet we ultimately decided to train models based on 2020↔2021 cross-validation, where we first predict between the 2020→2021 data and then between the 2021→2020 data. In both cases, we evaluate a model based on the correlation between the predicted and measured post-vaccination rankings, and our final cross-validation metric is the average of the correlations in both directions.
    * The 2020 data is missing some IgG features contained in both the 2021 and 2022 datasets. Since our validation scheme requires predicting between 2020↔2021 data, we had to forgo using these features in our model.
    * The 2022 pre-vaccination antibody titers are substantially higher than the 2021 pre-vaccination titers (and there is no way to directly compare it with the 2020 titers because of the different units). As such, the testing data may be quite different from the training data.
* Given these limitations, we searched for simple models (often linear regression or random forests) that predicted responses between 2020↔2021 data. Model performance was tuned using various tricks (e.g. using mean-centering in Task 1.1 but not in Task 1.2) that improved these cross-study predictions. As another example, we modeled the log10(titers) for both Tasks 1.1 and 1.2, and for these tasks we replaced titer=0 with titer=0.1 to be able to log all measurements.
* For Task 1, we found reasonably good predictions between the 2020↔2021 data, which gives some hope that this model may predict the 2022 data (assuming the higher pre-vaccination titers don't have too strong of an effect). Although we determined the model architecture using 2020↔2021 cross-validation, our final model was only trained on the 2020 data (because of the clear differences between the 2021 and 2022 pre-vaccination titers).
* For Task 2.1, we also found a good predictor. In this case, after fixing the model architecture based on 2020↔2021 cross-validation, we only used the 2021 data for training, since the 2021 pre-vaccination monocyte values were more similar to the 2022 data than the 2020 values. 
* For Task 2.2, we could not find a good predictor, likely because the fold-change of monocytes in 2020 and 2021 was always nearly 1, so there is little to learn and a very small chance to be able to predict such fine distinctions. In this case, it is clear that this model will perform poorly.
* Task 3 was challenging because there was so much data, and we ultimately opted for a simple model based almost entirely on the pre-vaccination expression of the gene-of-interest. We expect this model to perform poorly for both subtasks.
* We note that none of these models distinguished between the wP/aP vaccines, instead opting to combine all these data for additional training. Similarly, we did not use the age/sex of participants, the cytokine levels, nor the lower limit of detection for any assays.