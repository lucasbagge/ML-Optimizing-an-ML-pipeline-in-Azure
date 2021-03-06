# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

After running the models I can make a comparison and choose the model with the best result.

### Projects steps

For this projects I follow some steps that is illustrated in the schema seen in the educational material:

- Step 1) At first we need to custimize the **train script**, then create a **Tabular Dataset** and evaluate it with some custom Scikit model.
- Step 2) The we create a jupyter notebook and use **HyperDrice to find the best hyperparameters.
- Step 3) When we have our custimize model we explore the **AutoML** functions and create a similar model.
- Step 4) At the end we compare the two models.

## Summary

Our dataset is about direct marketing campaigns of a Portuguese banking instition. 

The business goal is to predict whether the client will make a subcription for a bank term deposit.

Our best model in the **HyperDrice** contents is **HD_7f9efeb6-5197-44d5-bd5d-dea48d1a2118** and it gets a accuracy of **0.91760**. For an comparison the **AutoML model** with the ID **AutoML_07c21b85-5957-4a43-8f4e-4f3a6fe87cb2_15** gets an accuracy of **0.91581**.

## Scikit-learn Pipeline
> Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.


Azure AML gives the user a freedom to tune a models hyperparameters. At first we need to define the **search space**, here it depend on wheather the hyperparameters are discreate or continouse. 

I have choosen discreate values as the business problem is a classification problem of discreate values and specify the **parameter sampler** that I have given as:

```
ps = RandomParameterSampling(
    {
        '--C' : choice(0.1, 1,5, 3, 9, 81),
        '--max_iter': choice(10, 100, 1000)
    }
)
```

I have choosen **RandomParameterSampling** because it is fast and support early termination of low performnce [runs](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-tune-hyperparameters).

There are two other methods one could pick and they are; *GridParameterSampling* and *BayesianParameterSampling* and they could be picked if budget was not an issue. 

We also need to pick a **early stopping policy** which is used to terminate bad runs and improving compute performance. I have chossen **BanditPolicy**

```
policy = BanditPolicy(evaluation_interval=2, slack_factor=0.1)
```

where the *evaluation_interval* is optional and is the frequency in applying the policy.

The *slack_factor* is a ratio in how much slack we allow for a traning run with respect to the best performing model. This mean that a run wil be executed until it has finished.

## AutoML

> In 1-2 sentences, describe the model and hyperparameters generated by AutoML

For building a AutoML model we need to use `AutoMLConfig` and make som configuration:

```
automl_config = AutoMLConfig(
    compute_target = compute_target,
    experiment_timeout_minutes=15,
    task='classification',
    primary_metric='accuracy',
    training_data=ds,
    label_column_name='y',
    enable_onnx_compatible_models=True,
    n_cross_validations=2)
```

For making sure an experiment dosen??t take to long I specify the `experiment_timeout_minutes` which define how long an experiment should be run and it helps us avoiding failures. 

After that I specify the `task` as a classification problem, and the primary_metric as `accuracy`. 

As a extra I enable the **ONNX-compatible models** which is an open standard of machine learning models and is an open source contributions where you can add machine learning models.

For the cross validation I pick 2 folds; thus the metrics are calculated with aveage of the 2 validation metrics.


## Pipeline comparison

>Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?

For the **HyperDrive HD_7f9efeb6-5197-44d5-bd5d-dea48d1a2118** we get and accuracy of 0.918 while for the **AutoML_07c21b85-5957-4a43-8f4e-4f3a6fe87cb2_15** we get and **0.91581**. The HyperDrice is slighly better but I am convinced that the AutoML is better because of it high **AUC_weighted: 0.947** and it works good with an imbalanced data set as ours. 

One other better things with AutoML is that it takes care of all the calculations etc. It is clearly an advangates where in the Scikit learng pipeline we had to make it ourselves.

## Future work

>What are some areas of improvement for future experiments? Why might these improvements help the model?

If we look at the data we can observe it is highly imbalanced meaning that we have a lot of *no* campared to *yes*. It is a huge problem in classification problems because it impact the model accuracy negatively. Here it is easy to predict the major class but harder for the minor class. Therefore the accuracy metric can be misleading.

For dealing with this we can choose another metric such as AUC_weighted. Otherwise we can consider under sampling or over sampling and other algoritmes. 

## Proof of cluster clean up
**If you did not delete your compute cluster in the code, please complete this section. Otherwise, delete this section.**
**Image of cluster marked for deletion**

![](Users/GGB/Screenshot 2021-07-25 at 22.17.38.png)

