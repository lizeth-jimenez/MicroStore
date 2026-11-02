# Documentación de la API y Persistencia en la Nube - MicroStore

Esta documentación detalla los servicios de persistencia, la arquitectura asíncrona y la lógica de integración del sistema de gestión de inventarios **MicroStore** con la base de datos en la nube **Google Cloud Firestore**.

---

## Información General

* **Versión del Sistema:** 2.0.0 (Firestore Cloud Synchronized)
* **Formato de Datos:** Objetos JSON / Documentos NoSQL (Firestore)
* **Base de Datos:** Google Cloud Firestore
* **Autenticación y Seguridad:** Hashing de contraseñas mediante `hash()` y Control de Acceso Basado en Roles (RBAC).

---

## Colecciones de Firestore

El sistema interactúa principalmente con dos colecciones dentro de Firestore:

1. **`users`**: Almacena los perfiles de los usuarios del sistema (Nombre, Email, Rol, Hashed Password).
2. **`products`**: Almacena el catálogo de productos con sus especificaciones de inventario (ID interno, Nombre, Categoría, Unidad, Stock, Stock Mínimo, Precio).

---

## Módulos y Funciones Principales

### 1. Autenticación y Carga Inicial (`enter`)

#### `enter(userData)`
* **Descripción:** Autentica al usuario en el sistema, establece la sesión en el navegador y sincroniza el estado de la aplicación con la base de datos en la nube.
* **Operación en la Nube:**
  ```javascript
  // Carga de usuarios desde la nube
  const snapUsers = await firebase.firestore().collection('users').get();
  S.users = snapUsers.docs.map(doc => doc.data());

  // Carga de productos desde la nube
  const snapProds = await firebase.firestore().collection('products').get();
  if (!snapProds.empty) {
    S.prods = snapProds.docs.map(doc => doc.data());
  }
  ```

---

### 2. Gestión de Productos (`products`)

Las operaciones CRUD de productos se comunican de forma asíncrona directamente con la colección `products` en Firestore.

#### `saveProd(producto)` (Create / Update)
* **Descripción:** Crea un nuevo producto o actualiza uno existente en la base de datos de la nube.
* **Operación en la Nube:**
  * **Creación:** Agrega un nuevo documento a la colección `products`.
  * **Actualización:** Busca el documento en Firestore por el campo `id` y aplica las modificaciones necesarias mediante el método `.update()`.

#### `del(id)` (Delete)
* **Descripción:** Elimina un producto del catálogo y de la base de datos de la nube.
* **Restricción RBAC:** Exclusivo para el rol de **Administrador** (`can('borrar')`).
* **Operación en la Nube:**
  ```javascript
  const querySnapshot = await firebase.firestore().collection('products').where('id', '==', id).get();
  if (!querySnapshot.empty) {
    const docId = querySnapshot.docs[0].id;
    await firebase.firestore().collection('products').doc(docId).delete();
  }
  ```

---

### 3. Movimientos de Inventario (`addMov`)

#### `addMov(tipo)` (Entradas y Salidas)
* **Descripción:** Registra una entrada o salida de mercancía, calculando el nuevo stock y persistiendo el cambio tanto en la interfaz como en Firestore.
* **Restricción RBAC:** Permitido para roles con permisos de movimiento (`can('mov')`).
* **Operación en la Nube:**
  ```javascript
  // Modifica el stock local y consulta el producto en Firestore para sincronizar
  const querySnapshot = await firebase.firestore().collection('products').where('id', '==', pid).get();
  if (!querySnapshot.empty) {
    for (const doc of querySnapshot.docs) {
      await firebase.firestore().collection('products').doc(doc.id).update({ 
        stock: p.stock 
      });
    }
  }
  ```

---

### 4. Gestión de Usuarios (`saveNewUser`)

#### `saveNewUser()`
* **Descripción:** Permite a los administradores crear nuevos operadores con asignación de roles.
* **Seguridad:** Aplica de manera obligatoria el algoritmo de hashing `hash()` sobre la contraseña antes de guardarla en la colección `users` de Firestore.

---

## Resumen de Permisos por Rol (RBAC)

| Rol | Ver Inventario / Reportes | Entradas / Salidas | Crear / Editar Productos | Eliminar Productos / Usuarios |
| :--- | :---: | :---: | :---: | :---: |
| **Administrador** | ✅ | ✅ | ✅ | ✅ |
| **Empleado** | ✅ | ✅ | ✅ | ❌ |
| **Consulta** | ✅ | ❌ | ❌ | ❌ |
