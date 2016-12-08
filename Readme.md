[![Build Status](https://travis-ci.org/fukatani/stacked_generalization.svg?branch=master)](https://travis-ci.org/fukatani/stacked_generalization)

# stacked_generalization
Implemented machine learning ***stacking technic[1]*** as handy library in Python.
Feature weighted linear stacking is also available. (See https://github.com/fukatani/stacked_generalization/tree/master/stacked_generalization/example)

Including simple model cache system Joblibed classifier and Joblibed Regressor.

## Feature

#####1) Any scikit-learn model is availavle for Stage 0 and Stage 1 model.

#####And stacked model itself has the same interface as scikit-learn library.

You can replace model such as *RandomForestClassifier* to *stacked model* easily in your scripts.
And multi stage stacking is also easy.

ex.
```python
from stacked_generalization.lib.stacking import StackedClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression, RidgeClassifier
from sklearn import datasets, metrics
iris = datasets.load_iris()

# Stage 1 model
bclf = LogisticRegression(random_state=1)

# Stage 0 models
clfs = [RandomForestClassifier(n_estimators=40, criterion = 'gini', random_state=1),
        GradientBoostingClassifier(n_estimators=25, random_state=1),
        RidgeClassifier(random_state=1)]

# same interface as scikit-learn
sl = StackedClassifier(bclf, clfs)
sl.fit(iris.target, iris.data)
score = metrics.accuracy_score(iris.target, sl.predict(iris.data))
print("Accuracy: %f" % score)
```

More detail example is here.
https://github.com/fukatani/stacked_generalization/blob/master/stacked_generalization/example/cross_validation_for_iris.py

https://github.com/fukatani/stacked_generalization/blob/master/stacked_generalization/example/simple_regression.py

#####2) Evaluation model by out-of-bugs score.
Stacking technic itself uses CV to stage0. So if you use CV for entire stacked model, ***each stage 0 model are fitted n_folds squared times.***
Sometimes its computational cost can be significent, therefore we implemented CV only for stage1[2].

For example, when we get 3 blends (stage0 prediction), 2 blends are used for stage 1 fitting. The remaining one blend is used for model test. Repitation this cycle for all 3 blends, and averaging scores, we can get oob (out-of-bugs) score ***with only n_fold times stage0 fitting.***

ex.
```python
sl = StackedClassifier(bclf, clfs, oob_score_flag=True)
sl.fit(iris.target, iris.data)
print("Accuracy: %f" % sl.oob_score_)

```
#####3) Caching stage1 blend_data and trained model. (optional)

If cache is exists, recalculation for stage 0 will be skipped.
This function is useful for stage 1 tuning.
```python
sl = StackedClassifier(bclf, clfs, save_stage0=True, save_dir='stack_temp')
```

## Feature of Joblibed Classifier / Regressor

Joblibed Classifier / Regressor is simple cache system for scikit-learn machine learning model.
You can use it easily by minimum code modification.

At first fitting and prediction, model calculation is performed normally.
At the same time, model fitting result and prediction result are saved as *.pkl* and *.csv* respectively.

**At second fitting and prediction, if cache is existence, model and prediction results will be loaded from cache and never recalculation.**


e.g.
```python
from sklearn import datasets
from sklearn.cross_validation import StratifiedKFold
from sklearn.ensemble import RandomForestClassifier
from stacked_generalization.lib.joblibed import JoblibedClassifier

# Load iris
iris = datasets.load_iris()

# Declaration of Joblibed model
rf = RandomForestClassifier(n_estimators=40)
clf = JoblibedClassifier(rf, "rf")

train_idx, test_idx = list(StratifiedKFold(iris.target, 3))[0]

xs_train = iris.data[train_idx]
y_train = iris.target[train_idx]
xs_test = iris.data[test_idx]
y_test = iris.target[test_idx]

# Need to indicate sample for discriminating cache existence.
clf.fit(xs_train, y_train, train_idx)
score = clf.score(xs_test, y_test, test_idx)
```

See also https://github.com/fukatani/stacked_generalization/blob/master/stacked_generalization/lib/joblibed.py

## Running with Spark
You can use the stacking interface transparently within your pyspark environment.
Use DistributedStackedClassifier respectively DistributedStackedRegressor.

The individual fitting of the estimators will be executed in parallel.


```python
from stacked_generalization.lib.stacking import DistributedStackedClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.linear_model import LogisticRegression, RidgeClassifier
from sklearn import datasets, metrics
iris = datasets.load_iris()

# Stage 1 model
bclf = LogisticRegression(random_state=1)

# Stage 0 models
clfs = [RandomForestClassifier(n_estimators=40, criterion = 'gini', random_state=1),
        GradientBoostingClassifier(n_estimators=25, random_state=1),
        RidgeClassifier(random_state=1)]

# same interface as scikit-learn
sl = DistributedStackedClassifier(sc, bclf, clfs)
sl.fit(iris.target, iris.data)
score = metrics.accuracy_score(iris.target, sl.predict(iris.data))
print("Accuracy: %f" % score)
```

## Software Requirement

* Python (2.7 or 3.4)
* numpy
* scikit-learn
* pandas
* findspark

## Installation

```
pip install git+https://github.com/teralytics/stacked_generalization
```

## License

MIT License.
(http://opensource.org/licenses/mit-license.php)


## Copyright

Copyright (C) 2016, Ryosuke Fukatani

Many part of the implementation of stacking is based on the following. Thanks!
https://github.com/log0/vertebral/blob/master/stacked_generalization.py

## Other
Any contributions (implement, documentation, test or idea...) are welcome.

## References
[1] L. Breiman, "Stacked Regressions", Machine Learning, 24, 49-64 (1996).
[2] J. Sill1 et al, "Feature Weighted Linear Stacking", https://arxiv.org/abs/0911.0460, 2009.
