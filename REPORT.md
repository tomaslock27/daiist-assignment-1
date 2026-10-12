# Assignment 1 Report

- **Name**: Tomas Lock
- **Student ID**: 18613
- **Email**: tlock.ieu2022@student.ie.edu
- **Group**: BBA BDBA A

## Dataset

The Seoul Bike Sharing Demand dataset, from the UCI Machine Learning Repository. Each row
is one hour of Seoul's public bike-sharing system, from 1 December 2017 to 30 November 2018:
8,760 rows and 14 columns. The target is the number of bikes rented in that hour. The other
columns are the date and hour, weather (temperature, humidity, wind speed, visibility,
dew point, solar radiation, rainfall, snowfall), the season, whether it was a public
holiday, and whether the system was operating.

I picked it because I'm getting into cycling, so I have a feel for why and when people
ride, and that helps with designing features. It also has a time dimension, which makes
the split a real decision rather than a default.

## Business / real-life framing

The model supports Seoul's bike-sharing operator in planning the next day. Each day, the
operations team uses a forecast of tomorrow's hourly rentals to decide how many bikes
(and free docks) need to be in service across the city at each hour, and when to schedule
staff to redistribute bikes.

- **Target:** the total number of bikes rented in each hour of the next day, citywide.
  The data is citywide, not per station, so the model supports city-level planning
  (fleet size, staffing), not decisions about individual stations.
- **When:** one day ahead. In practice, the weather inputs would come from a weather
  forecast. This dataset contains the actual weather, so I assume the forecast is
  accurate, which makes my results somewhat optimistic (see Limitations).
- **Only open hours:** days when the system is closed for maintenance are known in
  advance, so the forecast for them is simply 0 and they are removed from the model's
  data (295 hours, leaving 8,465).
- **Cost of errors:** predicting too few bikes is worse than predicting too many. If
  customers find no bike, the company loses that ride and may lose the customer to
  another type of transport for good. Too many bikes only wastes staff time and leaves
  bikes idle.
- **When errors matter most:** at rush hours, because that's when most customers (and
  potential customers) are affected. A miss at 6pm costs far more than a miss at 4am.

### How this framing drives the pipeline
- **Split:** chronological, by whole days, because the model is used to predict days it has
  never seen, after the days it learned from. A random split would let the model learn from
  days after the ones it is tested on, which is impossible in real use.
  Train: Dec 2017–Aug 2018. Validation: Sep 2018 (used for tuning). Test: Oct–Nov 2018
  (used once, at the end).
- **Consequence for features:** with this split, the training data contains no autumn, so the
  model can't learn anything about the "Autumn" category of the `seasons` column. I drop
  `seasons` and rely on the weather columns, which describe what each season is actually like.
- **Features:** only information available one day ahead (calendar, holidays, weather
  forecast). Nothing that would only be known on the day itself.
- **Metric:** RMSE in bikes, because it penalises large errors more than small ones, and the
  largest errors happen at rush hours, when they matter most. I also report RMSE on weekday
  rush hours only (7–9am and 5–7pm), to measure performance where it matters most for the
  business. MAE and R² are reported as supporting numbers.
- **Log target or not:** the target is right-skewed, so a log transform might help linear
  regression. But a log target makes the model focus on relative errors, which gives less
  weight to rush-hour misses. I let the validation set decide: I train both versions and
  keep the one with the lower validation RMSE in bikes (result in the Modeling section).
- **Safety buffer:** linear regression treats over- and under-predicting the same, but for
  this business under-predicting is worse. The dashboard has a buffer slider (plan for
  forecast + X%) that shows the trade-off between hours with too few bikes and bikes
  left idle. The model is unchanged; the buffer is a planning rule on top of it.

## Data preparation & feature engineering

*What you engineered and why, and any data-quality decisions you made along
the way — e.g. "segment X had defective data, so I excluded it and used a
population-average default for scope Y at inference time; the impact of
that choice is Z."*

## Modeling: three implementations, one model

*Which model (linear or logistic regression) and why. A results table
comparing scikit-learn, the manual PyTorch loop, and the standard
torch.nn.Module/torch.optim workflow, on the same test set, against the
naive baseline. Do the three agree? If not, why not?*

## Limitations & next steps

*Real limitations you found, and concretely how you'd address each one with
more time or data — not generic hedging.*

## Generative AI use disclosure

*Per the syllabus AI Policy: what you used and how, or "no AI content used."*
