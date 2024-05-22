# Merge transform

This topic tells you about the Application Accelerator `Merge` transform in Tanzu Application Platform (commonly known as TAP).

The `Merge` transform feeds a copy of its input to several other transforms and
merges the results together using set union.

A `Merge` of **T1**, **T2**, and **T3** applied to input **I** results in **T1(I) ∪ T2(I) ∪ T3(I)**.

![Diagram showing a merge transform.](images/merge.svg)

## <a id="syntax-reference"></a>Syntax reference

```plaintext
T1()
+ T2()
+ T3()
```
