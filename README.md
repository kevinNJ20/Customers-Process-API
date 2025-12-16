# Customers Process API

API Process qui orchestre la synchronisation des clients entre Core Banking, Global Data et Salesforce.

## Description

Cette API orchestre les opérations sur les clients en agrégeant et synchronisant les données entre plusieurs systèmes. Elle suit le pattern API-Led Connectivity comme Process API.

## Endpoints

### GET /api/process/customers
Récupère les clients depuis Global Data (vue unifiée).

**Query Parameters:**
- `globalId` (optional): ID global du client
- `limit` (optional, default: 100)
- `offset` (optional, default: 0)

### POST /api/process/customers
Synchronise un client vers tous les systèmes (Global Data, Core Banking, Salesforce).

**Body:** Customer object

### GET /api/process/customers/{customerGlobalId}
Récupère un client par son Global ID depuis Global Data.

## Configuration

### Connexions HTTP Requises

Cette API fait des appels HTTP vers:
- **Global Party System API** (port 8081)
- **Core Banking Customers System API** (port 8081)
- **Salesforce Customers System API** (port 8081)

Configurer dans `global.xml`:
- `Global_Party_System_API_Config`
- `Core_Banking_Customers_System_API_Config`
- `Salesforce_Customers_System_API_Config`

### Port

- **Port HTTP**: 8082

## Architecture Technique

### Flows Business-Logic

- `get-customers-business-logic`: Récupération depuis Global Data
- `sync-customer-business-logic`: Synchronisation multi-systèmes (parallel execution)
- `get-customer-by-global-id-business-logic`: Récupération par Global ID

### Stratégie de Synchronisation

Le flow `sync-customer-business-logic` utilise un **parallel** pour synchroniser simultanément vers:
1. Global Party System API (pour créer/mettre à jour la partie globale)
2. Core Banking Customers System API (si external ID CoreBanking existe)
3. Salesforce Customers System API (si external ID Salesforce existe)

## Exemples de Requêtes

### POST /api/process/customers (Sync)

```bash
curl -X POST http://localhost:8082/api/process/customers \
  -H "Content-Type: application/json" \
  -d '{
    "globalId": null,
    "customerNumber": "CUST001",
    "party": {
      "partyType": "Individual",
      "firstName": "John",
      "lastName": "Doe",
      "taxId": "123-45-6789"
    },
    "status": "Active",
    "externalIds": [{
      "system": "CoreBanking",
      "value": "CUST001"
    }]
  }'
```

