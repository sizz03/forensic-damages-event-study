**FORENSIC SECURITIES DAMAGES**

**RESEARCH PROJECT MANUAL**

_Quantifying Artificial Inflation and Aggregate Shareholder Loss_

_in ASX-Listed Securities Using Event Study Methodology_

A Comprehensive Step-by-Step Blueprint

From Data Acquisition to Court-Ready Damage Estimates

Prepared for: Sam Bizzell

Target Audience: Hans Weemaes, Principal, The Brattle Group (Sydney)

March 2026

# Part I: Project Overview and Strategic Context

## 1.1 What This Project Is

You are going to perform what is known in litigation consulting as a Securities Class Action Event Study. This is the exact type of work that Hans Weemaes does at Brattle. In plain English, you are going to:

- Find a company listed on the Australian Securities Exchange (ASX) that made misleading claims about AI capabilities, and then later had to reveal the truth (causing its stock price to crash).
- Build a mathematical model that shows what the stock price would have been if the company had never lied. This is called the "counterfactual" or the "but-for" price.
- Prove statistically that the price crash was caused by the corrective disclosure (the truth coming out), and not by general market movements, sector trends, or other unrelated news.
- Quantify the total dollar amount of damages suffered by shareholders who bought the stock while the lie was inflating its price and held it through the crash.

The output of this project will be a GitHub repository containing your Python code, a Jupyter notebook walking through the analysis, and a short executive memo summarising your findings. This is exactly what a first-week analyst at Brattle would be expected to produce.

## 1.2 Why This Will Impress Hans Weemaes

Hans Weemaes is a Principal at The Brattle Group who specialises in securities class actions, forensic data analytics, and "but-for" cost and profit estimation. His daily work involves building counterfactual price models for use as expert testimony in court, quantifying shareholder damages in class action lawsuits, performing forensic analysis of stock price movements around disclosure events, and dealing with confounding variables that opposing counsel will use to challenge his models.

By building this project, you are demonstrating that you already understand the core analytical workflow of his practice area. You are not just a student who can run a regression - you are showing that you understand how to frame a forensic question, control for noise, prove causation, and translate statistical output into a dollar figure that a court can act on.

## 1.3 The Conversation Starter

When you meet Hans, do not talk about your degree or your GPA. Lead with the project:

_"Hans, I've been following Brattle's expansion into the Australian market, particularly in securities litigation. I've been working on a project that uses Fama-French five-factor modelling paired with a GARCH framework to isolate the price impact of AI-related corrective disclosures in the ASX 200. I'm particularly interested in how you handle confounding effects in these newer 'AI-washing' cases - how do you statistically separate a general tech-sector downturn from a specific disclosure-related loss?"_

This works because it hits every keyword he uses professionally: "price impact," "confounding effects," "Fama-French," and "GARCH." It also asks him an open-ended question about his methodology, which signals intellectual curiosity rather than just name-dropping.

## 1.4 Key Terminology Cheat Sheet

Before diving into the technical detail, here are the terms you will encounter repeatedly. Memorise these:

| **Term**                         | **Plain English Meaning**                                                                                     |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Event Study                      | A statistical method for measuring how a specific event (like a disclosure) affects a stock price.            |
| Counterfactual / But-For Price   | The hypothetical price the stock would have been at if the misleading conduct had never occurred.             |
| Abnormal Return (AR)             | The difference between what the stock actually did and what our model predicted it should have done.          |
| Cumulative Abnormal Return (CAR) | The sum of all abnormal returns across the event window. This is your total "inflation" measure.              |
| Class Period                     | The time between the first misleading statement (T-start) and the corrective disclosure (T-disclosure).       |
| Estimation Window                | The "clean" period before the fraud, used to train your model on the stock's normal behaviour.                |
| Event Window                     | The period you are investigating (the Class Period and surrounding days).                                     |
| Artificial Inflation             | The amount by which the stock price was "pumped up" above its true value because of the misleading statement. |
| Loss Causation                   | Proving that the price drop was caused by the revelation of the truth, not by other factors.                  |
| Transaction Causation            | Proving that the investor relied on the misleading statement when deciding to buy the stock.                  |
| Corrective Disclosure            | The event where the truth comes out - a profit downgrade, contract loss, regulatory action, etc.              |

# Part II: Environment Setup and Data Acquisition

## 2.1 The Technology Stack

Your entire project will be built in Python. Here is every library you will need and what each one does:

| **Library**               | **Purpose**                                                 | **Install Command**                 |
| ------------------------- | ----------------------------------------------------------- | ----------------------------------- |
| pandas                    | Data manipulation and time series handling                  | pip install pandas                  |
| numpy                     | Numerical computation and matrix operations                 | pip install numpy                   |
| yfinance                  | Pull stock price and volume data (with caveats for ASX)     | pip install yfinance                |
| statsmodels               | Run the Fama-French regression, structural break tests, OLS | pip install statsmodels             |
| arch                      | Fit GARCH(1,1) and other volatility models                  | pip install arch                    |
| scipy                     | Statistical testing, probability distributions              | pip install scipy                   |
| matplotlib                | Create publication-quality charts                           | pip install matplotlib              |
| seaborn                   | Statistical visualisation built on matplotlib               | pip install seaborn                 |
| requests + beautifulsoup4 | Web scraping for ASX announcements if needed                | pip install requests beautifulsoup4 |
| jupyter                   | Interactive notebook for presenting your analysis           | pip install jupyter                 |

## 2.2 Python vs R vs Stata: Industry Standards

**Stata** is actually the most common tool in litigation consulting for event studies and regression-based analysis. Many expert reports filed in court cite Stata output. If you see academic papers on event study methodology, they almost always use Stata. The reason: Stata has built-in, well-documented, court-tested commands for exactly this type of analysis, and judges and opposing experts are familiar with its output format.

**Python** is increasingly used, especially for larger datasets, Monte Carlo simulations, and anything involving machine learning or complex data pipelines. It is the right choice for your project because (a) it demonstrates software engineering ability alongside statistical skill, (b) the arch library for GARCH modelling is excellent, and (c) it shows you can handle the full pipeline from data ingestion to visualisation in a single environment.

**R** is used in academic econometrics research but is less common in consulting practice. It is perfectly fine but does not signal the same "production-ready" capability as Python.

**Bottom line:** Use Python. If Hans asks, tell him you chose Python for the simulation flexibility and the arch library, but that you are comfortable working in Stata for standard event study workflows. This shows adaptability.

## 2.3 Data Sources: Where to Get Everything

This is where getting clean Australian data is harder than getting US data. Here is exactly where to find each piece:

### 2.3.1 Stock Prices and Volume

**Primary: yfinance.** For ASX stocks, append ".AX" to the ticker (e.g., "APX.AX" for Appen). Yfinance pulls from Yahoo Finance and usually has daily OHLCV data going back several years. However, it can have gaps and missing dividends for ASX stocks, so always check for missing dates and fill or flag them.

**Backup: ASX Historical Data.** The ASX website provides end-of-day data. You may need to pay for bulk historical downloads, but for a single stock over 2-3 years, yfinance will suffice.

**What to download:** Daily closing prices and daily trading volume for your target stock, going back at least 18 months before T-start (you need 252 trading days for the estimation window, plus buffer). Also download the same data for the S&P/ASX 200 index (ticker: "^AXJO" in yfinance) to use as your market return proxy.

### 2.3.2 Fama-French Factor Data

**The Problem:** There is no "Australian" Fama-French dataset on the Kenneth French Data Library. The closest proxy is "Asia Pacific ex Japan" factors, which includes Australia but also Hong Kong, Singapore, New Zealand, and others. This is a rough approximation.

**Solution A (Recommended for this project):** Use the Asia-Pacific ex-Japan factors from the Kenneth French Data Library (<https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html>). Download the "Fama/French 5 Factors (2x3) \[Daily\]" file for the Asia-Pacific ex-Japan region. This will give you daily values for MKT-RF, SMB, HML, RMW, and CMA, plus the risk-free rate RF.

**Solution B (More rigorous, if time allows):** Construct your own factors using ASX constituent data. This involves sorting ASX stocks into size/value/profitability/investment portfolios and computing the factor returns yourself. This is a substantial amount of work but would be a major differentiator. For your first pass, use Solution A and note the limitation in your write-up.

**The Risk-Free Rate:** Use the Reserve Bank of Australia (RBA) daily yield on 90-day Bank Accepted Bills. This is available from the RBA's Statistical Tables (Table F1). Divide the annual yield by 252 to get a daily risk-free rate.

### 2.3.3 ASX Announcements

You need the actual text and dates of the company's announcements (both the misleading one and the corrective disclosure). These are available on the ASX's company announcements platform. Search by company code and date range. The announcements are free to access and are PDFs.

## 2.4 Selecting Your Target Company

You need a company that satisfies all of the following criteria:

- Listed on the ASX (so all data is publicly available).
- Made a specific, identifiable claim about AI capabilities, AI-driven revenue, or AI-powered products.
- Subsequently experienced a material stock price decline around a "corrective disclosure" - a profit downgrade, contract loss, product failure announcement, or regulatory finding.
- The corrective disclosure can be plausibly linked to the earlier AI claim.
- The stock is liquid enough to have meaningful daily trading volume (needed for the PDM).

**Strong Candidates:**

- **Appen (APX):** Was the ASX's "AI darling." Made aggressive growth claims around its AI data-labelling business. Subsequently lost major contracts (including Google) and revised forecasts dramatically downward. Multiple corrective disclosure events to choose from. Highly liquid.
- **WiseTech Global (WTC):** Experienced governance-related disclosures alongside AI/tech claims. Check for corrective disclosures that triggered abnormal price declines.
- **BrainChip (BRN):** A speculative AI/neuromorphic computing company with a volatile price history. May have clearer "AI-washing" events but is less liquid.

**Recommendation: Start with Appen (APX).** It has the clearest narrative, the most data, and is the most likely to produce a clean event study with statistically significant results.

## 2.5 Setting Up Your Project Structure

Create the following directory structure in your GitHub repository:

forensic-damages-event-study/

data/

raw/ # Raw downloaded CSVs and factor files

processed/ # Cleaned and merged datasets

notebooks/

01_data_acquisition.ipynb

02_eda_and_tstart.ipynb

03_counterfactual_model.ipynb

04_monte_carlo.ipynb

05_damage_quantification.ipynb

06_visualisation.ipynb

src/

event_study.py # Core model functions

garch_model.py # GARCH fitting and simulation

pdm.py # Proportional Decay Model

utils.py # Data loading and cleaning helpers

output/

figures/ # Publication-ready charts

memo/ # Executive memo PDF

requirements.txt

README.md

# Part III: Phase 1 - Forensic Discovery

## 3.1 Understanding the Class Period

The "Class Period" is the legal term for the window of time between the first misleading statement and the corrective disclosure. In plain English: it is the period during which the stock price was artificially inflated because investors were making decisions based on false information.

Your entire analysis revolves around three time boundaries:

- **T-start:** The date the misleading statement was made (when the artificial inflation entered the stock price).
- **T-disclosure:** The date the corrective disclosure occurred (when the truth came out and the inflation was wiped out).
- **The Estimation Window end:** The day before T-start. Your model is trained on the period before the fraud.

Visually, the timeline looks like this:

\[...252 trading days of clean data...\] | T-start --- Class Period --- T-disclosure | \[post-event...\]

## 3.2 Identifying T-disclosure (The Corrective Disclosure Date)

This is usually the easier date to find. You are looking for the specific day when new information entered the market that corrected the earlier misstatement. Common triggers include a profit downgrade or earnings miss announcement, loss of a major contract or customer, a regulatory finding or investigation announcement, a whistleblower report or investigative journalism piece, or an admission by the company that prior guidance was overstated.

**How to find it:** Go to the ASX announcements platform, search for your target company, and look for announcements that coincide with sharp price drops. Cross-reference with yfinance price data. You are looking for a day where (a) the company released material information and (b) the stock dropped significantly.

**Critical nuance:** There may be multiple corrective disclosures. The truth often comes out in stages - a partial revelation, then a fuller admission. In a real litigation case, you might model each one separately. For this project, pick the single most dramatic corrective disclosure to keep the analysis clean.

## 3.3 Identifying T-start (The Misleading Statement Date)

This is the harder and more contentious date. In a real court case, T-start is primarily determined by the legal team, who identify the specific statement that was misleading. The econometrician then provides statistical support for that date.

**Your approach for this project:**

- **Start with the qualitative anchor.** Read the company's ASX announcements in the months before T-disclosure. Find the specific announcement where they made the AI-related claim you believe was misleading. This is your candidate T-start.
- **Verify with quantitative analysis.** Run the Quandt-Andrews structural break test (see next section) to check whether the stock's behaviour relative to the market actually shifted on or near that date.
- **Reconcile.** If the structural break occurs on or very near your qualitative date, you have strong evidence. If it occurs a few days before, this may indicate information leakage (see Section 3.6). If it occurs much later, the market may not have "believed" the lie immediately.

**Important:** The Fama-French model in Phase 2 will automatically control for other news events (interest rate changes, sector movements, etc.) because those are captured by the factor returns. So you do not need to manually account for every piece of unrelated news on or near T-start. The model subtracts the market's influence; whatever is left is the company-specific effect.

## 3.4 The Quandt-Andrews Structural Break Test

### The Intuition

Imagine you have a regression model that captures the relationship between your stock's returns and the market's returns. The stock normally moves 1.2% for every 1% the market moves. A structural break is the moment this relationship fundamentally changed - perhaps because artificial inflation entered the price, or because the stock decoupled from normal market behaviour.

The Quandt-Andrews test does not require you to specify when the break happened. Instead, it tests every possible date in a range and finds the one that maximises the evidence for a break.

### How It Works Mechanically

- **Set up your base regression.** Regress the stock's daily excess returns against the market excess return (or the full Fama-French factors) over a period that spans before and after your candidate T-start.
- **For every possible break date** in a trimmed range (typically the middle 70% of the sample, to avoid testing near the edges where the test has low power): split the sample into "before" and "after" subsamples, run the regression separately on each, and compute a Chow F-statistic that measures whether the coefficients are significantly different.
- **Take the maximum F-statistic** across all candidate dates. This is your "Sup-F" statistic. The date associated with the maximum is your estimated breakpoint.
- **Compare the Sup-F** to critical values (from Andrews, 1993, or Hansen, 1997) to determine if the break is statistically significant.

### Python Implementation

statsmodels does not have a direct Quandt-Andrews function. You will need to implement the Chow test loop manually:

import statsmodels.api as sm

import numpy as np

def chow_test(y, X, break_idx):

\# Full sample regression

model_full = sm.OLS(y, X).fit()

rss_full = model_full.ssr

\# Pre-break regression

model_pre = sm.OLS(y\[:break_idx\], X\[:break_idx\]).fit()

rss_pre = model_pre.ssr

\# Post-break regression

model_post = sm.OLS(y\[break_idx:\], X\[break_idx:\]).fit()

rss_post = model_post.ssr

\# Chow F-statistic

k = X.shape\[1\] # number of parameters

n = len(y)

f_stat = ((rss_full - rss_pre - rss_post) / k) / \\

((rss_pre + rss_post) / (n - 2 \* k))

return f_stat

\# Loop over candidate break dates (trim 15% from each end)

n = len(y)

trim = int(0.15 \* n)

f_stats = {}

for i in range(trim, n - trim):

f_stats\[dates\[i\]\] = chow_test(y, X, i)

\# The date with max F-stat is your estimated structural break

t_start_candidate = max(f_stats, key=f_stats.get)

## 3.5 Confounding Events and How to Handle Them

A confounding event is any news or market development that occurred on or near T-disclosure that could have caused the price drop independently of the corrective disclosure. Opposing counsel in a real case would use these to argue that the stock drop was not caused by the disclosure.

Examples include an RBA interest rate decision on the same day, a major sector-wide sell-off, earnings announcements by competitor companies, or changes to government policy affecting the company's sector.

**How the Fama-French model helps:** The five factors (market, size, value, profitability, investment) capture most macro and sector-level effects. If the ASX 200 dropped 3% on the same day as your corrective disclosure, the model accounts for that. The abnormal return is what's left after subtracting the market's contribution.

**What the model does NOT automatically handle:** Company-specific news that is unrelated to the AI claim. For example, if the company also announced a CEO resignation on the same day as the AI corrective disclosure, you have a confounding event that the Fama-French model will not separate. In this case, you need to: (1) acknowledge the limitation in your write-up, (2) attempt to disaggregate the price impact using intraday data if available, and (3) discuss alternative methodologies for separating the effects.

## 3.6 Information Leakage Detection

Sometimes the stock price starts moving before the official announcement. This could be due to insider trading, analyst leaks, or market rumour. If you observe abnormal returns or abnormal volume in the 1-5 days before T-start or T-disclosure, this is evidence of information leakage.

**How to test for it:**

- Calculate daily abnormal returns for the 5 days preceding your candidate T-start.
- Calculate the average daily trading volume over the estimation window. Compare the volume in the 5 days before T-start to this baseline.
- If abnormal returns are statistically significant (using the SAR test described in Part V) or volume is more than 2 standard deviations above the mean, you have evidence of leakage.

If you find leakage, consider adjusting T-start backward to capture the beginning of the information seepage. Document this clearly.

# Part IV: Phase 2 - Building the Counterfactual Model

## 4.1 The Estimation Window vs the Event Window

This distinction is absolutely critical. You train your model on the estimation window, and then you test it on the event window. Never mix the two.

**The Estimation Window** is a period of "clean" data where no fraud or misleading conduct was occurring. This is typically 252 trading days (one calendar year) ending the day before T-start. The model learns the stock's normal relationship with the market during this window.

**The Event Window** is the period you are investigating. At minimum, it includes T-disclosure and a few days on either side (e.g., T-disclosure ±5 days). For the inflation ribbon, it spans the entire class period from T-start to T-disclosure.

**Why this matters:** If you include any of the fraud period in your estimation window, you are training the model on contaminated data. The model will learn the "inflated" behaviour as if it were normal, which understates the true abnormal return. This is the single most common methodological error in student event studies.

## 4.2 The Fama-French 5-Factor Model

This is the core of your counterfactual. You are modelling the stock's expected return as a function of five systematic risk factors. The idea is that most of a stock's daily return can be explained by broad market forces. Whatever is left over (the residual) is the company-specific component.

**The equation:**

_R(i,t) - R(f,t) = α(i) + β_1\[R(M,t) - R(f,t)\] + β_2 SMB(t) + β_3 HML(t) + β_4 RMW(t) + β_5 CMA(t) + ε(i,t)_

**Variable definitions:**

| **Variable**    | **Name**                      | **What It Represents**                                                                                                      |
| --------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| R(i,t)          | Stock Return                  | The actual daily return of your target stock on day t. Calculated as: (Price today - Price yesterday) / Price yesterday.    |
| R(f,t)          | Risk-Free Rate                | The return on a riskless investment. Use the RBA 90-day bank bill rate, divided by 252 for daily.                           |
| R(i,t) - R(f,t) | Excess Return                 | Your dependent variable. The stock's return above and beyond the risk-free rate.                                            |
| R(M,t) - R(f,t) | Market Risk Premium           | How much the overall market returned above the risk-free rate on day t.                                                     |
| β_1             | Market Beta                   | The stock's sensitivity to the market. A beta of 1.2 means: for every 1% the market moves, this stock typically moves 1.2%. |
| SMB(t)          | Small Minus Big               | The return difference between small-cap and large-cap portfolios on day t.                                                  |
| HML(t)          | High Minus Low                | The return difference between high book-to-market (value) and low book-to-market (growth) stocks.                           |
| RMW(t)          | Robust Minus Weak             | The return difference between highly profitable and weakly profitable firms.                                                |
| CMA(t)          | Conservative Minus Aggressive | The return difference between firms that invest conservatively vs aggressively.                                             |
| α(i)            | Alpha / Intercept             | The stock's average daily return NOT explained by any factor. Should be close to zero.                                      |
| ε(i,t)          | Residual / Error Term         | The unexplained component. In the estimation window, this is noise. In the event window, this is your Abnormal Return.      |

## 4.3 Running the Regression

Here is the step-by-step Python workflow:

import pandas as pd

import numpy as np

import statsmodels.api as sm

\# 1. Compute daily returns for your stock

stock\['return'\] = stock\['Close'\].pct_change()

\# 2. Compute excess return

stock\['excess_return'\] = stock\['return'\] - ff_data\['RF'\]

\# 3. Filter to estimation window only (252 days before T_start)

est_data = merged\[(merged.index < T_start)\]\[-252:\]

\# 4. Define X (factors) and y (stock excess return)

X = est_data\[\['Mkt-RF', 'SMB', 'HML', 'RMW', 'CMA'\]\]

X = sm.add_constant(X) # Adds the alpha intercept

y = est_data\['excess_return'\]

\# 5. Run OLS

model = sm.OLS(y, X).fit()

print(model.summary())

**What to check in the output:**

- **R-squared:** Should be at least 0.15-0.40 for individual stocks. If very low, the five factors do not explain much of the stock's movement, which weakens your counterfactual.
- **F-statistic p-value:** Should be < 0.05, confirming the model is jointly significant.
- **Individual coefficient p-values:** The market factor (β_1) should almost always be significant. Others may or may not be.
- **Alpha:** Should be close to zero and statistically insignificant. If alpha is large and significant, the stock was systematically outperforming or underperforming even in the clean period.

## 4.4 Residual Diagnostics

Before moving to GARCH, examine the residuals from your OLS regression:

- **Plot the residuals over time.** Look for obvious patterns. If you see volatility clustering (periods of large residuals followed by more large residuals), this confirms you need GARCH.
- **Run a Ljung-Box test** on the squared residuals (statsmodels.stats.diagnostic.acorr_ljungbox). If significant, there are ARCH effects in the residuals - confirming GARCH is appropriate.
- **Plot the ACF/PACF** of the squared residuals. If you see significant autocorrelation at lag 1, GARCH(1,1) is the right specification.

## 4.5 The GARCH(1,1) Volatility Layer

The standard OLS regression assumes that the variance of the error term is constant over time (homoskedasticity). In financial data, this is almost never true. Volatility clusters: big price swings tend to follow big price swings.

**The GARCH(1,1) equation:**

_σ²(t) = ω + α ε²(t-1) + β σ²(t-1)_

| **Variable** | **Name**                | **Meaning**                                                                                                 |
| ------------ | ----------------------- | ----------------------------------------------------------------------------------------------------------- |
| σ²(t)        | Conditional Variance    | The model's prediction for today's variance, given everything that happened up to yesterday.                |
| ω (omega)    | Long-Run Variance       | The baseline level of variance the stock always reverts to. The stock's "resting heart rate" of volatility. |
| α (alpha)    | ARCH Parameter          | How much yesterday's shock affects today's volatility. High α = shocks have big immediate impact.           |
| ε²(t-1)      | Lagged Squared Residual | Yesterday's squared residual from the FF regression. The "news" or "shock" component.                       |
| β (beta)     | GARCH Parameter         | How persistent volatility is. High β = once volatility rises, it stays elevated for a long time.            |
| σ²(t-1)      | Lagged Variance         | Yesterday's conditional variance. Creates the "memory" in volatility.                                       |

**The intuition:** Imagine a pond. The long-run variance (ω) is the natural ripple level. When you throw a stone (ε²), it creates a splash (α). The ripples persist for a while (β). GARCH tells you exactly how big the ripples should be at any given moment. If the actual stock price movement is far larger than the GARCH-predicted ripple, you have statistical evidence that something extraordinary happened.

### Python Implementation

from arch import arch_model

\# Fit GARCH to the OLS residuals (from estimation window)

residuals = model.resid \* 100 # arch library prefers percentage returns

garch = arch_model(residuals, vol='Garch', p=1, q=1, dist='t')

\# Using Student's t-distribution for heavier tails (more realistic)

garch_fit = garch.fit(disp='off')

print(garch_fit.summary())

\# Extract conditional volatility

cond_vol = garch_fit.conditional_volatility

**Why Student's t-distribution?** Financial returns have "fat tails" - extreme events happen more often than a normal distribution predicts. Using a t-distribution gives more realistic confidence intervals. This is a PhD-level detail that Hans would expect to see.

**Stationarity condition:** α + β < 1. If this condition is violated, the volatility process is explosive and the model is misspecified. Typical financial data has α + β very close to 1 (around 0.95-0.99), indicating high persistence.

## 4.6 Generating the But-For Expected Returns

Now you have your trained model. To generate the counterfactual, push the actual market data from the event window through the model:

\# Get event window data

event_data = merged\[(merged.index >= T_start) &

(merged.index <= T_disclosure + pd.Timedelta(days=10))\]

\# Predicted (expected) excess return for each event day

X_event = event_data\[\['Mkt-RF', 'SMB', 'HML', 'RMW', 'CMA'\]\]

X_event = sm.add_constant(X_event)

expected_excess_return = model.predict(X_event)

\# Convert to expected price level (compound from pre-T_start price)

but_for_price = \[actual_price_at_T_start_minus_1\]

for r in expected_excess_return + ff_data.loc\[event_data.index, 'RF'\]:

but_for_price.append(but_for_price\[-1\] \* (1 + r))

The result is two price series: the actual price (what really happened) and the but-for price (what should have happened in the absence of the misleading statement). The gap between them is the artificial inflation.

# Part V: Phase 3 - Statistical Significance Testing

## 5.1 Abnormal Returns (AR) and Cumulative Abnormal Returns (CAR)

The Abnormal Return on any given day is simply the difference between the actual return and the expected return:

_AR(i,t) = R(i,t) - R̂(i,t)_

On days with no news, the AR should be small and randomly distributed around zero. On the day of a corrective disclosure, you expect a large negative AR.

The Cumulative Abnormal Return (CAR) is the sum of ARs over the event window:

_CAR(i, t1, t2) = Σ AR(i,t) for t = t1 to t2_

If you compute CAR over a window of \[-1, +3\] around T-disclosure, you capture the immediate impact plus any delayed reaction.

## 5.2 Standardised Abnormal Returns (SAR)

To determine if an abnormal return is statistically significant, divide by the GARCH conditional standard deviation:

_SAR(i,t) = AR(i,t) / σ(t)_

If SAR > 1.96, the abnormal return is significant at the 5% level. If SAR > 2.576, significant at the 1% level.

**Why this matters:** Suppose the stock dropped 5% on the day of the disclosure. Without GARCH, you might compare this to the average daily standard deviation of 2% and declare it significant. But what if the stock had been very volatile recently and the GARCH-predicted σ for that day was 4%? Then a 5% drop is only 1.25 standard deviations - not significant. The GARCH adjustment prevents you from over-claiming.

## 5.3 The Boehmer-Musumeci-Poulsen (BMP) Robust Test

The standard SAR test assumes the forecast error is homoskedastic. The BMP test corrects for event-induced variance - the fact that volatility itself tends to spike on event days.

The BMP test adjusts the cross-sectional standard deviation using the estimation window's variance as a baseline. In a single-stock event study, the BMP test uses a more robust estimate of the standard error.

**Reference:** Boehmer, E., Musumeci, J., and Poulsen, A. (1991). "Event-study methodology under conditions of event-induced variance." Journal of Financial Economics, 30(2), 253-272.

## 5.4 Cross-Sectional Tests (Extension)

If you extend the project to examine multiple AI-related corrective disclosures across several companies, you can compute the CAR for each company around its corrective disclosure date and test whether the average CAR across all companies is significantly different from zero. This strengthens the argument that AI-related disclosures systematically affect stock prices.

# Part VI: Phase 4 - Monte Carlo Simulation

## 6.1 Why a Single Counterfactual Line Is Not Enough

In a courtroom, an expert who presents a single "but-for" price line is vulnerable to attack. Opposing counsel will argue: "Your model gives one estimate, but stock prices are random. How confident are you?"

The Monte Carlo simulation addresses this by generating thousands of possible but-for price paths. Instead of saying "the but-for price on T-disclosure was \$4.50," you can say: "Based on 10,000 simulated paths, there is a 95% probability the but-for price on T-disclosure was between \$4.20 and \$4.80." This is much harder to discredit.

## 6.2 Parametric Simulation from the GARCH Process

This is the primary method. You use the GARCH model's parameters to generate simulated residuals:

- **Extract your fitted GARCH parameters:** ω, α, β, and the degrees of freedom (ν) from the Student's t-distribution.
- **For each of 10,000 simulation runs:** Start with the last conditional variance from the estimation window. For each day in the event window, simulate a new residual by drawing from a scaled t-distribution: ε(t) = σ(t) \* z(t), where z(t) ~ t(ν) and σ²(t) = ω + αε²(t-1) + βσ²(t-1). Add the simulated residual to the FF expected return to get a simulated total return. Compound to get a simulated price path.
- **Store all 10,000 price paths.**

**Python pseudocode:**

import numpy as np

from scipy.stats import t as t_dist

n_sims = 10000

n_days = len(event_window)

omega = garch_fit.params\['omega'\]

alpha = garch_fit.params\['alpha\[1\]'\]

beta = garch_fit.params\['beta\[1\]'\]

nu = garch_fit.params\['nu'\] # degrees of freedom

sigma2_init = garch_fit.conditional_volatility.iloc\[-1\] \*\* 2

simulated_prices = np.zeros((n_sims, n_days + 1))

simulated_prices\[:, 0\] = actual_price_before_T_start

for sim in range(n_sims):

sigma2 = sigma2_init

eps_prev = 0

for day in range(n_days):

z = t_dist.rvs(df=nu)

eps = np.sqrt(sigma2) \* z

sim_return = expected_excess\[day\] + rf\[day\] + eps / 100

simulated_prices\[sim, day+1\] = simulated_prices\[sim, day\] \* (1 + sim_return)

sigma2 = omega + alpha \* eps\*\*2 + beta \* sigma2

## 6.3 Bootstrap Residual Resampling

An alternative: instead of assuming a parametric distribution, resample (with replacement) from the actual estimation-window residuals. For each simulation run, randomly draw a sequence of residuals the same length as the event window, add them to the expected returns, and compound to get price paths.

**Advantage:** No distributional assumption required.

**Disadvantage:** Does not model volatility dynamics. Consider block bootstrap to preserve some time-series structure.

For maximum rigour, present both the parametric GARCH simulation and the bootstrap results side by side. If they agree, your conclusions are robust.

## 6.4 Constructing the Confidence Ribbon

lower_2_5 = np.percentile(simulated_prices, 2.5, axis=0)

upper_97_5 = np.percentile(simulated_prices, 97.5, axis=0)

median_path = np.percentile(simulated_prices, 50, axis=0)

Plot as a shaded ribbon (95% CI) with the median path as the central but-for estimate, and overlay the actual price. If the actual price on T-disclosure falls below the lower bound of the confidence ribbon, you have extremely strong evidence of a statistically significant loss.

## 6.5 Interpreting the Results

From the 10,000 simulations, you can compute:

- **The probability that the actual return was "normal":** What fraction of simulated paths produced a return as bad as or worse than the actual return on T-disclosure? If only 12 out of 10,000 paths are that bad, you can say: "The probability of this drop occurring by chance is 0.12%."
- **A confidence interval for total damages:** Compute the inflation (actual price - but-for price) at T-disclosure for each simulation. This gives you a distribution of damage estimates. Report the mean, median, and 95% CI.

# Part VII: Phase 5 - Damage Quantification

## 7.1 The Inflation Ribbon

The Inflation Ribbon is the visual representation of the artificial inflation in the stock price over the class period. On a chart, it is the shaded area between the actual price line and the but-for price line.

For each day t in the class period:

_Inflation(t) = Actual Price(t) - But-For Price(t)_

## 7.2 Constant Dollar vs Constant Percentage Inflation

**Constant Dollar:** Assumes inflation is a fixed dollar amount. If the corrective disclosure caused a \$2.00 abnormal drop, then the inflation was \$2.00 every day of the class period. Simpler but less realistic for volatile stocks.

**Constant Percentage:** Assumes inflation is a fixed percentage of the stock price. If the disclosure caused a 15% abnormal drop, then the inflation on any given day was 15% of that day's actual price. More common in practice for tech/growth stocks because it accounts for the fact that as the stock price rises, the dollar inflation also rises proportionally.

**Which to use:** For a tech stock like Appen, use constant percentage. Calculate the percentage drop on T-disclosure attributable to the abnormal return, and apply this percentage backward across the class period.

## 7.3 The Proportional Decay Model (PDM)

The PDM estimates which shareholders actually suffered a loss. Not everyone who bought during the class period held the stock until the crash. If someone bought at the inflated price and sold before the truth came out, they did not lose money.

**Core assumption:** On any given day, every outstanding share has an equal probability of being traded ("proportional trading" assumption).

### Step 1: Compute the Daily Turnover Rate

_τ(t) = Volume(t) / Shares Outstanding_

This is the probability that any given share changes hands on day t. Get Volume from yfinance and Shares Outstanding from the company's annual report.

### Step 2: Build the Share Density Matrix

For shares purchased on day t, the number still held on a later day k is:

_S(t,k) = N(t) × ∏(j=t+1 to k) \[1 - τ(j)\]_

**In plain English:** Take the number of shares bought on day t (which equals the volume on day t, N(t)). Then, for each subsequent day, multiply by the probability that those shares were NOT sold (1 minus the turnover rate). This decays the "surviving" shares over time, like radioactive decay.

### Step 3: The Intuition

Imagine you paint all the shares bought on Monday blue. On Tuesday, some blue shares are sold and replaced with "fresh" unpainted shares. By Friday, only a fraction of Monday's blue shares remain. The PDM calculates exactly how many blue shares survive to each future date.

### Python Implementation

import numpy as np

def build_share_density(volume, shares_outstanding, T_start_idx, T_disc_idx):

n_days = T_disc_idx - T_start_idx + 1

turnover = volume / shares_outstanding # daily turnover rate

\# S\[t, k\] = shares bought on day t still held on day k

S = np.zeros((n_days, n_days))

for t in range(n_days):

S\[t, t\] = volume\[T_start_idx + t\] # shares bought on day t

for k in range(t + 1, n_days):

S\[t, k\] = S\[t, k-1\] \* (1 - turnover\[T_start_idx + k\])

return S

## 7.4 The Restricted Two-Trader Model (RTTM)

The basic PDM assumes every share can trade. In reality, many shares are held by index funds, ETFs, insiders, and long-term institutional holders who essentially never trade. If you do not adjust for this, your turnover rate is artificially high.

**The adjustment:** Instead of total Shares Outstanding as the denominator, use only "Free Float" or "Tradeable Shares":

_τ_adjusted(t) = Volume(t) / Free Float Shares_

Free Float data can be obtained from financial data providers (Bloomberg, Refinitiv) or from the ASX's free float methodology. If unavailable, a reasonable approximation is 70-80% of total shares outstanding for a typical ASX 200 company.

**Why Hans would care:** If you use the unadjusted PDM, opposing counsel will point out that your model assumes index fund shares are trading when they are not. By using the RTTM, you pre-emptively address this criticism. This is the kind of methodological refinement that separates a student project from an expert report.

## 7.5 Aggregate Loss Calculation

The Aggregate Loss combines the inflation ribbon with the share density matrix:

_L = Σ(t=T_start to T_disc) Σ(k=t to T_disc) \[S(t,k) × Damage Per Share(t,k)\]_

**Simplification for your project:** The Damage Per Share is typically the inflation at T-disclosure (the full drop). So:

_L = Inflation per share × Σ(t=T_start to T_disc) S(t, T_disc)_

In other words: multiply the per-share inflation by the total number of shares that were bought during the class period and still held at T-disclosure.

**Using Monte Carlo:** Since you have 10,000 simulated but-for prices, compute the aggregate loss for each simulation. This gives you a distribution of damage estimates with a 95% confidence interval - exactly what an expert would present in court.

## 7.6 Loss Causation vs Transaction Causation

This is a critical legal distinction that Hans deals with constantly:

**Transaction Causation:** Did the investor rely on the misleading statement when deciding to buy? Would they have bought anyway even without the AI hype? This is a legal question, not a statistical one. Your model does not address this directly - it is typically established through legal arguments about market efficiency (the "fraud on the market" doctrine).

**Loss Causation:** Did the corrective disclosure actually cause the price drop? This IS the statistical question your model answers. Your event study proves that the abnormal return on T-disclosure was statistically significant and not explained by market factors.

In your write-up and conversation with Hans, make clear that your model addresses loss causation. If he asks about transaction causation, demonstrate that you understand the distinction and know it is a separate legal question.

# Part VIII: Visualisation and Presentation

## 8.1 The Key Charts to Produce

**Chart 1: The Inflation Ribbon.** Two lines (actual price vs but-for price) with the gap shaded in red/orange. This is the single most important chart.

**Chart 2: The Monte Carlo Confidence Ribbon.** The median simulated but-for price path, with a shaded 95% confidence interval, and the actual price overlaid. Where the actual price pierces the bottom of the confidence ribbon is your statistical significance moment.

**Chart 3: Daily Abnormal Returns Around T-disclosure.** A bar chart showing AR for each day in a \[-5, +5\] window around T-disclosure. The corrective disclosure date should show a large negative bar.

**Chart 4: GARCH Conditional Volatility.** A line chart showing how σ(t) evolved over the estimation window and event window. Shows the volatility spike at T-disclosure.

**Chart 5: Share Density Decay.** A heatmap or decay curve showing how shares bought at different points in the class period were progressively sold off.

**Chart 6: Structural Break Test.** A plot of the rolling Chow F-statistic over time, with a horizontal line for the critical value. The peak should coincide with your T-start date.

## 8.2 Structuring Your GitHub Repository

Hans (or his analysts) will look at your GitHub. It needs to look professional:

- **README.md:** Clear summary of the project, methodology, key findings, and how to run the code. Include a thumbnail of the inflation ribbon chart.
- **requirements.txt:** Every dependency with version numbers.
- **Numbered notebooks:** 01 through 06, each corresponding to a phase.
- **Clean code:** Docstrings, type hints, and clear variable names.
- **No hardcoded paths:** Use relative paths and a config file for dates and ticker symbols.

## 8.3 Writing the Executive Memo

Produce a 2-3 page memo. Structure:

- **Executive Summary** (1 paragraph): What you did, what you found, the estimated damages.
- **Methodology** (1 page): FF5 + GARCH + Monte Carlo + PDM at a high level.
- **Key Findings** (half page): The CAR, the SAR, the p-value, the aggregate loss estimate with 95% CI.
- **Limitations** (quarter page): Asia-Pacific factors vs Australian-specific, single stock vs portfolio, data quality caveats.
- **Charts** (appended): The inflation ribbon and Monte Carlo ribbon.

# Part IX: Two-Week Execution Timeline

| **Day** | **Phase**          | **Tasks**                                                                                                                                                         |
| ------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Day 1   | Setup              | Install all libraries. Set up GitHub repo and project structure. Pull stock price data with yfinance. Download FF5 factor data. Download RBA risk-free rate data. |
| Day 2   | Data Cleaning      | Merge stock returns with factor data. Handle missing dates. Compute daily excess returns. Verify data quality.                                                    |
| Day 3   | Forensic Discovery | Read ASX announcements. Identify candidate T-start and T-disclosure. Document the narrative.                                                                      |
| Day 4   | Structural Break   | Implement Quandt-Andrews/Chow test. Verify T-start statistically. Check for information leakage.                                                                  |
| Day 5   | Counterfactual     | Run FF5 regression on estimation window. Check diagnostics. Save coefficients.                                                                                    |
| Day 6   | GARCH Layer        | Fit GARCH(1,1) with Student's t to OLS residuals. Compute SARs.                                                                                                   |
| Day 7   | Rest               | Take a break. Let the concepts settle.                                                                                                                            |
| Day 8   | Monte Carlo        | Implement parametric GARCH simulation (10,000 runs). Implement bootstrap resampling as robustness check.                                                          |
| Day 9   | Confidence Ribbon  | Construct and plot the confidence ribbon. Overlay actual price. Compute probability of observed return.                                                           |
| Day 10  | PDM                | Implement PDM. Compute daily turnover rates. Build share density matrix. Adjust for free float (RTTM).                                                            |
| Day 11  | Aggregate Loss     | Compute per-share inflation. Multiply by surviving shares from PDM. Report aggregate loss with 95% CI.                                                            |
| Day 12  | Visualisation      | Produce all 6 charts. Polish formatting. Export to output/figures/.                                                                                               |
| Day 13  | Documentation      | Write the Executive Memo. Clean up notebooks with markdown. Write README. Push to GitHub.                                                                         |
| Day 14  | Final Review       | Review end-to-end. Prepare conversation starter for Hans. Practice the 60-second elevator pitch.                                                                  |

# Appendix A: Mathematical and Financial Concepts - Full Reference

## A.1 The Fama-French 5-Factor Model

**What it is:** An asset pricing model that explains a stock's expected return using five systematic risk factors: market, size, value, profitability, and investment patterns.

**What it tells you:** It isolates the "idiosyncratic" (company-specific) component of a stock's return.

**Equation:**

_R(i,t) - R(f,t) = α + β_1(Mkt-RF) + β_2(SMB) + β_3(HML) + β_4(RMW) + β_5(CMA) + ε(i,t)_

**Intuition:** A stock's price is heavily dictated by macroeconomic tides. By subtracting the impact of these five tides, whatever price movement is left over is the pure effect of the company's own actions.

**Key property:** The residual ε(i,t) in the estimation window should be mean-zero, approximately normal, and uncorrelated over time.

## A.2 GARCH(1,1)

**Full name:** Generalised Autoregressive Conditional Heteroskedasticity.

**What it is:** A model for the time-varying variance of financial returns.

**Equation:**

_σ²(t) = ω + α ε²(t-1) + β σ²(t-1)_

**Intuition:** Volatility has memory. A big shock yesterday increases today's predicted volatility. Once elevated, volatility stays elevated for a while. Think of a spring: you stretch it (big shock), it gradually returns to resting length (long-run variance).

**Stationarity:** Requires α + β < 1. Typical financial data: 0.95-0.99.

## A.3 Quandt-Andrews Structural Break Test

**What it is:** A test for finding an unknown breakpoint in a time series regression.

**What it tells you:** The exact date the stock's behaviour relative to the market fundamentally changed.

**How it works:** For every candidate date in a trimmed range, it computes a Chow F-statistic. The date with the maximum F-stat is the estimated break. Compare to Andrews (1993) critical values.

**Intuition:** Imagine driving with consistent steering. The structural break is when the tie-rod snaps. The test checks every second of the drive to find the most likely snap point.

## A.4 Abnormal Return (AR) and Cumulative Abnormal Return (CAR)

**AR(i,t):** The difference between actual return and model-predicted return on day t.

**CAR(i, t1, t2):** The sum of ARs over a window. Captures both immediate and delayed market response.

**SAR(i,t):** AR divided by GARCH-predicted conditional standard deviation. Converts AR into a z-score-like test statistic.

## A.5 Proportional Decay Model (PDM)

**What it is:** A simulation that estimates how many shares bought on a given day are still held by the original buyer at any future date.

**Formula:**

_S(t,k) = N(t) × ∏(j=t+1 to k) \[1 - τ(j)\]_

**Variables:**

- **S(t,k):** Shares purchased on day t still held on day k.
- **N(t):** Volume traded on day t.
- **τ(j):** Daily turnover rate on day j = Volume(j) / Tradeable Shares.

**Intuition:** Radioactive decay for shares. Each day, a fraction changes hands. The PDM tracks how much of each day's batch survives.

**RTTM Adjustment:** Replace total shares outstanding with free-float shares in the denominator of τ(j) to account for non-trading holdings.

## A.6 Monte Carlo Simulation

**What it is:** A computational technique generating thousands of random scenarios to estimate the distribution of an uncertain outcome.

**Two approaches:**

- **Parametric:** Draw residuals from the fitted t-distribution and propagate through the GARCH variance equation. Respects volatility dynamics.
- **Bootstrap:** Resample historical residuals with replacement. No distributional assumption, but loses volatility clustering.

**Output:** A distribution of but-for prices at T-disclosure. Report 2.5th and 97.5th percentiles as the 95% CI.

## A.7 The BMP Test

**What it is:** A robust test statistic for event studies that corrects for event-induced variance.

**Why needed:** Standard tests assume constant variance. On event days, volatility spikes. BMP adjusts for this.

**Reference:** Boehmer, Musumeci, and Poulsen (1991). "Event-study methodology under conditions of event-induced variance." Journal of Financial Economics, 30(2), 253-272.

## A.8 Event-Induced Variance

**What it is:** The phenomenon where the event you are studying (the corrective disclosure) itself causes a spike in volatility, making standard significance tests unreliable.

**Why it matters:** If you use the estimation-window standard deviation to test for significance on the event day, but the event day's true volatility is much higher, your test will over-reject the null hypothesis (too many false positives). The BMP test and GARCH framework both address this.

# Appendix B: Python Library Reference

| **Library** | **Key Functions You Will Use**                                 | **Documentation**         |
| ----------- | -------------------------------------------------------------- | ------------------------- |
| pandas      | pd.read_csv(), .pct_change(), .merge(), .loc\[\], .resample()  | pandas.pydata.org         |
| numpy       | np.percentile(), np.prod(), np.zeros(), np.sqrt()              | numpy.org                 |
| yfinance    | yf.download('APX.AX', start, end)                              | pypi.org/project/yfinance |
| statsmodels | sm.OLS().fit(), model.summary(), model.resid, acorr_ljungbox() | statsmodels.org           |
| arch        | arch_model(resid, vol='Garch', p=1, q=1, dist='t').fit()       | arch.readthedocs.io       |
| scipy.stats | t.rvs(df=nu), norm.cdf(), chi2.sf()                            | docs.scipy.org            |
| matplotlib  | plt.plot(), plt.fill_between(), plt.bar(), plt.savefig()       | matplotlib.org            |
| seaborn     | sns.heatmap(), sns.set_theme()                                 | seaborn.pydata.org        |

# Appendix C: Brattle-Specific Vocabulary for Hans

When speaking with Hans Weemaes, use these terms naturally. They signal that you understand his world:

| **Term**               | **What It Means**                                     | **When to Use It**                                                 |
| ---------------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| "But-For" Analysis     | The counterfactual price model                        | "I built a but-for price model using FF5."                         |
| Price Impact           | The abnormal return caused by a specific event        | "I measured the price impact of the corrective disclosure."        |
| Loss Causation         | Statistical proof that the disclosure caused the drop | "The GARCH-adjusted SAR confirms loss causation."                  |
| Idiosyncratic Risk     | Company-specific risk not explained by market factors | "The FF5 residuals capture the idiosyncratic component."           |
| Conditional Variance   | GARCH-predicted volatility for a specific day         | "I used the conditional variance to standardise the AR."           |
| Heteroskedasticity     | Non-constant variance in the error term               | "I used GARCH to address heteroskedasticity."                      |
| Event-Induced Variance | Volatility that spikes because of the event itself    | "I applied the BMP test to correct for event-induced variance."    |
| Inflation Ribbon       | The gap between actual and but-for prices             | "The inflation ribbon shows ~15% cumulative artificial inflation." |
| Proportional Decay     | The share turnover model for aggregate damages        | "I used a restricted two-trader PDM for the aggregate loss."       |
| Confounding Factors    | Other events that could explain the price movement    | "The FF5 model controls for the main confounding factors."         |
| Expert Testimony       | A consultant's sworn statement in court               | "The methodology is designed to be robust for expert testimony."   |

**Final Note:** This manual is your standalone reference. Follow it phase by phase, and you will produce a project that demonstrates PhD-level forensic econometrics to a Principal at The Brattle Group. The key is rigour over flash - Hans will be far more impressed by your GARCH diagnostic checks and your RTTM adjustment than by a pretty chart. Do the math right, and the story tells itself.