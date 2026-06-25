# API Documentation — Custom source

> Part of the [Instaclause Public API](./README.md). This page documents the **`custom`** source, used to push records that are already structured in Instaclause's own format. For the AdminConsult integration, see the [main README](./README.md).

Base URL: `https://app.instaclause.be/api/v1/custom`

## Authentication

### API Key

This API uses API keys to authenticate requests. You must include your API key in the `Authorization` header of each request, as the **raw value** (no `Bearer` prefix). To obtain an API key, access the [Instaclause Settings Page](https://app.instaclause.be/accountant/settings).
Each office gets a unique API key, which identifies the office and authorizes the requests.

## Routes

### /customers

- Method: POST
- Endpoint: `/customers`
- Headers:
  - `Authorization`: [API Key]
  - `Content-Type`: `application/json`
- Body: a single **Party** object **or an array** of Party objects (batch).
- Response
  - Success Status Code: `201 Created`
  - Body: `{ "response": "OK", "count": <number of records> }`

Each record **must** include an `id` (used as the unique identifier; `CustomerId` is also accepted). Posting a record with an existing `id` overwrites it.

#### Party schema

`type`: `"person"` or `"company"`.

Common fields: `id` (required), `street`, `number`, `zipCode`, `city`, `businessNumber`, `email`.

**Person** also has: `firstName`, `lastName`, `initials` (optional), `documentNumber`, `placeOfBirth`, `dateOfBirth` (format `DD/MM/YYYY`).

**Company** also has: `name`, `companyType`, `jurisdiction`, `website` (optional), `relations` (optional), `shareholders` (optional).
- `companyType` ∈ `BV, NV, CV, CommV, VOF, VZW, Stichting, SRL, SA, SC, SComm, SNC, ASBL, Fondation, Vereniging`.
- `relations[]` — the company's representatives. Each is a Party (`person` or `company`) plus a `function` (e.g. `"Bestuurder"`).
- `shareholders[]` — each is a Party plus `numberOfShares` and `numberOfVotes`.

Representatives and shareholders are **embedded** in the company object — there are no sub-routes to call (the payload is self-contained).

<details>
  <summary>Example — Person</summary>

```json
{
  "id": "P-1001",
  "type": "person",
  "firstName": "Jan",
  "lastName": "Peeters",
  "documentNumber": "90010112345",
  "placeOfBirth": "Gent",
  "dateOfBirth": "12/01/1990",
  "street": "Kerkstraat",
  "number": "10",
  "zipCode": "9000",
  "city": "Gent",
  "businessNumber": "",
  "email": "jan.peeters@example.be"
}
```

</details>

<details>
  <summary>Example — Company (with representative + shareholder)</summary>

```json
{
  "id": "C-2002",
  "type": "company",
  "name": "Acme BV",
  "companyType": "BV",
  "jurisdiction": "Brussel",
  "street": "Main Street",
  "number": "12",
  "zipCode": "1000",
  "city": "Brussel",
  "businessNumber": "0123456789",
  "email": "info@acme.be",
  "website": "https://acme.be",
  "relations": [
    {
      "type": "person",
      "id": "P-1001",
      "firstName": "Jan",
      "lastName": "Peeters",
      "function": "Bestuurder"
    }
  ],
  "shareholders": [
    {
      "type": "person",
      "id": "P-1001",
      "firstName": "Jan",
      "lastName": "Peeters",
      "numberOfShares": "100",
      "numberOfVotes": "100"
    }
  ]
}
```

</details>

<details>
  <summary>Example — Batch (array)</summary>

```json
[
  {
    "id": "P-1001",
    "type": "person",
    "firstName": "Jan",
    "lastName": "Peeters",
    "street": "Kerkstraat", "number": "10", "zipCode": "9000", "city": "Gent",
    "email": "jan.peeters@example.be"
  },
  {
    "id": "C-2002",
    "type": "company",
    "name": "Acme BV",
    "companyType": "NV",
    "street": "Main Street", "number": "12", "zipCode": "1000", "city": "Brussel",
    "businessNumber": "0123456789"
  }
]
```

</details>

### /customers (read back)

- Method: GET
- Endpoint: `/customers?id={id}`
- Headers:
  - `Authorization`: [API Key]
- Response
  - `200 OK` → `{ "response": { ...the stored record... } }`
  - `404 Not Found` if the `id` does not exist.

## Limits

- Maximum request size is ~10 MB and the default timeout is 60s. For large datasets, send the records in chunks (e.g. **500–2000 records per request**) — the endpoint accepts arrays, so just repeat the call per chunk.

## FAQ

### How do you differentiate between clients if data is uploaded from multiple locations? Does each client get a unique API key?
Each office has its own unique API key, which identifies the office and authenticates the request.

### How often should I sync?
Once a day is sufficient. After the initial upload you can post only the records that changed — a record is overwritten by its `id`.

### Do I need to post representatives and shareholders separately?
No. Embed them in the company's `relations` and `shareholders` arrays. The Custom payload is self-contained, so there are no sub-routes (unlike the AdminConsult source).

### What date format should I use for `dateOfBirth`?
`DD/MM/YYYY` (ISO `YYYY-MM-DD` is also accepted).
