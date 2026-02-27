# Development notes 

## First model and inital exploration
- Replicated benchmark LigthGBM classifier model
- Dropped SIGNED_VOLUME_1 (73% missing values)
- No feature engineering yet
- 5-fold cross-validation

#### Data Understanding
- **Training data:** 527,073 rows × 44 columns
- **Test data:** 31,870 rows × 44 columns
- **Target:** Continuous returns, converted to binary (1 = positive, 0 = negative)
- **Class balance:** 50.72% positive, 49.28% negative (well balanced!)
- **Structure:** Each row = one allocation on one date with 20 days of history

#### Key Discoveries

**Missing Values Problem:**
- SIGNED_VOLUME_1: 73.5% missing (387,506 rows!)
- Other SIGNED_VOLUME columns: <2% missing
- Pattern: Most recent day (day 1) systematically unavailable for most allocations
- **Decision:** Dropped SIGNED_VOLUME_1 entirely rather than filling 73% with fake zeros

**Other Missing Values:**
- Filled remaining missing values (<2%) with 0
- Final clean dataset: 41 features, 0 missing values

#### Models Trained

**Logistic Regression (Baseline):**
- Single train/val split (80/20)
- Accuracy: 50.97%
- Beat benchmark with simple model

**LightGBM:**
- 5-fold Cross-Validation
- Hyperparameters: n_estimators=100, learning_rate=0.05
- **CV Accuracy: 53.59%** ± 0.17% std dev
- **Leaderboard: 50.90%** (rank#210 (on 15.02.2026), dropped to rank#453 (on 27.02.2026))
- Benchmark: 50.79%

#### Key Learnings

**Overfitting:**
- CV score (53.59%) vs Leaderboard (50.90%) = 2.69% gap
- Classic overfitting to validation set
- Lesson: CV is optimistic; real test set is harder

**Why dates being shuffled matters:**
- Can use random K-fold validation (not time-based splits)
- But still need to be careful about generalisation

**Understanding the challenge:**
- Predicting direction of small returns is hard
- Benchmark at 50.79% barely beats random (50%)
- Even 51-52% would be valuable in real trading

#### What Worked
- Dropping SIGNED_VOLUME_1 (cleaner than filling 73% with zeros)
- Simple data cleaning approach
- K-fold validation for robust estimates
- LightGBM handles remaining missing values naturally

#### What Didn't Work
- No feature engineering = minimal improvement over benchmark
- Default hyperparameters might not be optimal
- CV score was misleadingly optimistic

#### Technical Details
- Features used: RET_1 to RET_20, SIGNED_VOLUME_2 to SIGNED_VOLUME_20, MEDIAN_DAILY_TURNOVER, GROUP
- Total: 41 features
- No feature engineering yet
- No hyperparameter tuning

### Questions to Explore
- Why is SIGNED_VOLUME_1 73% missing? Data collection issue?
- What patterns do the 4 different GROUPs represent?
- Are certain allocations more predictable than others?
- Does temporal information matter even though dates are shuffled?

## Second (V2) and V2.5 optimised but worst performing models

- Feature engineering and hyperparameter tunig
- Created 18 new engineered features
- Tested different hyperparameter configurations

#### Feature Engineering
**New features created:**
- Moving averages (RET_AVG_3, 5, 10, 15, 20)
- Volatility (RET_STD_5, 10, 20)
- Momentum (MOMENTUM_SHORT, MOMENTUM_LONG)
- Cross-allocation comparisons (RELATIVE_PERF_5, 10, 20)
- Trend direction (POSITIVE_DAYS_5, 10)

**Results:**
- Baseline CV: 53.65%
- With features CV: 54.61%
- Improvement: +0.96%

#### Hyperparameter Tuning
Tested 11 different configurations:
- Best CV: 55.92% (n_estimators=200, lr=0.07)
- Original: 54.61% (n_estimators=100, lr=0.05)

#### Leaderboard Submissions
1. Baseline (no features): 0.5090 (rank ~450)
2. With features: 0.5118 (rank#391) **BEST**
3. Optimised hyperparams: 0.5047 (rank would be ~500)  **OVERFITTING**

#### Key Learnings

**The Overfitting Problem:**
- CV scores kept improving (53.65% → 54.61% → 55.92%)
- Leaderboard scores got WORSE (50.90% → 51.18% → 50.47%)
- Gap between CV and leaderboard grew (2.7% → 3.4% → 5.5%)

**Critical lesson:** Optimising for CV score can hurt real performance

**What went wrong:**
- More trees + higher learning rate = memorised training patterns
- Features that work on training data don't generalise to test
- Need to be more conservative with complexity

#### What Worked
✅ Feature engineering (modest improvement: +0.28% on leaderboard)
✅ Keeping it simple (baseline features > over-optimised)

#### What Didn't Work
- Aggressive hyperparameter tuning (overfitted severely)
- Trusting CV scores blindly
- Adding complexity without validation on hold-out test

### Questions to Explore
- Why is CV-to-leaderboard gap so large?
- Are we validating on the wrong distribution?
- Would simpler features work better?
- Should we use fewer features, not more?

### Important Reminder
**Best model so far: Feature engineering with default hyperparameters**
- Don't always chase higher CV scores
- Simplicity often beats complexity
- Real test performance > validation performance