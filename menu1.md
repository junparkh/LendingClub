---
layout: page
title: Data Wrangling
permalink: /menu1/
---
{:.no_toc}
*  
{: toc}
### Description of Data

We downloaded the raw data from the Lending Club website. The data was structured as a large.csv with about 1.8 millions rows.  Data was provided for both the unfunded and funded loans. Unfortunately, as already mentioned, there was very little data provided related to the demographics of the borrowers.  This made performing analyses related to either discrimination or preferential treatment impossible.

#### Overview of the Raw Data
There were 1,873,290 records in the data file covering a timespan of 11 years.  The data in the &quot;funded&quot; file relates to loans which have been approved by LendingClub, assigned a credit score and then subsequently funded by investors.  The unfunded data (&#39;rejected&#39;) relates to borrowing applications which had been approved by LendingClub and assigned a credit score but not funded by investors.  The critical part of our modeling would focus on whether loan applications which were funded were actually repaid and, if they were repaid, the overall level of interest earned on these loans.  Unfortunately, the unfunded data was of little help in determining profitability as there is no subsequent record of whether these loan applicants would or would not have repaid their loans.

[If we had been able to pursue the discrimination issue further then the unfunded data would have been a potentially interesting source of information.  We believe that the most likely point where discrimination might take place would be at the point between &quot;approved by LendingClub&quot; and &quot;funded by investors&quot;.   That said, given LendingClub&#39;s poor track record with respect to data transparency there would also have been opportunities for discrimination in the initial phase from applicant to approval and credit scoring by LendingClub itself. For more details on the ongoing litigation against Lending Club please see the EDA section.]

### Data Cleansing
We concentrated data cleansing activities within the file Data\_cleansing.ipynb.  The data wrangling can be broken down into five different parts:

1. Credit-Worthiness
2. Loan Dates
3. Loans Profitability
4. Selected Predictors
5. Cleaned File

#### (1) Credit-Worthiness 
Some of the key metrics needed to be adjusted to allow the dates to show up as parametric variables. These included:

- Zip code: We were provided with a three digit zip code which was the extent of our demographic data. We created a function &quot;extract\_number&quot; which extracted the Zip Code from the data;
- Credit History: The timespan from the first credit application until today represents the length of time that the applicant has been active within the credit scoring system. In general, longer credit histories are viewed as better risks;
- Employment Length: Data needed to be cleansed to allow it to be continuous. Employment length is typically seen as one of the more important indicators of creditworthiness;
- Annual Income: Obviously an important credit score indicator. However, from an investor point of view we would prefer that the applicant did not repay his or her loan early. We imputed the missing values for this predictor using the mean; and
- Credit Score: We turned the A1, B2 etc credit scores into a linear/ordinal predicator and assigned it a score from 0.0 (A1 - the best) to 6.8 (G5 - the worst). We created a function &quot;subgrade\_score&quot; which performed this parametrization of the credit score variable.

#### (2) Loan Dates
There were a number of issues regarding the loan dates. In particular, we needed to understand which loans were actually completed and which were still current. We decided to exclude the current loans from our model. A couple of the key parameters are worth discussing:

- Last\_pymnt\_date: this refers strictly to the last payment made rather than the date that the last payment of the loan is due. To be clear, if a loan was made in Month=1 and had a term of 36 months then the final loan payment would be due in Month=36. If the current month were Month=10 and the most recent payment had been made then the parameter last\_pymnt\_date would record 10 rather than 36;
- Issue date: refers to the start date of the loan or when the money was drawn down; and
- Term\_months: refers to the length of the loan assuming that all payments are successfully made and the loan reaches its natural end point.

#### (3) Loan Profitability 
This was by far the most difficult metric to analyse. There is a saying in the finance community that, &quot;you cannot eat APY&quot;.  This refers to the fact that if the APY (Annual Percentage Yield) charged on a loan is very high but does not last very long then you can have exceptionally large APY but no nominal return. An example serves to illustrate this point:

Loan 1 = 100$; Term = 1 month; APY = 313%.

Loan 2 = 100$; Term = 12 months; APY = 20%.

The total cash generated by the first loan over the life of the loan is 10$. The APY is high because the 10$ was generated over a short period of time - 1 month. On an annual basis this works out to an APY of 313% (1.1^12). On the other hand, the cash generated by the second loan is 20$ or twice as much. The total amount earned over the two loans was 30$ or an overall APY of 15%.

Taking averages of APYs is therefore quite tricky. To facilitate our investment strategy and modeling over such a large database we decided to exclude loans with length less than 60 days. This is reasonable from an operational point of view we would prefer to have longer loans – our targets are in fact the regular 3 or 5 year loans.  Also, we excluded APRs\&gt;100%. These occur either due to penalty payments or due to very short loan periods. These types of loans are not what we are targeting.

Completed Loans: we assigned &quot;good&quot; to those loans which were fully paid-off.

#### (4) Selected Predictors
Primarily due to lack of data fields we were forced to reduce the predictors from the original data to the following 19 predictors:

- loan\_amnt
- term\_months
- int\_rate
- credit\_score
- emp\_length\_years
- home\_ownership
- annual\_inc
- verification\_status
- dti:  debt to interest following the LendingClub loan
- deling\_2yrs: whether the applicant had a delinquent payment in the last 2 years
- credit\_history\_years
- ing\_last\_6mths: whether there has been a credit inquiry in the past 6 months
- mths\_since\_last\_deling: months since the last delinquent payment
- mths\_since\_last\_record: months since the last public record of credit history
- pen\_acc: the number of open credit accounts in the borrower&#39;s credit file
- pub\_rec: Number of derogatory public records
- revol\_bal: total credit – revolving balance (outstanding debt)
- revol\_util\_perc: percentage of the revolving facility utilized e.g. a credit card line
- total\_acc: the total number of credit lines in the borrower&#39;s account

#### (5) Cleaned File
We then used this selected subset of parameters to create a new loan file and &quot;pickled&quot; it as analysis-predictors.bz2. The target data set (y) was saved as analysis-target.bz2).

We also saved the complete file as complete.bz2. This file would be important for EDA.

By reducing overall size of our working file we reduced the computational time required to run our models without gives up any critical information.

### Business Model Update

#### Mechanics of OnLine Lending
Following an in depth review of the LendingClub (LC) website as well as an analysis of their past two SEC Edgar 10K submissions our understanding of the overall business model reveals some important considerations that the investor should keep in mind and provided us with a certain level of domain knowledge for our models.

We will spend a bit of time reviewing the mechanics of how the OnLine Lending busienss works so that we have sufficient domain knowledge and a common vocabulary.

The overall LC business model works as follows:

|   | Example | Comment |
| --- | --- | --- |
|  (a)Fees paid by Borrower to LC |   | These fees are about 4% of the total payments made by the borrower. Source:10K  |
|   (b)Gross interest paid by Borrowed | 15%   | Assuming a 10,000 loan this would be 15% per annum. This is a function of the credit risk of the borrower and determined by LC. Higher credit risk means higher gross interest.  |
|  (c)Fees paid by Investor to LC | 1% | Paid on any payments (interest or principal) made by the borrower.   |
|  (d)Net Credit Costs | 9% | High due to the unsecured nature of the credit. Very difficult to collect against defaulters.  |
|  (e)Net Investor returns  | 5% | (b) – (c) – (d) = (e) |

**Comments** :

**(a):  Fees paid by Borrows to LC**: Most of LCs revenues come from fees paid by the borrower to LC rather than from the fees paid by investors.  Q4 2017 revenues were 121M$ from transaction and 24M$ from investor fees. LC also collects additional fees during any work-out loan cases, however, the total value of these fees was 1.4M$ for the same quarter or less than 2% of revenues.

**(b) &amp; (d) Gross interest paid by the borrower and Net Credit Costs**: **Prior** to being offered to investors individual loan applications are screened and given a credit score.  Some loans are rejected by LC outright and not offered to investors.  Of those loans offered to investors, the gross interest rate charged is set based on the creditworthiness of the borrower. LC, therefore, has already used an algorithm to assign gross interest charges to a borrower before these loans are seen by investors. &quot;Is this algorithm accurate?&quot;, will be a key aspect of our model.

How much additional interest should be applied to compensate for reduced credit-worthiness?  An example is useful at this point to illustrate the fat-tailed nature of the loss distribution function often found in finance.  (See references – Nassim Taleb).

#### Example Loan
Assumptions:
- 3-year loan;
- 36 equal payments of interest and principal, i.e. no final bullet payment
- Final payment = 1/36 of the total loan
- &quot;Excellent Credit&quot; has 100% chance of repaying the loan
- &quot;Poor Credit&quot; defaults 50% of the time after making 18 months of payment. No loan recover is made

|   | **Excellent Credit** | **Poor Credit** |
| --- | --- | --- |
| (e) Desired return | 5% | 5% |
| (d) Net Credit Costs (\*) | 0% | 25% |
| (c) Investor fee to LC | 1% | 1% |
| (b) Gross Interest Rate Required | 6% | 31% |

To earn the same investment return from the poor credit as from the excellent credit the gross interest charged would need to be 31% vs 6% respectively.  This is driven by the fact that 50% of the time there is a 50% loss of principal for the poor credit risks. Therefore, over the course of 54 payments (36 payments 50% of the time, and 18 payments 50% i.e. until default) sufficient additional income needs to be generated to cover the 5000$ principal loss that occurs 50% of the time.

(c) Fee paid by Investor to LC. This fee is not affected by credit worthiness as it is only collected in the event payment is made

(e) Each investor has their own targeted returns, however, for simplicity, we have used the examples given on the Investor video provided by LC = 5%.

#### Summary of Findings – Caps, and Floors
Each loan tranche that the investor funds have an implicit capped return, i.e. the borrower agrees to the loan with a fixed interest rate which cannot be increased over the life of the loan. This is the maximum yield that the investor can earn and therefore represents the cap. On the other hand, because it is an unsecured loan the potential loss for the investor is 100%.

#### Implications for Modeling
The main driver of future returns is ensuring that credit losses are minimised. The overall distributions of credit losses has a fat, left-tail distribution.  The maximum return is capped when the loan is made but the actual return can be -100%.  In addition, due to the nature of the credit market, instead of paying 25 % and then default, it makes more sense for the borrowers to default on the whole amount, which explains the shape of the fat tail. The reason for this is that the borrower will have a delinquent credit record whether they repay 25% of 0%. There is therefore often not much more to lose by defaulting completely from the loan.
