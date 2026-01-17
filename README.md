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
- Linear regression
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
This has been created and copied from DAX studio:

---- MODEL MEASURES BEGIN ----

MEASURE Mental_Health_and_Social_Media_Balance_Dataset[% High Happiness] = 
VAR Total = [Users]
VAR High = CALCULATE([Users], 'Mental_Health_and_Social_Media_Balance_Dataset'[Happiness Band] = "High (8–10)")
RETURN DIVIDE(High, Total)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[% High Stress] = 
VAR Total = [Users]
VAR High = CALCULATE([Users], 'Mental_Health_and_Social_Media_Balance_Dataset'[Stress Band] = "High")
RETURN DIVIDE(High, Total)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Days Offline] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Days_Without_Social_Media])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Exercise / Week] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Exercise_Frequency(week)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Happiness] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Screen Time] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Sleep Quality] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Sleep_Quality(1-10)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Avg Stress] = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Corr Offline–Happiness] = 
VAR T = ADDCOLUMNS('Mental_Health_and_Social_Media_Balance_Dataset', "X", 'Mental_Health_and_Social_Media_Balance_Dataset'[Days_Without_Social_Media], "Y", 'Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)])
VAR AvgX = AVERAGEX(T, [X])
VAR AvgY = AVERAGEX(T, [Y])
VAR Num = SUMX(T, ([X] - AvgX) * ([Y] - AvgY))
VAR Den = SQRT( SUMX(T, ([X] - AvgX)^2) * SUMX(T, ([Y] - AvgY)^2) )
RETURN DIVIDE(Num, Den)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Corr ScreenTime–Happiness] = 
VAR T = 
  ADDCOLUMNS(
    'Mental_Health_and_Social_Media_Balance_Dataset',
    "X", 'Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)],
    "Y", 'Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)]
  )
VAR AvgX = AVERAGEX(T, [X])
VAR AvgY = AVERAGEX(T, [Y])
VAR Num = SUMX(T, ([X] - AvgX) * ([Y] - AvgY))
VAR Den = SQRT( SUMX(T, ([X] - AvgX)^2) * SUMX(T, ([Y] - AvgY)^2) )
RETURN DIVIDE(Num, Den)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Corr Sleep–Happiness] = 
VAR T = ADDCOLUMNS('Mental_Health_and_Social_Media_Balance_Dataset', "X", 'Mental_Health_and_Social_Media_Balance_Dataset'[Sleep_Quality(1-10)], "Y", 'Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)])
VAR AvgX = AVERAGEX(T, [X])
VAR AvgY = AVERAGEX(T, [Y])
VAR Num = SUMX(T, ([X] - AvgX) * ([Y] - AvgY))
VAR Den = SQRT( SUMX(T, ([X] - AvgX)^2) * SUMX(T, ([Y] - AvgY)^2) )
RETURN DIVIDE(Num, Den)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Happiness Lower Bound] = [Avg Happiness] - [Happiness StdDev by Band]
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Happiness StdDev by Band] = 
VAR SelectedBand = SELECTEDVALUE('Mental_Health_and_Social_Media_Balance_Dataset'[Screen Time Band (hrs/day)])
RETURN
CALCULATE(
    STDEV.P('Mental_Health_and_Social_Media_Balance_Dataset'[Happiness_Index(1-10)]),
    'Mental_Health_and_Social_Media_Balance_Dataset'[Screen Time Band (hrs/day)] = SelectedBand
)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Happiness Upper Bound] = [Avg Happiness] + [Happiness StdDev by Band]
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Intercept Stress] = 
VAR XMean = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])
VAR YMean = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
RETURN
YMean - [Slope Stress] * XMean
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Predicted Stress] = 
[Slope Stress] * 'Mental_Health_and_Social_Media_Balance_Dataset'[Avg Screen Time] + [Intercept Stress]
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[R2] = 
VAR SS_Total =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)] - AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)]), 2)
    )
VAR SS_Residual =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)] - [Predicted Stress], 2)
    )
RETURN
1 - DIVIDE(SS_Residual, SS_Total)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Slope Stress] = 
VAR XMean = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)])
VAR YMean = AVERAGE('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
VAR Numerator =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        ('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)] - XMean) *
        ('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)] - YMean)
    )
VAR Denominator =
    SUMX(
        'Mental_Health_and_Social_Media_Balance_Dataset',
        POWER('Mental_Health_and_Social_Media_Balance_Dataset'[Daily_Screen_Time(hrs)] - XMean, 2)
    )
RETURN
DIVIDE(Numerator, Denominator)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Stress Color] = 
SWITCH (
    TRUE(),
    [Avg Stress] < 4, "Green",     // less than 4
    [Avg Stress] < 7, "Yellow",    // 4–6.9
    "Red"                          // 7 and above
)
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Stress Lower Bound] = [Avg Stress] - [Stress StdDev]
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Stress StdDev] = 
STDEV.P('Mental_Health_and_Social_Media_Balance_Dataset'[Stress_Level(1-10)])
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Stress Upper Bound] = [Avg Stress] + [Stress StdDev]
MEASURE Mental_Health_and_Social_Media_Balance_Dataset[Users] = COUNTROWS('Mental_Health_and_Social_Media_Balance_Dataset')

---- MODEL MEASURES END ----


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
<img width="1146" height="641" alt="image" src="https://github.com/user-attachments/assets/4c797e0d-1d43-439b-89cb-2fe4efe69fe4" />

<img width="1147" height="441" alt="image" src="https://github.com/user-attachments/assets/cd762758-39ad-41f9-8622-abde18bfe0ec" />



## Key Takeaways
- Stress increases as daily screen time increases (the positive slope suggests that as average screen time increases, average stress level rises)
- Happiness decreases with higher screen time (the negative slope suggests that higher screen time is associated with lower happiness)
- Both are correlation only, not a causal claim.
- Sleep quality declines sharply after 6+ hours of screen time.
- Gender differences appear in stress and happiness distributions.

## References
1. Wolfers, L.N. and Utz, S., 2022. Social media use, stress, and coping. Current Opinion in Psychology, 45, p.101305.
2. World Health Organization (WHO) (n.d.) Stress. World Health Organization. Available at: https://www.who.int/news-room/questions-and-answers/item/stress (Accessed: 2 December 2025).
3. Koo, M. and Yang, S.W., 2025. Likert-type scale. Encyclopedia, 5(1), p.18.
4. Prince, A. (n.d.) Mental Health and Social Media Balance Dataset. Kaggle. Available at: https://www.kaggle.com/datasets/prince7489/mental-health-and-social-media-balance-dataset (Accessed 02-12-2025)
