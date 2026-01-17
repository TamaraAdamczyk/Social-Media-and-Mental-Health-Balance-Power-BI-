# Social-Media-and-Mental-Health-Balance-Power-BI-

## Social Media, Screen Time & Mental Health Analysis
Power BI Project - Using Kaggle Dataset  
Dataset: Mental_Health_and_Social_Media_Balance_Dataset (Imran, 2023)

## Project Overview
This project explores how daily screen time, social media usage, and demographic factors relate to stress, happiness, sleep quality, and overall wellbeing.

The analysis was built entirely in Power BI, combining:
- Data cleaning in Power Query
- DAX measures for statistical modelling
- Visual analytics (scatter plots, bar charts, matrices)
- Linear regression and R² calculation
- Error bars and conditional formatting

The goal was to understand behavioural patterns and quantify relationships between screen time and mental health indicators.

## Data Cleaning & Transformation (Power Query)
All preprocessing was done in Power Query using the following steps:

1. Promote First Row to Headers
M
= Table.PromoteHeaders(Source, [PromoteAllScalars = true])
2. Set Correct Data Types
M
= Table.TransformColumnTypes(
    #"Promoted Headers",
    {
        {"User_ID", type text},
        {"Age", Int64.Type},
        {"Gender", type text},
        {"Daily_Screen_Time(hrs)", type number},
        {"Sleep_Quality(1-10)", Int64.Type},
        {"Stress_Level(1-10)", Int64.Type},
        {"Days_Without_Social_Media", Int64.Type},
        {"Exercise_Frequency(week)", Int64.Type},
        {"Social_Media_Platform", type text},
        {"Happiness_Index(1-10)", Int64.Type}
    }
)
3. Remove Duplicate Users
M
= Table.Distinct(#"Changed Type", {"User_ID"})
4. Create Age Category Column
M
= Table.AddColumn(
    #"Removed Duplicates",
    "Age category",
    each
        if [Age] < 17 then "0-17"
        else if [Age] <= 25 then "18-25"
        else if [Age] <= 35 then "26-35"
        else if [Age] <= 45 then "36-45"
        else if [Age] <= 55 then "46-55"
        else if [Age] <= 65 then "56-65"
        else if [Age] <= 75 then "66-75"
        else null
)

## DAX Measures Used
Averages
DAX
Avg Stress =
AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])

Avg Happiness =
AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)])

Avg Sleep =
AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Sleep_Quality(1-10)])

Avg Screen Time =
AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])

Conditional Formatting (Stress)
DAX
Stress Color =
SWITCH(
    TRUE(),
    [Avg Stress] < 4, "#66CC66",
    [Avg Stress] < 7, "#FFD700",
    "#FF6666"
)

Standard Deviation by Screen Time Band
DAX
Happiness StdDev by Band =
VAR SelectedBand =
    SELECTEDVALUE(
        'Mental_Health_and_Social_Media_Balance_Dataset'[Screen Time Band (hrs/day)]
    )
RETURN
    CALCULATE(
        STDEV.P(
            'Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)]
        ),
        'Mental_Health_and_Social_Media_Balance_Dataset'[Screen Time Band (hrs/day)]
            = SelectedBand
    )

Error Bar Bounds
DAX
Happiness Upper Bound =
[Avg Happiness] + [Happiness StdDev by Band]

Happiness Lower Bound =
[Avg Happiness] - [Happiness StdDev by Band]
Linear Regression (Slope, Intercept, Prediction)

Slope
DAX
Slope =
VAR XMean =
    AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])
VAR YMean =
    AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
VAR Numerator =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        (
            'Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)]
                - XMean
        )
            * (
                'Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)]
                    - YMean
            )
    )
VAR Denominator =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER(
            'Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)]
                - XMean,
            2
        )
    )
RETURN
    DIVIDE(Numerator, Denominator)

Intercept
DAX
Intercept =
VAR XMean =
    AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])
VAR YMean =
    AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
RETURN
    YMean - [Slope] * XMean

Predicted Stress
DAX
Predicted Stress =
[Slope] *
    'Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)]
    + [Intercept]
R² Calculation
DAX
R2 =
VAR SS_Total =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER(
            'Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)]
                - AVERAGE(
                    'Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)]
                ),
            2
        )
    )
VAR SS_Residual =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER(
            'Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)]
                - [Predicted Stress],
            2
        )
    )
RETURN
    1 - DIVIDE(SS_Residual, SS_Total)

## Visuals Created
- Scatter plots (Screen Time vs Stress, Screen Time vs Happiness)
- Trendlines with manually calculated regression
- R² displayed as a card
- Bar charts for wellbeing metrics by screen time band
- Error bars using upper/lower bound measures
- Matrix tables with conditional formatting
- Gender‑segmented scatter plots
- Platform usage comparisons

## Screenshots of Dashboard
<img width="1140" height="643" alt="image" src="https://github.com/user-attachments/assets/b92fcd09-2e41-40b1-a23f-db256496dcc5" />
<img width="1147" height="441" alt="image" src="https://github.com/user-attachments/assets/cd762758-39ad-41f9-8622-abde18bfe0ec" />



## Key Takeaways
- Stress increases as daily screen time increases (the positive slope suggests that as average screen time increases, average stress level rises)
- Happiness decreases with higher screen time (the negative slope suggests that higher screen time is associated with lower happiness)
- Both are correlation only, not a causal claim.
- Regression shows a positive linear relationship between screen time and stress.
- R² indicates the model explains a meaningful portion of variance. However, this dataset is synthetic so perfect linearity wouldn't normally be expected with 'real-world' data.
- Sleep quality declines sharply after 6+ hours of screen time.
- Gender differences appear in stress and happiness distributions.

## References
1. Wolfers, L.N. and Utz, S., 2022. Social media use, stress, and coping. Current Opinion in Psychology, 45, p.101305.
2. World Health Organization (WHO) (n.d.) Stress. World Health Organization. Available at: https://www.who.int/news-room/questions-and-answers/item/stress (Accessed: 2 December 2025).
3. Koo, M. and Yang, S.W., 2025. Likert-type scale. Encyclopedia, 5(1), p.18.
4. Prince, A. (n.d.) Mental Health and Social Media Balance Dataset. Kaggle. Available at: https://www.kaggle.com/datasets/prince7489/mental-health-and-social-media-balance-dataset (Accessed 02-12-2025)
