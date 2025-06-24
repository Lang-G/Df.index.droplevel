# Pandas 1.5.3 Compatibility Note

This repositary addresses an issue when we use `df.index.droplevel(0)`, where different versions of the **pandas** library may or may not raise an error due to a change in if `groupby().apply()` creates multi-index by default or not .

This issue may cause the code to return a error or ,in more complex structures, return below warming signal and a **empty dataframe**: 

```plaintext

Cannot remove 1 levels from an index with 1 levels: at least one level must be left. Fallback load also failed:
Cannot remove 1 levels from an index with 1 levels: at least one level must be left.

```

Any further steps may results in errors such as :

```plaintext
TypeError: Expected sequence or array-like, got <class 'NoneType'>
```


Based on my tests with two different devices: **Pandas 2.2.2**, **2.2.3**, and **2.0.0** work with this code, while **Pandas 1.5.3** does not. This is likely due to a change in how groupby and apply return the index mentioned in this [2.0.0 documentation](https://pandas.pydata.org/pandas-docs/version/2.0.0/user_guide/copy_on_write.html). 

---

## 🚨 Issue

Let's see a example:

```python
# Apply yearly grouping — this will truly reset accumulations
df = df.groupby(df.index.to_period('Y')).apply(year)

# Optional: remove MultiIndex if created
df.index = df.index.droplevel(0)
```

This worked under the assumption that `groupby(...).apply(...)` always returns a `MultiIndex`. However, older versions of **pandas** may return a flat index, causing:

```plaintext
ValueError: Cannot remove 1 levels from an index with 1 level
```

---

## ✅ Fix

Update the index-dropping line to first check whether the index is a `MultiIndex`. This ensures compatibility with **pandas** 1.5.3 and newer versions (≥2.0.0):

```python
# Apply yearly grouping — this will truly reset accumulations
df = df.groupby(df.index.to_period('Y'), group_keys=False).apply(year)

# Optional: remove MultiIndex if created
if isinstance(df.index, pd.MultiIndex):
    df.index = df.index.droplevel(0)
```

This fix ensures that the code works across different **pandas** versions.



---

## 📚 Reference

For further reading on `pandas` internal changes, including the new Copy-on-Write behavior that may affect DataFrame transformations, refer to the official documentation:  
https://pandas.pydata.org/pandas-docs/version/2.0.0/user_guide/copy_on_write.html
