# PanicTrader: Algorithmic Trading Bot

![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)
![C++](https://img.shields.io/badge/C++-17-blue.svg)
![Finance](https://img.shields.io/badge/Finance-Algorithmic_Trading-green.svg)

## 📊 Project Overview

**PanicTrader** is an algorithmic trading bot developed to identify, analyze, and capitalize on market patterns across diverse financial datasets. The project began with a fundamental question: Can we move beyond conventional trading approaches to discover and exploit market inefficiencies that aren't captured by standard techniques?

Through rigorous analysis of historical price data, bid-ask spreads, and volatility metrics, this project demonstrates how tailored trading strategies can be developed for specific market behaviors. Rather than applying a one-size-fits-all approach, each dataset was treated as a unique puzzle requiring its own solution.

**Author:** Stefan Ciutina  
**Presented To:** Qfin & IMC

### 🎯 Goals

The primary objective of this project was to explore the terrain of market inefficiencies beyond what conventional trading strategies typically address. Specifically, I aimed to:

- Discover and exploit different types of market inefficiencies across various financial instruments
- Design automated systems capable of identifying these inefficiencies and executing profitable trades
- Move beyond simplistic trend-following or mean-reversion tactics to uncover more nuanced market signals
- Apply statistical methods to process and analyze large financial datasets efficiently
- Implement trading strategies and iteratively refine them through rigorous testing and optimization

This project sits at the intersection of quantitative analysis, statistical modeling, and practical trading implementation—combining theoretical insights with real-world applicability.

## 📈 Trading Strategies

The project developed four distinct trading strategies, each tailored to the unique characteristics observed in different financial datasets. Each strategy represents a different approach to market inefficiency exploitation.

### UEC Stock Strategy: Exploiting Spread Patterns

#### Key Insights

The first breakthrough came while analyzing UEC stock data. While initial price chart analysis didn't reveal obvious patterns (what I candidly noted as "Meh" in my working notes), a deeper investigation into the bid-ask spread uncovered something remarkable.

The spread exhibited a highly non-random pattern where periods of normal, tight spread (around 1.00) consistently alternated with well-defined periods of widened spread (approximately 1.40). This pattern wasn't immediately obvious but became apparent through careful visualization and analysis.

Most importantly, I discovered a fascinating correlation: during low spread periods, UEC exhibited clear directional price movement (trending either up or down), while during high spread periods, the price tended to stagnate with minimal directional movement. This observation became the foundation of the trading strategy.

#### Strategy Development

The core hypothesis was straightforward yet powerful: if we could reliably determine the direction of price movement after a high spread period concluded (and a low spread period began), we could initiate a trade in that direction and hold it until the market showed signs of returning to a high spread environment.

The strategy implementation involved:

1. Continuous monitoring of UEC's bid-ask spread
2. Identification of transitions from high spread to low spread periods
3. Implementation of an observation window (waiting period) after transition to allow the new trend to establish
4. Direction determination using short-term moving averages
5. Position entry in the determined direction with full allocated capital
6. Position exit when high spread conditions reemerged

#### Development Process

The strategy development followed a systematic improvement path:

1. The initial implementation achieved a modest PNL of approximately $4,000, but suffered from significant drawdowns, suggesting issues with signal accuracy or risk management.

2. To improve performance, I identified key parameters for optimization, including the short-term moving average window, high spread exit threshold, moving average turn threshold, and the critical waiting period.

3. Recognizing the computational intensity of parameter optimization, I transpiled the Python-based strategy logic to C++. This decision proved invaluable, accelerating the optimization process from 0.2 parameter combinations per second in Python to around 1,000 combinations per second in C++—a 5,000x speed improvement.

4. Using this performance boost, I conducted a comprehensive grid search across approximately 1,000 different parameter combinations to identify the optimal configuration.

5. The optimized strategy achieved a significantly improved PNL of $8,677 on the training dataset, with more consistent profitable trades.

6. However, when tested on out-of-sample data, performance dropped to approximately $4,000, indicating potential overfitting—a common challenge in strategy development that highlighted the need for either more robust parameters or a simpler model.

#### Key Parameters

The strategy's performance was heavily influenced by several key parameters:

- `short_window`: Controls the lookback period for the short-term moving average, determining how quickly the strategy reacts to recent price changes.
- `HS_exit_threshold`: Defines how much the price needs to move after a high spread period to confirm a new direction.
- `ma_turn_threshold`: Sets the threshold for the rolling average pullback that triggers an early exit from positions.
- `waiting_period`: Determines the observation window after a high spread period before assessing direction, allowing noise to settle and the new trend to establish.

Fine-tuning these parameters proved essential for optimizing the strategy's risk-reward profile and overall performance.

### SOBER Stock Strategy: Volatility-Based Counter-Trend Trading

#### Key Insights

The analysis of SOBER stock revealed completely different characteristics compared to UEC. Unlike UEC, SOBER's bid-ask spread showed no remarkable patterns that could be exploited. However, when examining volatility metrics, I discovered distinct, sharp spikes in volatility that stood out clearly against the background.

Additionally, SOBER exhibited a general downward trend throughout the observed period, suggesting a bearish bias in the underlying market. These two observations—volatility spikes against a backdrop of a generally declining price—formed the foundation for a counter-trend strategy.

#### Strategy Design

The SOBER strategy was designed to capitalize on short-term counter-trend movements following volatility spikes, while maintaining alignment with the overall market direction:

1. The default position was to maintain a short position in SOBER, aligning with its general downward trend—essentially "riding" the predominant market direction.

2. Volatility was calculated as the standard deviation of recent returns over a defined lookback period.

3. When calculated volatility exceeded a predefined critical threshold, the system entered a "ready to entry" state, preparing for a potential counter-trend trade.

4. The strategy then measured the short-term moving average (SMA) of SOBER's price, waiting for it to stop decreasing and begin increasing—signaling a potential short-term bottom.

5. Upon this confirmation, the strategy would go long (counter to the general trend), effectively betting on a short-term reversal.

6. The exit signal came when the moving average of volatility itself reached a maximum and began decreasing, suggesting the period of extreme price fluctuation was subsiding.

7. Risk management was implemented via an absolute price threshold (stop-loss) and spread monitoring to avoid trading during unfavorable conditions.

This approach represents a sophisticated blend of trend alignment (via the default short position) and tactical counter-trend exploitation (via the volatility-triggered long entries).

#### Results

The SOBER strategy proved remarkably effective:

- On the training dataset, it generated a PNL of $92,000
- More importantly, on the out-of-sample (untested) data, it maintained strong performance with a PNL of $52,000

The relatively strong out-of-sample performance (maintaining ~56% of the in-sample performance) suggests a robust strategy that captures a genuine market inefficiency rather than simply fitting to historical noise.

#### Key Parameters

The strategy's effectiveness was highly dependent on proper parameter calibration:

- `short_window` (Default: 5): Controlled the lookback period for the short-term price moving average, with smaller values making it more sensitive to recent price changes—crucial for quickly detecting when prices bottomed out.

- `volatility_window` (Default: 50): Determined the period over which volatility was calculated, with longer windows providing smoother, more stable measures and shorter windows reacting more quickly to sudden changes.

- `volatility_threshold` (Default: 0.002): Set the specific level of volatility required to trigger the strategy's entry condition—essentially defining what constituted an "unusual" market state.

- `vol_ma_window` (Default: 5): Controlled the sensitivity of volatility's own moving average, used to identify when volatility was peaking and beginning to decline.

- `position_size` (Default: 100): Defined the trading size for both the default short position and the counter-trend long entries.

- `price_threshold` (Default: 95): Established an absolute price level for stop-loss protection, immediately exiting long positions if price dropped below this threshold.

### FAWA & SMIF Lead-Lag Strategy: Exploiting Inter-Asset Relationships

#### Key Insights

Moving beyond single-asset strategies, I next investigated potential relationships between pairs of assets. Through correlation analysis of FAWA and SMIF, I discovered a clear lead-lag relationship—a phenomenon where price movements in one asset tend to precede similar movements in another.

Using various statistical measures, I quantified this relationship:

- Price correlation analysis showed a maximum correlation of 0.9671 at a lag of -68 ticks
- Returns correlation similarly peaked at 0.3512 at the same -68 tick lag
- The R² value (coefficient of determination) reached 0.9353 at this lag, indicating that almost 94% of the variance in one series could be explained by the lagged values of the other

These findings suggested a potentially exploitable pattern: movements in FAWA could be used to predict subsequent movements in SMIF.

#### Strategy Design

Rather than directly implementing a fixed 68-tick lag (which might be too rigid given market dynamics), I developed an adaptive approach based on confirmed momentum:

1. The strategy continuously monitored FAWA's price movement using a short-term moving average to filter out noise and identify the underlying trend.

2. It identified significant moves in FAWA, defined as the current SMA moving a certain percentage (e.g., 1.4%) away from recent local extremes.

3. When such a significant move occurred, the system entered a "primed" state, noting the direction of FAWA's movement.

4. Instead of immediately trading, the strategy then waited for confirmation from SMIF, calculating SMIF's own SMA and determining its current trend direction.

5. A trade was only executed when SMIF's SMA direction matched the "primed" direction established by FAWA—essentially requiring confirmation from both assets.

6. The position was then maintained until exit conditions were met (typically related to reversal signals or risk management thresholds).

This confirmatory approach acted as a filter to avoid false signals, only trading when both assets showed alignment in their directional movement.

#### Results

The lead-lag strategy delivered consistent positive results:

- Achieved PNLs in the range of $8,000-$9,000 across various parameter configurations
- The best result recorded was $8,974.20 with 33 trades using the parameter set (Leader: 39, Follower: 14, Threshold: 1.4%)
- Interestingly, the originally identified 68-tick lag proved "not relevant at all" in the final implemented strategy—highlighting how statistical findings must often be adapted pragmatically for effective trading implementation

This strategy demonstrates how inter-asset relationships can be exploited without necessarily using complex statistical arbitrage techniques.

#### Key Parameters

Three primary parameters controlled the strategy's behavior:

- `Leader Window`: Determined FAWA's SMA lookback period, affecting how smoothed the price signal was and how quickly it responded to changes.
- `Follower Window`: Set SMIF's SMA lookback period, with different optimal values than the leader window due to the different characteristics of the asset.
- `Threshold %`: Established the percentage move in FAWA required to prime a signal—essentially setting the sensitivity of the trigger mechanism.

Optimizing these parameters required extensive testing and validation to find values that captured genuine signals while filtering out market noise.

### ORE, SHEEP, VP, WHEAT ETF/Correlation Strategy: Multi-Asset Statistical Arbitrage

#### Key Insights

The final and most complex strategy involved four distinct assets: ORE, SHEEP, VP, and WHEAT. Initial comparative analysis of mid-prices revealed that VP traded at significantly higher price levels and exhibited different dynamics compared to the other three assets, which clustered at lower price levels.

While spread analysis yielded no actionable patterns, correlation analysis uncovered significant relationships:

- A strong positive correlation of approximately 0.846 between ORE and VP
- A moderate positive correlation of around 0.593 between VP and WHEAT
- Weaker correlations among other asset pairs

These correlations suggested a potential "ETF situation" or underlying economic relationship—perhaps ORE was a key input for VP production, or an exchange-traded fund existed that tracked these commodities as a basket.

To quantify these relationships more formally, I employed multiple linear regression modeling with impressive results:

- When predicting ORE using the other assets, the model achieved an R² of 0.963
- For VP predicted by the other assets, the R² was even higher at 0.977

These extraordinarily high R² values indicated that almost 98% of VP's price movements could be statistically explained by the combined movements of SHEEP, ORE, and WHEAT—a relationship strong enough to potentially exploit through statistical arbitrage.

#### Strategy Development

To develop a robust trading strategy based on these statistical relationships, I followed a systematic process:

1. First, I built multiple linear regression models with each asset as the target variable, predicted by combinations of the others. This comprehensive approach allowed me to identify which relationships were strongest and most consistent.

2. Recognizing the dangers of in-sample overfitting, I implemented Forward Chaining Cross-Validation—a technique that respects the temporal nature of financial data by training on an initial segment and testing on subsequent periods, then expanding the training window forward.

3. This validation process revealed important insights about coefficient stability and how the model's predictive power varied across different time periods—sometimes maintaining high R² values (up to 0.9) and other times dropping significantly (near 0).

4. Using these insights, I developed a trading strategy based on deviations between actual prices and model-predicted values—essentially betting that when an asset's price diverged significantly from what the model predicted, it would eventually revert toward the predicted value.

5. The strategy parameters were optimized using advanced 3D visualization of the parameter space, plotting PNL as a function of key thresholds to identify optimal operating regions.

This approach represents a sophisticated application of statistical methods to multi-asset trading, moving well beyond simple correlation or technical analysis.

#### Results

The multi-asset strategy delivered exceptional results:

- Total PNL: $1,750,067.28, by far the most profitable strategy in the project
- The PNL was heavily dominated by trades in VP (~$1.73 Million), with much smaller contributions from SHEEP (~$21,000) and ORE (~$1,700)
- No profitable trades were found for WHEAT within the tested parameter ranges

The extraordinary performance of the VP component suggests that the statistical relationships identified were capturing a genuine market inefficiency—one that could be exploited through careful modeling and execution.

#### Key Parameters

The optimal parameter configuration included:

- `Positive DiffMAThreshold`: 34 (Threshold for positive deviations between actual and predicted prices)
- `Negative DiffMAThreshold`: -30.4 (Threshold for negative deviations)
- `RollingAvgWindow`: 1 (Window for smoothing the difference calculations)
- `FixedOrderQuantity`: 100 (Standard position size for trades)

Interestingly, extensive parameter "fuzzing" (randomized exploration) in C++ failed to find significantly more profitable configurations, suggesting the strategy was robust to minor parameter variations around the optimum.

## 🛠️ Technical Implementation

The project's technical implementation demonstrates a practical hybrid approach that leverages the strengths of different programming languages and visualization techniques.

### Programming Languages

My technical approach utilized the complementary strengths of two programming languages:

- **Python:** I used Python for initial data exploration, visualization, and rapid prototyping of strategies. Python's rich ecosystem of data analysis libraries (pandas, numpy, matplotlib, scipy, etc.) made it ideal for quickly testing ideas and generating visualizations that helped identify patterns. The language's flexibility and readability allowed for fast iteration on strategy concepts.

- **C++:** When it came time for parameter optimization and large-scale backtesting, I transpiled key algorithms to C++. This decision was driven by performance requirements: C++ provided dramatic speed improvements (up to 5,000x faster in some cases) compared to Python for computationally intensive tasks. This performance boost enabled much more comprehensive parameter searches across thousands of combinations.

This two-language approach represents a pragmatic workflow often used in quantitative finance: prototype in an accessible, high-level language, then optimize critical components in a faster, lower-level language when performance becomes crucial.

### Visualization Methods

Visualization played a central role throughout the project, from initial pattern discovery to final strategy validation:

- **Price and Spread Plots:** Basic time series visualizations of prices and spreads helped identify initial patterns, particularly the alternating spread regimes in UEC that became the foundation for the first strategy.

- **Volatility Analysis:** Specialized plots highlighting volatility spikes were essential for developing the SOBER strategy, making otherwise hidden patterns visually apparent.

- **Correlation Matrices:** Heat maps of correlation coefficients between different assets guided the development of the multi-asset strategies, quickly revealing which relationships were worth exploring further.

- **R² and Prediction Error Charts:** These visualizations helped assess model quality across different time periods, particularly during cross-validation, revealing when and how the statistical relationships changed over time.

- **3D PNL Surface Visualizations:** These advanced visualizations showed how strategy performance varied across different parameter combinations, making it possible to identify optimal parameter regions visually.

- **Interactive HTML Trading Performance Plots:** The final strategy visualizations included interactive elements showing price movements, model predictions, deviations, and position histories over time—providing a comprehensive view of strategy behavior.

Good visualization wasn't merely an aesthetic choice but a crucial analytical tool that enabled pattern discovery and strategic insights throughout the project.

## 🔍 Key Findings & Insights

This project yielded several valuable insights about market behavior and algorithmic trading development:

1. **Market Microstructure Signals:** The UEC strategy demonstrated that bid-ask spread patterns can provide valuable trading signals. Markets often exhibit microstructure patterns not visible in price charts alone, and these patterns can be systematically exploited. The alternating spread regimes in UEC correlated strongly with directional vs. range-bound price behavior, creating exploitable opportunities.

2. **Volatility as Alpha Source:** The SOBER strategy showed that spikes in volatility can indicate profitable counter-trend opportunities. Rather than viewing volatility merely as risk, recognizing it as a signal—particularly when combined with short-term momentum indicators—can generate significant alpha. The strategy's strong out-of-sample performance suggests it captured a genuine market inefficiency.

3. **Lead-Lag Relationships:** The FAWA & SMIF strategy demonstrated that price movements in one asset can successfully predict movements in another, but in ways that require adaptive implementation rather than fixed lags. Interestingly, while statistical analysis identified a 68-tick optimal lag, the final strategy performed best using a confirmation-based approach rather than a rigid timing offset.

4. **Statistical Arbitrage Potential:** The multi-asset correlation strategy revealed that linear regression models can identify significant mispricings with extraordinary predictive power (R² values approaching 0.98). These statistical relationships, when properly implemented with appropriate entry and exit thresholds, can generate exceptional returns—as evidenced by the $1.75 million PNL, primarily from VP trading.

5. **Implementation Efficiency Matters:** The project demonstrated that technical implementation decisions can dramatically impact strategy development. The 1000x faster parameter searches enabled by C++ implementation meant the difference between testing a handful of parameter combinations and thoroughly exploring thousands—directly contributing to strategy performance improvements.

6. **Cross-Validation Importance:** Forward Chaining Cross-Validation proved essential for understanding how statistical relationships evolve over time. The VP strategy's coefficient stability analysis revealed that while relationships were generally strong, they weren't constant—providing valuable insights for real-world strategy implementation.

These findings have broad implications beyond the specific strategies developed, offering insights into market behavior and quantitative strategy development approaches that could be applied to many different financial instruments and markets.

## 📝 Future Improvements

Several promising avenues exist for further refining and extending these strategies:

- **Dynamic Exit Strategies:** Replace fixed thresholds with adaptive exit mechanisms that respond to changing market conditions. For instance, implementing trailing stops based on volatility or using dynamic time-based exits could improve risk management, particularly for the UEC and SOBER strategies.

- **Regime-Based Coefficient Estimation:** For the multi-asset regression strategy, implementing automatic detection of regime changes and re-estimating model coefficients accordingly could improve performance during periods when the statistical relationships change significantly.

- **Model Simplification for UEC:** To address potential overfitting in the UEC strategy, exploring simpler models with fewer parameters might improve out-of-sample performance, even if in-sample results are somewhat reduced.

- **Volatility-Based Position Sizing:** Implement dynamic position sizing based on current volatility levels and signal conviction, rather than the fixed position sizes currently used. This could optimize the risk-reward ratio across different market conditions.

- **Alternative Stop-Loss Mechanisms:** Particularly for the SOBER strategy, exploring ratio-based stops (related to the magnitude of the initial volatile move) instead of fixed price thresholds could provide more adaptive risk management.

- **Ensemble Approach:** Combine multiple strategies into an ensemble, potentially allowing them to offset each other's weaknesses while capitalizing on their respective strengths across different market conditions.

- **Machine Learning Extensions:** Explore whether incorporating machine learning techniques (particularly for feature selection and non-linear pattern recognition) could enhance the existing statistical approaches.

---

*Note: This project was developed for educational purposes. Past performance is not indicative of future results.*