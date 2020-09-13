---
title: Hospital Admissions
parent: Data Sources and Signals
grand_parent: COVIDcast API
---

# Hospital Admissions
{: .no_toc}

* **Source name:** `hospital-admissions`
* **First issued:** June 30, 2020
* **Number of data revisions since 19 May 2020:** 0
* **Date of last change:** Never
* **Available for:** county, hrr, msa, state (see [geography coding docs](../covidcast_geography.md))

This data source is based on electronic medical records and claims data about
hospital admissions, provided to us by health system partners. We use this
inpatient data to estimate the percentage of new hospital admissions with a
COVID-associated diagnosis code in a given location, on a given day.

| Signal | Description |
| --- | --- |
| `smoothed_covid19` | Estimated percentage of new hospital admissions with COVID-associated diagnoses, based on data from health system partners, smoothed in time using a Gaussian linear smoother |
| `smoothed_adj_covid19` | Same, but with systematic day-of-week effects removed using [the same mechanism as in `doctor-visits`](doctor-visits.md#day-of-week-adjustment) |

## Table of contents
{: .no_toc .text-delta}

1. TOC
{:toc}

## Limitations

This data source is based on electronic medical records and claims data provided
to us by health system partners. The partners can report on a portion of
hospitalizations, but not all of them, and so this source only represents those
hospitalizations known to them. Their coverage may vary across the United
States.

## Lag and Backfill

Hospitalizations are reported and processed by the health system partners
several days after they occur, so the signal is typically available within
several days of lag. This means that estimates for a specific day are only
available several days later.

The amount of lag in reporting can vary, particularly whether the data comes
from electronic medical records or from processed claims. After we first report
estimates for a specific date, further hospitalization data may arrive for that
date, or diagnoses for admissions from that date may change. When this occurs,
we issue new estimates. This means that a reported estimate for, say, June 10th
may first be available in the API on June 14th and subsequently revised on June
16th.

## Qualifying Admissions

We receive two daily data streams of new hospital admissions recorded by the health system partners at each location. One stream is based on electronic medical records, and the other comes from claims records.

In the electronic medical records stream, admissions are considered COVID-associated if they meet the following criteria:

* If the admission has any ICD-10 code matching {U071, U072, B9729}, or
* If the primary ICD-10 code is one of {R05, R060, R509, Z9911, R0902, R0603,
  R0609, R062, R069, R0602, R05, R0600, J9691, J9692, J9621, J9690, J9601,
  J9600, J189, J22, J1289, J129, J1281, B9721, B9732, B342, B349, A419, R531,
  R6889} and there is a secondary ICD-10 code of Z20828, or
* If the primary ICD-10 code is Z20828.

For the claims stream, admissions are considered COVID-associated if the patient has a primary ICD-10 code matching {U071, U072, B9729, J1281, Z03818, B342, J1289}.


## Estimation

For a fixed location $$i$$ and time $$t$$, let $$Y_{it} = Y_{it}^{\text{emr}} + Y_{it}^{\text{claims}}$$ denote the number of
hospital admissions meeting the qualifying conditions, where the superscript denotes the respective data stream. Similarly, let $$N_{it} = N_{it}^{\text{emr}} + N_{it}^{\text{claims}}$$ denote the
total number of hospital admissions. 

Our estimate of the COVID-19 percentage is
weighted by the contribution from each data stream according to the magnitude of their total admissions.

$$
\hat p_{it} = 100 \cdot \frac{Y_{it} + 0.5}{N_{it} + 1} \approx 100\cdot\left(\frac{N_{it}^{\text{emr}}}{N_{it}}\cdot\frac{Y_{it}^{\text{emr}}}{N_{it}^{\text{emr}}} + \frac{N_{it}^{\text{claims}}}{N_{it}}\cdot\frac{Y_{it}^{\text{claims}}}{N_{it}^{\text{claims}}} \right)
$$

The additional pseudo-observation of 0.5 means this estimate can be interpreted
as the posterior mode of a binomial proportion with a $$\text{Beta}(1/2, 1/2)$$
Jeffreys prior. The practical effect is to prevent $$\hat p_{it}$$ from being
exactly zero or one, which would result in estimated standard errors of 0. The
estimated standard error is:

$$
\widehat{\text{se}}(\hat{p}_{it}) = 100 \sqrt{\frac{\frac{\hat{p}_{it}}{100}(1-\frac{\hat{p}_{it}}{100})}{N_{it}}}.
$$

## Backfill

This source undergoes the same backfill adjustment as the `doctor-visits`
source; see [its documentation](doctor-visits.md#backfill).

## Smoothing

This source undergoes the same smoothing adjustment as the `doctor-visits`
source (see [its documentation](doctor-visits.md#smoothing)), with the exception
that the smoothing is performed on the raw counts, rather than the raw rate.