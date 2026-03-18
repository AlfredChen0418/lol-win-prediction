This analysis examines one main question:

**Do teams with stronger early-game advantages at 15 minutes tend to win more often in professional League of Legends?**

---

## Introduction

This analysis uses Oracle’s Elixir match data from 2025 professional League of Legends games.

Since each game appears in both player rows and team rows, I only use the team rows here. My main question is whether stronger early-game advantage at 15 minutes is closely related to winning.

The original dataset has 120,636 rows. After restricting to complete team-level rows, my cleaned analysis DataFrame has 18,472 rows.

The main columns I use are:

- `side`: blue side or red side
- `result`: win or loss
- `gamelength`: game length
- `golddiffat15`: gold differential at 15 minutes
- `xpdiffat15`: experience differential at 15 minutes
- `csdiffat15`: CS differential at 15 minutes
- `killsat15`: kills at 15 minutes
- `turretplates`: turret plates taken

---

## Data Cleaning and Exploratory Data Analysis

I first kept only rows where `position == 'team'`, so each row represents one team in one game. Then I kept only rows where `datacompleteness == 'complete'`. After that, I selected the columns most related to my question and cleaned `side` and `result` to make them easier to read.

Here is the head of my cleaned DataFrame:

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>gameid</th>
      <th>league</th>
      <th>side</th>
      <th>result</th>
      <th>gamelength</th>
      <th>turretplates</th>
      <th>golddiffat15</th>
      <th>xpdiffat15</th>
      <th>csdiffat15</th>
      <th>killsat15</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>LOLTMNT03_179647</td>
      <td>LFL2</td>
      <td>blue</td>
      <td>loss</td>
      <td>1592</td>
      <td>1.0</td>
      <td>-3837.0</td>
      <td>-469.0</td>
      <td>-16.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <td>LOLTMNT03_179647</td>
      <td>LFL2</td>
      <td>red</td>
      <td>win</td>
      <td>1592</td>
      <td>8.0</td>
      <td>3837.0</td>
      <td>469.0</td>
      <td>16.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <td>LOLTMNT06_96134</td>
      <td>LFL2</td>
      <td>blue</td>
      <td>win</td>
      <td>1922</td>
      <td>8.0</td>
      <td>5069.0</td>
      <td>2014.0</td>
      <td>64.0</td>
      <td>10.0</td>
    </tr>
    <tr>
      <td>LOLTMNT06_96134</td>
      <td>LFL2</td>
      <td>red</td>
      <td>loss</td>
      <td>1922</td>
      <td>2.0</td>
      <td>-5069.0</td>
      <td>-2014.0</td>
      <td>-64.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <td>LOLTMNT06_95160</td>
      <td>LFL2</td>
      <td>blue</td>
      <td>loss</td>
      <td>1782</td>
      <td>7.0</td>
      <td>118.0</td>
      <td>1990.0</td>
      <td>-43.0</td>
      <td>10.0</td>
    </tr>
  </tbody>
</table>

### Univariate Analysis

<iframe
  src="assets/gold15_hist.html"
  style="width: 100%; max-width: 820px; height: 330px; border: none; display: block; margin: 0 auto;"
></iframe>

This histogram shows `golddiffat15`. It is centered around 0 and looks roughly symmetric. This makes sense because each game gives one positive value for one team and one negative value for the other team.

### Bivariate Analysis

<iframe
  src="assets/gold15_box.html"
  style="width: 100%; max-width: 820px; height: 330px; border: none; display: block; margin: 0 auto;"
></iframe>

This box plot compares `golddiffat15` for winning teams and losing teams. We can see that winning teams usually have much higher gold differential at 15 minutes. This means early-game gold lead is strongly related to the final result.

### Grouped Table

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>teams</th>
      <th>avg_gold_diff</th>
      <th>win_rate</th>
    </tr>
    <tr>
      <th>gold_bin</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>(-13867.001, -1552.0]</th>
      <td>4623</td>
      <td>-3191.3</td>
      <td>0.15</td>
    </tr>
    <tr>
      <th>(-1552.0, 0.0]</th>
      <td>4614</td>
      <td>-743.7</td>
      <td>0.39</td>
    </tr>
    <tr>
      <th>(0.0, 1552.0]</th>
      <td>4618</td>
      <td>745.1</td>
      <td>0.61</td>
    </tr>
    <tr>
      <th>(1552.0, 13867.0]</th>
      <td>4617</td>
      <td>3193.4</td>
      <td>0.85</td>
    </tr>
  </tbody>
</table>

This grouped table shows that win rate rises a lot as `golddiffat15` gets larger. Teams in the lowest gold-difference group win much less often than teams in the highest group.

---

## Assessment of Missingness

For the missingness analysis, I use `golddiffat25`, because it has clear missing values in the team-level data.

A simple reason is that some games do not last to 25 minutes, so the gold differential at 25 minutes cannot be recorded. Because this missingness can be explained by another observed column, `gamelength`, I do not think this column is MNAR. It is more likely MAR.

I tested whether the missingness of `golddiffat25` depends on `gamelength` and `side`. I found strong evidence that it depends on `gamelength`, but not enough evidence that it depends on `side`.

<iframe
  src="assets/missingness_perm.html"
  style="width: 100%; max-width: 820px; height: 330px; border: none; display: block; margin: 0 auto;"
></iframe>

The observed statistic is far from the center of the permutation distribution, which supports the idea that the missingness of `golddiffat25` depends on game length.

---

## Hypothesis Testing

In this section, I test whether winning teams usually have higher `golddiffat15` than losing teams.

**Null Hypothesis:** Winning teams and losing teams come from the same distribution of `golddiffat15`. Equivalently, the difference in mean `golddiffat15` between the two groups is 0.

**Alternative Hypothesis:** Winning teams have a higher mean `golddiffat15` than losing teams.

**Test Statistic:** Difference in mean `golddiffat15`

**Significance Level:** 0.05

I use this test statistic because I want to compare the average 15-minute gold differential between winners and losers.

The observed difference in mean `golddiffat15` was about 2690.55, and the empirical p-value was approximately 0. So I rejected the null hypothesis.

This means that, in this dataset, winning teams usually have much higher gold differential at 15 minutes than losing teams. This does not mean a 15-minute gold lead always guarantees a win, but it does show a strong relationship between early-game gold lead and match result.

<iframe
  src="assets/hypothesis_perm.html"
  style="width: 100%; max-width: 820px; height: 330px; border: none; display: block; margin: 0 auto;"
></iframe>

---

## Framing a Prediction Problem

In this analysis, I predict whether a team will win the game using only information available at 15 minutes.

This is a **binary classification** problem. The response variable is `result`, which I coded as win or loss.

At the time of prediction, I only use information that would already be known at 15 minutes, such as:

- `side`
- `golddiffat15`
- `xpdiffat15`
- `csdiffat15`
- `killsat15`
- `turretplates`

I use **accuracy** as my evaluation metric because this is a balanced binary classification problem at the team-row level.

---

## Baseline Model

For the baseline model, I use logistic regression.

The model uses two features:

- `side` (categorical)
- `golddiffat15` (numerical)

I one-hot encoded `side` and used `golddiffat15` directly.

The model achieved:

- training accuracy = 0.727
- test accuracy = 0.736

This is a decent starting point. Even with only one strong early-game feature and one categorical feature, the model already predicts game outcome reasonably well.

---

## Final Model

For the final model, I still use logistic regression, but I add more 15-minute information and two engineered features:

- `lead_count15`: how many of gold, experience, and CS a team is ahead in at 15 minutes
- `pressure15`: kills at 15 minutes plus turret plates taken

I also scaled the numerical features and tuned the regularization parameter `C` with `GridSearchCV`.

The best hyperparameter was:

- `C = 0.1`

The final model achieved:

- training accuracy = 0.736
- test accuracy = 0.739

This is only a small improvement over the baseline model, but it is still better on the same test set. This means the added features help a little and give a fuller picture of early-game advantage than `golddiffat15` alone.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>model</th>
      <th>train_accuracy</th>
      <th>test_accuracy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>baseline</td>
      <td>0.727</td>
      <td>0.736</td>
    </tr>
    <tr>
      <td>final</td>
      <td>0.736</td>
      <td>0.739</td>
    </tr>
  </tbody>
</table>

---

## Fairness Analysis

Finally, I checked whether the final model performs differently on blue-side teams and red-side teams.

I used **accuracy** as the metric.

**Null Hypothesis:** The model is fair with respect to side. Any accuracy difference between blue-side teams and red-side teams is due to chance.

**Alternative Hypothesis:** The model is unfair, and its accuracy is lower for red-side teams than for blue-side teams.

**Test Statistic:** Blue-side accuracy minus red-side accuracy

The observed test statistic was 0.00224, and the empirical p-value was 0.437.

Using a significance level of 0.05, I failed to reject the null hypothesis.

So, I do not have enough evidence to say the model is unfair across sides. The blue-side accuracy is a little higher than the red-side accuracy, but the difference is very small and can easily happen by chance.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>side</th>
      <th>accuracy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>blue</td>
      <td>0.740</td>
    </tr>
    <tr>
      <td>red</td>
      <td>0.738</td>
    </tr>
  </tbody>
</table>

<iframe
  src="assets/fairness_perm.html"
  style="width: 100%; max-width: 820px; height: 330px; border: none; display: block; margin: 0 auto;"
></iframe>

---

## Conclusion

Overall, this project shows that early-game advantage at 15 minutes is strongly related to winning in professional League of Legends.

From the exploratory analysis, teams with larger gold leads at 15 minutes clearly win more often. The hypothesis test supports this pattern, since winning teams have much higher `golddiffat15` than losing teams in this dataset.

For prediction, even a simple baseline model using only `side` and `golddiffat15` already performs reasonably well. After adding more 15-minute features and two engineered features, the final model improves slightly, which suggests that early-game information does help explain match outcome.

Finally, the fairness analysis does not show strong evidence that the final model performs differently across blue-side and red-side teams.

Taken together, the results suggest that 15-minute game state provides meaningful information about who will win, although it does not fully determine the final result.
