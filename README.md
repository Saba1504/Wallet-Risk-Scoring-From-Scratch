# Wallet-Risk-Scoring-From-Scratch
# Compound Wallet Risk Scoring System (0–1000)

## Objective

This project aims to assess the credit risk of Ethereum wallets interacting with the **Compound V2/V3** protocol. The final output is a **risk score between 0 (high risk) and 1000 (low risk)** based on borrowing, repayment, and liquidation history.

---

##  Tech Stack

- **Python**
- `pandas`, `numpy`
- Etherscan API for on-chain data
- CSV for input and output

---

## Input

A CSV file named `wallets.csv` with one column:

```csv
wallet
0xfaa0768bde629806739c3a4620656c5d26f44ef2
0xb8c77482e45f1f44de1745f52c74426c631bdd52
```

---

## Output

A CSV file named `wallet_risk_scores.csv` with two columns:

```csv
wallet,risk_score
0xfaa0768bde629806739c3a4620656c5d26f44ef2,712
0xb8c77482e45f1f44de1745f52c74426c631bdd52,301
```

---

## How It Works

1. **Load Input**: Read wallet addresses from the input CSV file.
2. **Fetch Transactions**: Use the Etherscan API to get all Compound protocol actions for each wallet.
3. **Feature Engineering**: Calculate key metrics:
   - `total_borrow`
   - `total_repay`
   - `total_liquidation`
   - `repay_borrow_ratio`
   - `liquidation_ratio`
   - `activity_level`
4. **Score Computation**: Compute a credit risk score based on activity patterns.
5. **Export**: Write results to `wallet_risk_scores.csv`.

---

## Methodology Explanation

### **Data Collection Method**
We collected transaction data for each wallet address using the [Etherscan API](https://docs.etherscan.io/) filtered for interactions with the **Compound V2/V3 smart contracts**. The types of actions parsed from the logs include:
- **Borrow**
- **Repay**
- **Liquidate**
- **Supply**
- **Redeem**

Only on-chain transaction logs specific to Compound's lending/borrowing activities were retained to maintain relevancy.

### **Feature Selection Rationale**
To determine risk, the following features were engineered from raw transaction logs:

| Feature                  | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| `total_borrow`           | Count of borrow transactions made by the wallet                            |
| `total_repay`            | Count of repayment transactions made                                        |
| `total_liquidation`      | Number of times the wallet was liquidated                                  |
| `repay_borrow_ratio`     | Ratio of repays to borrows (repayments show positive behavior)             |
| `liquidation_ratio`      | Liquidations divided by total transactions (higher = riskier)              |
| `activity_level`         | Total lending activity including supply/borrow/repay                        |

We chose these features based on real-world lending indicators. For example, a user with a high liquidation ratio or very few repayments is likely riskier than one who repays reliably.

### **Scoring Method**
Each wallet is assigned a **risk score between 0 and 1000** where:
- **Higher score = lower risk**
- **Lower score = higher risk**

#### Formula (simplified version):
```python
score = base_score + (repay_borrow_ratio * repay_weight) - (liquidation_ratio * liquidation_penalty)
```

We used `MinMaxScaler` to normalize features like `repay_borrow_ratio` and `liquidation_ratio` to bring everything onto a [0, 1] scale before applying scoring logic.

#### Example Weights:
- `repay_weight` = +500  
- `liquidation_penalty` = -400  
- `base_score` = 500  

Final score was clipped between **0 and 1000**.

### **Justification of the Risk Indicators**
| Indicator              | Why It Matters                                                              |
|------------------------|------------------------------------------------------------------------------|
| **Liquidations**       | Frequent liquidations signal poor risk management or over-leveraging        |
| **Repayment behavior** | Timely repayments indicate responsible borrowing and better creditworthiness |
| **Borrow frequency**   | Excessive borrowing with no repayment flags potential abuse or default risk  |
| **Supply actions**     | Supplying assets improves trust and signals active contribution to protocol  |
