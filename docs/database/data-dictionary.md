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