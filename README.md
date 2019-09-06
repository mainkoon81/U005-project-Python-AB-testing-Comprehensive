# 05-AB-testing

__Story:__ We want to understand the results of an A/B test run by an e-commerce website. Our goal is to work through this to help the company understand if they should implement the new page, keep the old page, or **perhaps run the experiment longer** to make their decision.

__Package:__ Pandas, Numpy, Matplotlib, statsmodels, scipy

## Part I. Data Wrangling
<img src="https://user-images.githubusercontent.com/31917400/34901191-0310e346-f7ff-11e7-8c37-41861c544ca4.jpg" width="500" height="100" />

 - the number of rows ?
```
df.info(),df.converted.unique(),df.landing_page.unique(),df.group.unique()
```
<img src="https://user-images.githubusercontent.com/31917400/34901245-afc6cb1e-f7ff-11e7-8bf9-aa1624cfb0ef.jpg" />

 - the number of unique USERID ? --- 290584
``` 
df.user_id.nunique()
```
 - the proportion of users converted ? --- `0.11965919355605512`
```
df.query('converted == 1').user_id.size / df.user_id.size
```
 - The number of times (new_page & treatment), (old_page & control) don't line up ? --- 1965 + 1928 = 3893
```
df.query('landing_page == "old_page" and group == "treatment"').user_id.size 
df.query('landing_page == "new_page" and group == "control"').user_id.nunique() 
```
For the rows where 'treatment' is not aligned with "new_page" or 'control' is not aligned with "old_page", we cannot be sure if this row truly received the new or old page. 

 - we should only use the rows that we can feel confident with. so remove them. and define the new df.
```
df_a = df.query('landing_page == "old_page" and group == "control"') 
df_b = df.query('landing_page == "new_page" and group == "treatment"')
df2 = df_a.append(df_b, ignore_index=True)
df2.info()
df2.head()
```
<img src="https://user-images.githubusercontent.com/31917400/34901313-babfca7e-f800-11e7-8d1c-62c8428b63ea.jpg" />

 - Double Check all of the correct rows were removed - this should be 0
```
df2[((df2['group'] == 'treatment') == (df2['landing_page'] == 'new_page')) == False].shape[0]
```
 - Ok. Check unique 'user_id' in df2 --- 290584
```
df2.user_id.nunique()
```
 - Is there any 'user_id' repeated ? --- one 
```
df2[df2.user_id.duplicated()]
```
<img src="https://user-images.githubusercontent.com/31917400/34901456-6f63ba7a-f802-11e7-8b99-e153c6834f09.jpg" />

 - the row information for the repeat user_id ?
```
df2.iloc[146678]
```
 - Remove one of the rows with a duplicate user_id - drop the desired value using the index directly.
```
df2 = df2.drop(146678)
df2 = df2.reset_index(drop=True)
df2.iloc[146678]
```
## Part II. Statistics
 - The probability of an individual converting regardless of the page they receive --- `0.11959708724499628`
```
df2.converted.mean()
```
 - Given that an individual was in the control group, the probability they converted ? or Given that an individual was in the treatment group, the probability they converted? --- 0.1203863045004612, 0.11880806551510564
```
df2.query('group == "control"').converted.mean(), df2.query('group == "treatment"').converted.mean()
```
 - The probability that an individual received the new page ? --- 0.5000619442226688
```
df2.query('landing_page == "new_page"').user_id.size / df2.user_id.size
```
The probability of conversion in general is 11.9%. In **"control Grp", it's 12%** and **in "treatment Grp", it's 11.8%.** Thus we don't have enough clue to say the new treatment page leads to more conversions. Here, the difference of their Conversion-probabilities is `0.118808 - 0.120386 = -0.001576` This is the **observed** difference of conversion rate!

>Notice that because of the time stamp associated with each event, you could technically run a hypothesis test continuously as each observation was observed. However, then the hard question is do you stop as soon as one page is considered significantly better than another or does it need to happen consistently for a certain amount of time? How long do you run to render a decision that neither page is better than another? These questions are the difficult parts associated with A/B tests in general.

>For now, consider you need to make the decision just based on all the data provided. If you want to assume that the old page is better unless the new page proves to be definitely better at a Type I error rate of 5%, our null and alternative hypotheses be...
<img src="https://user-images.githubusercontent.com/31917400/34901652-32d54eae-f805-11e7-9f43-32dc5d3723b9.jpg" />

## Part III. Bootstrapping (Additional psuedo Experiments ?)
>Under the null hypothesis, assume they are equal to the converted rate in df2 regardless of the page. (The conversion rate under the Null = `**0.11959708724499628**` as calculated above). 
 - Use a sample size for each page equal to the ones in df2.
 - Perform the sampling distribution for the difference in converted between the two pages over 10,000 iterations of calculating an estimate from the null. 
 - Here we are looking at the Null where there is no difference in conversion based on the page, which means the conversions for each page are the same.

 - 1st Bootstrapping - How many sample ? --- new:145310, old:145274
```
df2.query('group == "treatment"').user_id.size
df2.query('group == "control"').user_id.size
```
 - 1st Bootstrapping - Simulate transactions with a convert rate of both **under the null**. What's the difference of their conversion rate? --- It's -0.0003 which can change all the time. Is this difference significant ?  We will iterate this process multiple times - 10,000! 
```
new_page_converted = np.random.choice([0,1], size=145310, p=[1-0.1196, 0.1196])
old_page_converted = np.random.choice([0,1], size=145274, p=[1-0.1196, 0.1196])
new_page_converted.mean() - old_page_converted.mean()
```
 - Let's create a list consists of 10,000 different conversion rates yielded from multiple bootstrapping! then plot to compare this 'p_diffs' distribution with the observed difference!
```
p_diffs = []

for i in range(10000):
    control_df = np.random.choice([0,1], size=145274, p=[1-0.1196, 0.1196])
    treat_df = np.random.choice([0,1], size=145310, p=[1-0.1196, 0.1196])
    p_old = control_df.mean()
    p_new = treat_df.mean()
    p_diffs.append(p_new - p_old)

p_diffs = np.array(p_diffs) 

plt.hist(p_diffs)
plt.axvline(x=-0.001576, color='r')
plt.title('Distribution of p_diffs')
```
<img src="https://user-images.githubusercontent.com/31917400/34901989-70199144-f80a-11e7-8702-0e6b63c6f264.jpg" />

 - P-Value: What proportion of the 'p_diffs' are greater than the actual difference observed in df2 ? --- 90%
```
(p_diffs > -0.001576).mean()
```
>We cannot reject H0 because of P-Value(0.9 > 0.05) which means the difference is very insignificant with the probability of 0.9 and the new page is never better than the old.** In other words, the actual difference observed in df2 is -0.001576, which should be considered 'insignificant.' 

We could also use a built-in to achieve similar results. Though using the built-in might be easier to code, the above portions are a walkthrough of the ideas that are critical to correctly thinking about statistical significance. 

 - Let's prepare 'z-test'! SIZE ? --- (17489, 17264, 145274, 145310)
```
import statsmodels.api as sm

convert_old = df2.query('group == "control" and converted == 1').user_id.size
convert_new = df2.query('group == "treatment" and converted == 1').user_id.size
n_old = df2.query('group == "control"').user_id.size
n_new = df2.query('group == "treatment"').user_id.size

convert_old, convert_new, n_old, n_new
```
 - Find **Z-score** (Z-TEST) with `statsmodels`--- (1.3109241984234394, 0.18988337448195103) but here, P_value is for two_tailed test. We need one tailed test. 
```
z_score, p_value = sm.stats.proportions_ztest([convert_old, convert_new], [n_old, n_new])
z_score, p_value 
```
<img src="https://user-images.githubusercontent.com/31917400/34908094-64b9ada2-f882-11e7-9c15-273db7658538.jpg" />

 - Find **P-Value** with `scipy` --- (0.90505831275902449). Before this test began, we would have picked a significance level. Let's just say it's 95%. According to the Hypothesis setting, it's an one-tail test so a z-score past 1.64 will be significant. (if two-tail, then -1.96 to 1.96)
```
from scipy.stats import norm
norm.cdf(1.3109241984234394)  # 0.90505831275902449 # How significant our z-score is? Show the area until our z-score value.

norm.ppf(1-(0.05))  # 1.6448536269514722 (one-tail critical value at 95% confidence)
norm.ppf(1-(0.05/2)) # 1.959963984540054 (two-tail critical value at 95% confidence)
```
P-value is 0.905 so we cannot reject the null hypothesis that **the difference between the two proportions is no different from zero.** The result from this z-test matches exactly our previous simulation and the P-value here as well.

## Part IV. Regression Approach
>The result we acheived in the previous A/B test can also be acheived by performing regression ? Since each row is either a conversion or no conversion, we perform a Logistic regression with categorical predictors. 
 - Use `statsmodels` to fit the regression model to see if there is a significant difference in conversion based on which page a customer receives. We first need to create a column for the **intercept**, and create a **dummy variable** column for which page each user received. Add an intercept column, as well as an __ab_page__ column, which is 1 when an individual receives the treatment and 0 if control.
```
df2['intercept'] = 1

df2[['new_page','old_page']] = pd.get_dummies(df2['landing_page'])
df2['ab_page'] = pd.get_dummies(df2['group'])['treatment']
```
<img src="https://user-images.githubusercontent.com/31917400/34908277-e02a2370-f884-11e7-8702-36f10178b199.jpg" />

 - Instantiate the model, and fit the model using the two columns you created previously to predict whether or not an individual converts.
```
import statsmodels.api as sm
from scipy import stats
stats.chisqprob = lambda chisq, df: stats.chi2.sf(chisq, df)

log_model = sm.Logit(df2['converted'], df2[['intercept', 'ab_page']])
result = log_model.fit()
result.summary()
```
<img src="https://user-images.githubusercontent.com/31917400/34908322-71b57042-f885-11e7-8f49-29025e245777.jpg" />

P-value is 0.19 which means 'ab_page' is not that significant in predicting whether or not the individual converts. H0 in this model is that the **'ab_page' is totally insignificant in predicting the responses** and we cannot reject H0.  

 - Now along with testing if the conversion rate changes for different pages, also add an effect based on which country a user lives. You will need to read in the 'countries.csv' dataset and merge together your datasets on the approporiate rows. Does it appear that country had an impact on conversion? Don't forget to create dummy variables for these country columns. You will need **two columns for the three dummy variables.**
```
countries_df = pd.read_csv('C:/Users/Minkun/Desktop/classes_1/NanoDeg/1.Data_AN/L4/project 4/AnalyzeABTestResults 2/countries.csv')
df_new = countries_df.set_index('user_id').join(df2.set_index('user_id'), how='inner')
df_new.head()
```
<img src="https://user-images.githubusercontent.com/31917400/34908404-344b6732-f887-11e7-9e4b-4c5d34b78c1a.jpg" />

```
df_new.country.unique() 
df_new[['US','UK','CA']] = pd.get_dummies(df_new['country'])
df_new.head()
```
<img src="https://user-images.githubusercontent.com/31917400/34908466-d707056c-f887-11e7-8332-834c2679c943.jpg" />

```
log_model_2 = sm.Logit(df_new['converted'], df_new[['intercept','ab_page','US','UK']])
result = log_model_2.fit()
result.summary()
```
<img src="https://user-images.githubusercontent.com/31917400/34908472-f19e1b90-f887-11e7-8350-83c7c35dc424.jpg" />

Though you have now looked at the individual factors of country and page on conversion, we would now like to look at an interaction between page and country to see if there significant effects on conversion. 

