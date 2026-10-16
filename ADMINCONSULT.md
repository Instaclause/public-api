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
stored in a separate collection that is **isolated per office**, and they do not mix
with manually added CRM data or with other CRM integrations (HubSpot, Adminpulse,
FID-manager, Tess, Exact Online, etc.) that may be enabled in parallel in the same office.

<p align="center">
  <a href="#-before-you-start--turn-off-the-scheduled-pull">Before you start</a> ·
  <a href="#-getting-your-api-key">API key</a> ·
  <a href="#-authentication">Authentication</a> ·
  <a href="#-routes">Routes</a> ·
  <a href="#-faq">FAQ</a>
</p>

---

<br>

## ⚠️ Before you start — turn off the scheduled pull

Instaclause can also **pull** from AdminConsult on a nightly schedule. That pull writes to
the same records as this API, matched on the same `id` — so if it stays switched on, it
overwrites whatever you push, every night.

```
  UNTICKED — the pull competes with your push:

    AdminConsult ──── nightly pull ─────┐
                                        ├──▶  same records, matched by id
    You ──────────── POST /customers ───┘     ⚠ the pull wins overnight

  TICKED — push-only:

    AdminConsult      (scheduled pull off)
    You ──────────── POST /customers ───────▶  records stay as you pushed them
```

Before pushing anything, the office must tick **Disable data fetching** on the
**AdminConsult & AdminIS** integration under **Integration Settings**.

![The AdminConsult & AdminIS integration expanded, with Disable data fetching ticked](./docs/images/adminconsult-disable-data-fetching.png)

- The checkbox only appears on the AdminConsult row, and only once the Public API is enabled for the office.
- It switches off the scheduled pull, and hides the AdminConsult credential fields so no connection test or manual import runs.
- It affects the AdminConsult integration only — no other integration changes behaviour.
- It does **not** gate this API. The push routes keep working whether the box is ticked or not; ticking it is what stops the pull from competing with your writes.

---

<br>

## 🔑 Getting your API key

Open the [Instaclause Settings Page](https://app.instaclause.be/accountant/settings) and go
to **Integration Settings**. For an office using AdminConsult, the key sits under its own
**Credentials** heading — there is no Custom APIs involvement.

### What you now have

- **One key for the whole office.** It is not per user and not per integration — the same
  key also authenticates the `custom` source, since the source is only a URL segment.
- **It does not expire.** It stays valid until someone presses Refresh.
- **You can come back for it.** Re-opening the page shows the *same* key, so this is not a
  one-time reveal.
- **It is long** — a few hundred characters. Make sure whatever stores it does not truncate it.

Treat it like a password: keep it in your secret manager, never commit it, never put it in
a URL.

> ⚠️ **Refresh breaks running integrations immediately.** There is no grace period. The
> moment anyone in the office presses **Refresh**, every previously issued key stops working
> and your sync fails until the new key is handed over. That is why the button asks for
> confirmation — and it is the most common cause of a sync that "suddenly broke".

If the key is not under **Credentials**, the office has Custom APIs enabled and it has moved
— see [the README](./README.md#getting-the-key) for all three cases.

---

<br>

## 🛡️ Authentication

Include the API key in the `Authorization` header of every request, as the **raw value** —
no `Bearer` prefix.

```
Authorization: <your API key>
```

A request fails with `401 Unauthorized` if the header is missing, the key is malformed or
not one Instaclause issued, the key has been superseded by a **Refresh**, or the Public API
is switched off for that office.

> ⚠️ **The `adminconsult` segment in the URL is case-sensitive.** Use it in lowercase, exactly as written. A capitalised variant still passes authentication and can return `201 Created` while the data is written nowhere — you would see successful responses and nothing appearing in Instaclause.

---

<br>

## 🛣️ Routes

**Order matters: post the customer before its sub-resources.**

```
  1. POST /customers                                  ← must land first
       │
       ├── 2. POST /customers/{id}/addresses
       └── 3. POST /customers/{id}/customerlinkcustomer
```

`/addresses` and `/customerlinkcustomer` attach to a customer that must already exist. A
sub-route call for a customer that has not been posted yet **still answers `201 Created`,
but stores nothing** — so a `201` on these two routes is not on its own proof that the
write landed. Always create the customer first, then its addresses and links.

### /customers

- Method: POST
- Endpoint: /customers
- Headers:
  - Authorization: [API Key]
- Body: the object for a single customer returned from AdminConsult API /customers
- Response
  - Success Status Code: 201 Created

Forward AdminConsult's own customer object as-is. These are the fields it is read for:

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

Instaclause reads only a few of these fields. `CustomerId` is required: it becomes the customer's id and is the id you use in the other two routes.

| Field | Used for |
| --- | --- |
| `CustomerId` | The customer's id (required) |
| `IsCompany` | Company or person |
| `Name`, `Firstname` | Company name, or the person's last and first name |
| `Title` | Company type (e.g. `BV`, `NV`, `V.Z.W.`, `CMP_BE_015`). Unknown values leave it empty |
| `RegistrationNr` | Business number |
| `RPR` | Jurisdiction of a company |
| `Email` | Email address |
| `SocialSecurityNumber`, `PlaceOfBirth`, `DateOfBirth` | Person only |

#### Example: a company

```json
{
  "CustomerId": 1001,
  "IsCompany": true,
  "Name": "Acme BV",
  "Title": "BV",
  "RegistrationNr": "0123456789",
  "RPR": "Brussel",
  "Email": "info@acme.be"
}
```

#### Example: a person

```json
{
  "CustomerId": 2001,
  "IsCompany": false,
  "Firstname": "Jan",
  "Name": "Peeters",
  "SocialSecurityNumber": "85.01.01-123.45",
  "PlaceOfBirth": "Gent",
  "DateOfBirth": "1985-01-01",
  "Email": "jan@example.be"
}
```

### /customers/{customerId}/addresses

Adds the address of a customer. Instaclause uses one address per customer: the one with `RegisteredOffice: true`, otherwise the first in the list.

- Method: POST
- Endpoint: /customers/{customerId}/addresses
- Headers:
  - Authorization: [API Key]
- Body: the **list** of addresses returned from AdminConsult API /customers/{customerid}/addresses. A single object, not in a list, is ignored.
- Response
  - Success Status Code: 201 Created

Fields that are read:

| Field | Meaning |
| --- | --- |
| `Street1` | Street. May include the number ("Kerkstraat 12 bus 3") |
| `HouseNr`, `HouseBox` | Number and box, used when `Street1` has no number |
| `ZipCode`, `City` | Postal code and city |
| `RegisteredOffice` | `true` marks the address to use |

Example for customer 1001:

`POST /customers/1001/addresses`

```json
[
  {
    "Street1": "Main Street",
    "HouseNr": "12",
    "HouseBox": "3",
    "ZipCode": "1000",
    "City": "Brussel",
    "RegisteredOffice": true
  }
]
```

### /customers/{customerId}/customerlinkcustomer

Links a company to the customers related to it: its directors, shareholders and contacts. Instaclause shows them as the company's **representatives** and **shareholders**.

- Method: POST
- Endpoint: /customers/{customerId}/customerlinkcustomer
- Headers:
  - Authorization: [API Key]
- Body: the list returned from AdminConsult API /customers/{customerid}/customerlinkcustomer
- Response
  - Success Status Code: 201 Created

How the links are read:

- `{customerId}` is the company. Each link's `CustomerIdFk` is the related customer, who must also be posted to `/customers`, otherwise the link is ignored.
- `CustomerLinkTypeId` sets the role: `4` director, `6` shareholder, `8` contact. Any other code takes its AdminConsult label (e.g. `74` Lasthebber); unknown codes become "Contact".
- A link with `StockOwned` above 0 also makes that customer a shareholder: `StockOwned` is the number of shares, `StockVotingNr` the number of votes.

#### Example

Acme BV (`CustomerId` 1001) with Jan Peeters (`CustomerId` 2001) as director and sole shareholder. Post both to `/customers` first, then:

`POST /customers/1001/customerlinkcustomer`

```json
[
  {
    "CustomerIdFk": 2001,
    "CustomerLinkTypeId": 4,
    "StockOwned": 100,
    "StockVotingNr": 100
  }
]
```

Instaclause then shows Acme BV with:

```json
{
  "relations": [
    { "id": "2001", "firstName": "Jan", "lastName": "Peeters", "function": "Bestuurder" }
  ],
  "shareholders": [
    { "id": "2001", "firstName": "Jan", "lastName": "Peeters", "numberOfShares": "100", "numberOfVotes": "100" }
  ]
}
```

---

<br>

## ❓ FAQ

### Does each office get its own API key?
Yes. Each office has its own unique API key, which identifies the office and authenticates the request. This is what keeps offices apart when one application uploads on behalf of several of them.

### Is syncing once a day enough?
Yes.

### Do I re-post everything, or only what changed?
For `/customers`, only the customers that changed. For `/customers/{customerId}/addresses` and `/customers/{customerId}/customerlinkcustomer`, always post the **full list**.

### If I re-post a customer, do I have to re-post its addresses and links too?
Yes. Posting to `/customers` overwrites the customer data, which means all addresses and `customerlinkcustomer` entries must be posted again afterwards.

### Can I delete a customer?
No. There is no DELETE route, so a client pushed once cannot be removed through the API — it stays available in the party picker. Decide up front how your sync should handle clients that leave the office.

### Our sync returns 201 but nothing shows up in Instaclause.
Two known causes: the source segment in the URL was not lowercase `adminconsult`, or addresses / links were posted for a customer that did not exist yet. See the notes under [Authentication](#-authentication) and [Routes](#-routes).
