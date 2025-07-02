# Aurora Bank Analysis 
![1000191964](https://github.com/user-attachments/assets/fe9e2d3d-9fc0-4e27-a5c3-79c35025b5f9)
## Table of Content
- [Overview](#overview)
- [Project Objectives](#project-objectives)
- [Data Cleaning](#data-cleaning)
- [Key Questions](#key-questions)
- [Key Findings](#key-findings)
- [Recommendations](#recommendations)

## Overview 
This project provide a comprehensive analysis of Aurora Bank customer base, transaction behaviour and card operations. Analyzed using Sql queries for actionable insight to help to make decisions.
## Project Objectives
The primary ojectives of the project is to:
- Understand customer demographics and financial profile
- Monitor spending trends, transaction errors and customer at risk based on credit score and debt
- Assess card owned by customer and usage

## Data Cleaning 
- Rename table and columns
```
- Rename table name-
exec sp_rename '[dbo].[user$]','customer';
exec sp_rename '[dbo].[Transaction Data]','customertransaction';
exec sp_rename '[dbo].[Card]','Card_data';
 exec sp_rename '[dbo].[mcc_codes$]','mcc'

- Rename column name
    exec sp_rename '[dbo].[user$].id','Client id';
	exec sp_rename 'customertransaction.client_id','Client id';
	exec sp_rename'card_data.date','card_date'
exec sp_rename 'mcc.mcc_id','merchant_id'
```
- Add New Column Retirement status 
```
-- Create Retirement status column
alter table customer
add retirement_status varchar(50)
update customer
set Retirement_Status =
case
when current_age > retirement_age then'Retired'
else 'Not Retired'
end
```
- Add New Column Credit score rating
```
---- create credit score rating-
alter table customer
add [Credit score rating]varchar(100)
update customer
set[Credit score rating]=
case
when credit_score between 800 and 850 then 'Excellent'
when credit_score between 740 and 799 then 'Very Good'
when credit_score between 670 and 739 then 'Good'
when credit_score between 580 and 699 then 'Fair'
when credit_score between 300 and 579 then 'poor'
else 'unknown'
end
```
- Add New column Risk Levels
```
--- Customer risk column
alter table customer
add [Risk level] varchar(100)
update customer
set [Risk level] =
case 
when credit_score >=740 
then 'Low Risk'
when credit_score between 670 and 739 then 'Middle Risk'
when credit_score <670 then 'High Risk'
end
```
## Key Questions 
- Customer
1. What is the Total number of customers and thier gender distribution
2. What is the total debt by customer
3. What is the Total debt based on gender and identify the gender with the highest debt
4. How many customer fall into each credit score rating category 
5. How many customers are classified under each risk level based on thier credit score

- Transactions
1. What is the total transaction amount across all customer
2. What are the most common transactions error during transactions
3. Which month and year has the highest transaction amount
4. which card brand recorded the highest transaction volume in 2024

- Card Analysis
1. What is the total number of card issued to customer by card type and brand between 2020 and 2024
2. How many card has EMV Chips
3. what is the total number of customers having different card brands
## SQL Query 
```

--- Customer

---CUSTOMER ANALYSIS

--1. what is the total number of customers and gender distribution--
  select gender,count(distinct[client id]) as [total customer] from customer
  group by gender

--2. what is the Total debt by customer-
select sum(Total_debt) as [Total Debt] from Customer

--3. What is the total debt based on genderand identify the gender with the highest total debt -
select gender, sum(Total_debt) as [Total Debt] from customer
group by gender
order by [Total Debt] desc

--4.-- How many customer fall into each credit score rating category 
select[credit score rating], count(*)as [Total customer] from customer
group by [credit score rating]
order by [Total customer]desc

--5.	How many customer are classified under each risk level based on thier credit score
select [risk level] ,count([client id]) as total_customer from customer
group by [risk level]


----Card Analysis--
--1. what is the Total card type and card brand issued to customer between 2020 and 2024
select card_type,card_brand,year (card_date)as year, sum( num_cards_issued) as [total Card] from card_data
where year( card_date) between 2020 and 2024
group by card_type, card_brand,year(card_date)

--2.Total card issued between 2020 and 2024
 select year(card_date)as year, sum( num_cards_issued) as [total Card] from card_data
 where year(card_date)between 2020 and 2024
 group by year(card_date)
order by [total card]asc

---3. How many card has EMV chips
 select has_chip, count(has_chip) as[Card with chip] from card_data
 group by has_chip


 --4. what is the total number of customer with different card brand
 select card_brand, count(client_id) as TOtal_customer from Card_data
 group by card_brand


 --- transaction analysis
 --1.  what is the total transaction amount
 select sum (amount) as total_transaction_amount from customertransaction

 --2. what are the most common transaction errors during transaction
 select top 5 errors,count(errors) as total_errors from customertransaction
 group by errors
 order by total_errors desc


 --3. which month and year has the highest transaction amount 
 select top 1 month([date]) as month,year([date])as year, sum (amount)as [TOtal transaction amount]from customertransaction
 group by month([date]) ,year([date])
 order by [TOtal transaction amount]desc

 --4. which card brand has the highest transaction volume in 2024
 select year([date])as year,Card_data.card_brand, count(customertransaction.id) as transaction_volume from customertransaction
 join  card_data on customertransaction.[client id]=card_data.[client_id]
 where year([date])=2024
 group by card_data.card_brand,year([date])
 order by transaction_volume desc ,year([date])
```

 
 ## Key Findings
1.**Customer Demographics and Financial Profile**
- A total of 2000 customers (1,016 Females and 984 males)
- A total of $127.42M debt as female has the highest debt
- 81 customers have poor credit score <579 and 166 have excellent credit score >800
- A total of 640 customer at low risk and 429 are high risk based on their credit score rating 

2. **Card Distribution**
- 7,566 card was issued to customers between 2020 and 2024
- 3,209(52%)of the customer use mastercard and majority make use of debit card
- 5,500(89%) of card has EMV chips ensuring higher security during transactions

3. **Transaction Analysis**
- The most frequent transaction error is insufficient balance
- The highest number of transaction was recorded in july 2023
- Mastercard has the highest transaction volume  in 2024

## Recommendations
- Tailor loan offering based on credit score and risk level of customers
- Enhance fraud detection measures to monitor transaction of card without EMV chips
- Educate customers on security advantages of EMV technology through emails
  
