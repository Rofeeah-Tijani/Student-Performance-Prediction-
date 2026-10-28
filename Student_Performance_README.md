# Predicting Student Performance Using Machine Learning

A classification project that tests whether machine learning can flag students who are likely to fail their final exam early enough for teachers to step in. I built and compared two models, logistic regression and a decision tree, and tested each one with and without earlier grades to see how well an early-warning system could work before any grades exist.

## Problem

Schools often learn that a student is struggling only after the final exam, when it is too late to help. This project asks two questions:

1. Can a model identify students at risk of failing early?
2. Which factors, such as study time, past class failures, absences and family background, are most closely linked to failing?

## Dataset

- **Source:** [UCI Machine Learning Repository, Student Performance dataset](https://archive.ics.uci.edu/dataset/320/student+performance) (Cortez and Silva, 2008), collected from two secondary schools in Portugal.
- **File used:** the Portuguese language file with 649 students. I chose it over the Mathematics file (395 students) because it has more records.
- **Variables:** 33 original attributes, including age, sex, home address type, parents' education and jobs, weekly study time, past class failures, absences, internet access and extra school support, plus three grades (G1, G2 and G3, each scored 0 to 20). G3 is the final grade.
- **Added column:** Final Grade (Pass or Fail), created from G3, which brings the file to 34 columns.

## What I did

1. **Created the target.** I turned the final grade into two classes: Pass (G3 of 10 or above) and Fail (below 10). I built the label in Excel with an `IFS` formula, then rebuilt it in Python and checked that both matched in all 649 rows. Fail is coded as 1 and Pass as 0.
2. **Checked and cleaned the data.** There were no missing values and no duplicate rows. Text columns were converted to numbers with one-hot encoding, and numeric columns were standardised.
3. **Explored the data** with matplotlib and seaborn: class balance, grade distribution, study time, absences, past failures, fail rates by group and correlations.
4. **Prevented data leakage.** I removed Final Grade and G3 from the inputs before encoding and training, and added a check that stops the code if either one appears among the inputs.
5. **Split the data** into training, validation and test sets using stratified sampling, so each set keeps about 15% failing students.

| Set | Purpose | Students |
|---|---|---|
| Training (60%) | The model learns patterns from it | 389 |
| Validation (20%) | Used to choose settings such as tree depth | 130 |
| Test (20%) | Used once at the end for the final scores | 130 |

6. **Built two models.** Logistic regression gives the probability of failing and a weight for each factor. The decision tree asks a series of yes/no questions to reach a prediction, which makes it easy to explain and draw. Both gave extra weight to the Fail class, because only 15% of students fail. Their settings (regularisation strength and tree depth) were chosen on the validation set.
7. **Trained each model in two scenarios.**
   - **Scenario A, with G1 and G2:** all variables.
   - **Scenario B, without G1 and G2 (early warning):** only background and behaviour variables, which is what a school has before any grades exist.
8. **Evaluated both models** on the unseen test set using accuracy, precision, recall, F1-score, ROC-AUC and confusion matrices.

## Exploratory findings

**Class balance.** 549 students passed (84.6%) and 100 failed (15.4%). A model that predicted "everyone passes" would be 85% accurate and catch no failing students, so I used recall and precision for the Fail class as the main measures instead of accuracy alone.

**Grades.** The average final grade is 11.9 out of 20. Fifteen students scored exactly 0, which may mean withdrawal or no-shows.

**Study time.** More study time goes with a lower fail rate and higher grades.

| Weekly study time | Students | Fail rate | Average G3 |
|---|---|---|---|
| Under 2 hours | 212 | 23.6% | 10.8 |
| 2 to 5 hours | 305 | 13.4% | 12.1 |
| 5 to 10 hours | 97 | 7.2% | 13.2 |
| Over 10 hours | 35 | 5.7% | 13.1 |

**Past class failures.** This is the strongest background signal.

| Past class failures | Students | Fail rate |
|---|---|---|
| 0 | 549 | 9.3% |
| 1 | 70 | 45.7% |
| 2 | 16 | 50.0% |
| 3 | 14 | 64.3% |

The average final grade drops from 12.5 with no past failures to about 8.6 after even one.

**Other groups with higher fail rates**

| Group | Fail rate | Comparison |
|---|---|---|
| Does not want higher education | 47.8% | 11.6% for those who do |
| School MS | 30.1% | 7.6% for school GP |
| Rural home address | 22.3% | 12.4% for urban |
| No internet at home | 21.2% | 13.7% with internet |
| Male | 18.8% | 13.1% for female |

**Absences.** The link to the final grade is weak (correlation of -0.09), so absences alone are a poor warning sign in this dataset.

**G1 and G2.** The earlier grades are very closely related to the final grade (correlation of 0.83 for G1 and 0.92 for G2), which is why I tested the models with and without them.

## Model results (test set: 98 students, 15 failing)

| Scenario | Model | Accuracy | Precision (Fail) | Recall (Fail) | F1 (Fail) | ROC-AUC |
|---|---|---|---|---|---|---|
| A: with G1 and G2 | Logistic Regression | 0.878 | 0.571 | 0.800 | 0.667 | 0.909 |
| A: with G1 and G2 | Decision Tree | 0.867 | 0.542 | 0.867 | 0.667 | 0.927 |
| B: early warning | Logistic Regression | 0.776 | 0.333 | 0.467 | 0.389 | 0.757 |
| B: early warning | Decision Tree | 0.643 | 0.250 | 0.667 | 0.364 | 0.709 |

**Confusion matrices**

| Scenario and model | Failing students caught | Failing students missed | False alarms | Correct Pass |
|---|---|---|---|---|
| A, Logistic Regression | 12 | 3 | 9 | 74 |
| A, Decision Tree | 13 | 2 | 11 | 72 |
| B, Logistic Regression | 7 | 8 | 14 | 69 |
| B, Decision Tree | 10 | 5 | 30 | 53 |

## What the results show

- **Earlier grades make prediction much easier.** With G1 and G2, both models reach about 87% accuracy and catch 12 to 13 of the 15 failing students. Without them, accuracy falls to between 64% and 78%.
- **Early warning is possible but imperfect.** Without grades, the decision tree still caught 10 of 15 failing students (recall of 67%) and logistic regression caught 7 (47%). Background and behaviour data carry real signal, but they are not enough to predict failure reliably on their own.
- **Accuracy and recall tell different stories.** In the early-warning scenario, logistic regression has the better accuracy but misses more failing students. The decision tree catches more of them but raises many more false alarms (30 against 14).
- **Neither model wins on every measure.** Logistic regression has higher accuracy and precision in both scenarios, and a better AUC in the early-warning scenario (0.757 against 0.709). The decision tree has higher recall in both scenarios. I judge logistic regression to be the more balanced model, while the decision tree is the better choice when missing an at-risk student is the main worry, and it is easier to explain to teachers.
- **What drives failure.** In the early-warning logistic regression, attending school MS and having past class failures raise the risk of failing. Wanting higher education, more study time, an urban address and a more educated father lower it. These match the exploratory findings.

## Practical use

A school could run the early-warning model at the start of the year, before any grades exist, to build a watch list of students who need attention. The strongest signals are past class failures, low study time and low interest in higher education. Support such as tutoring, study-skills sessions or conversations with families could then go to those students while there is still time to change the result.

## Limitations

- **Small test set.** Only 15 students in the test set fail, so each student changes recall by about 7 percentage points. Treat the numbers as indicative.
- **Class imbalance.** With only 15% failing, the minority class is harder for the models to learn.
- **Two schools in one country.** Results may not carry over to other education systems.
- **Correlation is not causation.** The models show which factors go with failure, not that they cause it. For example, school MS may reflect differences in resources or intake rather than teaching quality.
- **Sensitive factors.** Variables such as sex and home address should be used with care. A model like this should guide support, not label or penalise students.

## Lesson learned: data leakage

My first attempts scored 100% accuracy for both models. The cause was data leakage: after encoding the whole table, a numeric copy of Final Grade, a column I had not meant to create, was still among the inputs, so the model was reading the answer. I fixed it by removing the answer columns before encoding and adding an automatic check that stops the code if they appear. A perfect score is a warning sign, not a success.

## Next steps

- Test more models, such as random forests.
- Use cross-validation for steadier estimates.
- Repeat the approach on the Mathematics file and on data from other schools.

## Tools

Python (pandas, matplotlib, seaborn, scikit-learn), Microsoft Excel

## Author

Rofee'ah Tijani
GitHub: [Rofeeah-Tijani](https://github.com/Rofeeah-Tijani)
