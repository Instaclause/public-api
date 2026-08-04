# AdminConsult API — relay AdminConsult data to Instaclause

One of the [two ways into Instaclause](./README.md). This page documents the **`adminconsult`** source.

> ✅ **You are in the right place if** you are forwarding objects that came out of the
> AdminConsult / AdminIS API, unchanged. You post AdminConsult's own response objects and
> Instaclause interprets them.
>
> ↩️ **Pushing data from your own CRM or an in-house system?** That is the other API —
> see the **[Custom API guide](./CUSTOM.md)**. Different payload, different routes,
> different setup.

Base URL: `https://app.instaclause.be/api/v1/adminconsult`

Records posted through this API create customer entities in Instaclause — companies
(BV, CommV, NV, etc.) and persons (shareholders, directors, etc.). These entities are
stored in a separate collection that is **isolated per customer**, and they do not mix
with manually added CRM data or with other CRM integrations (HubSpot, Adminpulse,
FID-manager, Tess, Exact Online, etc.) that may be enabled in parallel in the same office.

---

## Before you start — turn off the scheduled pull

Instaclause can also **pull** from AdminConsult on a nightly schedule. That pull writes to
the same records as this API, matched on the same `id`. If it stays switched on, the
nightly sync will overwrite whatever you push.

Before pushing anything, the office must tick **Disable data fetching** on the
**AdminConsult & AdminIS** integration under **Integration Settings**.

![The AdminConsult & AdminIS integration expanded, with Disable data fetching ticked](./docs/images/adminconsult-disable-data-fetching.png)

- The checkbox only appears on the AdminConsult row, and only once the Public API is enabled for the office.
- It switches off the scheduled pull, and hides the AdminConsult credential fields so no connection test or manual import runs.
- It affects the AdminConsult integration only — no other integration changes behaviour.
- It does **not** gate this API. The push routes keep working whether the box is ticked or not; ticking it is what stops the pull from competing with your writes.

---

## Authentication

Include the API key in the `Authorization` header of every request, as the **raw value** —
no `Bearer` prefix.

```
Authorization: <your API key>
```

There is one key per office and it does not expire. **See
[Authentication in the README](./README.md#authentication) for how to obtain it, where it
appears in the settings page, and what the Refresh button does to keys already in use.**

A request fails with `401 Unauthorized` if the header is missing, the key is malformed or
not one Instaclause issued, the key has been superseded by a **Refresh**, or the Public API
is switched off for that office.

> ⚠️ **The `adminconsult` segment in the URL is case-sensitive.** Use it in lowercase, exactly as written. A capitalised variant still passes authentication and can return `201 Created` while the data is written nowhere — you would see successful responses and nothing appearing in Instaclause.

---

## Routes

> **Order matters: post the customer before its sub-resources.** `/addresses` and
> `/customerlinkcustomer` attach to a customer that must already exist. A sub-route call
> for a customer that has not been posted yet still answers `201 Created`, but nothing is
> stored — so a `201` on these two routes is not on its own proof that the write landed.
> Always create the customer first, then its addresses and links.

### /customers

- Method: POST
- Endpoint: /customers
- Headers:
  - Authorization: [API Key]
- Body: the object for a single customer returned from AdminConsult API /customers
  <details>
    
    <summary>Example</summary>

  ```json
    {
      "AccCode": "string",
      "AccountancySoftware": 0,
      "AccountancySoftwareLabel": "string",
      "CommercialName": "string",
      "CompanyId": 0,
      "CreationDate": "string",
      "CupboardNumber": "string",
      "Currency": "string",
      "CustCode": "string",
      "CustKind": "string",
      "CustomerCrmType": 0,
      "CustomerGroup": 0,
      "CustomerGroupLabel": "string",
      "CustomerId": 0,
      "DateOfBirth": "string",
      "DisabledDate": "string",
      "Distance": 0,
      "Email": "string",
      "Fax": "string",
      "Firstname": "string",
      "Holding": 0,
      "Homepage": "string",
      "IsActive": true,
      "IsCompany": true,
      "Language": "string",
      "Mobile": "string",
      "NaceCode": "string",
      "Name": "string",
      "Nationality": "string",
      "Newsletter": true,
      "Phone": "string",
      "Phone2": "string",
      "PlaceOfBirth": "string",
      "ReasonForLeaving": 0,
      "RegistrationNr": "string",
      "Remarks": "string",
      "RPR": "string",
      "Sector": "string",
      "SectorId": 0,
      "Sex": "string",
      "SocialSecurityNumber": "string",
      "Title": "string",
      "VATNr": "string"
    }
  ```

  </details>
- Response
  - Success Status Code: 201 Created
  
### /customers/{customerId}/addresses

- Method: POST
- Endpoint: /customers/{customerId}/addresses
- Headers:
  - Authorization: [API Key]
- Body: the object returned from AdminConsult API /customers/{customerid}/addresses
- Response
  - Success Status Code: 201 Created
  
### /customers/{customerId}/customerlinkcustomer

- Method: POST
- Endpoint: /customers/{customerId}/customerlinkcustomer
- Headers:
  - Authorization: [API Key]
- Body: the object returned from AdminConsult API /customers/{customerid}/customerlinkcustomer
- Response
  - Success Status Code: 201 Created

---

## FAQ

### How do you differentiate between clients if that application is going to upload to Instaclause from multiple locations? Does each client get a unique API key?
Each office has its own unique API key, which identifies the office and authenticates the request.

### Frequency of sync once a day sufficient?
Yes.

### After the initial upload, should only the clients with changes be posted? Or a full run with all data each time (for customeraddress and customerlinkcustomer)?
For /customers/{customerId}/addresses and /customers/{customerId}/customerlinkcustomer requests the full list must be posted.

### Does this also apply to the customers-call which returns a List? (So always the complete list or only the customers containing changes)?
For requests to /customers, it's possible to only post customers with changes. This will overwrite the customer data, therefore all adresses and customerlinkcustomer needs to be posted again.

### Can I delete a customer?
No. There is no DELETE route, so a client pushed once cannot be removed through the API — it stays available in the party picker. Decide up front how your sync should handle clients that leave the office.

### Our sync returns 201 but nothing shows up in Instaclause.
Two known causes: the source segment in the URL was not lowercase `adminconsult`, or addresses / links were posted for a customer that did not exist yet. See the notes under [Authentication](#authentication) and [Routes](#routes).
