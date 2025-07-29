# 📈 Dhan Auto Exit System

This Python script is an automated **MTM-based risk management system** for intraday trading using the [Dhan API](https://dhan.co/). It monitors live positions and automatically closes them based on **maximum loss, profit target**, and a **trailing profit lock mechanism**.

---

## 🚀 Features

- ✅ Auto-closes all positions if **loss exceeds ₹3,000**
- ✅ Auto-books profit if **gain exceeds ₹7,000**
- ✅ Implements a **dynamic trailing profit lock**:
  - Starts locking when MTM reaches ₹1,000
  - Locks ₹500 profit
  - For every ₹500 additional MTM, it updates the locked profit
  - If MTM falls below locked profit, all positions are exited

---

## 🛠️ Requirements

- Python 3.x
- `requests` library
- Dhan account with valid API access

Install dependencies:

```bash
pip install requests
