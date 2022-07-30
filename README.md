# Nodeschool SF API Interaction

In this lesson you're going to build a Node.js program to read data from the San Francisco Data API, and then write it to the Map Buddy API. These sorts of interactions are common amongst services written using Node.js.

Here's a diagram of what your application will be doing:

![Swimlane Diagram](./swimlane.png)

The first step when working with APIs is to read the documentation. In this case, there are two sets of documentations that you'll read about. For the SF data, we're going to work with data reports made from SF 311 calls. You'll also need the documentation for Map Buddy:

- [SF 311 Cases API Docs](https://dev.socrata.com/foundry/data.sfgov.org/vw6y-z8j6)
- [Map Buddy API Docs](https://docs.mapbuddy.app/)

## Application Preparation

First, make a new directory somewhere for your project. Then, using your terminal, traverse into the directory. Once you're in the directory, initialize a new npm project. And finally, install version 2 of the `node-fetch` package. This process may look something like the following:

```sh
mkdir node-project
cd node-project
npm init -y
npm install node-fetch@2
```

We'll use an older version of `node-fetch` to ensure it works with the version of Node.js that everyone could have installed.

## SF 311 Preparation

> The SF 311 API allows you to get information about SF 311 requests. These requests are associated with the physical location that the request was made for, and the time that the request was made. Additional data about the request is also provided in the response.

The SF Data APIs are all hosted by a service called Socrata. They do allow you to optionally create a user account which allows you to authorize your requests. The main benefit seems to be to allow higher requests rates, which we don't necessarily need, so for now we'll just not make an account.

Here's the URL to get data from:

```
https://data.sfgov.org/resource/vw6y-z8j6.json
```

Just making a request to this URL gives you a JSON response, which is an array of objects. Here's a truncated example of the response:

```json
[
  {
    "service_request_id": "15631421",
    "requested_datetime": "2022-07-29T12:01:33.000",
    "updated_datetime": "2022-07-29T12:01:34.000",
    "status_description": "Open",
    "status_notes": "open",
    "agency_responsible": "Clear Channel - Transit Queue",
    "service_name": "Damaged Property",
    "service_subtype": "Damaged Transit_Shelter_Platform",
    "service_details": "Transit_Shelter_Platform",
    "address": "Intersection of CHURCH ST and DUBOCE AVE",
    "street": "CHURCH ST",
    "supervisor_district": "8",
    "neighborhoods_sffind_boundaries": "Duboce Triangle",
    "police_district": "PARK",
    "lat": "37.76943268329",
    "long": "-122.429140599703",
    "point": {
      "latitude": "37.76943268",
      "longitude": "-122.4291406",
      "human_address": "{\"address\": \"\", \"city\": \"\", \"state\": \"\", \"zip\": \"\"}"
    },
    "source": "Mobile/Open311"
  }
]
```

The API lets you perform further queries based on parameters that you feed to it. For example, here's a complex query:

```
https://data.sfgov.org/resource/vw6y-z8j6.json?$select=service_request_id,requested_datetime,service_name,service_subtype,service_details,address,lat,long&$order=service_request_id desc&$where=service_request_id > 14813843&$limit=1000
```

Read the API docs for further details on how to use it.

Here's an example of how to make a basic request to this URL to get data:

```javascript
const request = require('request');
const response = await fetch('https://data.sfgov.org/resource/vw6y-z8j6.json');
const records = await response.json();
console.log(records);
```

## Map Buddy Preparation

To use Map Buddy, you'll first need to create an account. This is common with APIs where you're actually creating resources or writing changes, and Map Buddy is no different
