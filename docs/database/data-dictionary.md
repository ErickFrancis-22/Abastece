# Diccionario de datos — Abastece

Diseño inicial. Las tablas aún no han sido implementadas

## Convenciones

- Los nombres de tablas y campos se escriben en inglés.
- PK significa clave primaria.
- FK significa clave foránea.
- Los identificadores int se generan mediante IDENTITY.
- Los textos utilizan nvarchar, excepto los códigos de país.
- Los importes utilizan decimal, no float.
- Las fechas se guardan en UTC mediante datetime2.
- Los códigos de país utilizan dos letras.
- Las relaciones con historial no se eliminan en cascada.

## 1. Categories

Agrupa productos del catálogo.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(100) | No | Nombre único y no vacío. |
| IsActive | bit | No | Indica si admite nuevas asignaciones. |

Una categoría puede tener muchos productos.
Cada producto pertenece a una categoría.

Desactivar una categoría no desactiva automáticamente sus productos
ni elimina sus relaciones existentes.

## 2. Products

Representa un artículo específico del catálogo.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| CategoryId | int | No | FK hacia Categories.Id. |
| Sku | nvarchar(50) | No | Único y no vacío. |
| Name | nvarchar(150) | No | Nombre no vacío. |
| Description | nvarchar(500) | Sí | Descripción adicional. |
| MinimumStock | int | No | Mayor o igual que cero. |
| TargetStock | int | No | Mayor que MinimumStock. |
| CountryOfOriginCode | char(2) | Sí | País de fabricación, si se conoce. |
| ImageUrl | nvarchar(500) | Sí | Referencia a una imagen. |
| IsActive | bit | No | Habilita el producto para nuevas operaciones. |

El producto no almacena el costo de cada proveedor.

Las existencias, la apertura y los movimientos se modelan por separado.

En esta versión, cada SKU representa un producto con un origen definido,
si se conoce. No se rastrea el país por lote.

## 3. Suppliers

Representa una empresa que suministra productos.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(150) | No | Nombre no vacío. |
| CountryCode | char(2) | No | País donde está ubicado el proveedor. |
| ContactName | nvarchar(150) | Sí | Persona de contacto. |
| Email | nvarchar(254) | Sí | Correo de contacto. |
| Phone | nvarchar(30) | Sí | Teléfono; se trata como texto. |
| IsActive | bit | No | Habilita al proveedor para nuevas compras. |

El nombre del proveedor no se considera un identificador único.

El país del proveedor no determina el país de fabricación del producto.

## 4. ProductSuppliers

Relaciona productos y proveedores, y conserva sus condiciones
actuales de suministro.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ProductId | int | No | FK hacia Products.Id. |
| SupplierId | int | No | FK hacia Suppliers.Id. |
| UnitCost | decimal(18,2) | No | Costo unitario en USD, mayor que cero. |
| LeadTimeDays | int | No | Plazo estimado en días calendario, no negativo. |
| MinimumOrderQuantity | int | No | Entero mayor que cero. |
| UpdatedAt | datetime2 | No | Última actualización de las condiciones, en UTC. |
| IsActive | bit | No | Indica si la condición de suministro está vigente. |

### Restricciones

- La combinación ProductId + SupplierId es única.
- La compra mínima no implica múltiplos de empaque.
- Un plazo de cero representa entrega estimada el mismo día.
- Desactivar una relación conserva su historial.
- Si vuelve a utilizarse la misma combinación, se reactiva el registro
  existente en lugar de crear otro.

### Historial de condiciones

Esta tabla contiene las condiciones actuales de referencia.

Las solicitudes y órdenes conservan sus propias condiciones históricas.
Los cambios relevantes del catálogo también se registrarán en auditoría.

Actualizar UnitCost no modifica solicitudes ni órdenes anteriores.

## 5. Actors

Identifica a las personas y agentes responsables de acciones.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorType | nvarchar(20) | No | User o Agent. |
| DisplayName | nvarchar(150) | No | Nombre visible, no vacío. |
| CreatedAt | datetime2 | No | Fecha de creación en UTC. |

Cada actor corresponde exclusivamente a un usuario o a un agente,
según ActorType.

La identidad permite atribuir acciones, pero no concede permisos.

Los actores con historial no se eliminan físicamente.

## 6. Users

Contiene los datos de acceso de las personas.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorId | int | No | FK hacia Actors.Id; único. |
| Email | nvarchar(254) | No | Correo de acceso. |
| NormalizedEmail | nvarchar(254) | No | Correo normalizado; único. |
| PasswordHash | nvarchar(max) | No | Hash generado por el mecanismo de autenticación. |
| IsActive | bit | No | Permite o bloquea el acceso. |
| CreatedAt | datetime2 | No | Fecha de creación en UTC. |

### Restricciones

- El actor relacionado debe tener ActorType = User.
- El usuario y su actor se crean en una misma transacción.
- La normalización del correo se realiza mediante una política
  consistente de la aplicación.
- La contraseña original no se almacena ni se registra en logs.
- Desactivar al usuario no elimina sus acciones anteriores.
- Los permisos se obtienen de sus asignaciones de roles vigentes.

## 7. Roles

Define los roles disponibles en la primera versión.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| Name | nvarchar(50) | No | Nombre único. |

Roles iniciales:

- Administrator: usuarios, asignaciones y configuración.
- Manager: aprobación de compras y ajustes; supervisión.
- InventoryOperator: catálogos y operación de inventario.

La interfaz puede mostrar los nombres en español.

Los roles son predefinidos. La primera versión permite asignarlos
y retirarlos, pero no crear roles ni editar sus permisos.

Administrator no incluye automáticamente los otros roles.

## 8. UserRoles

Asigna roles a usuarios.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| UserId | int | No | FK hacia Users.Id; parte de la PK. |
| RoleId | int | No | FK hacia Roles.Id; parte de la PK. |

### Restricciones

- Clave primaria compuesta: UserId + RoleId.
- Un usuario puede tener varios roles.
- Un rol puede estar asignado a varios usuarios.
- Retirar una asignación no elimina al usuario ni al rol.
- Las asignaciones y retiradas se registran en auditoría.
- Un administrador no puede modificar sus propias asignaciones.
- No se puede retirar el último administrador activo.

## 9. AgentProfiles

Representa la configuración básica de una identidad de agente.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| Id | int | No | PK, IDENTITY. |
| ActorId | int | No | FK hacia Actors.Id; único. |
| IsEnabled | bit | No | Permite o impide nuevas acciones del agente. |

### Restricciones

- El actor relacionado debe tener ActorType = Agent.
- El agente no tiene correo ni contraseña de usuario humano.
- Sus herramientas y permisos se controlan desde la aplicación.
- Deshabilitarlo conserva sus casos, solicitudes y ejecuciones.
- Su habilitación y permisos se verifican antes de cada herramienta.

La configuración del modelo y sus credenciales se gestionan
por separado; no se guardan como datos públicos del perfil.

## 10. InventoryBalances

Conserva la existencia registrada actual y el estado de apertura
del inventario de cada producto.

| Campo | Tipo SQL Server | Permite NULL | Restricciones / descripción |
|---|---|---|---|
| ProductId | int | No | PK y FK hacia Products.Id. No es IDENTITY. |
| QuantityOnHand | int | No | Existencia registrada, mayor o igual que cero. |
| OpeningStatus | nvarchar(20) | No | Pending o Opened. |
| RowVersion | rowversion | No | Valor generado por SQL Server para detectar cambios concurrentes. |

### Restricciones

- Al crear un producto se crea su saldo en la misma transacción,
  con cantidad cero y apertura Pending.
- ProductId como clave primaria impide dos saldos del mismo producto.
- Mientras la apertura esté pendiente, no se permiten movimientos
  operativos ni alertas de reposición.
- La aprobación de la apertura cambia OpeningStatus a Opened.
- Después de la apertura, el saldo solo cambia mediante movimientos
  autorizados y registrados en la misma transacción.
- Las cantidades pendientes de recibir no forman parte de QuantityOnHand.
- RowVersion no representa una fecha ni una cantidad de movimientos.