MC Metallicity
====================

This code allows the user to calculate metallicity according to a number of <i> strong line metallicity diagnostics </i> from spectroscopy line measurements, and obtain uncertainties from the line flux errors in a Monte Carlo framework. If you use this code, please cite <b>OUR PAPER (link to be provided when published)!</b>
This code is released under MIT licence: see LICENSE.txt

====================
Usage:
====================
Place the  _meas and _err  files in the "input" directory (example files are provded in the directory).


From the commandline simply use as:
```
python mcz.py <filename> nsample --path PATH --clobber --delog --verbose
```
\<filename\>: the SN name which should be the common name of the _meas and _err files (e.g. testdata for testdata_meas.txt and testdata_err.txt)

nsample: the number of MC samples desired 

  --clobber             replace existing output
  --unpickle            if it exists, a pickle output files for this SN and this 
                        number of samples is read, instead of recalculating the metalicity

  --binmode             how to choose the number of bins for plotting the histogram:
                            'd' is based on Doane's formula (wilipedia's version),  
                            's' is the sqrt of number of data,        
                            't' is on 2*n**1/3 , 
                            'kb' uses Knuth's block rule (default), 
                            'bb' uses bayesian blocks (must have astroML installed or it defaults to 'kb')
                            'kd' is the kernel density, which requires sklearn installed and additioinally plots the                             histogram with 't' mode

  --clobber             If set to true, will overwrite existing output files. Default False.

  --verbose          verbose mode. Default False.

  --path PATH           the directory in which subdirectory "input" is located. If not provided, will default
                        environmental variable MCMetdata that should be set to point to that directory. 
                        _err.txt _meas.txt must live in <path>/input

  --unpickle            read the pickled realization instead of making a new
                        one
  --nodust              don't do dust corrections (default is to do it)
  --noplot              don't plot individual distributions (default is to
                        plot all distributions)
  --asciiout            write distribution in an ascii output (default is not
                        to)
  --md MD               metallivity diagnostic to calculate. default is 'all',
                        options are: D02, Z94, M91, M08,C01, P05,P10, PP04, M13, D13, KD02,
                        KD02comb, DP00 (deprecated), P01
  --multiproc           multiprocess, with number of threads nps=max(available
                        cores-1, MAXPROCESSES)



====================
Input file format
====================
each flux data should be stored in the directory sn_data that exists in the directory provided by --path arg. 

with common filename \<fi\>:

\<fi\>_err.txt

\<fi\>_meas.txt 

The format for each of the txt files should be as follows:


;# galnum,[OII]3727,Hb,[OIII]4959,[OIII]5007,[OI]6300,Ha,[NII]6584,[SII]6717,[SII]6731,[SIII]9069,[SIII]9532,E(B-V),dE(B-V)
       1     0.0     0.0     0.0     0.0     0.0   5.117   0.998     0.0     0.0     0.0     0.0
       2     0.0     0.0     0.0     0.0     0.0   5.031   1.012     0.0     0.0     0.0     0.0
       
       
galnum is the index used for matching the flux with the corresponding radius for calculating the gradient.

The first row is optional - if can contain more keys than column, as long as the columns are listed in the same order as specified above, up to whichever line is of interest. Missing data should be filled in with 'nan's.

the data should be in the above order, separated by any number of white spaces.


====================
Output
====================
All the results will be saved in the directory "\<fi\>" inside "outputs" which will be creates in the directory provided by the --path arg.

For a given \<fi\> and nSample, the following output files are generated:

"\<fi\>_n"nSample"_X.csv": the metallicity and its uncertainty that was calculated, where X will correspond to the number of rows the input data contains

"\<fi\>_n"nSample"_X.pkl": the metallicity and its uncertainty that was calculated stored in binary (pickled) format in a python dictionary

"hist" folder, containing all the histograms generated


====================
Tests implemented
====================

A few tests are implemented to make sure your inputs are valid, and your outputs are robust. 
In order to assess if the number of samples requested is sufficiently large, we provide the modeul testcompleteness.py. 

Use as, for example: 

import testcompleteness as tc

tc.fitdistrib(<path to pickle file>)


This reads in the pickle file generated by the main code, and compares the cumulative distributions for 4 diagnostics: E(B-V), D02, Z94 and KD02comb_updated, choosing the best measurement among the imput ones (i.e. the oue that allows most complete results).  

The cumulative distribution for 1/10, 1/4, 1/2, 3/4 and the full sample generated by the code are compared with a KS tests and plotted. 

If the number of samples is sufficient, you should expect the probability of the 3/4 sample and full sample to come from the same parent distribution to be close to 1, and generally the probability to increse with the sample fraction (but not always it turns out...). More importantly though this can be used visually: the cumulative distribution should all be smooth, and nearly identical: then you are sure that you are well above the critical sample size that achieve smoothness (by a factor 10).

The figures below show an undersampled realization and a well- (possibly over-) sampled realization.




![alt tag](https://github.com/fedhere/MC_Metalicity/blob/master/output/exampledata/exampledata_n200_testcomplete.png)
![alt tag](https://github.com/fedhere/MC_Metalicity/blob/master/output/exampledata/exampledata_n2000_testcomplete.png)
![alt tag](https://github.com/fedhere/MC_Metalicity/blob/master/output/exampledata/exampledata_n20000_testcomplete.png)


 
====================
Known Issues and TODO
====================

Plot formatting is designed for a platform using latex and with Times New Roman serif font available to matplotlib. The module pylabsetup loaded early on assures the matplotlib rcparameters are set up appropriately, including font choice, and in case of missing fonts an error message is streamed (but not paused upon). If your plots don't look good change the necessary parameters in pylabsetup.py. This is an unfortunately common occurrence when not running on a Mac.
