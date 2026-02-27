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