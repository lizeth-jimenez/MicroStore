# Documentación de la API - MicroStore

Esta documentación detalla los endpoints REST para la integración del sistema de gestión de inventarios **MicroStore** con la base de datos **MongoDB Atlas** (`microstore_db`).

---

## Información General

* **Versión de la API:** 1.0.0
* **Formato de datos:** JSON (`application/json`)
* **Base de Datos:** MongoDB Atlas (`microstore_db`)
* **Autenticación:** Api-Key / Bearer Token (para endpoints de modificación)

---

## Endpoints de la API

### 1. Productos (`/api/productos`)

#### `GET /api/productos`
Obtiene el listado completo de productos registrados en el inventario.

* **Respuesta Exitosa (200 OK):**
```json
[
  {
    "_id": "650c1f2e9b1d8f001c8e4a11",
    "id": 1695200000000,
    "nombre": "Café Colombiano Premium",
    "cat": "Bebidas",
    "uni": "bolsa",
    "st": 15,
    "min": 5,
    "pre": 25000
  }
]