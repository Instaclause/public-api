# API Documentation

Base URL: `http://app.instaclause.be/api/v1/adminconsult`

## Authentication

### API Key

This API uses API keys to authenticate requests. You must include your API key in the `Authorization` header of each request. To obtain an API key, access [Instaclause Settings Page](http://app.instaclause.be/accountant/settings)

## Routes

### /customers

- Method: POST
- Endpoint: /customers
- Headers:
  - Authorization: [API Key]
- Body: the object returned from AdminConsult API /customers
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
