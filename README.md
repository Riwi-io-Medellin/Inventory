# Inventory — Práctica de la capa Domain

Proyecto para practicar la **capa Domain** de Clean Architecture en **.NET 10**.

Una empresa tiene varias bodegas y registra la **entrada** y **salida** de productos. El dominio no tiene usuarios ni personas; eso se manejará más adelante con Identity.

## Cómo empezar

Requisitos: [.NET SDK 10](https://dotnet.microsoft.com/download).

```bash
git clone git@github.com:Riwi-io-Medellin/Inventory.git
# o por HTTPS
git clone https://github.com/Riwi-io-Medellin/Inventory.git

cd Inventory
dotnet build
```

## Estructura

```
src/
├── Inventory.Domain/           ← aquí se trabaja
├── Inventory.Application/      → referencia a Domain
├── Inventory.Infrastructure/   → referencia a Application
└── Inventory.Api/              → referencia a Application e Infrastructure
```

Solo se trabaja en **Domain**. Las demás capas están vacías para usarse más adelante. Domain no depende de ningún proyecto ni paquete NuGet.

### Carpetas de Domain

| Carpeta | Qué va |
|---|---|
| `Abstractions/` | Clase base `Entity`, `Result`, `Error` e interfaces que describen a la entidad (`IAuditable`, `ISoftDeletable`). |
| `Entities/` | Las entidades: `Producto`, `Bodega`, `ItemStock`. |
| `ValueObjects/` | Objetos inmutables que se comparan por valor: `Sku`, `Dinero`, `Direccion`. |
| `Enums/` | Estados: `EstadoProducto`, `EstadoBodega`. |
| `Errors/` | Errores esperados, una clase por entidad: `ProductoErrors`, `BodegaErrors`… |
| `Exceptions/` | `DomainException` (abstracta) y las excepciones que heredan de ella. |
| `Constants/` | Límites y valores fijos: longitudes máximas, formatos, etc. |

Los `.gitkeep` solo existen para que Git suba las carpetas vacías; bórralos cuando agregues código.

## Convenciones

- Carpetas en inglés; el código puede ir en español o inglés, pero sé consistente.
- **Encapsulamiento:** setters privados, constructores privados y métodos de fábrica estáticos que retornan `Result`.
- Las colecciones se exponen como `IReadOnlyCollection<T>`.
- Las interfaces de repositorios **no** van en Domain (irán en Application).

## Error vs. Exception

> **¿El que llama debería esperar este caso y manejarlo?** Sí → `Error`. No → `Exception`.

- **Error** (se retorna en un `Result`): falla esperada que el usuario puede provocar. Ej.: stock insuficiente.
- **Exception** (se lanza): invariante rota, es decir, un bug. Ej.: sumar dinero de monedas distintas.

## Enunciado

### Abstracciones

- `Entity`: `Id` (`Guid`); dos entidades son iguales si tienen el mismo `Id`.
- `Error`: `Code` y `Description`, más `Error.None`.
- `Result` / `Result<T>`: `IsSuccess`, `IsFailure`, `Error` y `Value`.
- `IAuditable`: `CreatedAt`, `UpdatedAt`.
- `ISoftDeletable`: `IsDeleted`, `DeletedAt`.

### Value objects

- `Sku`: obligatorio, en mayúsculas, solo letras, números y guiones, de 3 a 20 caracteres.
- `Dinero`: `Monto` no negativo y `Moneda` ISO de 3 letras (`COP`, `USD`).
- `Direccion`: `Calle`, `Ciudad` y `Pais` obligatorios.

### Entidades

**Producto** — `Nombre`, `Sku`, `PrecioUnitario` (`Dinero`), `Estado`. Auditable y con borrado lógico.

- `Crear`: nombre obligatorio, precio mayor que cero, nace `Activo`.
- `CambiarPrecio`, `Renombrar`, `Activar`, `Desactivar`, `Eliminar` (borrado lógico).
- Un producto eliminado no se puede modificar.

**Bodega** — `Nombre`, `Ubicacion` (`Direccion`), `Capacidad` (unidades), `Estado`, `Items`. Auditable. Es la raíz del agregado: el stock solo cambia a través de ella.

- `Crear`: nombre obligatorio, capacidad mayor que cero, nace `Activa` y sin ítems.
- `RegistrarEntrada(producto, cantidad)`: cantidad > 0, bodega activa, producto activo y no eliminado, sin superar la capacidad. Suma al ítem existente o crea uno nuevo.
- `RegistrarSalida(productoId, cantidad)`: cantidad > 0, bodega activa, el ítem existe y hay stock suficiente. Si queda en cero, el ítem se quita.
- `Activar`, `Desactivar` (no se puede desactivar con stock).

**ItemStock** — `BodegaId`, `ProductoId`, `Cantidad`. Solo `Bodega` lo crea y modifica. Si la cantidad quedara negativa es un bug → excepción.

## Checklist

- [ ] `dotnet build` compila sin errores.
- [ ] Sin setters ni constructores públicos en las entidades.
- [ ] Entidades y value objects se crean con métodos de fábrica que retornan `Result`.
- [ ] Errores en `Errors/` y excepciones heredando de `DomainException`.
- [ ] Sin números mágicos: los límites están en `Constants/`.

## Retos extra

- Registrar un historial de `MovimientoStock` (entrada/salida, cantidad, fecha).
- Transferir stock entre bodegas.
- Agregar pruebas unitarias con xUnit.

---

## Autor

**Javier Cómbita Téllez**

[![GitHub](https://img.shields.io/badge/GitHub-jcomte23-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/jcomte23)\
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Javier_C%C3%B3mbita-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/javier-c%C3%B3mbita-t%C3%A9llez-4b4aa3258)\
[![Web](https://img.shields.io/badge/Web-javiercombita.pro-512BD4?style=for-the-badge&logo=googlechrome&logoColor=white)](https://javiercombita.pro)
