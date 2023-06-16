# Outage Prediction

## Framing the Problem

In this project, I will be using regression techniques in an effort to predict the number of customers affected by a particular power outage. 

I think this variable is the best and most interesting way to evaluate the severity of a power outage. While the duration of a power outage is likely to be dependent mainly on technical factors on the provider's side of things, the number of customers affected is also based on the demographics and conditions of the area where the outage happens. Thus, I expect a different set of feature vectors to be relevant for this question, taking more sociological and economic factors into account.

I will be evaluating my model's performance using the coefficient of determination, $R^{2}$. $R^{2}$ is the statistic of choice for regression models, as its an accurate and intuitive identifier of how well a model fits a given prediction problem.

## Baseline Model

For my baseline model, I wanted to choose one quantitative and one categorical variable to try to predict the number of consumers affected by an outage.

1. **Quantitative:** The variable I chose as my quantitative variable was the duration of the outage. I thought this variable was interesting to think about in terms of the overall "intensity" of an outage. If more intense outages both tend to last longer *and* be spread across a larger area, then it makes sense that there would be some correlation between the outage duration and the number of customers affected. I standardized this variable as is best practice; it shouldn't affect my $R^{2}$ but it would help to interpret my coefficient should I need to in the case I add more variables.

2. **Categorical:** The variable I chose as my categorical variable was the type of event that caused the outage (called CAUSE.CATEGORY in the original dataset). If certain causes for outages tend to result in more intense outages, then there is likely a relationship between certain categories and the number of customers affected. I encoded this categorical variable using sklearn's OneHotEncoder, dropping a column in order to prevent multicolinearity. One-Hot encoding is the best choice as this variable is not ordinal.

Instead of calculating the $R^{2}$ for my base model once, I did it many times and plotted a histogram of the error to get a sense for its distribution.

<iframe src="assets/base_coef.html" width=800 height=600 frameBorder=0></iframe>

On average, the $R^{2}$ was around 0.08. I don't think this value is very good, but I hope to improve it in the final model.

## Final Model

For my final model, I added just two more feature vectors (I tried doing more at first, but ran into overfitting problems). The two I went with were TOTAL.SALES and TOTAL.REALGSP. These feature vectors find correlation with the response variable not through the severity of the outage itself, but with how the state's electricity departments would be able to deal with it.

I thought that states with more total earnings, as well as states with more lucrative electricity industries, would probably be more well-funded and therefore more well equipped to combat power outages. Thus, there would be correlation with the number of customers affected not because of the causes of power outages, but because of what was preventing them. 

I kept the pre-processing for the cause event the same. However, each of the quantitative variables was assigned a separate PolynomialFeatures() preprocesser. That way, if one of the variables worked best at degree 3, but another worked best at degree 5, they would each have their own value instead of being stuck with the same one. I then used GridSearchCV to get the values for these parameters.

I ended up with $R^{2}$ values that averaged around 0.135. While I wish this number was higher, I was glad that It was significantly larger than my baseline model. Unfortunately, my model also isn't very consistent. I tried to improve this by playing around with the value of my train-test-split and the cv parameter in GridSearchCV, and was able to make some improvements. However, the variance in the model's $R^{2}$ is still higher that I would like.

## Fairness Analysis

My two groups for my fairness analysis were states that were "more urban", meaning they had a higher proportion of their population living in urban environments, and states that were less urban. I calculated this by considering states in the 60th percentile in terms of the proportion of their population living in urban environments as urban and the rest as less urban. I used sklearn's binarizer to accomplish this.

My evaluation metric for the test is the absolute difference in coefficients of correlation. This results in a two-sided permutation test, where large skew in either direction will be noticed.

I chose this metric because I'm not precisely sure how exactly the urbanness will affect the level of prediction. I didn't just choose it at random - I thought it would be interesting to see if the two feature vectors I added for my final model, which were both based on a state's wealth and economic capacity, would cause bias with respect to urban states. However, I could see this bias going both ways.

$H_{0}$: Our model is fair. Its coefficient of determination for power outages in more urban states is the same as its coefficient of determination for less urban ones.
$H_{1}$: Our model is biased. Its coefficient of determination for power outages in more urban states is not the same as that of less urban ones.

My significance level was $\alpha = 0.05$. I ended up with a p-value of 0.735, so I most definitely failed to reject the null hypothesis. You can see the test statistic with respect to the permuted data in the visualization below. 

<iframe src="assets/fairness.html" width=800 height=600 frameBorder=0></iframe>