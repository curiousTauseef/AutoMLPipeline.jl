# Preprocessing
Let us start by loading the `diabetes` dataset:
```@setup preprocessing
using Random
ENV["COLUMNS"]=1000
Random.seed!(123)
```
```@example preprocessing
using AutoMLPipeline
using CSV
diabetesdf = CSV.read(joinpath(dirname(pathof(AutoMLPipeline)),"../data/diabetes.csv"))
X = diabetesdf[:,2:end]
Y = diabetesdf[:,1] |> Vector
nothing #hide
```
We can check the data by showing the first 5 rows:
```@repl preprocessing
show5(df)=first(df,5); # show first 5 rows
show5(diabetesdf)
```

This dataset is a collection diagnostic tests among the 
Pima Indians to investigate whether the patient shows 
sign of diabetes or not based on certain features:
- Number of times pregnant
- Plasma glucose concentration a 2 hours in an oral glucose tolerance test
- Diastolic blood pressure (mm Hg)
- Triceps skin fold thickness (mm)
- 2-Hour serum insulin (mu U/ml)
- Body mass index (weight in kg/(height in m)^2)
- Diabetes pedigree function
- Age (years)
- Class variable (0 or 1) indicating diabetec or not

What is interesting with this dataset is that one or more numeric columns
can be categorical and should be hot-bit encoded. One way to verify is 
to compute the number of unique instances for each column and look for 
columns with relatively smaller count:
```@repl preprocessing
[n=>length(unique(x)) for (n,x) in eachcol(diabetesdf,true)] |> collect
```

Among the input columns, `preg` has only 17 unique instances and it can
be treated as a categorical variable. However, its description indicates
that the feature refers to the number of times the patient is pregnant
and can be considered numerical. With this dillema, we need to figure
out which representation provides better performance to our classifier.
In order to test the two options, we can use the Feature Discriminator
module to filter and transform the `preg` column to either numeric
or categorical and choose the pipeline with the optimal performance.

### CatNumDiscriminator for Detecting Categorical Numeric Features
*Transform numeric columns with small unique instances to catergories.*

Let us use `CatNumDiscriminator` which expects one argument to indicate
the maximum number of unique instances in order to consider a particular
column as categorical. For the sake of this discussion, let us use its 
default value which is 24.
```@example preprocessing
using AutoMLPipeline, AutoMLPipeline.FeatureSelectors
using AutoMLPipeline.EnsembleMethods, AutoMLPipeline.CrossValidators
using AutoMLPipeline.DecisionTreeLearners, AutoMLPipeline.Pipelines
using AutoMLPipeline.BaseFilters, AutoMLPipeline.SKPreprocessors
using AutoMLPipeline.Utils

disc = CatNumDiscriminator()
@pipeline disc
tr_disc = fit_transform!(disc,X,Y)
nothing #hide
```
```@repl preprocessing
show5(tr_disc)
```

