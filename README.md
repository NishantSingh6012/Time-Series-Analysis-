# TIME SERIES ANALYSIS OF NON – SEASONAL LONG TERM INTEREST DATA


**NISHANT R. SINGH**

---

## INTRODUCTION

This project focuses on the time series analysis of daily real long-term interest rate data (for maturities over 10 years), which reflects market expectations about inflation, growth, and future monetary policy. Unlike datasets with seasonal trends, this series displays non-seasonal behavior but exhibits volatility clustering, especially around economic events.

The goal of this analysis is:

* To explore and understand the behavior of long-term interest rates using time series visualization, ACF/PACF plots, and statistical tests.
* To verify stationarity in both mean and variance using transformations and formal tests such as the ADF and ARCH tests.
* To model and forecast the conditional volatility using GARCH-family models, allowing us to capture risk dynamics present in financial data.


---

## DATA EXPLORATION AND PREPROCESSING

The analysis begins with a thorough exploration of the raw time series data, which consists of daily real long-term interest rates spanning several years. Visual inspection is a critical first step in any time series analysis, as it uncovers patterns such as trends, structural shifts, or volatility clustering that inform subsequent modeling choices.

---

### TIME SERIES VISUALIZATION

The time series of daily real long-term interest rates from 1999 to 2024 shows notable structural shifts. From 1999 to around 2012, the trend is predominantly downward, reflecting a long period of accommodative monetary policy and subdued inflation. The series then enters a low-rate regime that persists through the mid-2010s, eventually dropping sharply around the COVID-19 pandemic in 2020. Following this, there is a strong rebound in rates, with a steep increase beginning in 2021 — likely due to inflationary pressures and tightening monetary policy.

Although the series does not display seasonal repetition, its behavior over time offers important insights:

* **No visible seasonality**: There is no repeating annual cycle, confirming the data is non-seasonal.
* **Clear trend and regime shifts**: Periods of structural change are visible, particularly around 2008 and 2020.
* **Volatility clustering**: Certain windows show sustained high or low volatility, indicating non-constant variance.
* **Near-zero or negative real rates**: A key feature during 2020, aligning with global economic uncertainty.

These observations suggest that while modeling the mean may not be critical, capturing the changing variance through models like GARCH is essential for an accurate understanding of volatility dynamics in the interest rate series.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/b902e7ff-01f2-4224-b5ef-742f9af2215c" />


---

## STATIONARITY TESTING

Before fitting ARIMA or SARIMA models, one of the most critical requirements is that the time series must be stationary. A stationary series has a constant mean, constant variance, and constant autocovariance over time. This ensures the model’s assumptions hold true and that forecasting remains meaningful beyond the observed period.

**Augmented Dickey-Fuller (ADF) Test – Original Series**

To formally test for stationarity, the Augmented Dickey-Fuller (ADF) test is applied to the original time series. The hypotheses are:

* **Null hypothesis (H₀)**: The series has a unit root (i.e., it is non-stationary).
* **Alternative hypothesis (H₁)**: The series is stationary.

**Result on original series**:
ADF Test Statistic: -0.386947856872
p-value: 0.912255314887

---

## FIRST ORDER DIFFERENCING AND SEASONAL DIFFERENCING

To make our data stationary and handle the trends, the first difference of the series is computed. This process helps eliminate the deterministic trend and stabilize the mean.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/84343c0f-86bd-4187-af19-2ae66da7fea1" />
<img width="550" alt="image" src="https://github.com/user-attachments/assets/f7780b11-ff71-4ffe-81ae-73569ff40b69" />


The Autocorrelation Function (ACF) and Partial Autocorrelation Function (PACF) plots are used to assess serial dependence. In this case, both plots indicate no significant autocorrelation, suggesting that the mean model could be simplified, possibly to a zero-mean or constant mean model.

---

## SARIMA AND GARCH MODEL FITTING

### ACF and PACF

In the current case, both ACF and PACF plots of the return series show very little activity beyond lag 0, with all subsequent spikes well within the 95% confidence interval:

* **ACF observations**: Almost all lags are non-significant due to the differencing operation itself; all further lags lie within the bounds, indicating no strong moving average structure.
* **PACF observations**: Similarly, there is no meaningful spike beyond lag 0, suggesting the absence of autoregressive behavior in the returns.

This combination suggests that the underlying data might include either AR or MA components alone. Also, a random walk can be considered as a candidate.

---

### Model Evaluation

Several SARIMAX configurations were tested on the differenced daily real interest rate series: ARIMA (0,1,0), ARIMA (1,1,0), and ARIMA (0,1,1). Despite slightly different specifications, all three models yield virtually identical log-likelihood values (\~8186) and comparable AIC scores.

| Model         | Log-Likelihood | AIC       | BIC       | AR/MA Coeff. | p-value | Ljung-Box p |
| ------------- | -------------- | --------- | --------- | ------------ | ------- | ----------- |
| ARIMA (0,1,0) | 8185.808       | -16369.62 | -16363.15 | —            | —       | 0.27        |
| ARIMA (1,1,0) | 8186.439       | -16368.88 | -16355.95 | AR(1) = .016 | 0.10    | 0.99        |
| ARIMA (0,1,1) | 8186.473       | -16368.95 | -16356.01 | MA(1) = .016 | 0.10    | 0.99        |

**Model Notes:**

* High Ljung-Box test p-values (0.27–0.99) confirm no significant autocorrelation in residuals.
* ARCH test p-value = 0.0000 → strong evidence of heteroskedasticity.
* Jarque-Bera test and kurtosis (\~8.45) suggest heavy tails and non-normal residuals.

Conclusion: No mean model is needed, but modeling volatility is essential → Use GARCH.

---

### Residual Diagnostics

To evaluate the adequacy of the ARIMA (0,1,0) model, a comprehensive residual analysis was conducted. This includes visual diagnostics such as standardized residual plots, Q-Q plots, correlograms, histograms, and formal statistical tests.

<img width="224" alt="image" src="https://github.com/user-attachments/assets/3316a948-168d-47eb-891a-4ebd03a7cb91" />

<img width="238" alt="image" src="https://github.com/user-attachments/assets/f83a6895-dbe9-4d08-949d-e2b6b4b55354" />

The standardized residual plot shows no obvious trend or pattern, which suggests that the mean structure has been adequately captured. The residuals fluctuate randomly around zero with some large spikes, indicating potential changes in variance over time.

The Q-Q plot, however, reveals heavy tails and the sample quantiles deviate substantially from the theoretical normal line, particularly in the tails. This aligns with the Jarque-Bera test, which confirms significant deviation from normality (JB ≈ 5923.52, p < 0.001), and a kurtosis value of ~8.5 further supports the presence of extreme values.

The histogram with KDE overlay also illustrates that the residual distribution is more peaked and fat-tailed than a standard normal distribution. This reinforces the decision to use a Student's t-distribution in later volatility modeling.

The correlogram and Ljung-Box test results support the absence of autocorrelation in residuals.
None of the first 20 lags show significant correlation and all Ljung-Box p-values are well above the 5% threshold most are near or above 0.99.

<img width="900" alt="image" src="https://github.com/user-attachments/assets/45dcc089-911c-4c96-8e51-680bb129ff54" />

Given these conditions, and the lack of autocorrelation in ARIMA residuals, the analysis proceeds with a zero-mean GARCH model applied directly on returns, which preserves stationarity while accurately modeling the time-varying volatility structure.
---

### GARCH Model

Following the residual diagnostics, it became evident that while the return series does not exhibit autocorrelation in the mean, it does exhibit strong evidence of time-varying volatility. The ARCH test p-value of 0.0000, visible volatility clustering in residual plots, and high kurtosis from the normality tests all support this.
Given these findings, a GARCH (1,1) model with a zero-mean assumption was fit directly to the return series. This specification is one of the most widely used for financial time series and captures both the effect of recent shocks (ARCH component) and long-term volatility persistence (GARCH component).

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/51c1d4c6-201f-4024-b2a6-1720be9c933a" />

 
For most of the time, the returns hover closely around zero, reflecting relatively stable changes in interest rates. However, a distinct episode of heightened volatility is observed between approximately 2011 and 2014, where the return values spike dramatically in both directions with extremes exceeding ±8%. This cluster of sharp fluctuations stands in contrast to the otherwise subdued behavior of the series and highlights a period of intense market instability.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/5fe88ca0-ab0a-4900-a05f-d9259d4b72d3" />
 
The significant autocorrelation in the squared returns indicates that the variance of the series is not constant over time, a classic signature of heteroskedasticity. This supports the use of a GARCH-family model, which models the conditional variance as a function of past squared shocks and past variances.

Based on this autocorrelation structure, a GARCH (1,1) model was selected. This choice is well-supported by the data, as it captures the immediate effect of recent shocks (ARCH term: α₁) and the long-term persistence in volatility (GARCH term: β₁).

The GARCH (1,1) order strikes a balance between model parsimony and adequacy, aligning with the observed ACF behavior and avoiding overfitting.

The table below summarizes the fitted Zero-Mean GARCH (1,1) model on the return series

Parameter	Estimate	Understanding the parameters 
ω (omega)	7.74e-05	Baseline variance; long-run average level of volatility
α₁ (alpha)	0.1297	Short-term reaction to market shocks (ARCH component)
β₁ (beta)	0.8703	Persistence of volatility over time (GARCH component)
ν (nu)	6.12	Degrees of freedom from t-distribution — accounts for fat tails


All coefficients are highly statistically significant (p < 0.01), and the α + β ≈ 1 implies strong persistence, meaning that volatility shocks decay slowly over time a common trait in financial time series.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/5737ca7b-7769-4f5c-89b7-546f70c86db0" />

  
The volatility plot shows the actual returns in blue, overlaid with the conditional volatility in red, as estimated by the GARCH (1,1) model. 

Volatility spikes dramatically during the 2012 to 2014 window, capturing periods of heightened market turbulence. The conditional volatility adapts smoothly over time, rising in response to extreme return movements and gradually decaying afterward. 

This pattern becomes even more evident in the second zoomed-in plot, where volatility visibly increases following sharp return shocks and then slowly subsides, illustrating the memory effect inherent in GARCH processes. 

The red conditional volatility curve closely tracks the intensity and timing of real-world return fluctuations, demonstrating the model’s ability to respond accurately to changing market conditions. Overall, the visualization confirms that the GARCH model not only captures the historical clustering of volatility but also reacts effectively to dynamic shifts in return behavior.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/7189ab70-5120-4fd1-8a85-1ecfcd29a293" />

The plot below illustrates forecast generated by the GARCH (1,1) model using predicted volatility bands. 

Notably, the forecasted volatility bands dynamically adjust to the level of market uncertainty widening during periods of elevated volatility and narrowing during calmer phases. This behavior highlights the model’s ability to anticipate shifts in risk, offering a more realistic and responsive estimate of future variability compared to models that assume constant variance. Most of the observed returns lie well within the forecasted bands, confirming that the GARCH model effectively captures the conditional heteroskedasticity in the return series.
NOTE: -
To generate these volatility forecasts, simulation-based forecasting was used by setting the method to 'simulation' and running 10,000 simulations through the forecast object in python        model_fit.forecast(horizon=949, method='simulation', simulations=10000). 
This approach improves the robustness of the predicted variance, allowing the model to better capture uncertainty and future volatility dynamics. 
      
 
<img width="1000" alt="image" src="https://github.com/user-attachments/assets/b38e712f-2940-4a7a-b47c-f2695a13010d" />

---
## FORECASTING

The final plot displays the forecast from an ARIMA (0,1,0) model, which generates a flat mean forecast accompanied by fixed-width confidence intervals.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/0f3bf6a6-129d-4a11-af01-fe3e8f8dcb55" />


These static bands assume constant variance and remain unchanged throughout the forecast horizon. As a result, the model fails to account for fluctuations in volatility, offering no insight into periods of elevated or reduced risk. This limitation makes ARIMA-based forecasting less suitable for financial time series, where market uncertainty is known to vary over time.

In contrast, the GARCH-based forecast provides a more nuanced perspective by explicitly modeling the conditional variance. It produces time-varying confidence bands that expand during volatile periods and contract during calmer intervals, better aligning with observed market behavior. This dynamic adaptation makes the GARCH (1,1) model more appropriate for high-frequency financial data.

---

## CONCLUSION

This analysis examined the behavior of daily real long-term interest rates with a focus on volatility modeling. Initial exploratory steps, including time series visualization and autocorrelation analysis, revealed that while the returns themselves exhibit no serial correlation, the variance is clearly time-varying. Standard ARIMA models failed to improve predictive performance due to the absence of autocorrelation in the mean structure. However, the presence of volatility clustering and significant ARCH effects indicated the need for a more appropriate model for the variance dynamics.

To address this, a GARCH (1,1) model with a zero-mean specification was implemented and evaluated using simulation-based forecasting. The model successfully captured conditional heteroskedasticity in the return series, with time-varying volatility that responded appropriately to real-world financial shocks. Compared to the ARIMA (0,1,0) forecast, the GARCH model provided dynamic confidence intervals and a more realistic risk assessment. These findings support the suitability of GARCH models for financial time series analysis, where modeling risk and uncertainty is as important as modeling central tendencies.

---

## FUTURE IMPROVEMENTS

While the GARCH (1,1) model provided a solid foundation for modeling volatility in daily long-term interest rate returns, several avenues exist for improving and extending the analysis. One important enhancement would be to explore asymmetric GARCH models such as EGARCH or GJR-GARCH. These models can capture the differing impacts of positive and negative shocks on volatility, a common feature in financial markets where bad news tends to increase uncertainty more than good news. Applying such models could lead to a more nuanced understanding of risk behavior, particularly during crisis periods.

Additionally, although a Student’s t-distribution was used to better account for fat tails in the return distribution, alternative distributional assumptions such as the generalized error distribution (GED) or skewed-t distribution could be explored. These may offer better fit in cases where the return series displays both heavy tails and asymmetry.

The current analysis also relies on univariate modeling. In the future, a multivariate GARCH approach, such as DCC-GARCH, could be used to study the co-movement of long-term interest rates with other macroeconomic variables like inflation, short-term rates, or equity indices. This would provide a more comprehensive view of interconnected market dynamics and portfolio-level risk.

---


