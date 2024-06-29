# Sandy CRM 🥐 

> Sandy is the personal assistant for small to medium consulting companies

Built with a modern stack, cloud ready, self hostable

```mermaid
graph TD
  subgraph "User Interface"
    User --> ReactFrontEnd[React Front End]
  end

  subgraph "Middleware"
    ReactFrontEnd --> APIGateway[API Gateway]
    APIGateway --> CloudFunctions[Cloud Functions]
  end

  subgraph "Back End"
    CloudFunctions --> ApacheIcebergStorage[Apache Iceberg Storage]
  end
```