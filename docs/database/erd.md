# Diagrama entidad-relación — Abastece

Estado: diseño inicial de catálogo, identidades y acceso.
Los campos completos se describen en data-dictionary.md.

## 1. Catálogo y proveedores

```mermaid
erDiagram
    Categories ||--o{ Products : agrupa
    Products ||--o{ ProductSuppliers : tiene
    Suppliers ||--o{ ProductSuppliers : ofrece

    Categories {
        int Id PK
        nvarchar Name UK
        bit IsActive
    }

    Products {
        int Id PK
        int CategoryId FK
        nvarchar Sku UK
        nvarchar Name
        int MinimumStock
        int TargetStock
        char CountryOfOriginCode
        nvarchar ImageUrl
        bit IsActive
    }

    Suppliers {
        int Id PK
        nvarchar Name
        char CountryCode
        bit IsActive
    }

    ProductSuppliers {
        int Id PK
        int ProductId FK
        int SupplierId FK
        decimal UnitCost
        int LeadTimeDays
        int MinimumOrderQuantity
        datetime2 UpdatedAt
        bit IsActive
    }
```

### Restricciones adicionales

- ProductId + SupplierId es una combinación única.
- TargetStock debe ser mayor que MinimumStock.
- Los costos deben ser mayores que cero.
- Cada producto pertenece a una categoría.
- Un producto puede existir sin proveedores asociados.

## 2. Identidades y perfiles

```mermaid
erDiagram
    Actors ||--o| Users : identifica
    Actors ||--o| AgentProfiles : identifica

    Actors {
        int Id PK
        nvarchar ActorType
        nvarchar DisplayName
        datetime2 CreatedAt
    }

    Users {
        int Id PK
        int ActorId FK, UK
        nvarchar Email
        nvarchar NormalizedEmail UK
        nvarchar PasswordHash
        bit IsActive
        datetime2 CreatedAt
    }

    AgentProfiles {
        int Id PK
        int ActorId FK, UK
        bit IsEnabled
    }
```

### Restricciones adicionales

- Cada actor corresponde exactamente a un usuario o a un agente,
  nunca a ambos.
- ActorType debe coincidir con el perfil relacionado.
- El actor y su perfil se crean en una misma transacción.
- El diagrama muestra las relaciones individuales; la exclusividad
  y la existencia de un perfil requieren validaciones adicionales.

## 3. Asignación de roles

Users es la misma entidad presentada en el diagrama anterior.

```mermaid
erDiagram
    Users ||--o{ UserRoles : recibe
    Roles ||--o{ UserRoles : asigna

    Users {
        int Id PK
        int ActorId FK, UK
    }

    Roles {
        int Id PK
        nvarchar Name UK
    }

    UserRoles {
        int UserId PK, FK
        int RoleId PK, FK
    }
```

### Restricciones adicionales

- UserRoles tiene una clave primaria compuesta por UserId + RoleId.
- Un usuario puede tener cero o varios roles.
- Tener una cuenta sin roles no concede permisos de negocio.
- Los roles son predefinidos en la primera versión.

## 4. Saldo y apertura de inventario

```mermaid
erDiagram
    Products ||--|| InventoryBalances : tiene
    Products ||--o{ InventoryOpenings : registra
    Users ||--o{ InventoryOpenings : solicita

    Products {
        int Id PK
    }

    InventoryBalances {
        int ProductId PK, FK
        int QuantityOnHand
        nvarchar OpeningStatus
        rowversion RowVersion
    }

    InventoryOpenings {
        int Id PK
        int ProductId FK
        int InitialQuantity
        datetime2 CountedAt
        nvarchar Status
        int RequestedByUserId FK
        int ReviewedByUserId FK
        int CancelledByUserId FK
        rowversion RowVersion
    }

    Users {
        int Id PK
    }
```

### Restricciones adicionales

- Cada producto tiene exactamente una fila de saldo, creada
  en la misma transacción que el producto.
- Un producto puede tener varias solicitudes de apertura históricas.
- Solo puede tener una solicitud pendiente y una apertura aplicada
  como máximo.
- RequestedByUserId identifica al solicitante.
- ReviewedByUserId es opcional mientras no exista una revisión.
- CancelledByUserId es opcional y solo se informa al cancelar.
- Los tres campos de responsable referencian Users.Id.
- Para simplificar el dibujo, se muestra únicamente la relación
  del solicitante con Users.
- Los campos completos se describen en data-dictionary.md.

## 5. Conteos físicos

```mermaid
erDiagram
    Products ||--o{ StockCounts : tiene
    Users ||--o{ StockCounts : abre

    Products {
        int Id PK
    }

    Users {
        int Id PK
    }

    StockCounts {
        int Id PK
        int ProductId FK
        int ReferenceQuantity
        int CountedQuantity
        nvarchar Status
        int OpenedByUserId FK
        datetime2 OpenedAt
        int ConfirmedByUserId FK
        datetime2 ConfirmedAt
        int CancelledByUserId FK
        datetime2 CancelledAt
        rowversion RowVersion
    }
```

Los responsables de apertura, confirmación y cancelación
referencian Users.Id. Solo se dibuja la relación de apertura
para simplificar la vista.

Solo puede existir un conteo abierto por producto.
Los campos completos y restricciones están en el diccionario.

## 6. Ajustes de inventario

```mermaid
erDiagram
    Products ||--o{ InventoryAdjustments : tiene
    StockCounts o|--o{ InventoryAdjustments : fundamenta
    InventoryMovements o|--o{ InventoryAdjustments : origina
    Users ||--o{ InventoryAdjustments : solicita

    Products {
        int Id PK
    }

    StockCounts {
        int Id PK
        int ProductId FK
    }

    InventoryMovements {
        int Id PK
        int ProductId FK
    }

    Users {
        int Id PK
    }

    InventoryAdjustments {
        int Id PK
        int ProductId FK
        int StockCountId FK
        int OriginalMovementId FK
        int QuantityDelta
        nvarchar Status
        int RequestedByUserId FK
        int ReviewedByUserId FK
        int CancelledByUserId FK
        rowversion RowVersion
    }
```

### Restricciones adicionales

- Cada ajuste referencia exactamente un origen: conteo o movimiento.
- El origen debe corresponder al mismo producto del ajuste.
- Un conteo puede tener solicitudes históricas rechazadas o canceladas,
  pero como máximo una pendiente y una aplicada.
- Un movimiento puede tener varias correcciones, siempre respetando
  la cantidad máxima que puede compensarse.
- Las relaciones opcionales del dibujo no expresan por sí solas
  la exclusividad entre ambos orígenes.
- Solo se dibuja la relación del solicitante para simplificar.
  Revisor y responsable de cancelación también referencian Users.Id.
- InventoryMovements se presenta como referencia parcial;
  su estructura completa se documentará posteriormente.
```

## 7. Movimientos de inventario

```mermaid
erDiagram
    Products ||--o{ InventoryMovements : registra
    Actors ||--o{ InventoryMovements : realiza
    InventoryOpenings o|--o| InventoryMovements : origina
    InventoryAdjustments o|--o| InventoryMovements : origina

    Products {
        int Id PK
    }

    Actors {
        int Id PK
    }

    InventoryOpenings {
        int Id PK
    }

    InventoryAdjustments {
        int Id PK
        bigint OriginalMovementId FK
    }

    InventoryMovements {
        bigint Id PK
        int ProductId FK
        nvarchar MovementType
        int Quantity
        datetime2 OccurredAt
        datetime2 CreatedAt
        int CreatedByActorId FK
        int InventoryOpeningId FK
        int GoodsReceiptDetailId FK
        int InventoryIssueDetailId FK
        int InventoryAdjustmentId FK
        bigint ReversesMovementId FK
    }
```

### Restricciones adicionales

- Cada movimiento tiene un producto, un actor y exactamente
  un documento de origen.
- Los documentos de origen posibles son apertura, detalle
  de recepción, detalle de salida o ajuste.
- Cada documento de origen genera como máximo un movimiento.
- Una apertura con cero unidades no genera movimiento.
- Un ajuste aplicado genera exactamente un movimiento.
- Las relaciones con detalles de recepción y salida se dibujarán
  al incorporar esas entidades.
- OriginalMovementId identifica el movimiento que motivó
  una solicitud de corrección.
- ReversesMovementId identifica el movimiento compensado
  por una corrección aplicada.
- Las referencias al movimiento original y las restricciones
  entre tipos y orígenes se describen en el diccionario.
- Para simplificar esta vista, no se dibujan las relaciones
  al movimiento original.
```