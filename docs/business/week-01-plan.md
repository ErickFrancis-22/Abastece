# Plan de trabajo — Semana 1

## Objetivo

Definir la base del negocio y construir una API inicial conectada
a SQL Server, con autenticación y gestión de productos y proveedores.

## Disponibilidad estimada

- Entre 4 y 5 horas diarias.
- Entre 28 y 35 horas durante la semana.
- El tiempo incluye estudio, implementación, pruebas y correcciones.

## Día 1 — Alcance y organización

### Actividades

- Definir el propósito y alcance de la primera versión.
- Documentar roles y reglas de negocio.
- Preparar el README y el plan semanal.
- Configurar las exclusiones de Git.
- Realizar el commit inicial.

### Entregables

- README.md
- docs/business/scope.md
- docs/business/business-rules.md
- docs/business/week-01-plan.md
- .gitignore

### Criterio de terminado

La documentación permite explicar el problema, los usuarios,
el recorrido principal y los límites de la primera versión.
El repositorio contiene el primer commit publicado.

## Día 2 — Modelo de datos

### Actividades

- Identificar entidades y relaciones.
- Definir claves primarias, foráneas y restricciones.
- Modelar usuarios, roles, productos, categorías y proveedores.
- Diseñar inicialmente movimientos, conteos, ajustes, solicitudes,
  órdenes y recepciones.
- Recorrer ejemplos para comprobar el modelo.

### Entregables

- DER inicial.
- Diccionario de datos inicial.

### Criterio de terminado

El modelo permite representar una compra con recepción parcial,
un conteo con diferencia y un usuario con varios roles.
Se distingue qué tablas se implementan esta semana.

## Día 3 — Solución y persistencia

### Actividades

- Revisar el entorno de desarrollo y elegir versiones compatibles.
- Crear la solución y sus proyectos.
- Configurar las referencias entre proyectos.
- Configurar Entity Framework Core y SQL Server.
- Crear la primera migración para las entidades de esta semana.
- Configurar credenciales fuera del repositorio.

### Entregables

- Solución que compila.
- API que arranca.
- Primera migración aplicada.
- Instrucciones iniciales de configuración.

### Criterio de terminado

Una base vacía puede prepararse mediante migraciones.
La aplicación verifica su conexión con SQL Server.

## Día 4 — Productos y categorías

### Actividades

- Implementar registro, consulta y actualización.
- Incorporar desactivación de productos.
- Definir DTOs y validaciones.
- Incorporar filtros y paginación básica.
- Establecer respuestas consistentes de error.

### Entregables

- Endpoints de productos y categorías.
- Ejemplos de peticiones.
- Verificación de restricciones principales.

### Criterio de terminado

Se puede registrar y consultar un producto.
Se rechazan SKU duplicados y datos inválidos.
Todavía no se modifica inventario desde estos endpoints.

## Día 5 — Proveedores

### Actividades

- Implementar registro, consulta, actualización y desactivación.
- Asociar proveedores con productos.
- Registrar costo, plazo de entrega y compra mínima.
- Validar asociaciones y condiciones de suministro.

### Entregables

- Endpoints de proveedores.
- Endpoints de condiciones por producto y proveedor.
- Ejemplos de peticiones.

### Criterio de terminado

Un producto puede tener varios proveedores.
No se admiten asociaciones duplicadas ni condiciones inválidas.

## Día 6 — Autenticación y autorización

### Actividades

- Implementar usuarios y roles.
- Proteger contraseñas mediante un mecanismo estándar de hashing.
- Implementar login y emisión de JWT.
- Aplicar permisos a los endpoints existentes.
- Preparar un mecanismo seguro de creación del administrador inicial.

### Entregables

- Autenticación funcional.
- Endpoints protegidos.
- Asignación de varios roles a un usuario.
- Configuración de desarrollo documentada.

### Criterio de terminado

Las peticiones sin autenticación reciben 401.
Los usuarios autenticados sin permiso reciben 403.
No existe registro público de administradores.
No se publican credenciales ni claves de firma.

## Día 7 — Integración y cierre

### Actividades

- Ejecutar pruebas de integración prioritarias.
- Corregir errores encontrados.
- Revisar migraciones y configuración.
- Completar instrucciones de ejecución.
- Revisar y fusionar los cambios terminados a main.

### Entregables

- Pruebas de autenticación, permisos y restricciones del catálogo.
- README actualizado.
- Ejemplos reproducibles de uso de la API.
- Versión estable del avance semanal.

### Criterio de terminado

Se puede preparar la base de datos, iniciar la API, autenticarse
y gestionar productos y proveedores siguiendo la documentación.

## Forma de trabajo con Git

- El commit inicial de documentación se publica en main.
- Los cambios posteriores se desarrollan en ramas cortas por tarea.
- Cada rama se integra mediante un pull request.
- Cada commit representa un cambio coherente.
- No se publican secretos, archivos de compilación ni datos privados.

## Prioridades si hay retrasos

1. Conservar el alcance y las reglas documentadas.
2. Conseguir una API conectada a SQL Server.
3. Completar productos, proveedores y sus validaciones.
4. Completar autenticación y permisos antes de considerar
   el avance listo para utilizarse.
5. Reducir filtros secundarios y detalles de presentación.

Las tareas incompletas se registran y se replanifican.
No se presentan como terminadas.

## Fuera de esta semana

- Operaciones completas de inventario y compras.
- Agente de IA y herramientas.
- RAG e índice vectorial.
- Panel web.
- Entorno completo con Docker.

Estas capacidades forman parte de las siguientes etapas del proyecto.

## Seguimiento

- [ ] Día 1: alcance y organización.
- [ ] Día 2: modelo de datos.
- [ ] Día 3: solución y persistencia.
- [ ] Día 4: productos y categorías.
- [ ] Día 5: proveedores.
- [ ] Día 6: autenticación y autorización.
- [ ] Día 7: integración y cierre.