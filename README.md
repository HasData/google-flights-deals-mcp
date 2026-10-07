# Google Flights Deals MCP Server

<!-- mcp-name: com.hasdata/google-flights-deals -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client one Google Flights Deals tool. Describe a trip in plain language, such as "cherry blossom in Japan" or "beach escape", and get dated fares from one origin airport with the typical price beside them, the airline, the stops and a booking link, as structured JSON.

**1,000 free credits every month, no card required**, which is about 66 searches.

```
https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals
```

[![Glama score](https://glama.ai/mcp/servers/HasData/google-flights-deals-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/google-flights-deals-mcp)
[![tool contract](https://github.com/HasData/google-flights-deals-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/google-flights-deals-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-1-10b981?style=flat-square)](#tools)
- [Prompts and resources](#prompts-and-resources)
[![npm](https://img.shields.io/npm/v/@hasdata/google-flights-deals-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/google-flights-deals-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-google-flights-deals-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-google-flights-deals-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp), free to create with no card, and the free tier covers about 66 calls a month at the 15-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run and no Google account anywhere in the flow. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/google-flights-deals-mcp` on npm and `hasdata-google-flights-deals-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http google-flights-deals "https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

```json
{
  "mcpServers": {
    "google-flights-deals": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

```json
{
  "mcpServers": {
    "google-flights-deals": {
      "type": "streamable-http",
      "url": "https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

```json
{
  "servers": {
    "google-flights-deals": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=google_travel_flights_deals",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 15 credits.

> I want to see cherry blossom in Japan, flying from LAX. What are the cheapest dates?

*One call, 15 credits. Google picks the destinations and the date range from the description.*

> Beach escape from London in February, nothing over 600 pounds, direct only.

*One call, 15 credits. The price ceiling and the stops filter both need a destination pinned with `arrivalId`, so the agent has to name one or drop the filters.*

> Same trip but to Tokyo specifically, ten days, business class.

*One call, 15 credits. With `arrivalId` set, the trip length and the cabin become available.*

> Compare the cherry blossom idea from LAX and from JFK.

*Two calls, 30 credits. One origin per call.*

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_google_travel_flights_deals_getGoogleFlightsDeals` | Dated deals with the fare, the typical price for the route, flight duration, trip length, stops, the operating airline, the airports and a Google Flights booking link, plus the destinations and date range Google derived from the query. 15 credits a call |

One tool, 15 credits per successful call.

### Find flight deals

[`hasdata_google_travel_flights_deals_getGoogleFlightsDeals`](https://docs.hasdata.com/apis/google-travel/flights-deals?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp)

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `q` | string | yes | Free-text trip description, from a bare place name to a full sentence |
| `departureId` | string | yes | Origin as a three-letter uppercase IATA code, for example `LAX` |
| `arrivalId` | string | | Pins the destination instead of letting Google choose. **Every filter below needs it** |
| `type` | string | | `roundTrip` (default) or `oneWay` |
| `travelClass` | string | | `economy`, `premiumEconomy`, `business` or `first` |
| `outboundDate` | string | | An exact date, or a window as `2026-12-01,2026-12-10` |
| `returnDate` | string | | Same spelling as `outboundDate`, and required alongside it |
| `travelDuration` | string | | Preset length, `week`, `weekend` or `twoWeeks` |
| `tripLength` | string | | Length in days, instead of `returnDate` or `travelDuration` |
| `stops` | string | | `nonStop`, `oneStopOrFewer` or `twoStopsOrFewer` |
| `maxDuration` | number | | Ceiling per flight in minutes |
| `maxPrice` | number | | Ceiling in the currency of the request |
| `includeAirlines` / `excludeAirlines` | string | | One or the other, never both |

The filters are the thing to get right. **All of them require `arrivalId`.** A prompt that asks for direct flights under a price cap while letting Google pick the destination is asking for two things that cannot be combined, and the API says so with a 422 naming the offending pair. Pin a destination or drop the filters.

`searchInformation` reports what Google made of the query: the `departure` airport in full with its city and coordinates, the `dateRange` it settled on, the `priceRange` it found and the `airlines` involved.

```json
{
  "searchInformation": {
    "query": "cherry blossom in Japan",
    "departure": { "airport": "LAX", "city": "Los Angeles", "country": "United States" },
    "dateRange": { "from": "2027-03-01", "to": "2027-04-30" }
  },
  "flightDeals": [
    {
      "outboundDate": "2027-03-01",
      "returnDate": "2027-03-08",
      "price": { "value": 960, "currency": "USD" },
      "typicalPrice": { "value": 1335, "currency": "USD" },
      "discountPercent": 28,
      "durationMinutes": 1320,
      "tripLengthDays": 7,
      "stops": 1,
      "airlineCode": "CX",
      "airline": "Cathay Pacific",
      "multipleAirlines": false,
      "departureAirport": "LAX",
      "arrivalAirport": "HIJ",
      "destination": { "city": "Hiroshima", "country": "Japan", "highlight": "Peace Park Memorial & Shukkei-en garden" },
      "bookingLink": "https://www.google.com/travel/flights?tfs=…"
    }
  ]
}
```

## Prompts and resources

The server exposes 7 resources, one per parameter whose accepted values are a fixed list. Reading one is cheaper than learning the vocabulary from a rejected call, and it costs no credits. Each URI is `hasdata://google_travel_flights_deals/<parameter>`.

| Parameter | Values | What it selects |
| --- | ---: | --- |
| `type` | 4 | Flight type. Requires `arrivalId`. - `1` / `roundTrip` — round trip (default) - `2` / `oneWay` — one way A one-way deal carries no `returnDate` and no `tripLengthDays`. |
| `travelClass` | 8 | Travel class. Requires `arrivalId`. - `1` / `economy` — economy (default) - `2` / `premiumEconomy` — premium economy - `3` / `business` — business - `4` / `first` — first Fares climb steeply: on LAX-NRT the same search ran $730 in economy against $2882 in business. |
| `travelDuration` | 6 | Preset trip length. Requires `arrivalId`. Cannot be combined with `returnDate` or `tripLength`. - `1` / `week` — about a week (6-8 days) - `2` / `weekend` — a weekend (2-3 days) - `3` / `twoWeeks` — about two weeks (13-15 days) Pairs with `outboundDate` to limit the departure period. Ignored when `type` is `oneWay`. |
| `stops` | 6 | Maximum number of stops. Requires `arrivalId`. Omitted, any number is allowed. - `1` / `nonStop` — direct flights only - `2` / `oneStopOrFewer` — at most one connection - `3` / `twoStopsOrFewer` — at most two connections A route with nothing at that depth returns an empty `flightDeals` array, not an error — `nonStop` on a route without a direct flight is a valid, empty answer. |
| `gl` | 245 | The two-letter country code for the country you want to limit the search to. |
| `hl` | 159 | The two-letter language code for the language you want to use for the search. |
| `currency` | 71 | Parameter defines the currency of the returned prices |

The list is served without an API key, so a client can read it before a user has signed up.

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text.

**`discountPercent` is the exception, not the rule.** Across 351 deals measured on 2026-10-05 it was present on 109 of them. Where it appears it matches the gap between `price` and `typicalPrice` to within a rounding step, and where it is absent that gap ran from 4% to 15%, so Google appears to publish the field only past a threshold of its own. `typicalPrice` was on every deal, so compute the comparison yourself rather than reporting "no discount" because the key is missing.

**`airline` and `airlineCode` go missing together.** In that same sample 269 deals named a carrier and 82 did not, and the split is exactly the `multipleAirlines` flag: every deal that set it to true omitted both fields, every deal that did not carried both. Read the flag before reporting a carrier, and say "several airlines" rather than leaving the field blank.

**A filter without `arrivalId` is rejected before it runs.** The API answers 422 with `rule: "requiredIfExists"` naming both `arrivalId` and the filter that triggered it, and nothing is billed. All eleven filters behave this way, so the failure is loud and cheap rather than silent.

**Dates are what Google chose unless you pinned them.** `searchInformation.dateRange` is the range it searched, which can be months away from today. Quote it when presenting prices, because a fare for next March is not a fare for next month.

Each successful call spends credits from the connected account. Neither a 422 from validation nor a 400 from upstream is billed, measured by reading the balance either side of a call.

## Pricing, free tier and limits

The Google Flights Deals tool costs **15 credits per successful call**, the same as the Google Flights API. The number of deals returned does not change the price.

The free tier is **1,000 credits every month with no card**, which is about 66 searches.

Paid plans start at **$59 a month** for 200,000 credits, which is about 13,000 searches. The unit price falls on larger plans. Current numbers are on the [plans page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp).

## How it compares

| | Google Flights API | This server |
| :--- | :--- | :--- |
| Input | An origin, a destination and dates | A sentence, with the destination optional |
| Who picks the destination | You | Google, from the description |
| Who picks the dates | You | Google, unless you pin them |
| Best for | A trip someone has already decided on | A trip someone is still imagining |

The two sit next to each other rather than competing. Use this one to find where and when, then the [Google Flights API](https://hasdata.com/apis/google-flights-api?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) to price the itinerary once the trip is decided.

## FAQ

### Do I need a Google account?

No. The only credential involved is your HasData key.

### Why were my filters ignored?

They almost certainly went out without `arrivalId`. Every filter on this tool needs a pinned destination, and a request that omits it still succeeds.

### Can I search from several origins at once?

No. `departureId` takes one airport, so comparing origins means one call each.

### How far ahead does it search?

As far as Google decides from the query. Read `searchInformation.dateRange` rather than assuming the next few weeks.

### Is HasData affiliated with Google?

No. HasData is an independent web data provider and is not affiliated with, endorsed by or sponsored by Google. All trademarks belong to their owners.

### Compliance and personal data

The server reads the public Google Flights deals interface. It carries no passenger data and books nothing.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [Google Flights API](https://hasdata.com/apis/google-flights-api?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) |
| Endpoint documentation | [Google Flights Deals API docs](https://docs.hasdata.com/apis/google-travel/flights-deals?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) |
| Every tool in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-flights-deals-mcp) |

## Development

```bash
npm install
npm test
```

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=google_travel_flights_deals` returns the one expected tool, that its name and required parameters have not changed, that it carries a description, and that a real search still returns `flightDeals` with the documented fields.

## Contributing

The parameter table and the sample above were read from the live schema and from a real call rather than from documentation. A correction is welcome when a field or a failure mode has changed. Open an issue with the response you saw.

## License

MIT
