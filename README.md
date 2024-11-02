
# HR Analytics Using Power BI

## Project Overview
This project focuses on building a data model and analyzing HR data of a fictitios compane named Atlas Labs using Power BI. The report design includes insights into employee attrition, hiring trends, and key HR metrics.

### Key Objectives
- **Data Modeling:** Developing a comprehensive data model
- **Report Design:** Creating an interactive and insightful report for HR analytics

---

## Steps

### 1. Loading CSV Files

Atlas Labs intends to create a report to track HR analytics through an end-to-end development process. The first step is loading and preparing the dataset. 

There are five CSV files that must be imported:
- `EducationLevel.csv`
- `Employee.csv`
- `PerformanceRating.csv`
- `RatingLevel.csv`
- `SatisfiedLevel.csv`

Each file is imported into Power BI as a table. The tables are formatted as text, numbers, or dates and renamed with prefixes to indicate whether they are **Fact** or **Dimension** tables.

- **Fact Table:** `PerformanceRating` (contains performance ratings of employees)
- **Dimension Tables:** `EducationLevel`, `Employee`, `RatingLevel`, and `SatisfiedLevel`

---

### 2. Creating a Date Table and Managing Relationships

For accurate date and time reporting, a calculated Date table (`DimDate`) is created. Below is the DAX code used to define the table:

```DAX
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(
    CALENDAR(
        DATE(_minYear,1,1),
        DATE(_maxYear,12,31)
    ),
    "Year", YEAR([Date]),
    "Year Start", DATE(YEAR([Date]),1,1),
    "YearEnd", DATE(YEAR([Date]),12,31),
    "MonthNumber", MONTH([Date]),
    "MonthStart", DATE(YEAR([Date]), MONTH([Date]), 1),
    "MonthEnd", EOMONTH([Date],0),
    "DaysInMonth", DATEDIFF(DATE(YEAR([Date]), MONTH([Date]), 1), EOMONTH([Date],0), DAY)+1,
    "YearMonthNumber", INT(FORMAT([Date],"YYYYMM")),
    "YearMonthName", FORMAT([Date],"YYYY-MMM"),
    "DayNumber", DAY([Date]),
    "DayName", FORMAT([Date],"DDDD"),
    "DayNameShort", FORMAT([Date],"DDD"),
    "DayOfWeek", WEEKDAY([Date]),
    "MonthName", FORMAT([Date],"MMMM"),
    "MonthNameShort", FORMAT([Date],"MMM"),
    "Quarter", QUARTER([Date]),
    "QuarterName", "Q"&FORMAT([Date],"Q"),
    "YearQuarterNumber", INT(FORMAT([Date],"YYYYQ")),
    "YearQuarterName", FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
    "QuarterStart", DATE(YEAR([Date]), (QUARTER([Date])*3)-2, 1),
    "QuarterEnd", EOMONTH(DATE(YEAR([Date]), QUARTER([Date])*3, 1),0),
    "WeekNumber", WEEKNUM([Date]),
    "WeekStart", [Date]-WEEKDAY([Date])+1,
    "WeekEnd", [Date]+7-WEEKDAY([Date]),
    "FiscalYear", IF(_fiscalStart=1, YEAR([Date]), YEAR([Date])+QUOTIENT(MONTH([Date])+(13-_fiscalStart),13)),
    "FiscalQuarter", QUARTER(DATE(YEAR([Date]),MOD(MONTH([Date])+(13-_fiscalStart)-1,12)+1,1)),
    "FiscalMonth", MOD(MONTH([Date])+(13-_fiscalStart)-1,12)+1
)
```

- The `DimDate` table is connected to `FactPerformanceRating` and `DimEmployee` tables, with only one active relationship allowed at a time.
- **Additional Relationships:**
  - `DimEducation` table to `DimEmployee`
  - `FactPerformanceRating` -> `DimSatisfiedLevel` (using `EnvironmentSatisfaction` as the active relationship)
  - `FactPerformanceRating` -> `DimRatingLevel` (using `SelfRating` as the active relationship)

---

### 3. Exploring the Data

The goal is to gain visibility into high-level metrics about employee status and attrition within the company. Key measures are created in an empty table called `_Measures`:

```DAX
TotalEmployees = DISTINCTCOUNT(DimEmployee[EmployeeID])
ActiveEmployees = CALCULATE([TotalEmployees], FILTER(DimEmployee, DimEmployee[Attrition] = "No"))
InactiveEmployees = CALCULATE([TotalEmployees], FILTER(DimEmployee, DimEmployee[Attrition] = "Yes"))
Attrition Rate = DIVIDE([InactiveEmployees], [TotalEmployees])
```

These measures are displayed as card visuals, showing:
- **Attrition Rate:** 16.1%

---

#### 3.1 Hiring Trends Over Time

To analyze hiring trends over time, a stacked column chart is created using `DimDate` and `FactPerformanceRating` tables. Since `DimDate` already has an active relationship, additional relationships are activated using `USERELATIONSHIP()` within the `CALCULATE()` function.

The visualization of employee hiring trends over the yers based on attrition, department and gender are given below by using stacked column charts.

![Employee Hiring Trends](<assets/Employee_Hiring _Trends_1.png>)
*Stacked Column chart visualizing employee attrition by year*

![Employee Hiring Trends](<assets/Employee_Hiring _Trends_2.png>)
*Stacked Column chart visualizing employee attrition by department*

![Employee Hiring Trends](<assets/Employee_Hiring _Trends_3.png>)
*Stacked Column chart visualizing employee attrition by gender*

---

### 3.2 Analyzing Departments and Job Roles

A treemap visualization is created to analyze typical roles and hiring patterns within departments at Atlas Labs.

![Employee Hiring Trends](<assets/Active_Employees_By_Dept_and_Job_Role.png>)
*Stacked Column chart visualizing employee attrition by gender*

#### Initial Insights
- **Total Employees:** 1470 since inception
- **Active Employees:** 1200 currently
- **Largest Department:** Technology
- **Attrition Rate:** 16%

---

## 4. Factors impacting employee attrition - Key HR Metrics 

### 4.1 Diversity & Inclusion

To understand diversity, metrics related to age and gender are analyzed. A conditional column is created to categorize employee ages into bins. Key visuals include:
- Card visuals for **minimum and maximum age**
- Column charts for **employee distribution by age**
- Clustered charts for **employee distribution across age bins and gender**

![Employee Hiring Trends](<assets/Factors_affecting_attrition_page-level_filter.png>)


### 4.2 Marital Status and Gender** distribution

![Employee Hiring Trends](<assets/Employees_by_maritas_status.png>)

### 4.3 Employees by Ethnicity and Average Salary

![Employee Hiring Trends](<assets/Employees_by_ethnicity_and_average_salary.png>)
** The lowest average salary ($106k) is among mixed/multiple ethnic groups.

---

### 5. Performance Tracker

The performance tracker monitors individual employee performance based on yearly reviews.

- **Last Review Date** and **Next Review Date** calculated as follows:

    ```DAX
    LastReviewDate = IF(MAX(FactPerformanceRating[ReviewDate]) = BLANK(), "No Review Yet", MAX(FactPerformanceRating[ReviewDate]))
    
    NextReviewDate = 
    VAR reviewOrHire = IF(MAX(FactPerformanceRating[ReviewDate]) = BLANK(), MAX(DimEmployee[HireDate]), MAX(FactPerformanceRating[ReviewDate]))
    RETURN reviewOrHire + 365
    ```

- Measures track **job satisfaction, work-life balance, relationship satisfaction, and environment satisfaction**.

---

### Insights

- Majority of employees are between 20-29 years old.
- **Gender Balance:** Atlas Labs employs 2.7% more women than men among active employees.
- **Non-Binary Employees:** 8.5% of employees identify as non-binary.
- **Salary by Ethnicity:** White employees have the highest average salary; mixed/multiple ethnic groups have the lowest.

---

### Employee Attrition Analysis

Employee attrition, whether voluntary or involuntary, is analyzed with insights into factors contributing to attrition.

- **Attrition by Department and Job Role**

![Employee Hiring Trends](<assets/Percentage_attrition_by_dept_and_job_role.png>)

It can be noted that sales representatives have the highest attrition rates.


- **Attrition by Hire Date**

![Employee Hiring Trends](<assets/Attrition_by_hire_date.png>)


- **Other factors contributing to attrition**

![Employee Hiring Trends](<assets/Other_attrition_factors.png>)

- Highest attrition rates are among employees who travel frequently.


