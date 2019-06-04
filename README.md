# geostatistical_inverse_modeling
This repository contains Matlab code for conducting geostatistical inverse modeling using large atmospheric datasets.

----------------------------------------------------------
Fair use
----------------------------------------------------------

Please send us an email (scot.m.miller [at] gmail.com and asaibab [at] ncsu.edu) if you are thinking of using these scripts in your research. We would love to hear more about what you are working on and if we can help in any way. 

Also, please make sure to cite the companion article in Geoscientific Model Development if you do use these scripts in your work.


----------------------------------------------------------
A note on the structure of the scripts
----------------------------------------------------------

These scripts provide four different ways to run a geostatistical inverse model (GIM).
(1) Solve the GIM equations directly: The direct or analytical solution involves solving a system of linear equations.
This approach works (a) if the number of observations is less than ~50,000 and (b) if an explicit H matrix is readily available. To solve the GIM in this way, modify and run the script inversion_direct.m. This script also provides an option for computing the posterior uncertainties directly.

(2) Estimate the fluxes using the minimum residual approach: In this approach, the fluxes are estimated by iterating toward the solution using a minimum residual method. This approach works well if (a) the number of observations is large (e.g., > 50,000) and/or (b) if an explicit H matrix is not readily available (i.e., if one is using an adjoint model that does not produce an explicit H matrix). Note that this approach is not recommended if the fluxes have known bounds (e.g., if the fluxes cannot be negative). To solve the GIM in this way, modify and run the script inversion_dual_eq.m. This script also provides an option for approximating the uncertainties using a reduced rank approach.

(3) Estimate the fluxes using a limited memory Broyden–Fletcher–Goldfarb–Shanno (L-BFGS) algorithm with a variable transformation: In this approach, the fluxes are estimated by iterating toward the solution using a quasi-Newton algorithm known as L-BFGS.  This approach works well if (a) the number of observations is large (e.g., > 50,000) and/or (b) if an explicit H matrix is not readily available (i.e., if one is using an adjoint model that does not produce an explicit H matrix). This approach should exhibit similar computational performance to the minimum residual approach (2). However, this script does require inverting the covariance matrix R and the components of the Q covariance matrix (D and E). If these matrix inverses are burdensome to compute, the minimum residual approach is likely the better option. The L-BFGS is used in many 4-D variational (4D-Var) inverse models; these scripts may therefore be easier to integrate into existing 4D-Var code. Note that this approach is not recommended if the fluxes have known bounds (e.g., if the fluxes cannot be negative). To solve the GIM in this way, modify and run the script inversion_LBFGS.m.

(4) Estimate the fluxes using the L-BFGS algorithm with bounds: This approach is similar to L-BFGS but will additionally enforce bounds on the inverse problem. For example, this option may be a good choice for methane or N2O, gases that do not have large negative fluxes. This script also iterates toward the solution. However, it converges much more slowly than either options (2) or (3). For example, this script may require ten times as many iterations to converge on a solution, and it is therefore not recommended for very large inverse problems. To solve the GIM in this way, modify and run the script inversion_LBFGS.m. One would need to remove the variable transformation from this script, swap out calls to the functioncost_gradient_fun_transform.m with calls to the function cost_gradient_fun.m, and swap out calls to the L-BFGS algorithm with calls to an L-BFGS-B algorithm (the bounded version). 


----------------------------------------------------------
Which set of scripts should I use?
----------------------------------------------------------

Q: Are there bounds on the fluxes? For example, do the fluxes need to be non-negative?
Use the L-BFGS-B (option 4). One could also use option (1) with a variable transformation to enforce non-negativity (see Snodgrass and Kitanidis 1997 [https://doi.org/10.1029/96WR03753] for one example).

Q: Is the number of observations less than ~50,000, and is an explicit H matrix available?
Use the direct solution (option 1). Sometimes, inverse problems with between 25,000 and 50,000 observations are too large for the direct solution (depending upon the problem and the computer system involved), and one may need to use option 2 or 3.

Q: What if my atmospheric model does not produce an explicit H matrix?
Use option (2) or (3) (or option 4 for a problem with bounds). Note that you will need both a forward model and an adjoint model to use any of the inverse modeling approaches outlined above.


----------------------------------------------------------
A note on the H matrix
----------------------------------------------------------

Some available atmospheric models will produce an explicit H matrix. These models include STILT (the Stochastic, Time-Inverted, Lagrangian Transport Model) and FLEXPART (the FLEXible PARTicle dispersion model). Other atmospheric models do not produce an explicit H matrix (e.g., TM-5 or GEOS-Chem). Instead, these models can pass a vector through the forward atmospheric model, and in some cases, pass a vector through the adjoint model. Specifically, these models will calculate the product H or H^T and a vector but will not formulate H or H^T explicitly.

You will need to modify the scripts here depending upon which type of model that you use. Most of these scripts can be used with either type of model.

The scripts here have been written for a model that produces an explicit H matrix, but these scripts can be modified to account for other types of atmospheric models. 

The scripts included here are written assuming the following format for the H matrix:
- An explicit H matrix is available.
- The H matrix has been broken into smaller pieces or strips, and each strip has been written to a different file. Each strip should have n rows (corresponding to the total number of observations in the inverse model), and each strip should have columns corresponding to a single time period of fluxes to be estimated. 
- These H "strips" are saved in Matlab files (with the ".mat" extension). The files should have the following naming convention: "H_1.mat" where 1 indicates the time period that this particular file corresponds to. In this case, "H_1.mat" would refer to the first time period and "H_2.mat" would correspond to the second time period of the inverse model, etc. For example, if one were estimating 3-hourly CO2 fluxes, H_1.mat would correspond to the first three-hourly time period, and H_2.mat to the second three-hourly time period.
- Again, this setup can be modified for atmospheric models that do not produce an explicit H matrix or for inverse problems where the H matrix is stored in a different format. I have set up the scripts this way solely out of convenience; the setup described above lends itself to the case study included here using the STILT model. I have flagged all of the sections of code that would need to be modified to run the GIM with different types of atmospheric models that do not fit the H matrix format described above. Look for sections of code denoted with the comment "%! Note: Edit this section to match the actual format of H in your problem."


----------------------------------------------------------
List of scripts in the repository
----------------------------------------------------------

* OPTION 1: Solve the GIM equations directly

SCRIPT: inversion_direct.m
PURPOSE: Estimate the fluxes using the direct solution (i.e., by solving a system of linear equations).
CALLED BY: None.
CALLS: HQHR.m, uncert_direct.m

SCRIPT: HQHR.m
PURPOSE: Calculate (H*Q*H' + R). This matrix is often referred to with the Greek letter Psi.
CALLED BY: inversion_direct.m
CALLS: None.

SCRIPT: uncert_direct.m
PURPOSE: Calculate the posterior uncertainties using direct calculations. This script will provide an exact answer
	 and will not use any approximations to improve computational tractability.
CALLED BY: inversion_direct.m
CALLS: None.


* OPTION 2: Minimum residual approach

SCRIPT: inversion_dual_eq.m
PURPOSE: Launch the geostatistical inverse model using the minimum residual approach to estimating the fluxes.
	 This script also provides an option for estimating the uncertainties using a reduced rank approach.
CALLED BY: None.
CALLS: Ax.m, weights_to_fluxes.m, uncert_est.m

SCRIPT: Ax.m
PURPOSE: Calculate the left-hand side of the GIM equations used in the minimum residual approach.
CALLED BY: inversion_dual_eq.m
CALLS: None.

SCRIPT: weights_to_fluxes.m
PURPOSE: Convert the weights estimated using the minimum residual approach into estimated fluxes.
CALLED BY: inversion_dual_eq.m
CALLS: None.

SCRIPT: uncert_est.m
PURPOSE: Estimate uncertainties using the reduced rank approach. This approach estimates approximate
	 uncertainties. It is computationally tractable even for very large inverse problems.
CALLED BY: inversion_dual_eq.m
CALLS: QBABQ.m, randeigdecomp.m, eigfun.m

SCRIPT: QBABQ.m
PURPOSE: Calculate Q^(1/2)*B*A*B'*Q^(1/2)*B where B is some vector or matrix.
CALLED BY: uncert_est.m
CALLS: None.

SCRIPT: eigfun.m
PURPOSE:  Function for computing Ax where A is the matrix we want eigenvectors/values for. 
CALLED BY: uncert_est.m, randeigdecomp.m
CALLS: None.

SCRIPT: randeigdecomp.m
PURPOSE: Compute the eigenvectors and eigenvalues of matrix A using a randomized approach.
CALLED BY: uncert_est.m
CALLS: eigfun.m


* OPTION 3-4: L-BFGS and L-BFGS-B

SCRIPT: inversion_LBFGS.m
PURPOSE: Estimate fluxes using a geostatistical inverse model, implemented using 
	the L-BFGS algorithm and a variable transformation.
CALLED BY: None.
CALLS: cost_gradient_fun_transform.m

SCRIPT: cost_gradient_fun_transform.m
PURPOSE: Calculate the cost function and gradient for the geostatistical inverse model.
CALLED BY: inversion_LBFGS.m
CALLS: None.

SCRIPT: cost_gradient_fun.m
PURPOSE: Calculate the cost function and gradient function without a variable transformation.
CALLED BY: This function is not called by any other script. Rather, it is an alternative to the script
cost_gradient_fun_transform.m. Specifically, this script is useful for inverse problems with bounds (e.g., non-negativity). If one wants to enforce bounds on an inversion, use this script can be used in place of cost_gradient_fun_transform.m and use with L-BFGS-B (the bounded version of L-BFGS) instead of L-BFGS. 
CALLS: None.


----------------------------------------------------------
Scripts customized to run the included case study
----------------------------------------------------------

Case study description:
The case study here is identical to the first case study in Miller et al. (GMD). As a result, the fluxes estimated by these scripts can be compared directly with the first case study in Miller et al. (GMD). 

In this case study, we estimate 6 weeks of CO2 fluxes (July through mid-August 2015) across North America using observations from the OCO-2 satellite. Specifically, these scripts will estimate CO2 fluxes at a 3-hourly time resolution and 1 degree by 1 degree spatial resolution across the US. The observations used in this case study are synthetic; they were generated using fluxes from NOAA's CarbonTracker product and using the Weather Research and Forecasting (WRF) model paired with the STILT model. We further added noise to the synthetic observations to give them qualities more similar to real-world observations.

In this particular case study, we generated an explicit H matrix using WRF-STILT model output from Arlyn Andrews and Mike Trudeau (NOAA). 

Here is a summary of the different scripts that can be used to run this case study. To run each of these scripts, one only needs to provide a path to the model and data input files; all of the other inputs should be customized to the case study. These scripts will write the estimated fluxes (averaged over the entire 6-week time period) as a netcdf file. This file should contain all of the information needed to plot the estimated fluxes. 
 
- casestudy_inversion_direct.m: Estimate fluxes and uncertainties using the direct approach.

- casestudy_inversion_dual_eq.m: Estimate fluxes and uncertainties using the minimum residual method. Also estimate uncertainties using a reduced rank approach.

- casestudy_inversion_LBFGS.m: Estimate fluxes and uncertainties using the L-BFGS algorithm.


----------------------------------------------------------
Contact information
----------------------------------------------------------

Scot Miller
Johns Hopkins University
Department of Environmental Health and Engineering
Email: smill191 [at] jhu.edu OR scot.m.miller [at] gmail.com
http://greenhousegaslab.org/

Arvind Saibaba
North Carolina State University
Depatment of Mathematics 
Email: asaibab [at] ncsu.edu
https://asaibab.math.ncsu.edu/


