# QRT-Data-Challenge-2026

Predicting whether to short or follow asset allocation potfolios based on historical performance. 

## Challenge
- **Goal:** Predict the sign (positive/negative) of next-day returns for asset allocations
- **Metric:** Accuracy
- **Data:** X_train(527073, 44): each row is a unique allocation on a specific day and the columns are the features such as TS/ALLOCATION/20 returns/20 volumes/turnover/GROUP, X_test(31870, 44): rows to predict and same columns as X_train, y_train(527073,1): same rows as X_train with 1 column of the target - next day's return

## Setup
python -m venv venv
source venv/bin/activate #on mac
pip install pandas numpy matplotlib seaborn scikit-learn lightgbm jupyter