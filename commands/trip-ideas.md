---
name: trip-ideas
description: Turn a vague trip idea into dated flight deals from one airport, with the typical price beside each fare
---

Take a trip description and come back with concrete dates and fares.

Ask me for the idea and the departure airport if I have not given them. The idea can be as loose as "somewhere warm in February" or as specific as a city. The airport has to be a three-letter IATA code.

Then:

1. Call `hasdata_google_travel_flights_deals_getGoogleFlightsDeals` with the description as `q` and the airport as `departureId`.
2. Before adding any filter, check whether I named a destination. Every filter on this tool needs `arrivalId`, and without it the call is rejected with 422. If I asked for direct flights or a price cap without naming a destination, say so and either pin one or leave the filters off.
3. Read `searchInformation.dateRange` and report it up front. Google chooses the window from the description, and for a seasonal query it can be months away.
4. List the deals with the destination city, the dates, the fare, the trip length, the stops and the airline. Where `multipleAirlines` is true there is no airline name, so write that the itinerary uses several carriers rather than leaving a gap.
5. Put the fare next to `typicalPrice` so the reader sees whether it is actually cheap. Roughly two deals in three carry no `discountPercent` while still sitting under the typical fare, so compute the gap yourself for those instead of calling them undiscounted.

Convert `durationMinutes` into hours before showing it.

One origin per call. If I ask to compare two departure airports, say that it costs two calls before making them.
