#!/usr/bin/env python
# coding: utf-8

# ### Note
# * Instructions have been included for each segment. You do not have to follow them exactly, but they are included to help you think through the steps.

# In[1]:


# Dependencies and Setup
import pandas as pd
import numpy as np
import os


# In[2]:


# Loading Files

schools_path = os.path.join("Resources","schools_complete.csv")
students_path = os.path.join("Resources","students_complete.csv")


# In[3]:


# Read School and Student Data File and store into Pandas DataFrames

school_data_df = pd.read_csv(schools_path)
student_data_df = pd.read_csv(students_path)


# In[4]:


# Combine the data into a single dataset. 

completedata_df = pd.merge(student_data_df, school_data_df, how="left", on=["school_name", "school_name"])


# ## District Summary
# 
# * Calculate the total number of schools
# 
# * Calculate the total number of students
# 
# * Calculate the total budget
# 
# * Calculate the average math score 
# 
# * Calculate the average reading score
# 
# * Calculate the percentage of students with a passing math score (70 or greater)
# 
# * Calculate the percentage of students with a passing reading score (70 or greater)
# 
# * Calculate the percentage of students who passed math **and** reading (% Overall Passing)
# 
# * Create a dataframe to hold the above results
# 
# * Optional: give the displayed data cleaner formatting

# ## School Summary

# In[5]:


#Calculating Total school count

unique_school = len(completedata_df['school_name'].unique())


#Calculating Total student count

count_student = completedata_df['student_name'].count()


#Calculating Budget

total_budget = completedata_df['budget'].unique().sum()


#Calculating Average Math Score

avg_math =  completedata_df["math_score"].mean()


#Calculating Average Rading Score

avg_read =  completedata_df["reading_score"].mean()


#Calculate the percentage of students with a passing math score (70 or greater)

math_pass =  completedata_df.loc[completedata_df['math_score'] >= 70]['student_name'].count() 
math_perc =  math_pass / count_student * 100

#Calculate the percentage of students with a passing reading score (70 or greater)

read_pass =  completedata_df.loc[completedata_df['reading_score'] >= 70]['student_name'].count() 
read_perc =  read_pass / count_student * 100


#Calculate the percentage of students who passed math and reading (% Overall Passing)
passed_math = completedata_df.loc[(completedata_df['math_score'] >= 70) & (completedata_df['reading_score'] >= 70)]['student_name'].count()
perc_total  = passed_math/ count_student*100


# Dataframe to hold the above results

summary_df = pd.DataFrame ({ "Total Schools": [unique_school],
    "Total Students": [count_student],
    "Total Budget": [total_budget],
    "Average Math Score": [avg_math],
    "Average Reading Score": [avg_read],
    "% Passing Math": [math_perc],
    "% Passing Reading":[read_perc],
    "Overall Passing Rate": [perc_total] })



#formatting cells
summary_df.style.format ({ "Total Budget": "${:,.2f}", 
                       "Average Math Score": "{:.6f}", 
                       "Average Reading Score": "{:.6f}", 
                       "% Passing Reading": "{:.6f}",
                       "% Passing Math": "{:.6f}",
                       "Overall Passing Rate":"{:.6f}" })
        
           


# * Create an overview table that summarizes key metrics about each school, including:
#   * School Name
#   * School Type
#   * Total Students
#   * Total School Budget
#   * Per Student Budget
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * % Overall Passing (The percentage of students that passed math **and** reading.)
#   
# * Create a dataframe to hold the above results

# In[6]:


#School Type

school_type =  school_data_df.set_index(['school_name'])['type']

#Total Students

count_student = completedata_df['school_name'].value_counts()

#Total School Budget

school_budget =  school_data_df.set_index(['school_name'])['budget']

#Per Student Budget


student_budget =  school_budget/ count_student

#Average Math Score

math_avg = completedata_df.groupby(['school_name']).mean()["math_score"]

# Average Reading Score

read_avg = completedata_df.groupby(['school_name']).mean()["reading_score"]



#% Passing Math

math_sum = completedata_df[completedata_df['math_score'] >= 70].groupby(['school_name']).count()['student_name'] / count_student * 100


#% Passing Reading
read_sum = completedata_df[completedata_df['reading_score'] >= 70].groupby(['school_name']).count()['student_name'] / count_student * 100


#% Overall Passing (The percentage of students that passed math and reading.)

total_sum = completedata_df[(completedata_df['reading_score']>= 70 )& (completedata_df['math_score'] >= 70) ] .groupby(['school_name']).count()['student_name'] / count_student * 100

#Dataframe to hold the above results

school_sum = pd.DataFrame({
    "School Type": school_type,
    "Total Students": count_student,
    "Total School Budget": school_budget,
    "Per Student Budget": student_budget,    
    "Average Math Score": math_avg ,
    "Average Reading Score": read_avg,
    "% Passing Math": math_sum,
    "% Passing Reading": read_sum,
   "% Overall Passing": total_sum })


#formatting cells

school_sum.style.format({'Total Students': '{:}', 
                          "Total School Budget": "${:,.2f}", 
                          "Per Student Budget": "${:,.2f}",
                          "Average Math Score": "{:.6f}", 
                          "Average Reading Score": "{:.6f}", 
                          "% Passing Math": "{:.6f}", 
                          "% Passing Reading": "{:.6f}", 
                          "Overall Passing Rate": "{:.6f}"})


# ## Top Performing Schools (By % Overall Passing)

# * Sort and display the top five performing schools by % overall passing.

# In[7]:


# sorting top 5 

top_five = school_sum.sort_values("% Overall Passing", ascending = False)


#formatting cells

top_five.head().style.format({"Total Students": '{:}',
                           "Total School Budget": "${:,.2f}", 
                           "Per Student Budget": "${:.2f}", 
                           "% Passing Math": "{:.6f}", 
                           "% Passing Reading": "{:.6f}", 
                           "% Overall Passing": "{:.6f}"})


# ## Bottom Performing Schools (By % Overall Passing)

# * Sort and display the five worst-performing schools by % overall passing.

# In[8]:


#bottom 5 schools from worse to best


bottom_five = school_sum.sort_values("% Overall Passing", ascending = True)

#formatting cells

bottom_five.head().style.format({"Total Students": '{:}',
                           "Total School Budget": "${:,.2f}", 
                           "Per Student Budget": "${:.2f}", 
                           "% Passing Math": "{:.6f}", 
                           "% Passing Reading": "{:.6f}", 
                           "% Overall Passing": "{:.6f}"})


# ## Math Scores by Grade

# * Create a table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school.
# 
#   * Create a pandas series for each grade. Hint: use a conditional statement.
#   
#   * Group each series by school
#   
#   * Combine the series into a dataframe
#   
#   * Optional: give the displayed data cleaner formatting

# In[9]:


#creates grade level average math scores for each school 

ninth_math = student_data_df.loc[student_data_df["grade"] == '9th'].groupby("school_name").mean()["math_score"]
tenth_math = student_data_df.loc[student_data_df["grade"] == '10th'].groupby("school_name").mean()["math_score"]
eleventh_math = student_data_df.loc[student_data_df["grade"] == '11th'].groupby("school_name").mean()["math_score"]
twelfth_math = student_data_df.loc[student_data_df["grade"] == '12th'].groupby("school_name").mean()["math_score"]

#Dataframe to hold the above results

math_scores = pd.DataFrame({
        "9th": ninth_math,
        "10th": tenth_math,
        "11th": eleventh_math,
        "12th": twelfth_math
})

math_scores = math_scores[['9th', '10th', '11th', '12th']]


#formatting

math_scores.style.format({"9th": '{:.6f}', 
                          "10th": '{:.6f}', 
                          "11th": "{:.6f}", 
                          "12th": "{:.6f}"})


# ## Reading Score by Grade 

# * Perform the same operations as above for reading scores

# In[10]:


#creates grade level average reading scores for each school

ninth_reading = student_data_df.loc[student_data_df["grade"] == '9th'].groupby("school_name").mean() ["reading_score"]
tenth_reading = student_data_df.loc[student_data_df["grade"] == '10th'].groupby("school_name").mean()["reading_score"]
eleventh_reading = student_data_df.loc[student_data_df["grade"] == '11th'].groupby("school_name").mean()["reading_score"]
twelfth_reading = student_data_df.loc[student_data_df["grade"] == '12th'].groupby("school_name").mean()["reading_score"]

#Dataframe to hold the above results

reading_scores = pd.DataFrame({
        "9th": ninth_reading,
        "10th": tenth_reading,
        "11th": eleventh_reading,
        "12th": twelfth_reading
})
reading_scores = reading_scores[['9th', '10th', '11th', '12th']]



#formatting

reading_scores.style.format({"9th": '{:.6f}', 
                             "10th": '{:.6f}', 
                             "11th": "{:.6f}", 
                             "12th": "{:.6f}"})


# ## Scores by School Spending

# * Create a table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * Overall Passing Rate (Average of the above two)

# In[12]:


# create spending bins

bins = [0,585, 630, 645, 999999]
group_name = ['< $585', "$585 - 629", "$630 - 644", "$645-675"]
completedata_df["spending_bins"] = pd.cut(completedata_df["budget"]/completedata_df["size"], bins, labels = group_name)

#group by spending

by_spending = completedata_df.groupby("spending_bins")

#calculations

avg_math = by_spending["math_score"].mean()
avg_read = by_spending["reading_score"].mean()
pass_math = completedata_df[completedata_df["math_score"] >= 70].groupby("spending_bins").count()["Student ID"]/by_spending["Student ID"].count()*100
pass_read = completedata_df[completedata_df["reading_score"] >= 70].groupby("spending_bins").count()["Student ID"]/by_spending["Student ID"].count()*100
overall = completedata_df[(completedata_df["reading_score"] >= 70) & (completedata_df["math_score"] >= 70)].groupby("spending_bins").count()["Student ID"]/by_spending["Student ID"].count()*100

            
#Dataframe to hold the above results

scores_by_spend = pd.DataFrame({
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    "% Passing Math": pass_math,
    "% Passing Reading": pass_read,
    "% Overall Passing": overall
            
})
            
#reorder columns
scores_by_spend = scores_by_spend[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing"
]]

scores_by_spend.index.name = "Per Student Budget"
scores_by_spend = scores_by_spend.reindex(group_name)

#formatting
scores_by_spend.style.format({'Average Math Score': '{:.2f}', 
                              'Average Reading Score': '{:.2f}', 
                              '% Passing Math': '{:.2f}', 
                              '% Passing Reading':'{:.2f}', 
                              '% Overall Passing': '{:.2f}'})


# * Perform the same operations as above, based on school size.

# ## Scores by School Size

# In[13]:


# create size bins
bins = [0, 999, 1999, 99999999999]
group_name = ["Small (<1000)", "Medium (1000-2000)" , "Large (2000-5000)"]
completedata_df["size_bins"] = pd.cut(completedata_df["size"], bins, labels = group_name)

#group by spending
by_size = completedata_df.groupby('size_bins')

#calculations 
avg_math = by_size["math_score"].mean()
avg_read = by_size["math_score"].mean()
pass_math = completedata_df[completedata_df["math_score"] >= 70].groupby('size_bins').count()['Student ID']/by_size['Student ID'].count()*100
pass_read = completedata_df[completedata_df["reading_score"] >= 70].groupby('size_bins').count()['Student ID']/by_size['Student ID'].count()*100
overall = completedata_df[(completedata_df["reading_score"] >= 70) & (completedata_df['math_score'] >= 70)].groupby('size_bins').count()['Student ID']/by_size['Student ID'].count()*100

            
#Dataframe to hold the above results

scores_by_size = pd.DataFrame({
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    "% Passing Math": pass_math,
    "% Passing Reading": pass_read,
    "% Overall Passing": overall })
            
#reorder columns
scores_by_size = scores_by_size[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing"]]

scores_by_size.index.name = "Total Students"
scores_by_size = scores_by_size.reindex(group_name)

#formating
scores_by_size.style.format({"Average Math Score": '{:.6f}', 
                              "Average Reading Score": '{:.6f}', 
                              "% Passing Math": '{:.6f}', 
                              "% Passing Reading":'{:.6f}', 
                              "% Overall Passing": '{:.6f}'})


# ## Scores by School Type

# * Perform the same operations as above, based on school type

# In[14]:


# group by type of school
by_type = completedata_df.groupby("type")

#calculations 
avg_math = by_type["math_score"].mean()
avg_read = by_type["math_score"].mean()
pass_math = completedata_df[completedata_df["math_score"] >= 70].groupby("type").count()["Student ID"]/by_type["Student ID"].count()*100
pass_read = completedata_df[completedata_df["reading_score"] >= 70].groupby("type").count()["Student ID"]/by_type["Student ID"].count()*100
overall = completedata_df[(completedata_df["reading_score"] >= 70) & (completedata_df["math_score"] >= 70)].groupby("type")["Student ID"].count()/by_type["Student ID"].count()*100

#Dataframe to hold the above results

scores_by_type = pd.DataFrame({
    "Average Math Score": avg_math,
    "Average Reading Score": avg_read,
    "% Passing Math": pass_math,
    "% Passing Reading": pass_read,
    "% Overall Passing": overall})
    
#reorder columns

scores_by_type = scores_by_type[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing"]]

scores_by_type.index.name = " School Type "



#formating
scores_by_type.style.format({"Average Math Score": '{:.6f}', 
                              "Average Reading Score": '{:.6f}', 
                              "% Passing Math": '{:.6f}', 
                              "% Passing Reading":'{:.6f}', 
                              "% Overall Passing": '{:.6f}'})

