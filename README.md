# Chit&Go: entrega

Chit&Go completo, con la base (MySQL 8.4), el back y el front, cada uno en su contenedor. El back y el front son
submódulos: `backend/` es [Chit-Go-Backend](https://github.com/AlanWeishein/Chit-Go-Backend) y `frontend/` es
[Chit-Go-Frontend](https://github.com/AlanWeishein/Chit-Go-Frontend). Solo el front sale a la compu, en el puerto
80. MySQL y el back quedan adentro de la red del compose.

## Qué hace falta

- **Docker Desktop**, abierto. Si no está corriendo, `docker compose` da error al conectarse.
- **git**.

## Clonar

```
git clone --recurse-submodules https://github.com/EzeCasado/Chit-Go.git
cd Chit-Go
```

Sin `--recurse-submodules`, `backend/` y `frontend/` quedan vacías y el compose no arranca. Si ya se clonó así,
se bajan con:

```
git submodule update --init
```

## Configurar

1. Copiar `.env.example` como `.env`. El `.env` no se sube al repo, y sin él `docker compose up` no arranca.

   ```powershell
   Copy-Item .env.example .env
   ```

   En bash: `cp .env.example .env`.

2. Generar una `JWT_CLAVE` nueva y pegarla en el `.env`, en lugar de la que viene:

   ```
   openssl rand -base64 32
   ```

   **Ojo:** la del ejemplo está publicada en el repo, y con ella cualquiera puede armar una sesión válida.
   Si PowerShell no encuentra `openssl`, viene con Git: correrlo desde Git Bash.

Las demás variables del `.env` tienen su explicación al lado. Para una prueba no hace falta cambiar ninguna.

## Levantar

```
docker compose up --build -d
```

- La primera vez tarda unos minutos, porque arma las imágenes del back y del front. Las siguientes, no.
- El back arranca recién cuando MySQL está listo. Con la base vacía, Flyway crea las tablas al arrancar.
- **Hasta que el back termina de arrancar, el front da 502 en todo lo que va a `/api`.** Para saber cuándo está:

  ```
  docker compose logs -f back
  ```

  Cuando aparece `Started ChitGoApplication`, ya se puede entrar (`Ctrl+C` corta el log, no el back).
- `docker compose ps` muestra los tres servicios, con `mysql` en `healthy`.

## Entrar

`http://localhost` solo no lleva a ninguna pantalla: cada uno entra por su URL, la que en el local sale de un QR.

<!-- ST-28: completar con los usuarios y las URL de la semilla de la entrega. -->
> **Falta:** la tabla de usuarios y las URL de la semilla de la entrega. Mientras tanto, con
> `SPRING_PROFILES_ACTIVE=dev`, valen las de abajo, que son las de La Esquina en la semilla de desarrollo del back
> (la tabla completa está en "Datos de prueba (dev)" del [README del back](backend/README.md)).

| Quién | URL | Entra con |
|---|---|---|
| Mozo, Cocina | `http://localhost/staff/83f39b25-deac-4fde-80cc-4cfdf8117713` | Juan (Mozo), PIN `1234`; Hugo (Cocina), PIN `1357` |
| Caja | `http://localhost/staff/83f39b25-deac-4fde-80cc-4cfdf8117713/caja` | Carla, PIN `2468` |
| Admin | `http://localhost/admin/login` | `admin@laesquina.test` y `Chitgo2026!` |
| Cliente, mesa 1 | `http://localhost/mesa/d4964b6b-fdae-4b4f-83e9-1c1e78a21723` | sin login |

- La primera vez en cada navegador, el mozo pone también su usuario y contraseña (`juan` y `Chitgo2026!`), y
  después el PIN. Se vuelve a pedir cada 7 días.
- **Una sesión por navegador:** la cookie es una sola, así que si entra Carla donde estaba Juan, Juan queda afuera.
  Para tener varios roles a la vez, cada uno en otro navegador, en una ventana de incógnito o en otro dispositivo.
- Desde un celular en la misma red, lo mismo con la IP de la compu en lugar de `localhost` (por ejemplo
  `http://192.168.1.50/mesa/...`).

## Bajar y borrar

```
docker compose down
```

Borra los contenedores y la red, pero **no la base**: vive en un volumen, y con el próximo `up` los datos siguen.

```
docker compose down -v
```

Borra también la base. El próximo `up` arranca con la base vacía.

## Si el puerto 80 está ocupado

Si el `up` dice que el puerto 80 ya está en uso, cambiar `FRONT_PUERTO` en el `.env`, por ejemplo
`FRONT_PUERTO=8088`, volver a correr `docker compose up -d` y entrar a `http://localhost:8088`.

## Para el grupo

Cada submódulo queda fijo en un commit: no sigue su rama solo. Hoy siguen `develop`. Para traer lo último de
`develop` de los dos repos:

```
git submodule update --remote
git commit -am "Back y front actualizados"
```

Para la versión final, cuando el back y el front estén en `main` y probados, se pasan los dos submódulos a `main`:

```
git submodule set-branch --branch main backend
git submodule set-branch --branch main frontend
git submodule update --remote
git commit -am "Back y front en main"
```

Después de actualizar, `docker compose up --build -d` arma las imágenes con el código nuevo.
