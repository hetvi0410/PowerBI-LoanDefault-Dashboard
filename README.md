# Loan Default & Financial Risk Analysis

### Dashboard Link: https://app.powerbi.com/groups/b192cb89-7173-4c5a-afbc-889cc1856102/reports/24e75446-8575-4d67-887f-c1e8fc3330e3/bcb9ed15231d6684065a?experience=power-bi

## Overview

This is an end-to-end Power BI project that analyzes loan default patterns and borrower risk profiles. The dashboard helps financial institutions understand which customer segments are more likely to default and provides actionable insights for improving lending decisions and portfolio management.

## Problem Statement

This dashboard helps the business understand loan default trends and borrower risk more clearly. It shows how factors such as loan purpose, employment type, age group, credit score bins, marital status, and education level affect loan amount, default rate, and repayment behavior.

Through different KPIs and visuals, the report helps identify which borrower segments are more likely to default and where financial risk is concentrated. It also highlights trends such as average income by employment type, average loan amount by age group, median loan amount by credit score bins, year-over-year default changes, and year-to-date loan amount patterns.

Since the project focuses on default risk analysis, it helps decision-makers improve loan approval strategies, reduce exposure to risky borrowers, and manage the loan portfolio more effectively.

## Objectives

- Analyze loan default behavior across different borrower segments
- Identify high-risk groups based on financial and demographic factors
- Track loan amount and default trends over time
- Support better lending and credit risk decisions using data-driven insights

### Steps followed

- Step 1: Loaded loan default dataset into SQL Server database 'Loan' and created Power BI Dataflow using on-premises gateway connection.

- Step 2: Imported Loan_Default table into Power BI Desktop from the Dataflow for report development.

- Step 3: Checked data types, performed data profiling, and added Year column from Loan_Date_DD_MM_YYYY.

- Step 4: Created dedicated Measures Table1 to organize all DAX calculations for Page 1 (Loan Default & Overview) Report.

	A DAX measure was added using SUMX, FILTER, NOT, and ISBLANK to calculate loan amount by purpose while excluding blanks.

            Loan Amount by Purpose = SUMX(FILTER('Loan_default',NOT(ISBLANK('Loan_default'[LoanAmount]))),'Loan_default'[LoanAmount])

- Step 5: The measure was checked against a table visual and cross-verified with the source data using a Pivot Table in Excel.

- Step 6: A DAX measure using CALCULATE, AVERAGE, and ALLEXCEPT was created to find average income by employment type.

            Average Income by Employment type = CALCULATE(AVERAGE('Loan_default'[Income]),ALLEXCEPT('Loan_default',Loan_default[EmploymentType]))

- Step 7: A DAX measure was built using COUNTROWS, FILTER, ALLEXCEPT, and DIVIDE to calculate the default rate by employment type.

            Default Rate by Employment type = 
            var totalrecords = COUNTROWS(ALL('Loan_default'))
            var DefaultCases = COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE()))

            RETURN 
            CALCULATE(DIVIDE(DefaultCases,totalrecords),ALLEXCEPT('Loan_default',Loan_default[EmploymentType])) *100

- Step 8: A new Age Groups column was added using IF logic to categorize customers as Teen, Adults, Middle Age Adults, or Senior Citizens.  

            Age Groups = 
            IF('Loan_default'[Age]<=19,"Teen",
                IF('Loan_default'[Age]<=39,"Adults",
                    IF('Loan_default'[Age]<=59,"Middle Age Adults",
                        "Senior Citizens")))

- Step 9: A DAX measure using AVERAGEX and VALUES was created to calculate the average loan amount for each age group.

            Average Loan by Age Group = 
            VERAGEX(VALUES('Loan_default'[Age Groups]),
            AVERAGE('Loan_default'[LoanAmount]))


- Step 10: A DAX measure was built using COUNTROWS, ALLEXCEPT, FILTER, and DIVIDE to analyze how default rates changed by year.

            Default Rate by Year = 
            var totalloans = CALCULATE(COUNTROWS('Loan_default'), ALLEXCEPT('Loan_default','Loan_default'[Year]))

            var Default = CALCULATE(COUNTROWS(FILTER('Loan_default',Loan_default[Default]=TRUE())),ALLEXCEPT('Loan_default',Loan_default[Year]))

            RETURN
            DIVIDE(Default,totalloans)*100

- Step 11: Created dedicated Measures Table2 to organize all DAX calculations for Page 2 (Applicant Demographics & Financial Profile) Report.
	Created the Median Loan Amount by Credit Score Bins measure.
A MEDIANX measure was created and validated using a card visual.

- Step 12:  A DAX measure was added to calculate the average loan amount for customers in the High credit score category.

        Credit Score Bins = 
        IF('Loan_default'[CreditScore]<=400,"Very Low",
            IF('Loan_default'[CreditScore]<=450,"Low",
                IF('Loan_default'[CreditScore]<=650,"Medium",
                "High")))

- Step 13: Average loan amount for High credit score calculated using DAX measure AVERAGX.

            Average Loan Amount (High Credit) = 
            AVERAGEX(FILTER('Loan_default','Loan_default'[Credit Score Bins]="High"),'Loan_default'[LoanAmount])

- Step 14: Loan amount by credit bins was analyzed using DAX and line charts to compare loan patterns across customer groups.

            Total Loan (Credit Bins) = 
            CALCULATE(SUM('Loan_default'[LoanAmount]),'Loan_default'[Age Groups]="Adults",ALLEXCEPT('Loan_default',Loan_default[Age],Loan_default[Age Groups],Loan_default[CreditScore],Loan_default[Credit Score Bins]))

- Step 15: A measure was created to calculate total loan amount for middle age adults with mortgage and dependents analysis. 

            Total Loan (Middle Age Adults) = 
            SUMX(FILTER('Loan_default',Loan_default[Age Groups]="Middle Age Adults"),'Loan_default'[LoanAmount])

- Step 16: A measure was added to count loans by education type for demographic analysis.

            Loans by Education type = COUNTROWS(FILTER('Loan_default',NOT(ISBLANK('Loan_default'[LoanID]))))

- Step 17: Created dedicated Measures Table3 to organize all DAX calculations for Page 3 (Financial Risk Metrics) Report. 
	Year-over-year loan amount change was calculated using DAX to compare current and previous year values.

            YOY Loan Amount Change = 
            DIVIDE(
                CALCULATE(SUM('Loan_default'[LoanAmount]),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) - 
                CALCULATE(SUM('Loan_default'[LoanAmount]),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1)

                , CALCULATE(SUM('Loan_default'[LoanAmount]),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),0) * 100

- Step 18: A similar YOY measure was created to track changes in defaulted loans over time.

            YOY Default Loans Change = 
            DIVIDE(
                CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) -
                CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1)

                , CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=TRUE())),'Loan_default'[Year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),0) * 100

- Step 19: A year-to-date measure was built using DATESYTD and ALLEXCEPT to analyze loan amount by credit score bins and marital status.

            YTD Loan Amount = 
            CALCULATE(SUM('Loan_default'[LoanAmount]),DATESYTD('Loan_default'[Loan_Date_DD_MM_YYYY].[Date]),ALLEXCEPT('Loan_default',Loan_default[Credit Score Bins],'Loan_default'[MaritalStatus]))

- Step 20: Added an Income Bracket calculated column in Loan_Default table. A SWITCH function was used to classify customers into Low Income, Medium Income, and High Income groups.

            Income Bracket = 
            SWITCH(
                TRUE(),
                'Loan_default'[Income]<30000,"Low Income",
                'Loan_default'[Income]>=30000 && 'Loan_default'[Income]<60000,"Medium Income",
                'Loan_default'[Income]>=60000,"High Income")

Step 21: Schedule refresh was set up so the dataflow and semantic model stay updated when the source data changes.

Step 22: After completing the dashboard in Power BI Desktop, the report was published to Power BI Service.


## Key Insights

- Default rates vary significantly across employment types and age groups
- Credit score bins strongly correlate with loan amounts and default risk
- Middle age adults represent a key segment for targeted risk analysis
- Year-over-year trends help track improvement in lending quality
- Income brackets provide deeper understanding of repayment capacity

## Dashboard Features

- **Page 1**: Different Line charts were created to represent Loan trends by purpose and Age Groups, Average Income by employment type, Default rate by Employment type and yearly defaults

**Report Snapshot Page 1 (Power BI DESKTOP)**

||
|---|
|![Desktop](https://github.com/hetvi0410/PowerBI-LoanDefault-Dashboard/blob/main/images/Desktop1.png)

- **Page 2**: Created line charts, Donut chart and Clustered Column chart  to represent Applicant demographics, credit score analysis, segment-wise loan amounts.

**Report Snapshot Page 2 (Power BI DESKTOP)**

||
|---|
|![Desktop](https://github.com/hetvi0410/PowerBI-LoanDefault-Dashboard/blob/main/images/Desktop2.png)

- **Page 3**: Created line charts, ribbon chart to represent Financial risk metrics YOY Loan Amount Change, YOY default Loan Change, YTD Loan Amount by Credit Score bins and Marital Status and decomposition tree for Income Bracket classification.

**Report Snapshot Page 3 (Power BI DESKTOP)**

||
|---|
|![Desktop](https://github.com/hetvi0410/PowerBI-LoanDefault-Dashboard/blob/main/images/Desktop3.png)


## Tools & Technologies

- **Power BI Desktop & Service**
- **SQL Server** (Data source)
- **Power BI Dataflow**
- **DAX** (Advanced calculations)
- **On-premises Data Gateway**

## Conclusion

This comprehensive loan default analysis dashboard provides financial institutions with actionable insights to reduce default risk, optimize lending strategies, and improve portfolio quality. The end-to-end implementation demonstrates advanced Power BI techniques including dataflows, complex DAX measures, time intelligence, and production deployment best practices.


