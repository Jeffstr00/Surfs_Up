# Surfs_Up

## Overview

I (apparently) love both surfing and ice cream.  Who wouldn't?  In an attempt to bring together my two loves into a match made in heaven that will leave me with an absolute dream job where I never "work" another day in my life, I come up with the idea of starting a surf & shake shop in Oahu, Hawaii.  Mostly based on his perfect name (but also because he has both a love of surfing and a history of investing in surf shops), we found a man named W. Avy who was potentially interested in helping us with our investment.

However, he does have a bit of trepidation regarding the project.  He previously invested in a surf shop without doing his due diligence to check on the weather, and poor weather forecasts soon turned into poor business forecasts.  When the clouds rained on Hawaii (and his surf shop), they also rained on his parade and ultimately led to the closing of his business.

In an attempt to make sure that he doesn't make the same mistake twice, he wants us to run some analytics on the weather at Oahu so that he can get an idea of what to expect regarding the weather (and, accordingly, business prospects) before making an investment.  He has provided us access to an SQLite database holding nearly a decade's worth of data on local weather, and it is our job to retrieve and interpret that data in order to assist in the decision making process.  After first asking us to find out how much precipitation there has been over the last year, he also asks us to look at historical temperature trends in both typically warmer (June) and colder (December) months.

## Results

### Retrieving Data

![June All Temperatures](https://github.com/Jeffstr00/Surfs_Up/blob/main/Resources/june_all_temps.png)

After connecting to the SQLite database, we were able to ask it for all of the June temperatures throughout the years using the following query: `results_jun = session.query(Measurement.date, Measurement.tobs).filter(extract('month',Measurement.date) == 6)`.  (A similar query for December was conducted simply by switching the 6 to 12.)  After using .all() to convert our return to a list, we converted it to a dataframe  using `temp_jun = pd.DataFrame(results_jun, columns=['date','temperature'])`.  From there, a simple .describe() gave us an overall view of the month's temperatures by showing its average, minimum/maximum, and distribution statistics.

### Interpreting Data

![June Weather](https://github.com/Jeffstr00/Surfs_Up/blob/main/Resources/june_weather.png) ![December Weather](https://github.com/Jeffstr00/Surfs_Up/blob/main/Resources/dec_weather.png)

* When comparing the results of June to those of December, the most shocking difference is that there were 1700 temperature counts in June but only 1517 in December, meaning that temperature happens more often in June for some reason.  Just kidding.

* In what should come as a surprise to absolutely no one, Oahu is warmer in June than it is in December.  However, the average difference is not even 4°F (74.94 vs 71.04, or 3.9° total).  Compared to somewhere like New Jersey ( [New Jersey Climate Information](http://www.worldclimate.com/climate/us/new-jersey) ), where average temperatures by month vary by more than ten times that at ~40° (and thus pretty much has to operate seasonally), Oahu's more consistant temperatures should mean that they'll be able to operate the business throughout the year.

* However, in Oahu, June is not actually THAT much hotter than it is in December.  Whether you're looking at temperatures at the first, second, or third quartile breaks, the difference is only 4°, 4°, and 3° respectively.  In fact, the difference when it comes to maximum temperature is only 2°!  That means that, even in the winter, there will still be plenty of days when it is warm enough for surfing (and hopefully ice cream as well).

* However, there is more temperature variation in December (as evidenced by a standard devation of 3.75 vs 3.26 for June), and most of that variation seems to come at the lower end of the spectrum.  While warmer temperatures only differed by 3-4°, the minimum temperature was 64 vs 56, for a difference of 8° (so roughly double the difference of warmer temperatures).  As a result, there might be some days where, while it might not be too cold to keep people out of the water entirely, you'd  expect fewer surfers (and certainly fewer people craving ice cream).

## Summary

So far, the prognosis seems good for the surf & shake shop.  W. Avy previously seemed satisfied when looking at the precipitation data, and looking at the temperatures seems to indicate that the shop should be able to operate year round (even if some months would inevitably be busier than others).  However, further analysis could be conducted to get an even better idea about how the weather might impact success.

![Rain Days](https://github.com/Jeffstr00/Surfs_Up/blob/main/Resources/rain_days.png)
* While we previously looked at levels of precipitation to see how much it might be expected to rain, you might also want to simply see how many days where it did rain so that you would know how many days you would expect to not have much business.  A query of `results = session.query(Measurement.date, Measurement.prcp).filter(Measurement.prcp >= .01).all()` would show all days where it rained more than .01 inches, and obviously that amount could easily be changed if it was determined that the little bit of rain wouldn't be enough to dissuade potential surfers from coming out.

![Good Days](https://github.com/Jeffstr00/Surfs_Up/blob/main/Resources/good_days.png)
* While it is good that we have looked at both precipitation levels and temperatures, so far they have been looked at independantly.  As a result, we may have overlapping data.  For instance, let's say that we looked at 100 days and found 20 to be too rainy and 20 to be too cold for surfing.  Are we left with only 60 days that are good for business?  Or do those bad weather days overlap, leaving us with 80 good days?  As a result, it would be a good idea to look at both simultaneously so that we can see how many "good" days we are left with.  For instance, if we determined that business will be good as long as it doesn't rain and temperatures are at least in the 70's, we can isolate those days by using the query `results = session.query(Measurement.date, Measurement.prcp, Measurement.tobs).filter(Measurement.prcp < .01).filter(Measurement.tobs >= 70).all()`.