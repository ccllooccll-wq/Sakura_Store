# Guía de Plantilla CRUD Reutilizable — Clean Architecture & POO (Sakura Store)

Documento explicativo para el equipo de desarrollo (**Derik Bernabe, Marcelo Calle, Alvaro Sillerico, Ronny Mamani**) para replicar la arquitectura base establecida en el **Módulo de Usuario** hacia los demás módulos de la tienda (Productos, Categorías, Proveedores, Ventas, Clientes).

---

## 🏛️ Flujo Arquitectónico por Capas (Clean Architecture)

```
[ Frontend Angular ]
        │ HTTP REST (JSON / DTOs)
        ▼
[ Presentation Layer ] (Controller + DTO Request/Response + ExceptionHandler)
        │ Invoca Caso de Uso
        ▼
[ Application Layer ]  (Use Cases / Interactors + Input Commands)
        │ Opera sobre el Dominio e Invoca Puertos (Interfaces)
        ▼
[ Domain Layer ]       (Entidades POO Puras + Reglas de Negocio + Ports)
        ▲ Implements Ports
        │
[ Infrastructure Layer ] (Adapter + JPA Entity + Spring Data Repository + PostgreSQL)
```

---

## 📋 Pasos para Crear un Nuevo Módulo (Ejemplo: `Producto`)

### 1. Capa de Dominio (`domain`)
Ubicación: `backend/src/main/java/com/sakurastore/backend/domain`
- **Paso 1.1:** Crear la Entidad POO Pura (ej: `Producto.java` en `domain/model/`).
  - Atributos privados (`id`, `name`, `code`, `price`, `stock`, `active`).
  - Constructor con validación de invariantes de negocio (ej. `price > 0`, `stock >= 0`, `name` no vacío).
  - Métodos de negocio explicitos (ej. `aumentarStock(int cantidad)`, `reducirStock(int cantidad)`, `desactivar()`).
- **Paso 1.2:** Crear la interfaz de Puerto de Salida en `domain/port/`:
  - `ProductoRepositoryPort.java` (`save`, `findById`, `findAll`, `existsByCode`, etc.).

### 2. Capa de Aplicación (`application`)
Ubicación: `backend/src/main/java/com/sakurastore/backend/application`
- **Paso 2.1:** Crear DTOs de entrada y salida en `application/dto/` (`CreateProductoCommand`, `UpdateProductoCommand`, `ProductoResponseDto`).
- **Paso 2.2:** Crear Casos de Uso en `application/usecase/`:
  - `CreateProductoUseCase.java`
  - `UpdateProductoUseCase.java`
  - `ListProductosUseCase.java`
  - `ToggleProductoStatusUseCase.java`

### 3. Capa de Infraestructura (`infrastructure`)
Ubicación: `backend/src/main/java/com/sakurastore/backend/infrastructure`
- **Paso 3.1:** Crear la entidad JPA en `infrastructure/persistence/entity/` (`ProductoJpaEntity.java`).
- **Paso 3.2:** Crear la interfaz `SpringDataProductoRepository.java` extendiendo `JpaRepository`.
- **Paso 3.3:** Crear el mapper `ProductoPersistenceMapper.java` (convierte entre `Producto` POO y `ProductoJpaEntity`).
- **Paso 3.4:** Crear `ProductoRepositoryAdapter.java` implementando `ProductoRepositoryPort`.
- **Paso 3.5:** Registrar los beans de Casos de Uso en `infrastructure/config/UseCaseConfig.java`.

### 4. Capa de Presentación (`presentation`)
Ubicación: `backend/src/main/java/com/sakurastore/backend/presentation`
- **Paso 4.1:** Crear `ProductoController.java` en `presentation/controller/` exponiendo endpoints REST (`/api/products`).

### 5. Frontend en Angular (`frontend/src/app`)
- **Paso 5.1:** Crear interfaz del modelo en `core/models/producto.model.ts`.
- **Paso 5.2:** Crear servicio en `core/services/producto.service.ts`.
- **Paso 5.3:** Crear componente en `features/inventory/producto-list.component.ts`.

---

## ⚖️ Reglas de Oro del Proyecto
1. **Dominio Puro:** `domain/` NUNCA debe importar paquetes de Spring (`org.springframework.*`) ni de JPA (`jakarta.persistence.*`).
2. **Encapsulamiento:** No usar setters públicos indiscriminados; preferir métodos de negocio que validen estados e invariantes.
3. **Manejo de Errores:** Lanzar `DomainException` ante fallos de reglas de negocio para ser capturadas por `GlobalExceptionHandler`.
