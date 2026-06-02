# Project 3: Serverless Application

## Architecture
```
Client (Browser/Mobile)
        │
        ▼
┌───────────────┐
│  API Gateway  │  ← REST API endpoints
│               │
│ GET /items    │──────┐
│ POST /items   │──┐   │
│ GET /items/:id│  │   │
└───────────────┘  │   │
                   │   │
            ┌──────▼───▼──────┐
            │     Lambda      │  ← Business logic
            │   Functions     │
            │                 │
            │ create_item()   │
            │ get_items()     │
            │ get_item()      │
            └────────┬────────┘
                     │
              ┌──────▼──────┐
              │  DynamoDB   │  ← NoSQL database
              │             │
              │ Items Table │
              │ PK: item_id │
              └─────────────┘

All deployed via Terraform
Zero servers to manage
Pay only for what you use
```

## What to Build
```
Serverless CRUD API:
├── Lambda functions (Python or Node.js)
│   ├── create_item
│   ├── list_items
│   └── get_item
├── API Gateway (REST API)
├── DynamoDB table
├── IAM roles for Lambda
├── Terraform for all infrastructure
├── GitHub Actions for deployment
└── README with API documentation + curl examples
```
