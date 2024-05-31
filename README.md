# API Documentation

Base URL: `http://app.instaclause.be/api/v1/adminconsult`

## Authentication

### API Key

This API uses API keys to authenticate requests. You must include your API key in the `Authorization` header of each request. To obtain an API key, access [Instaclause Settings Page](http://app.instaclause.be/accountant/settings).
Each client get an unique API key, which is used to identify the user and authorize the requests.

## Routes

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

## FAQ

### How do you differentiate between clients if that application is going to upload to Instaclause from multiple locations? Does each client get a unique API key?
Each client has its unique API key, which is used to identify the user and also authenticate the requests.

### Frequency of sync once a day sufficient?
Yes.

### After the initial upload, should only the clients with changes be posted? Or a full run with all data each time (for customeraddress and customerlinkcustomer)?
For /customers/{customerId}/addresses and /customers/{customerId}/customerlinkcustomer requests the full list must be posted.

### Does this also apply to the customers-call which returns a List? (So always the complete list or only the customers containing changes)?
For requests to /customers, it's possible to only post customers with changes. This will overwrite the customer data, therefore all adresses and customerlinkcustomer needs to be posted again.
