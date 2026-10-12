## Data exploration: first look

### Dataset
- 8,760 rows: one row per hour, from Dec 2017 to Nov 2018.
- No missing values in any column.

### Rentals by Hour of Day
- The pattern is not a straight line: it has two clear peaks, around 8am and 6pm.
  These match commuting times (going to and coming back from work).
- The evening peak (around 1,500 bikes) is bigger than the morning one (around 1,000), which could maybe be because people also ride for leisure after work rather than solely as a transportation method.
- Rentals are lowest around 4–5am, but still surprisingly high around midnight (around 500 bikes), maybe its people going home after a party or something.

### Distribution of rentals
- It's right-skewed: most hours have fewer than 500 rentals, and a few very busy hours reach around 3,500.

### Rentals over the year
- Clear seasonal pattern: low in winter (Dec–Feb), rising in spring, highest in summer (Jun),
  then dropping again towards Nov.
- The sudden drops to exactly zero are not caused by weather: they are the days the system
  was not operating (the "Functioning days" metric).
- The other sharp dips (not to zero) might be rainy days. This could be corsschecked with rainfall data for that period for a better analysis.
- I only have one year, so I can see the seasonal pattern once but can't confirm it repeats.

### Temperature vs rentals
- Warmer temperatures generally mean more rentals, but the relationship is not a straight line: rentals rise up to around 25–30°C and then level off or fall when it gets very hot, which is understandable as people prefer riding bikes when there is a "clima moderado" rather than any of the extremes.
- There is a flat line of zeros along the bottom between 10 and 25°C. These are the non-functioning days again, not cold weather.
- So temperature is clearly related to rentals, just not in a simple linear way.

### Functioning days
- 295 hours are marked as non-functioning, and every one of them has exactly 0 rentals.
- This means the system was closed or not operational, not that nobody wanted a bike.

### Weekday vs weekend
- Weekdays: two sharp peaks at 8am and 6pm, matching commuting to and from work.
- Weekends: no morning peak; a smooth rise to a peak around 5pm (leisure riding).
- Weekends have more rentals late at night and in the early morning (nights out).
- Conclusion: the effect of the hour depends on the type of day. A model needs separate
  hour patterns for weekdays and weekends (an interaction), not just a weekend on/off column.
- To check: does the "holiday" column behave like a weekend?

### Dew point
- Temperature + humidity explain 99% of dew point (R² = 0.990).
- Decision: drop dew point. It adds almost no new information and would make
  coefficients unstable, which matters when comparing the three methods.

### Non-functioning days
- 13 days, all full days (24h) except one partial day (7h on 6 Oct).
- 10 of the 13 are between mid-September and November: maintenance or failures cluster in autumn.

## Decisions so far
- Remove non-functioning hours (295 rows): the system was closed, so 0 rentals doesn't
  measure demand. The operator knows in advance when it's closed, so the forecast is
  simply 0 then; the model only needs to handle open hours. Also, most closed days are
  in autumn and would distort a test set taken from the end of the year.
- Drop dew point: 99% explained by temperature + humidity (R² = 0.990).
- Hour as one-hot columns, split by weekday/weekend (interaction): weekdays have two
  commuting peaks, weekends have one smooth afternoon peak. One-hot lets the data show
  the shape instead of me choosing hour groups, which would be subjective and could
  differ between weekdays and weekends.
- Drop one hour column as reference to avoid duplicating the intercept.
