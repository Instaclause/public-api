# Instaclause Public API

Push your client data into Instaclause so users can select it as a contract party instead
of typing it in by hand. Records posted through the API create companies (BV, CommV, NV,
etc.) and persons (shareholders, directors, etc.), stored in a collection that is **isolated
per office**. They do not mix with manually added CRM data, or with other CRM integrations
(HubSpot, Adminpulse, FID-manager, Tess, Exact Online, etc.) that may be enabled in parallel
in the same office.

<p align="center">
  <a href="./ADMINCONSULT.md"><strong>AdminConsult guide</strong></a> ·
  <a href="./CUSTOM.md"><strong>Custom guide</strong></a> ·
  <a href="#-two-ways-in--pick-one-before-you-write-any-code">Which one do I need?</a> ·
  <a href="#-test-account">Test account</a> ·
  <a href="#-authentication">Authentication</a>
</p>

### Terms used in these docs

| Term | Means |
|---|---|
| **office** | The accountancy firm using Instaclause. An API key belongs to exactly one office. |
| **client** / **customer** | A record you push — a company or a person. |
| **party** | A client once it has been selected into a contract. |
| **source** | `adminconsult` or `custom` — the URL segment that picks which API you are using. |

---

<br>

## 🔀 Two ways in — pick one before you write any code

They are separate APIs with different payloads, different routes and different setup steps.

```
                    Where does the client data come from?
                                     │
              ┌──────────────────────┴──────────────────────┐
              │                                             │
   AdminConsult / AdminIS                        Your own CRM, in-house
   (you forward AdminConsult's                   system, or another vendor
    API responses as-is)                         (you map the fields)
              │                                             │
              ▼                                             ▼
   ┌──────────────────────┐                    ┌──────────────────────┐
   │  adminconsult        │                    │  custom              │
   │                      │                    │                      │
   │  → ADMINCONSULT.md   │                    │  → CUSTOM.md         │
   └──────────────────────┘                    └──────────────────────┘
```

### 📘 [ADMINCONSULT.md — the AdminConsult API guide](./ADMINCONSULT.md)
For relaying AdminConsult / AdminIS data.

### 📗 [CUSTOM.md — the Custom API guide](./CUSTOM.md)
For **everything else**: your own CRM, an in-house system, or another vendor.

> 🛑 **These are not interchangeable.** Following the AdminConsult guide with custom-shaped
> data — or the reverse — will not work. If your data does not come from AdminConsult, you
> want **[CUSTOM.md](./CUSTOM.md)**.

### The difference at a glance

| | [`adminconsult`](./ADMINCONSULT.md) | [`custom`](./CUSTOM.md) |
|---|---|---|
| **For** | Relaying AdminConsult / AdminIS data | Any other source: your CRM, in-house system, another vendor |
| **Base URL** | `…/api/v1/adminconsult` | `…/api/v1/custom` |
| **Payload** | AdminConsult's own objects, forwarded unchanged | Instaclause's Party schema — **you do the mapping** |
| **Addresses & relations** | Separate sub-routes, posted after the customer | Embedded in the customer, one single call |
| **Turned on via** | *AdminConsult & AdminIS* toggle | *Custom APIs* toggle |
| **Scheduled pull to disable first** | Yes | Not applicable |
| **API key** | The same key — see below | The same key — see below |

---

<br>

## 🧪 Test account

You do not have to build against a live office. **Request a test account by emailing
[support@instaclause.com](mailto:support@instaclause.com)** and you can develop and test
your integration without touching real client data.

Mention which source you are integrating — `adminconsult` or `custom` — so the account is
set up with the right integration switched on.

---

<br>

## 🔑 Authentication

**This section applies to both APIs.** There is only one key, and the source is nothing
more than a segment in the URL.

Include the key in the `Authorization` header of every request, as the **raw value** — no
`Bearer` prefix.

```
Authorization: <your API key>
Content-Type: application/json
```

A request fails with **`401 Unauthorized`** — before anything is written — if the header is
missing, the key is malformed or not one Instaclause issued, the key has been superseded by
a **Refresh**, or the Public API is switched off for that office.

### Getting the key

Go to the [Instaclause Settings Page](https://app.instaclause.be/accountant/settings) →
**Integration Settings**. Where the key is displayed depends on the office's setup:

| Office setup | Where the key appears |
|---|---|
| Public API on, Custom APIs not enabled | Under its own **Credentials** heading |
| Custom APIs enabled **and** switched on | Inside the **Custom APIs** block |
| Custom APIs enabled but switched off | Not shown anywhere — switch the toggle on to reveal it |

**Step-by-step, with screenshots, is in the guide you need:**
[Custom](./CUSTOM.md#-getting-your-api-key) ·
[AdminConsult](./ADMINCONSULT.md#-getting-your-api-key)

### About the key

The key is a signed token (a JWT) issued per office. It is long — a few hundred characters
— so make sure whatever stores it does not truncate it.

| | |
|---|---|
| **Scope** | One key per **office**, granting read + write on that office's pushed client data — nothing else. There is no per-endpoint or read-only scoping, and no separate key per source. |
| **Expiry** | None. The key stays valid until it is refreshed. |
| **Re-showing it** | Safe. Opening the page regenerates the *same* key, so it is not a one-time reveal — an office can come back and copy it again. |
| **Rotation** | **Refresh** wipes the stored credentials and issues a new key. |

> ⚠️ **Refresh breaks running integrations immediately.** There is no grace period and no
> rotation window: the moment an office presses Refresh, every previously issued key stops
> working. That is why the button asks for confirmation. If an office refreshes, the new key
> has to be handed over before the sync will work again. This is the most common cause of a
> sync that "suddenly broke".

Treat the key like a password: store it in your secret manager, never commit it, never put
it in a URL.

> ⚠️ **The source segment in the URL is case-sensitive.** Use `adminconsult` / `custom` in
> lowercase, exactly as written. A capitalised variant still passes authentication and can
> return `201 Created` while the data is written nowhere — you would see successful
> responses and nothing appearing in Instaclause.

---

<br>

## 📋 Also true of both APIs

- **No DELETE route.** A client pushed once cannot be removed through the API. Decide up
  front how your sync should handle clients that leave the office.
- **Re-posting an `id` replaces the record**, it does not merge into it.
- **A daily sync is sufficient.** After the initial upload you can post only what changed.

---

<br>

## 👉 Where to go next

| I want to… | Read |
|---|---|
| Relay AdminConsult / AdminIS data | **[ADMINCONSULT.md](./ADMINCONSULT.md)** |
| Push from my own CRM or system | **[CUSTOM.md](./CUSTOM.md)** |
