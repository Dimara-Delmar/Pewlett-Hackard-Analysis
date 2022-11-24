# Pewlett Hackard Analysis

## Overview of Project

### Purpose
Pewlett Hackard is a company looking to prepare retirement packages for all their eligible employees. Due to this anticipated large-scale drop in workers, they are also looking to fill in all the necessary job positions that will be vacated by their retiring employee base. The purpose of this analysis is to determine the number of employees retiring based on their work title, and the number of current employees that are eligible to participate in the company’s mentorship program. 

## Resources
- Data Sources: departments.csv, dept_emp.csv, dept_manager.csv, employees.csv, salaries.csv, titles.csv
- Software: PostgreSQL11 and pgAdmin

## Summary

### Employee Retirement

Using SQL, the data in the `employee.csv` was combined with the data in the `titles.csv` to find the people ready for retainment by filtering their given birthdates between the years 1952 and 1955 and placing that data into a new table called `retirement_tables`.

    -- Create retirement_titles table
    SELECT e.emp_no, e.first_name, e.last_name, t.title, t.from_date, t.to_date
    INTO retirement_titles
    FROM employees as e
    INNER JOIN titles as t
    ON (e.emp_no = t.emp_no)
    WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
    ORDER BY e.emp_no;

To account for employees having multiple job titles based on things like promotion, a new table titled `unique_titles` was created to remove duplicates and only retain the most recent job titles.

    -- Use Distinct with Orderby to remove duplicate rows + Create unique_titles table
    SELECT DISTINCT ON (emp_no) emp_no, first_name, last_name, title
    INTO unique_titles
    FROM retirement_titles
    WHERE (to_date = '9999-01-01')
    ORDER BY emp_no, to_date DESC;

Once the retirement-aged employees and their unique job titles were accounted for, we used the `COUNT()` function to find the number of employees and filter them by their most recent job titles, excluding employees who are no longer working with the company. The table created to contain the data was named `retiring_titles`.

    -- Create retiring_titles table
    SELECT COUNT (title), title
    INTO retiring_titles
    FROM unique_titles
    GROUP BY title
    ORDER BY COUNT (title) DESC;

### Mentorship Eligibility 

Now that we’ve found the number of people eligible for retirement and what their most current job titles are, we need to find the number of those people eligible for the mentorship program used to prepare new employees needed to fill their roles. To do this, SQL was used to create a new table titled `mentorship_eligibility` that holds the number of currently working employees born in the year 1965.

    -- Create mentorship_eligibility table
    SELECT DISTINCT ON (e.emp_no) e.emp_no, e.first_name, e.last_name, e.birth_date, de.from_date, de.to_date, t.title
    INTO mentorship_eligibilty
    FROM employees as e
    INNER JOIN dept_emp as de
    ON (e.emp_no = de.emp_no)
    INNER JOIN titles as t
    ON (e.emp_no = t.emp_no)
    WHERE (de.to_date = '9999-01-01') 
        AND (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
    ORDER BY e.emp_no;

### Results

By looking at the CSV tables created throughout this process, we can conclude that:
- There are a total of 72,458 employees eligible for retirement.
- Senior engineers have the highest number of retiring employees with a total of 25,916 eligible for benefits. 
- Only 2 managers need to be replaced.
- There are a total of 1,549 people eligible for the mentorship program. 

## Questions
1) How many roles will need to be filled as the "silver tsunami" begins to make an impact?

The `retiring_titles.csv` gives us a clear table on how many people are retiring based on what job title they hold.  

<img width="167" alt="retiring_titles_table" src="https://user-images.githubusercontent.com/108738297/203684524-23e4fcc7-eb99-415b-aff7-3e2de93443c9.PNG">

Adding up the total number of people in the count column, we can see that there are a total of 72,458 job roles that would need to be replaced. 

2) Are there enough qualified, retirement-ready employees in the departments to mentor the next generation of Pewlett Hackard employees?

There do not seem to be enough people ready and qualified to act as mentors for the number of jobs this company needs to replace. If we take a look at the `mentorship_eligiblity.csv`, we can see that there are a total of 1,549 people eligible for the mentorship program, and since there are 72,458 people leaving, there will be a substantial gap between the amount of people who can fulfill those roles and the amount of people who are leaving the company for retirement. 
