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