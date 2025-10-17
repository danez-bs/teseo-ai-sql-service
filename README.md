# Specifiche Tecniche API

## Integrazione Teseo <-> AI-Sql-Microservice

**Versione:** 1.0  
**Data:** 17 Ottobre 2025

---

## Indice

1. [Introduzione](#introduzione)
2. [API AI-SQL-SERVICE](#api-ai-sql-service)
3. [API TESEO](#api-teseo)
4. [Strutture Dati JSON](#strutture-dati-json)
5. [Codici di Errore](#codici-di-errore)

---

## 1. Introduzione

Questo documento descrive le specifiche tecniche per l'integrazione tra i servizi **AI-SQL-SERVICE** e **TESEO**.  
Entrambi i servizi utilizzano il formato **JSON** per input e output, con autenticazione tramite **token JWT**.

### 1.1 Base URL

- **AI-SQL-SERVICE** → `https://api-ai-sql.example.com/api-rest/v1`
- **TESEO** → `https://api-teseo.example.com/api-rest/v1`

### 1.2 Autenticazione

Tutte le API richiedono un token di autenticazione nell’header HTTP:

```
Authorization: Bearer {token}
```

---

## 2. API AI-SQL-SERVICE

### 2.1 getSqlAI

Genera una query SQL a partire da una domanda in linguaggio naturale, utilizzando il contesto SQL e il codice entità forniti.

**Endpoint:** `POST /getSqlAI`

#### 2.1.1 Request

**Headers:**

```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer {token}"
}
```

**Body:**

```json
{
  "entity_code": "string",
  "sql_context": "string",
  "question": "string",
  "sql_teseo": "string (opzionale)"
}
```

| Campo         | Tipo   | Obbligatorio | Descrizione                                                 |
| ------------- | ------ | ------------ | ----------------------------------------------------------- |
| `entity_code` | string | Sì           | Codice identificativo dell’entità (es: `fatture`, `ordini`) |
| `sql_context` | string | Sì           | Contesto SQL (schema, tabelle, relazioni)                   |
| `question`    | string | Sì           | Domanda in linguaggio naturale                              |
| `sql_teseo`   | string | No           | Query SQL di riferimento generata da Teseo (TBD)            |

#### 2.1.2 Response

**Successo con risultati (200 OK):**

```json
{
  "success": true,
  "data": {
    "is_valid": true,
    "execution_time_ms": 45,
    "rows_count": 150,
    "result": [{ "id": 1 }, { "id": 2 }]
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

**Errore di validazione SQL (200 OK con `is_valid = false`):**

```json
{
  "success": true,
  "data": {
    "is_valid": false,
    "error": "Syntax error near 'FORM' at line 1",
    "error_code": "SQL_SYNTAX_ERROR",
    "result": []
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

**Errore di sistema (500 INTERNAL_SERVER_ERROR):**

```json
{
  "success": false,
  "error": {
    "code": "DATABASE_CONNECTION_ERROR",
    "message": "Impossibile connettersi al database",
    "details": "Timeout connessione dopo 30 secondi"
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

---

## 3. API TESEO

### 3.1 getSqlContext

Restituisce il contesto SQL associato a una specifica entità.

**Endpoint:** `POST /getSqlContext`

#### 3.1.1 Request

**Headers:**

```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer {token}"
}
```

**Body:**

```json
{
  "entity_code": "string"
}
```

| Campo         | Tipo   | Obbligatorio | Descrizione                       |
| ------------- | ------ | ------------ | --------------------------------- |
| `entity_code` | string | Sì           | Codice identificativo dell’entità |

#### 3.1.2 Response

**Successo (200 OK):**

```json
{
  "success": true,
  "data": {
    "entity_code": "fatture",
    "sql_context": "## Schema e Relazioni\n\n### Tabella: fatture\n- fatture.id (INTEGER): chiave primaria\n- Relazione: one-to-many con ai_fatture_righe..."
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

**Errore (404 ENTITY_NOT_FOUND):**

```json
{
  "success": false,
  "error": {
    "code": "ENTITY_NOT_FOUND",
    "message": "Entità non trovata",
    "details": "Il codice entità 'fatture_xyz' non esiste nel sistema"
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

---

### 3.2 testSql

Esegue un test di validazione di una query SQL verificandone la correttezza sintattica e logica.

**Endpoint:** `POST /testSql`

#### 3.2.1 Request

**Headers:**

```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer {token}"
}
```

**Body:**

```json
{
  "sql": "string"
}
```

| Campo | Tipo   | Obbligatorio | Descrizione          |
| ----- | ------ | ------------ | -------------------- |
| `sql` | string | Sì           | Query SQL da testare |

#### 3.2.2 Response

**Successo con risultati (200 OK):**

```json
{
  "success": true,
  "data": {
    "is_valid": true,
    "execution_time_ms": 45,
    "rows_count": 150,
    "result": [{ "id": 1 }, { "id": 2 }]
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

**Errore di validazione SQL (200 OK con `is_valid = false`):**

```json
{
  "success": true,
  "data": {
    "is_valid": false,
    "error": "Syntax error near 'FORM' at line 1",
    "error_code": "SQL_SYNTAX_ERROR",
    "result": []
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

**Errore di sistema (500 INTERNAL_SERVER_ERROR):**

```json
{
  "success": false,
  "error": {
    "code": "DATABASE_CONNECTION_ERROR",
    "message": "Impossibile connettersi al database",
    "details": "Timeout connessione dopo 30 secondi"
  },
  "timestamp": "2025-10-11T14:30:00Z"
}
```

---

## 4. Strutture Dati JSON

### 4.1 Struttura di Successo

```json
{
  "success": true,
  "data": { ... },
  "timestamp": "ISO-8601 UTC"
}
```

### 4.2 Struttura di Errore

```json
{
  "success": false,
  "error": {
    "code": "STRING",
    "message": "STRING",
    "details": "STRING (opzionale)"
  },
  "timestamp": "ISO-8601 UTC"
}
```

---

## 5. Codici di Errore

| Codice                      | HTTP Status | Descrizione                                  |
| --------------------------- | ----------- | -------------------------------------------- |
| `INVALID_TOKEN`             | 401         | Token di autenticazione non valido o scaduto |
| `MISSING_PARAMETER`         | 400         | Parametro obbligatorio mancante              |
| `INVALID_INPUT`             | 400         | Dati in input non validi                     |
| `ENTITY_NOT_FOUND`          | 404         | L'entità specificata non esiste              |
| `SQL_SYNTAX_ERROR`          | 200         | Errore di sintassi nella query SQL           |
| `DATABASE_CONNECTION_ERROR` | 500         | Errore di connessione al database            |
| `INTERNAL_SERVER_ERROR`     | 500         | Errore generico del server                   |

---

**Fine del documento**
