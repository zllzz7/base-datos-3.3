# Tarea de PostgreSQL / pgAdmin 4 — Base de Datos `accommodations_tourism`

**Autor:** Kevin Alvarez

## Descripción

Este proyecto consiste en la restauración y explotación de una base de datos PostgreSQL orientada a la gestión de un sistema de alojamientos turísticos (hoteles, apartamentos, villas, cabañas, etc.), incluyendo propietarios, ubicaciones, habitaciones, reservas, huéspedes, pagos y reseñas.

A partir del script de restauración (`accommodation_database_restore.sql`) se crea el esquema completo de la base de datos, se cargan los datos iniciales y, finalmente, se ejecuta un conjunto de 20 consultas SQL que cubren operaciones de inserción, actualización, eliminación, `JOIN`, funciones de agregación y subconsultas.

## Requisitos

- PostgreSQL 14 o superior (dump generado con formato personalizado v1.16, PG 18.3)
- pgAdmin 4
- Codificación `UTF8`, locale `en_US.UTF-8`

## Restauración de la base de datos

1. Crear la base de datos:

```sql
CREATE DATABASE accommodations_tourism
  WITH TEMPLATE = template0 ENCODING = 'UTF8'
  LOCALE_PROVIDER = libc LOCALE = 'en_US.UTF-8';
```

2. Restaurar el script:

```bash
psql -U postgres -d accommodations_tourism -f accommodation_database_restore.sql
```

También puede hacerse desde pgAdmin 4 usando **Query Tool** sobre la base de datos ya creada, o mediante la opción **Restore** del menú contextual de la base de datos.

## Estructura de la base de datos

### Tablas principales

| Tabla | Descripción |
|---|---|
| `accommodation_types` | Catálogo de tipos de alojamiento (Hotel, Hostel, Apartment, Villa, Cabin, Resort, Guesthouse, House) |
| `amenities` | Catálogo de comodidades (WiFi, Pool, Parking, AC, Kitchen, etc.) |
| `owners` | Propietarios de los alojamientos |
| `locations` | Ubicaciones geográficas de los alojamientos |
| `accommodations` | Alojamientos publicados (precio, capacidad, tipo, ubicación, propietario) |
| `accommodation_amenities` | Relación N:M entre alojamientos y comodidades |
| `rooms` | Habitaciones asociadas a cada alojamiento |
| `guests` | Huéspedes registrados |
| `booking_statuses` | Catálogo de estados de reserva (Pending, Confirmed, CheckedIn, CheckedOut, Cancelled, NoShow) |
| `bookings` | Reservas realizadas |
| `booking_guests` | Huéspedes adicionales asociados a una reserva |
| `payments` | Pagos asociados a cada reserva |
| `reviews` | Reseñas dejadas por los huéspedes |
| `staff_users` | Usuarios del personal del sistema |

### Otros objetos

- **Función** `set_updated_at()`: actualiza automáticamente el campo `updated_at` en cada `UPDATE`.
- **Triggers**: aplicados en `accommodations`, `bookings`, `guests`, `owners`, `rooms` y `staff_users` para mantener actualizado el campo `updated_at`.
- **Secuencias**: una por cada tabla con clave primaria autoincremental.
- **Índices**: sobre claves foráneas y campos de búsqueda frecuente (fechas de reserva, estado, propietario, ubicación).
- **Llaves foráneas**: garantizan la integridad referencial entre alojamientos, propietarios, ubicaciones, reservas, huéspedes, pagos y reseñas.

## Consultas realizadas

A continuación se listan las 20 consultas desarrolladas sobre la base de datos:

1. **INSERT** — Registro de un nuevo propietario (`owners`).
2. **INSERT** — Registro de un nuevo alojamiento (`accommodations`).
3. **INSERT** — Registro de un huésped adicional en una reserva (`booking_guests`).
4. **INSERT** — Registro de una nueva reserva (`bookings`).
5. **SELECT** — Alojamientos activos (`is_active = true`), ordenados por ID.
6. **SELECT** — Huéspedes filtrados por nacionalidad (México, España, Japón).
7. **SELECT** — Reservas cuya fecha de salida está dentro de un rango específico.
8. **UPDATE** — Actualización del precio por noche de un alojamiento.
9. **UPDATE** — Actualización del nombre de un estado de reserva.
10. **DELETE** — Eliminación de una reseña específica.
11. **INNER JOIN** — Reservas junto con los datos del huésped que las realizó.
12. **INNER JOIN** — Detalle completo de alojamiento junto con su tipo.
13. **INNER JOIN** — Pagos relacionados con su reserva y alojamiento correspondiente.
14. **LEFT JOIN** — Alojamientos que no tienen ninguna reseña registrada.
15. **LEFT JOIN** — Alojamientos que no tienen ninguna reserva registrada.
16. **Función de agregación** — Suma total de ingresos por reservas (`SUM`).
17. **Función de agregación** — Promedio de calificación de las reseñas (`AVG`).
18. **GROUP BY** — Top 5 alojamientos con más reservas.
19. **HAVING** — Alojamientos con más de 3 reservas.
20. **Subconsulta** — Alojamiento con el precio por noche más alto.

## Notas

- Los montos de las reservas y pagos están expresados en distintas monedas (`USD`, `EUR`, `GBP`, `MXN`, `BRL`) según el campo `currency_code` de cada alojamiento.
- El campo `total_nights` en `bookings` es una columna generada automáticamente (`GENERATED ALWAYS AS`), calculada como la diferencia entre `check_out_date` y `check_in_date`.
- Las consultas 1 a 4 modifican datos (inserciones); se recomienda ejecutarlas una sola vez para evitar duplicados o conflictos con las llaves primarias/foráneas ya existentes.
