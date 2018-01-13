# project-Python-05-AB-testing

__Story:__ We want to understand the results of an A/B test run by an e-commerce website. Our goal is to work through this to help the company understand if they should implement the new page, keep the old page, or **perhaps run the experiment longer** to make their decision.

__Package:__ Pandas, Numpy, Matplotlib, 

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

## Part III. Bootstrapping
>Under the null hypothesis, assume they are equal to the converted rate in df2 regardless of the page. (The conversion rate under the Null = `**0.11959708724499628**` as calculated above). 
 - Use a sample size for each page equal to the ones in
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
```
<img src="https://user-images.githubusercontent.com/31917400/34901942-9f445342-f809-11e7-977f-620be13487f4.jpg" />

 - P-Value: What proportion of the 'p_diffs' are greater than the actual difference observed in df2 ? --- 90%
```
(p_diffs > -0.001576).mean()
```















