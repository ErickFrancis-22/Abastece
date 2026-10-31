# Abastece

Sistema empresarial de inventario y operaciones para una distribuidora
de repuestos y suministros, con un agente de IA para apoyar el
abastecimiento.

> Estado: en desarrollo. Actualmente en la etapa de definición del
> negocio y preparación de la base técnica.

## Propósito

Centralizar el control de existencias, proveedores y compras, y facilitar
la detección y atención de riesgos de desabastecimiento.

El proyecto busca combinar una operación de inventario consistente con
un agente que consulte información, recupere políticas internas y cree
solicitudes de reposición mediante herramientas autorizadas.

## Alcance de la primera versión

- Una empresa y una bodega.
- Productos y categorías.
- Proveedores y condiciones de suministro.
- Entradas, salidas e historial de inventario.
- Conteos físicos y ajustes sujetos a aprobación.
- Solicitudes de reposición y órdenes de compra.
- Recepciones parciales o completas.
- Usuarios con múltiples roles.
- Alertas y seguimiento de operaciones.
- Agente de abastecimiento con autonomía limitada.
- Recuperación de documentos mediante RAG.
- Auditoría de acciones de usuarios y del agente.

Las compras y los proveedores utilizados en la demostración serán
simulados. No se realizarán pagos ni pedidos externos reales.

## Recorrido principal previsto

1. El encargado registra una salida de productos.
2. El sistema detecta existencias bajas.
3. El agente consulta existencias, reposiciones abiertas y proveedores.
4. Recupera las políticas relevantes y crea una solicitud justificada.
5. El gerente revisa y aprueba la solicitud.
6. Se genera y emite una orden de compra simulada.
7. El encargado registra la recepción.
8. El sistema actualiza las existencias y conserva la trazabilidad.

## Roles

| Rol | Responsabilidad principal |
|---|---|
| Administrador | Gestionar usuarios, roles y configuración. |
| Gerente | Aprobar compras y ajustes; supervisar la operación. |
| Encargado de inventario | Gestionar catálogos, movimientos, recepciones y conteos. |

Un usuario puede tener varios roles.

El agente utiliza una identidad propia y permisos específicos.
Inicialmente no puede aprobar compras ni ajustes, emitir órdenes
o confirmar recepciones.

## Tecnologías previstas

| Área | Tecnología |
|---|---|
| Backend | C# y ASP.NET Core |
| Persistencia | Entity Framework Core |
| Base de datos operativa | SQL Server |
| Autenticación | JWT y autorización por roles |
| IA | Modelo local con herramientas y recuperación de documentos |
| Entorno reproducible | Docker |
| Control de versiones | Git y GitHub |

Las versiones, el frontend y la implementación del índice vectorial
se definirán durante la preparación técnica.

## Organización prevista

- `backend/`: solución .NET y pruebas.
- `frontend/`: panel web.
- `database/`: scripts SQL complementarios y datos de demostración.
- `docs/`: documentación del negocio, arquitectura y API.
- `infra/`: configuración de infraestructura y contenedores.

Las carpetas se incorporarán conforme tengan contenido.
Las migraciones de EF Core vivirán en el proyecto de infraestructura
del backend.

## Documentación

- [Alcance del proyecto](docs/business/scope.md)
- [Reglas de negocio](docs/business/business-rules.md)
- [Plan de la semana 1](docs/business/week-01-plan.md)

## Ejecución

La aplicación todavía no está implementada.

Las instrucciones para configurar SQL Server, aplicar migraciones
y ejecutar la API se incorporarán junto con la base técnica.

## Objetivo de aprendizaje

Construir y comprender una aplicación completa que permita practicar:

- Diseño de dominio y programación orientada a objetos.
- APIs REST, validación y manejo de errores.
- SQL, transacciones y concurrencia.
- Autenticación y autorización.
- Pruebas, logging y ejecución reproducible.
- Agentes de IA, herramientas, RAG y aprobación humana.

## Autor

Erick Francis Hernández Angel