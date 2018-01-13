# project-Python-05-AB-testing

__Story:__ We want to understand the results of an A/B test run by an e-commerce website. Our goal is to work through this to help the company understand if they should implement the new page, keep the old page, or **perhaps run the experiment longer** to make their decision.

__Package:__ Pandas, Numpy, Matplotlib, 

>Part I. Data Wrangling
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
 - the proportion of users converted ? --- 0.11965919355605512
```
df.query('converted == 1').user_id.size / df.user_id.size
```
 - The number of times the new_page and treatment don't line up ? --- 3893
```
df.query('landing_page == "old_page" and group == "treatment"').user_id.size 
df.query('landing_page == "new_page" and group == "control"').user_id.nunique() 
```
For the rows where 'treatment' is not aligned with "new_page" or 'control' is not aligned with "old_page", we cannot be sure if this row truly received the new or old page. 

 - we should only use the rows that we can feel confident in the accuracy of the data. so remove them. and define the new df.
```
df_a = df.query('landing_page == "old_page" and group == "control"') 
df_b = df.query('landing_page == "new_page" and group == "treatment"')
df2 = df_a.append(df_b, ignore_index=True)
df2.info()
df2.head()
```
<img src="https://user-images.githubusercontent.com/31917400/34901313-babfca7e-f800-11e7-8d1c-62c8428b63ea.jpg" />
















































