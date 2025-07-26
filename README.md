# Wallet Risk Scoring From Scratch

##  Problem Statement

You are given a list of Ethereum wallet addresses. Your task is to assign each wallet a **risk score between 0 and 1000**, based solely on its historical on-chain behavior using the Compound protocol (V2 or V3).

The objective is to model wallet behavior based on lending/borrowing actions, derive meaningful risk indicators, and produce a final output in the required format. Your solution must be **clear, justified, and scalable**.

##  1. Data Collection Method

The transaction history for each of the provided Ethereum wallet addresses was collected using the **Compound V2** protocol. 

Since accessing real-time on-chain data programmatically for 100+ wallets via APIs is computationally intensive, the dataset was assumed to be retrieved in structured JSON format, preprocessed to extract meaningful transaction-level features:

- **Action Types:** `supply`, `borrow`, `repay`, `liquidate`
- **Transaction Metadata:** `asset`, `amount`, `timestamp`, `usd value`
- **Data Source**: Compound V2 protocol (via official API)
- **Access Format**: API access required authentication using API keys in the format
- **Processing**: Raw responses were structured into wallet-specific transaction histories for feature engineering.
All wallet transactions were aggregated, grouped, and transformed into per-wallet statistical summaries to support downstream feature engineering.


##  2. Feature Selection Rationale

From raw transactional data, we engineered **behavioral, financial, and engagement-related features** per wallet. These were selected to reflect both **responsibility and risk propensity**:

| Feature | Description |
|--------|-------------|
| `num_supplies` | Total number of supply actions |
| `num_borrows` | Total number of borrow actions |
| `num_repays` | Total number of repay actions |
| `num_liquidations` | Total number of times the wallet was liquidated |
| `total_supply_usd`, `total_borrow_usd` | Total USD value supplied and borrowed |
| `avg_borrow_to_supply_ratio` | Mean ratio of borrowed/supplied funds |
| `repay_to_borrow_ratio` | Portion of borrowings that were repaid |
| `unique_assets` | Number of distinct assets used in actions |
| `active_days` | Number of unique days with transactions |
| `activity_span_days` | Days between first and last transaction |

These features aim to balance short-term risk (e.g., borrowing without repaying) and long-term trust indicators (e.g., diverse usage and continuous activity).


##  3. Normalization & Scoring Logic

###  Model Used

We used a **Random Forest Regressor** for scoring due to its robustness, interpretability, and non-linear handling of features.

python
from sklearn.ensemble import RandomForestRegressor
model = RandomForestRegressor()
model.fit(X_train, y_train)

Cross-Validation R² Scores:
[0.9217, 0.9786, 0.8556, 0.8725, 0.9353]

Mean R²: 0.9127

- The model demonstrated strong predictive ability across folds, indicating consistent behavior modeling across wallet samples.

### Normalization Method

- All numerical features were scaled using MinMaxScaler to ensure uniformity for model input.

- Categorical flags (like liquidation status) were retained as binary values (0 or 1).

- Ratios were clipped between 0–1 to avoid skew due to outliers.
### Scoring Logic
A custom weighted scoring function was designed. Riskier behavior contributes to a higher score, while safe behavior lowers it. The scoring logic is:
risk_score = (
    w1 * normalized_borrow_supply_ratio +
    w2 * (1 - normalized_repay_ratio) +
    w3 * (1 - normalized_asset_diversity) +
    w4 * (1 - normalized_active_days) +
    w5 * normalized_loan_to_value_ratio
)
The result was scaled to a 0–1000 range:
final_score = int(risk_score * 1000)

## Justification of Risk Indicators

Each selected feature contributes toward understanding a wallet’s reliability:

| **Risk Indicator**             | **Justification**                                       |
|-------------------------------|----------------------------------------------------------|
| High Borrow with Low Repay    | Indicates potential default behavior                    |
| Frequent Liquidations         | Strong negative signal for risk                         |
| Low Borrow/Supply Ratio       | Healthier loan-to-value behavior                        |
| Diverse Asset Use             | Suggests higher engagement and knowledge                |
| Higher Active Days            | More consistent users are typically lower risk          |


## Final Output File Overview
-  A CSV file `wallet_risk_scores.csv` with the following format:

| wallet_id | score |
|-----------|--------|
| 0xfaa0768bde629806739c3a4620656c5d26f44ef2 | 732 |


## Scalability
The current solution is scalable to any number of wallet addresses with proper API integration or access to transaction logs. The model can be continuously improved by integrating real-time price feeds, credit delegations, and flash loan flags.


##  License

This project is submitted as part of **Round 2 Assignment – Wallet Risk Scoring** for the **AI Engineer Internship**.  
All data handling, analysis, and modeling are performed strictly for evaluation and learning purposes. Redistribution or commercial use of this content without explicit permission is prohibited.


###  Author

[**Jasmine Savathallapalli**](https://github.com/JasmineSavathallapalli)  

