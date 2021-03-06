# EBG - Issue #5
Compile a list of useful scripts to add to our toolbox.

In this issue we will compile a list of programs/scripts to add to our toolbox and discuss them so that we then add them as issues. For example, generate_cdf.sh

#### scriptGetInterarrivalTraces-Animation.py --- DEPRECATED

#### scriptGetInterarrivalsSummaryStatistics.py --- Replace scriptGetInterarrivalTraces-Animation.py

This script takes the tracefile with three columns like these: 

	1169602866.764024004 0 1890460369ba9e1b2000000002408bcc0407e714badc0779620000002ad6ee00
	1169603076.857360676 0 b0950301da59470520000000000a77acec71750660f8073eb0950301da594700
	1169603076.865715928 0 b0950301da59470520000000028d3897abe44a0100fb07e7b0950301da594700
	1169603076.873845130 0 d5554d0089f4a8052000000001aba9a5e624590cbadc075ed5554d0089f4a800
	...

Obtains the interarrival times and the summary statistics to generate the dataset to be used with the machine learning clustering techniques.

It should be able to read files with a slightly different format like:

	1201639675.424053,READ,HSOw4LO05Ws
	1201639757.082532,READ,1bhOE7xcZyg
	1201639761.780669,READ,3TKi92CP-vc
	1201639762.360242,READ,1bhOE7xcZyg
	...

It also should get the SPAN, the access count number, and the "DELAY" (Delay is the time difference between the first access in the dataset and the fir access of the object) for each object. We currently make this with a different script: scriptGetSpanAccessCount.sh, scriptGetDelayTraces-Animation.py.

There are some additional considerations that should be added to the final script:

- The dataset for clustering includes the object-id and the summary statistics. It should also include the new columns SPAN, and Access Count.
- Summary statistics can only be calculated on "objects" with more than 1 interarrival (> 2 accesses). 
- Some Summary Statistics give "NaN"s and/or warning messages. Objects with NaNs should be excluded from the new dataset, but added to a separate group (see below).
- Besides the dataset for clustering, there should be 2 additional files:
	- Objects with a single access should be included in a separate file, with columns: "object-id, delay". These objects have SPAN = 0, and accessCount = 1
	- Objects with a single interarrival time (2 accesses), and those with "NaN"s values in Summary Statistics should be put together in a separate file, with columns: "object-id, delay, SPAN, AccessCount"
- The fortmat of output files should be as standard as possible.
- Interarrivals should be in miliseconds, instead of seconds.

#### scriptKmeans.py

This script takes the output file from the "scriptGetInterarrivalTraces-Animation.py", with the format:

	id mean median mid-range gmean std iqr range mad coeficiente_variacion skewness kurtosis
	d5554d0089f4a8052000000001668d8d8807b105badc075ed5554d0089f4a800 498.38129096 13.8858120441 1151.76963568 32.468978124 905.369996108 161.596295357 2303.53927135 12.5654661655 1.81662115439 1.48280905868 0.225775235498
	1890460369ba9e1b2000000002faeeea2ca2f91460f80753620000002ad6ee00 20642.2612459 15953.156997 21169.0236591 10859.0741992 17599.5897049 21169.0236591 42338.0473183 14135.3672858 0.852599891808 0.380735813174 -1.5
	62000000e280200d20000000001f6eb63f8d4b0d60f8073d62000000e2802000 205.395387888 9.77516174316e-06 976.770716667 0.00305335934816 489.810111449 1.00845932961 1953.54143333 3.09944152832e-06 2.38471815986 2.74771472591 6.71973115235
	...

And apply the machine learning clustering techniques:

    - Dimensionality reduction:
	- VarianceThreshold with a variance threshold value set to .9 (http://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.VarianceThreshold.html#sklearn.feature_selection.VarianceThreshold)
	- PCA with the same variance value.
    - Data Scaling (Original and PCA datasets)
    - Clustering:
	- K-means for: the complete dataset (11 features/SummaryStatistics), the dataset with the columns after the VarianceThreshold reduction (if any column was removed), the PCA transformed dataset, and a dataset with only 2 features (Skewness and AverageInterarrival/mean).

This script requires the following improvements:

	- Write logs to a file, it currently shows them in the console. 
	- Receive the "variace threshold" as argument.
	- Receive the "K" clusters number as argument.
	- Receive an argument to decide if data should be scaled.
	- MAYBE: select the technique to be used (PCA, Variance threshold).
	- Use the new dataset with additional columns generated by the improved version of "scriptGetInterarrivalTraces-Animation.py"
	- MAYBE: receive a minimal column number for PCA when the variance threshold is reached with a single column.|
