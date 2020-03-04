---
layout:     post
title: Considerations about OneHotEncoder
subtitle: 使用OneHotEncoder的一些注意事项
date:       2020-03-04
author:     Loopy
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - StackOverflow

---

> When I was answering questions on StackOverflow, I found [this question](https://stackoverflow.com/questions/60516821/python-one-hot-encoder-pandasarray-object-has-no-attribute-reshape) interesting. I also found that there are lots of questions about `sklearn.preprocessing.OneHotEncoder`, which seemed to be pretty confusing, so I reprinted my answer here.


These following informations might be helpful:

1. The type of some of the objects:
    - `data[feature]`: `pandas.Series`
    - `data[feature].values`: `numpy.ndarray`
2. You can `reshape` a `numpy.ndarray` but not a pandas.Series, so you need to use `.values` to get a `numpy.ndarray`
3. When you assign a `numpy.ndarray` to `data[feature]`, automatic type conversion occurs, so `data[feature] = data[feature].values.reshape(-1, 1)` doesn't seem to do anything.
4. `fit_transform` takes an array-like(Need to be a 2D array, e.g. `pandas.DataFrame` or `numpy.ndarray`) object as argument because `sklearn.preprocessing.OneHotEncoder` is designed to fit/transform multiple features at the same time, input `pandas.Series`(1D array) will cause error.
5. `fit_transform` will return sparse matrix(or 2-d array), assign it to a `pandas.Series` may cause a disaster.

(**Not Recommended**) If you insist on processing one feature after another:
```python
for f in categorical_feats:
    encoder = OneHotEncoder()
    tmp_ohe_data = pd.DataFrame(
        encoder.fit_transform(data[f].values.reshape(-1, 1)).toarray(),
        columns=encoder.get_feature_names(),
    )
    data = pd.concat([ohe_data, data], axis=1).drop([feature], axis=1)
```

I **Recommended** do encoding like this:
```python
encoder = OneHotEncoder()

ohe_data = pd.DataFrame(
    encoder.fit_transform(data[categorical_feats]).toarray(),
    columns=encoder.get_feature_names(),
)
data = pd.concat([ohe_data, data], axis=1).drop(categorical_feats, axis=1)
```

`pandas.get_dummies` is also a good choice, but the downside is that, you can't `pickle` an encoder for later use.

```python
for f in categorical_feats:
    dummies = pd.get_dummies(data[f], prefix=f)
    data = data.join(dummies)
```