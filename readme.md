# Alsoa WebHook Integration

Alsoa WebHook integration for HTTP JSON payloads. API endpoint accepts flat JSON objects of strings/arrays that represent a conversion event and submits it to relevant ad tracking conversion APIs.

---

## Quickstart

```bash
curl --location --request POST 'https://us-central1-alsoa-dev.cloudfunctions.net/eventapi/test-webhook' \
--header 'Content-Type: application/json' \
--data-raw '{"email": "john.doe@gmail.com"}'
```

* Use the included `postman_collection.json` with Postman to immediately make test calls to our testing API
* Test API endpoint is https://us-central1-alsoa-dev.cloudfunctions.net/eventapi/test-webhook
* Once you are familiar with the above, **PLEASE** read full documentation!

---

## Terms Glossary

* **JSON Payload:** The data submitted to the Alsoa API via HTTP call
* **Field:** A piece of data regarding the conversion event, mapped to a key/value pair within the JSON payload
* **Originator:** the HTTP client calling us on a conversion event (ie: your integrating client)
* **Tracker:** the tracking API that we submit the conversion event to (ex: Google Ads, Facebook)
* **User Identifier:** a piece of data identifying a user submitted with a lead (ex: email address, phone number)
* **CLID:** Click identifier (set by URL parameter coming into lead generation form, submitted along to our WebHook)

---

## WebHook URLs to Integrations

Each unique conversion action will get it's own unique URL provided by Alsoa. This means that each lead/purchase conversion will have one unique WebHook URL to submit events to.

* Example URL: https://us-central1-alsoa-prod.cloudfunctions.net/eventapi/track/xkdT3ytSmzCcc743q0sD
* Unique per-integration; this means each unique lead conversion or purchase event type will have it's own WebHook URL. Examples:
   - Given http://form-a.com, http://form-b.com, http://form-c.com - three (3) unique WebHook URLs
   - Given http://store-a.com, http://store-b.com - two (2) unique WebHook URLs

---

## "Submit What You Can" Strategy

If you have access to the data outlined in the field definitions below **PLEASE** send it as it helps us have higher data fidelity in regards to tracking events. However, you will likely not always have all of this data (ie: missing fields) - in that circumstance send as many fields as you can.

The only assumed field is the conversion `at` date time value. If this is omitted we assume the conversion happened at the time you called our WebHook since timestamps are a hard requirement for the trackers.

#### Partial CLID Submission:

In the circumstance that we do not have *any* CLIDs submitted to our WebHook we submit the tracking event to all configured trackers. This means that we may submit a partial tracking event to trackers where the event did not originate.

**Example:** Clicked on Google Search Ad but `g_clid` was not set and both Google/Facebook integrations are configured for WebHook URL. We don't know who the desired tracker would be in this situation so we send the conversion event to each configured tracker.

---

## Singular/Multiple Field Support

Some of the fields we collect are allowed to be of an array type to support multiple user identifiers per-event. An example of this would be if there were multiple email or phone #'s associated with a single conversion event.

In the case that a tracker does not support multiple user identifiers for a field, the last array item is used to submit the tracking event.

If duplicates exist for email/phone fields within a multiple field array then they will be removed before tracking.

##### Examples:

Assuming just the email field, both are valid JSON payloads:

```json
{ "email" : "email.singular@gmail.com" }
```

```json
{
   "email" : [ "email.a@gmail.com", "email.b@gmail.com", "email.c@gmail.com" ]
}
```

---

## Field Definitions

All fields are requested to be of **string datatype** `typeof x === "string"` even if the value is purely numeric (phone).

| Field key    | Field                  | Singular/Multiple | Notes                                            |
| -------------|------------------------|-------------------|--------------------------------------------------|
| `first_name` | First name             | both              |                                                  |
| `last_name`  | Last name              | both              |                                                  |
| `email`      | Email address          | both              |                                                  |
| `city`       | City                   | both              |                                                  |
| `state`      | State (or Province)    | both              |                                                  |
| `zip`        | Zip (or Postal Code)   | both              |                                                  |
| `phone`      | Phone #                | both              |                                                  |
| `at`         | Date converted         | singular          | Date > JSON.stringify "2022-05-09T22:01:23.561Z" |
| `url`        | URL converted at       | singular          |                                                  |
| `ip`         | With IP                | singular          |                                                  |
| `uas`        | With user agent string | singular          |                                                  |
| `fb_clid`    | Facebook CLID          | singular          |                                                  |
| `g_clid`     | Google Ads CLID        | singular          |                                                  |
| `fbclid`     | Facebook CLID          | singular          | (alternative key)                                |
| `gclid`      | Google Ads CLID        | singular          | (alternative key)                                |


---

## HTTP API Spec:

* **HTTP method** - `POST`
* **URL:** - Provided by Alsoa when setting up your integration (notes above)
* **Headers:**
   - `User-Agent: Your Client` (optional, but preferred)
   - `Content-Type: application/json` (optional, but preferred)

---

## API Call Examples:

Different fields chosen for illustrative purposes:

### Fully Singular w/Facebook CLID:

```json
{
    "first_name" : "John",
    "last_name"  : "Doe",
    "email"      : "john.doe@gmail.com",
    "city"       : "Flagstaff",
    "state"      : "Arizona",
    "zip"        : "86001",
    "phone"      : "3195551234",
    "fb_clid"    : "PAAabyzfvPy7wke2y3m6sNN0WQ6znA8bfELKDnN8-PrDicr1j_hM21oO-GYkc_aem_AVv-AD2XmEbD4E56cvyev"
}
```

### Singular/Multiple Without CLID:

```json
{
    "first_name" : [ "John", "Sally" ],
    "last_name"  : [ "Doe", "Mae" ],
    "email"      : [ "john.doe@gmail.com", "sally.mae@gmail.com" ],
    "phone"      : [ "3195551234", "3195551235" ],
    "ip"         : "182.27.44.223",
    "uas"        : "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36"
}
```

### Singular/Multiple w/Google Ads CLID:

```json
{
    "first_name" : "John",
    "last_name"  : "Doe",
    "email"      : "john.doe@gmail.com",
    "city"       : "Flagstaff",
    "state"      : "Arizona",
    "zip"        : "86001",
    "phone"      : "3195551234",
    "at"         : "2022-05-09T22:01:23.561Z",
    "g_clid"     : "CjwKCAjw682TBhATEiwA9crl3wR5SGkqta7gqGqd3yoRABj-zJ_8tjevUKY8cfZXj0_p1jeTDRLEUxoC27sQAvD_BwE"
}
```

