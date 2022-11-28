# Forward Starting CDO Model Implementation


The forward starting CDO (FSCDO) valuation model serves the purpose of pricing FSCDO trades, which are defined as a forward agreement to enter into a CDO trade at some time in the future. Any obligor defaults before the forward starting date will be replaced by a risk free asset with the same notional amount.

A non-parametric approach is adopted in the model. A Monte Carlo (MC) simulation is first employed to generate a large number of prior correlated default scenarios with a set of different correlation assumptions. The weight of each MC scenario is computed by the weighted Monte Carlo (WMC) method, such that the credit spreads of each obligor and the market implied spot tranche values with different terms can be repriced. Then the MC scenarios associated with the calibrated weights can be used to price the FSCDO trade.

To our best knowledge to date, there is no directly observable market information on the FSCDO.
The methodology of the model is considered acceptable, based on the fact that the model can be calibrated to the pertinent spot market information with different terms, effectively deal with market implied correlation skew structure, and reasonably model the default events before the forward starting date. However, there are two embedded model uncertainties. One is the calibration process of WMC in which the optimization of hundreds of instruments is involved for a normal FSCDO trade. This task is difficult and it is unlikely to achieve a perfect calibration to all instruments. The selection of the spot calibrating instruments, specially the spot tranches, also contributes to the uncertainties of the calibration.  The other uncertainty resides in the generation of the prior MC scenario. Two different choices would give different prices of the forward trade, even if both can be successfully calibrated to the spot market information. 

As an effective way of managing the model uncertainties, a model risk reserve is set up by benchmarking against the most conservative parameter scenarios. At the present time those scenarios are found through an extensive testing on a trade by trade basis. When we start to accumulate positions in FSCDOs, well defined conservative scenarios will be justified and established.

In the model setup, the results of the optimization can be tracked in two ways. First a log file can be generated with the example given in Figures 1(a) and 1(b). In Fig. 1(a), a successful calibration is achieved while in Fig. 1(b) the calibration fails. In the latter example, the optimization stops because objective function exceeds limits (100) and shows no sign of convergence. Second, a message is shown to imply in the pricing spreadsheet that how many iterations are performed before the convergence is reached. If this number reaches the maximum,  it indicates that most likely no convergence is reached. The calibration results of the spot tranche values are exported to show how good the calibration of those tranche values. However,  we should be cautious to take these results the criterions as the calibration, because the largest portion of calibration instruments are acutally the credit spreads of each obligor. The calibration results of those credit spreads are not exported in the model. Normally the calibration of a credit spread for an obligor can be manually verified by setting the notional amount of all other oblgors to zero. 


In the model the prior distribution of WMC is simulated using the normal copula model. There is a standard way of MC simulation to generate default times of the obligors in the collateral pool using the marginal unconditional default probability of each obligor, which can be found in Ref. [7]. 

When a MC scenario is generated, a correlation assumption has to be used. On the other hand, it is well known that there is a market implied correlation skew implied by iTraxx or CDX indexes, which can be described by the base correlation term structure. Hence, in a non-parametric approach intended to fit market implied spot tranche values with different maturities, a very wide distribution of the prior MC scenarios has to be simulated, so that we have enough degree of freedom to find the best fit to the market.  For each similar correlated default scenarios there are a fairly large number of paths so that the theoretical basis of the MC applies. 

It would be ideal if we can use many correlation values between 0 and 1 with considerable large number of the simulation paths number. However, like all other MC simulations, the choice is limited by the computational efficiency and simulation memory capacity. Instead, several correlation values are assumed, each with a fairly large number of simulation so that all possible correlated default scenarios can be generated. Currently we choose five correlation values 0, 0.2, 0.4, 0.6, and 0.95, each associated with at least 100,000 paths.  Our test in the next section shows that the current choice is adequate.

Due to the feature of the MC simulation, we have no control over the actual output correlation default scenarios, even those with the same correlation assumptions. Two different prior MC scenarios would predict a different forward values, even both of them can be successfully calibrated to the same spot market information.  In order to manage this model uncertainty caused by the choice of prior MC scenario, a model risk reserve has been set up. 

Once the correlation values are chosen, the correlated default times of the obligors are simulated and recorded. For the   MC scenario, the default time of the obligor is denoted as . Then for each scenario, the accumulated loss up to time t can be written as 

(16)	 .


There are two subsets of calibrating instruments. As discussed above, the prerequisite of the WMC is that the optimized path weights should be able to reprice the instruments used to generate prior MC scenarios. For all the structured credit derivatives, they are defined as the unconditional default probabilities (or, equivalently, credit spread curve) of each obligor in the collateral pool. Each obligor has a term structure of the credit spread and so does the unconditional default probability. Normally the terms of a credit spread curve include 1w, 1m, 3m, 6m, 1y, 2y, 3y, 4y, 5y, 7y, 10y, and 20y. For example, if a FSCDO trade is from 2y to 7y, the terms we need to calibrate are 2y, 3y, 4y, 5y, and 7y.

For each obligor in the collateral pool, the probability of default before the term   can be described by   and the constraint is defined as

(17)			 


with

(18)			   

where   is the default time of  the  obligor in the  -th simulation path.

In the pricing of a FSCDO, the default probabilities from forward starting date to maturity are calibrated. It would be ideal if we can calibrate the full term structure of the credit spread curve of each obligor, which is the case when the WMC are initially used in the sensitivity computation. However, practically we have to calibrate to fewer terms for a successful calibration of appropriate spot tranche values.  By doing this, the number of constraints is reduced and only the ones most relevant to the target FSCDO are considered.  For example, the CDX5 index pool has 125 obligors hence there are 125 curves to fit. 

In the test trade Test-I with CDX5 index pool as the collateral pool, defined as a 3 x 4y FSCDO, the full term structure has ten points hence there are 10 x125= 1250 credit spread constraints to fit. By ignoring the terms before the forwarding date, we only calibrate to four terms at December 20, 2008, December 20, 2009, December 20, 2010, and December 20, 2012. So the total number of constraints of credit spreads becomes 4 x125=600.  This can be viewed as an approximation and the test in the next section have shown that the approximation is acceptable. 

The second type of calibration instruments are expected losses of spot tranches with different terms. Apparently for an FSCDO, we need the spot tranche values with at least two maturities: the forwarding starting date S and the trade maturity T.  In the model setup, the calibrating spot tranche values are first calculated with given implied base correlation. For example, the   calibrating tranche is defined as attachment point , detachment point , and maturity S. The base correlations for this spot tranche is   and . Then within the base correlation framework the expected loss of this tranche   and the total loss of the collateral at time   can be calculated. The constraint of this tranche is defined as

(19)	 

Where 

(20)	 



 is the accumulated tranche loss for a given   MC simulation path at time t. 

The FSCDO value is dependent on the choice of calibrating tranches in two ways. First, it would be best to calibrate to the whole capital structure with two maturities. However, in reality some times there is no available market data for the whole capital structure. It is also difficult to successfully calibrate to the entire capital structure. Like other non-parametric approaches a perfect match of all market quotes of the indexes is unlikely without further assumptions outside of the current modeling framework. For example, in the perfect copula model a negative correlation between the default rate and recovery rate has to be made to fit the whole capital structure of the indexes with one maturity. In a many cases, the target FSCDO tranche has a low attachment point and detachment point and the expected losses for those higher spot tranches are very small. Ignoring those high tranches can be a good approximation. 

Second, the choice of thickness of each calibrating tranche has a very large effect on the FSCDO. For example a target FSCDO trade could be a thin tranche below 3%. It won’t give us much market implied correlation information if we only calibrate to 0~3% tranche and other standard tranches. In this case, the value of FSCDO is highly dependent on the prior MC distribution.  

In order to manage the model uncertainty caused by the calibration error and choice of calibrating tranches, a model risk reserve has to be set up. A procedure to determine the calibrating tranches has been set up so that appropriate capital structure and tranche thickness can be chosen on a trade by trade basis. The procedure proposed by the FO is attached as Appendix III.

If the path weight of each scenario is calibrated successfully, the expected loss function used in Eqs. (3) and (4) can be calculated by summing up all scenarios 

(21) 	 .

Then the MTM of the FSCDO can be calculated readily. It is important to note that in Eq.(21), the weight of each MC path   is no longer uniform after the WMC calibration and reflects the market implied correlation information.

The credit spread sensitivity has been implemented in the model. As shown in the above section, instead of the market implied correlation information the model is calibrated to the normalized expected loss of the spot tranches. Hence the sensitivities are simply a risk factor perturbing and revaluation of the trade. In the submitted model, if the risk factor is perturbed, the model has to be recalibrated by holding the market implied base correlations unchanged.  

In order to increase the computational efficiency and stability of the sensitivities, the initial base MC scenarios and calculated weights are fixed and perturbed, when a risk factor is shocked. For example, the credit spread sensitivity the credit spread curve is perturbed by parallel shift of 5bps. For the   obligor, because the prior MC scenario is fixed, the default time of the   path   is changed to   such that

(22)	 

Where  is the perturbed hazard rate curve.  In the actual implementation, the default times are simply remapped. This is consistent with the method proposed by Leif Anderson [14]. Once the remapped default times are calculated, the optimization is re-run starting from the base case hence it will be much faster to get the calibrated results.

There is  a possibility that the model could be miscalibrated. In the credit spread sensitivity, normally a small shock such as 5bps is used. There is a possibility that such a small perturb to the correlated default times would lead to no re-calibration at all.  In order to avoid such cases, a higher tolerance control is used by setting fucntion tolerance and gradient tolerance to  . 

Leif Anderson (LA) proposed a model within the factor copula and base correlation framework. The FSCDO defined in this report is called type II trade in his paper. Losses on the FSCDO are redefined to be computed only from the portfolio losses that take place after the forward start date, which can be written as 

(23)	 

With   for  . Eq.(2) for FSCDO can be re-written as

(24)	 

In the factor-setting it follows that all existing algorithms will work, if we replace the default probability conditional on the latent variable is replaced by

(25)	 

This is a straightforward extension of existing algorithms of semi-closed form solution. 

Note that for normal copula, which is the case of FSCDO model,

(26)	 ,

with   being the unconditional default probability of the   obligor and   the correlation. The details of the semi-closed form solution for one factor normal copula mode can be found in Ref. [16].

The model needs a well defined correlation skew term structure model. As shown in Eq.(25), within the base correlation framework the computation will have exposure to the correlation skew term structures, that is, a carefully implied base correlations with different detachment points and different terms to maturity has to be used. Because Leif Anderson does not elaborate on how to use the correlation skew in his model, we simply propose a linear interpolation of the base correlation for any time between the forward starting date and maturity, and ignore any possible interpolation within each term. For the purpose of the comparison to the model, we also ignore any market implied base correlation curve between forward starting date and maturity.  

Note that, unlike the model, this model can not guarantee arbitrage free. It is well known that, no matter what interpolation scheme on the base correlation curves is used, the base correlation curve is not arbitrage free. 

There is another type of FSCDO, named Type-I in Leif Anderson’s paper.  The value of protection can be written as

(27)	 

The pricing for this type of FSCDO is straightforward (see https://finpricing.com/lib/FiZeroBond.html). We could price the trade as two spot trades with two different maturities. Then the value of two legs can be calculated by simply subtracting the value of protection and risky annuity of the two trades. 

Note that this trade can be viewed as a replica of a portfolio, in which we long a tranche and short the same tranche with different maturities. However, as discussed above it is not the same as the FSCDO covered in this report. In type I trade, the attachment point has to be adjusted if a default happens before the forward starting date hence this type of trade does not serve the purpose of buying/selling a protection starting from some time in the future.

Because the value of this type of trade is transparent, it is helpful to compare its value with the type II trade. As discussed in the paper by Leif Anderson, we could expect that the protection leg for an equity tranche of a type II trade is larger than that of a type I and vise versa for the super senior tranche. Hence the type I trade value shown in Eq.(27) can be consider as a kind of theoretical boundary.

If we consider the FSCDO be the entire collateral pool, that is, setting the attachment point 0 and detachment 100%, then Eq.(2) becomes

(27)	 

It is well know that the expected loss of the whole pool won’t change on any correlation assumption. We could first calculate Eq. (27) with flat compound correlation and compare this value to the scenarios with correlation skew. Any consistent FSCDO model should be able to reproduce the forward starting total collateral pool losses under various correlations skew. In other words, Eq. (27) can be considered as a prerequisite of any FSCDO model.

The FSCDO valuation model has been investigated. Our tests have shown that the model is consistent with the one outlined in the report and has been implemented correctly. By comparing the model with the benchmark models and the theoretical limits for the FSCDO trades, we are confident that the value of the FSCDO predicted by the model is reasonable if the model can be successfully calibrated. All the input parameters of the model have been assessed and found adequate for a typical FSCDO trade. Therefore, the model is approved for Marking-to-Market and computing credit spread sensitivities.

The model is considered as an acceptable one, based on the fact that the model can be calibrated to the pertinent spot market implied correlation information with different terms, effectively deal with the correlation skew, and reasonably model the default before the forwarding starting date.  Furthermore, a detailed testing against a FSCDO model proposed by Leif Anderson shows that the two models are reasonably consistent, although they are based on totally different assumptions.  We have also done two other tests. One is to test against the Type-I FSCDO trade, in which the expected forward losses can be considered as the boundaries of that of the FSCDO trade discussed in this report (Type-II trade defined by Leif Anderson). The other test is against a prerequisite of a FSCDO trade, that is, the total forward losses of the collateral pool should remain unchanged for all correlation skew scenarios. Our tests have shown that both model and Leif Anderson’s model perform as expected. Hence we are confident that the forward CDO value predicted by the model is reasonable, although there is no direct market information to verify this.  

The model employs a WMC method, which is regarded as a non-parametric approach. The advantage of this approach is that, if successfully calibrated, the model ensures no-arbitrage. It is well known that the base correlation is not arbitrage free. Further, the WMC technology enables one to fit the market with multiple terms of the calibrating instrument, which can be considered a progress of the modeling of such approaches.

However, there are two major uncertainties in the model. One is the calibration process in which the optimization of hundreds of instruments is involved for a typical CDO trade. This task is difficult and time consuming. More importantly, it is unlikely to achieve a perfect calibration to all instruments. In order to achieve a reasonably good calibration to the market information, an acceptable calibration tolerance has to set up first. Second, some of the calibrating instruments have to be dropped and only the most relevant ones are selected. This will bring uncertainties to the model. For example, in certain cases we would get unstable Val01, because we only calibrate the expected losses of the spot tranches. Another uncertainty in the calibration is the choice of the calibrating spot tranches. The value of a FSCDO trade is dependent on the choice of the capital structure and attachment points and detachment points.

The second uncertainty is the subjective choice of the correlation set in the generation of the prior MC scenarios. Two different choices would give different prices of a forward trade, even if both can be successfully calibrated to the spot market information. As a non-parametric approach intended to fit market implied correlation skew structure, it would be ideal if we can choose many correlation values between 0 and 1 with considerable large number of the simulation paths number. However, like all other MC simulations this is limited by the computational efficiency and simulation memory capacity hence a subjective choice has to be made.

Linked with those uncertainties, the model encounters many parameters which are chosen quite subjectively.  All of them have been tested and current choices are found adequate for a typical FSCDO trade. In this report, a 3x4 year FSCDO with CDX5 index as the collateral pool is regarded as a typical FSCDO trade and used as a test trade. The tests summarized in the report are:

	choices of optimization parameters;
	choices of correlation set to generate prior MC scenarios;
	choices of calibrating tranches;
	choices of calibrating credit spreads;
	MC simulation convergence for MTM and credit spread sensitivity.

An effective way of managing the model uncertainties is to set up an appropriate model risk reserve, which is calculated by benchmarking against the most conservative parameter scenarios. At the present time the most conservative scenarios are found through an extensive testing on a trade by trade basis. GMRMR London and GMRMR QA have agreed that, for each FSCDO trade, two most conservative scenarios are searched by 1) using various calibration instruments with the different levels of calibration tolerances, and 2) using various copulas and correlations to generate prior MC scenario. The model risk reserve is then determined by a combination of the above two. When we begin to accumulate positions in FSCDOs, the scenarios will be tested and well defined conservative scenarios will be established.

Through testing, the restriction and limitations are defined as follow.

	The generation of the prior MC scenario

1.	The default correlation model is the normal copula model;
2.	The least set of correlation is 0, 0.2, 0.4, 0.6, and 0.95;
3.	The minimum number of paths for each correlation value is 100,000.
	
	The calibration of WMC

1.	There are two sets of calibrating instruments. One includes the full term structure of the credit default probabilities of each obligor starting from the forward starting date to the trade maturity. The other set of calibrating instrument should at least include the spot tranches with terms at the forward starting date and the trade maturity.
2.	Among those optimization parameters, the most import two are objective function tolerance and objective function gradient tolerance, which are set to be at least  for MTM and  for the credit spread sensitivity.


The choice of calibrating tranche attachments and detachment points is determined on a trade by trade basis. Ideally the calibrating instruments should cover the entire capital structure for each term with appropriate attachment and detachment points. Practically it is hard and/or not necessary to do that due to the market data availability and the target tranching position of the FSCDO. 

The model should not be used if the calibration fails. A successful calibration is dependent on both optimization parameters and the choice of the calibrating instruments. Any calibration error 

