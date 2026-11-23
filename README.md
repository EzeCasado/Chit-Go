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

3. Elegir el modo, en `SPRING_PROFILES_ACTIVE`:

   - **`demo`**, el que viene: con la semilla de la demo, cuatro locales con su personal, su carta y cuentas de
     los 30 días anteriores. Para probar Chit&Go es este, y no hace falta cambiar nada más.
   - **Vacío** (`SPRING_PROFILES_ACTIVE=`): sin semilla, para darle la app a un cliente. Arranca con la base
     vacía y crea el primer local y su Gerente/Dueño con estas cinco variables del `.env`:

     | Variable | Qué es |
     |---|---|
     | `CHITGO_LOCAL_NOMBRE` | El nombre del local |
     | `CHITGO_LOCAL_DIRECCION` | Su dirección |
     | `CHITGO_GERENTE_NOMBRE` | El nombre del Gerente/Dueño |
     | `CHITGO_GERENTE_EMAIL` | Su email, con el que entra |
     | `CHITGO_GERENTE_CONTRASENA` | Su contraseña provisoria, de 8 a 64 caracteres |

     Van las cinco o ninguna: con algunas sí y otras no, el back no arranca y el log nombra las que faltan.
     Con `demo` quedan vacías.

   **El modo se elige antes del primer `up` y no se cambia sobre la misma base.** La semilla corre una sola
   vez, sobre la base vacía: con `demo` sobre una base que ya tiene locales, el back no arranca y dice "Esta
   base ya tiene datos: sacá el perfil demo". Para pasar de un modo al otro, `docker compose down -v` (borra
   la base) y `up` de nuevo.

Las demás variables del `.env` tienen su explicación al lado. Para una prueba no hace falta cambiar ninguna.

## Levantar

```
docker compose up --build -d
```

- La primera vez tarda unos minutos, porque arma las imágenes del back y del front. Las siguientes, no.
- El back arranca recién cuando MySQL está listo. Con la base vacía, Flyway crea las tablas al arrancar y, con
  `demo`, carga la semilla. Esa carga corre una sola vez: con los próximos `up`, los datos son los que quedaron.
- **Hasta que el back termina de arrancar, el front da 502 en todo lo que va a `/api`.** Para saber cuándo está:

  ```
  docker compose logs -f back
  ```

  Cuando aparece `Started ChitGoApplication`, ya se puede entrar (`Ctrl+C` corta el log, no el back).
- `docker compose ps` muestra los tres servicios, con `mysql` en `healthy`.

## Entrar

`http://localhost` solo no lleva a ninguna pantalla: cada uno entra por su URL, la que en el local sale de un QR.

- El Gerente/Dueño y el Admin entran por `http://localhost/admin/login`. Los reportes y la auditoría están en
  su menú.
- Mozo y Cocina, por el login del personal de su local, con su PIN; Caja, por el mismo con `/caja` al final.
- La primera vez en cada navegador, el mozo pone también su usuario y contraseña, y después el PIN. Se vuelve
  a pedir cada 7 días.
- **Una sesión por navegador:** la cookie es una sola, así que si entra Caja donde estaba un mozo, el mozo
  queda afuera. Para tener varios roles a la vez, cada uno en otro navegador, en una ventana de incógnito o en
  otro dispositivo.
- Desde un celular en la misma red, lo mismo con la IP de la compu en lugar de `localhost` (por ejemplo
  `http://192.168.1.50/mesa/...`).

### Con la semilla (`demo`)

Cuatro locales con 20 mesas cada uno. **Ninguna cuenta queda abierta**: todas las mesas arrancan libres. La
Esquina, La Pala y La Colina tienen cuentas cerradas y pedidos de los 30 días anteriores a la carga, para que
los reportes y la auditoría tengan qué mostrar; hoy arranca vacío. El detalle está en "Datos de la demo" del
[README del back](backend/README.md).

Contraseña de todos los que entran con contraseña: `Chitgo2026!`. Nadie tiene que cambiarla al entrar. Los PIN
de La Pala y La Colina son el local (3 La Pala, 4 La Colina), el rol (1 Mozo, 2 Caja, 3 Cocina) y el número
dentro del rol.

#### La Esquina

Los mismos usuarios, PIN, tokens y platos que la semilla de dev, así sirven los casos de Postman. Suma las mesas
7 a 20, sin mozo, y platos hasta llegar a 30.

Login del personal: `http://localhost/staff/83f39b25-deac-4fde-80cc-4cfdf8117713`

Login de Caja: `http://localhost/staff/83f39b25-deac-4fde-80cc-4cfdf8117713/caja`

| Rol | Nombre | PIN | Usuario o email |
|---|---|---|---|
| Gerente/Dueño | Dueño La Esquina |  | `dueno@laesquina.test` |
| Admin | Admin La Esquina |  | `admin@laesquina.test` |
| Mozo | Juan | `1234` | `juan` |
| Mozo | Ana | `5678` | `ana` |
| Caja | Carla | `2468` |  |
| Cocina | Hugo | `1357` |  |

<details>
<summary>Las 20 mesas de La Esquina</summary>

| Mesa | Mozo | URL |
|---|---|---|
| 1 | Juan | `http://localhost/mesa/d4964b6b-fdae-4b4f-83e9-1c1e78a21723` |
| 2 | Juan | `http://localhost/mesa/7ac79c59-b48d-4db2-8365-5c6a6565d9f4` |
| 3 | Juan y Ana | `http://localhost/mesa/22f4331e-4464-43b8-a7e2-eea2682b1a40` |
| 4 | Ana | `http://localhost/mesa/f0c89fba-24a4-4cb4-acff-dee757071597` |
| 5 | (sin mozo) | `http://localhost/mesa/c8b58174-70d8-431f-8b8f-b0d3531c8409` |
| 6 | (sin mozo) | `http://localhost/mesa/1a8af780-91b5-4815-a6cb-128a6ef92e59` |
| 7 | (sin mozo) | `http://localhost/mesa/a9101e94-fd2a-470e-8345-7886b098f176` |
| 8 | (sin mozo) | `http://localhost/mesa/9a48f77f-8fdb-42f8-b37e-32bc784b182c` |
| 9 | (sin mozo) | `http://localhost/mesa/3abc56b6-9ca3-4dfd-ada3-d459c148f5f9` |
| 10 | (sin mozo) | `http://localhost/mesa/c6a85b5f-58dd-48ae-bb25-c3c305ecfe54` |
| 11 | (sin mozo) | `http://localhost/mesa/28c32028-7d0d-420f-8368-e08acefa36f9` |
| 12 | (sin mozo) | `http://localhost/mesa/6eacfac2-1905-4e70-b764-763c044d25d9` |
| 13 | (sin mozo) | `http://localhost/mesa/70349c09-0e4c-48f6-80a2-ebae3bae7e79` |
| 14 | (sin mozo) | `http://localhost/mesa/838acc0a-0d60-4222-8a7f-52746eccaa1a` |
| 15 | (sin mozo) | `http://localhost/mesa/1411e744-4d5a-4ede-b705-2ac183ddbd22` |
| 16 | (sin mozo) | `http://localhost/mesa/3ee23e2b-2e7a-4c3d-82ec-81dfedd41299` |
| 17 | (sin mozo) | `http://localhost/mesa/3afc2af3-a049-4389-9086-2dc68144795c` |
| 18 | (sin mozo) | `http://localhost/mesa/ec5122a1-3afe-4992-a5a4-ca8e1c9dfda1` |
| 19 | (sin mozo) | `http://localhost/mesa/55775c6e-e35a-4e1e-aaf4-96f970b7b150` |
| 20 | (sin mozo) | `http://localhost/mesa/63a57323-0271-474b-b38b-3a90fd91bbb6` |

</details>

#### El Puerto

El segundo local de dev, para ver que los datos no se mezclan. Suma las mesas 3 a 20, sin mozo, y platos hasta
llegar a 29. **No tiene historial**: no tiene Caja ni Cocina que aprueben, cocinen y cierren.

Login del personal: `http://localhost/staff/de076a51-1fbd-46f5-98c5-c895ce59a007`

| Rol | Nombre | PIN | Usuario o email |
|---|---|---|---|
| Gerente/Dueño | Dueño El Puerto |  | `dueno@elpuerto.test` |
| Mozo | Juan | `4321` | `juan` |

<details>
<summary>Las 20 mesas de El Puerto</summary>

| Mesa | Mozo | URL |
|---|---|---|
| 1 | Juan | `http://localhost/mesa/ab0124a8-6033-4d73-a1f6-3513140e62f7` |
| 2 | Juan | `http://localhost/mesa/9426856c-793f-405c-bc07-46be50499627` |
| 3 | (sin mozo) | `http://localhost/mesa/596e0328-1822-4e18-90f3-ce192a28b6ba` |
| 4 | (sin mozo) | `http://localhost/mesa/305786ef-06f7-4326-992d-113de2df14cc` |
| 5 | (sin mozo) | `http://localhost/mesa/8199bf98-5cdb-419b-a750-b50ec7a747d9` |
| 6 | (sin mozo) | `http://localhost/mesa/a6bdc286-2bd5-4a69-abd9-9c6cd9815344` |
| 7 | (sin mozo) | `http://localhost/mesa/3dc8cbb1-7709-4feb-b505-11d80a47ada6` |
| 8 | (sin mozo) | `http://localhost/mesa/f336c913-e8c6-417b-930c-13a8b292bd21` |
| 9 | (sin mozo) | `http://localhost/mesa/f6fe3f82-3323-4f13-97d6-814e2dd64c5e` |
| 10 | (sin mozo) | `http://localhost/mesa/0cb0c4e2-35d5-4f1f-b64d-3dc8fd4c609f` |
| 11 | (sin mozo) | `http://localhost/mesa/0fccf298-692c-4346-b644-deeb15e3992d` |
| 12 | (sin mozo) | `http://localhost/mesa/65292bf7-0b67-4564-b34b-2206c59ac8cf` |
| 13 | (sin mozo) | `http://localhost/mesa/a37652f0-ab40-4b8e-9111-882f508152aa` |
| 14 | (sin mozo) | `http://localhost/mesa/a2a75942-b4e5-4411-ae72-597dfdf3cbb2` |
| 15 | (sin mozo) | `http://localhost/mesa/28d5dce2-c7bc-4640-ae45-ab9731c6963a` |
| 16 | (sin mozo) | `http://localhost/mesa/1a8ba11a-2ad4-4d02-ab28-29e3962d418b` |
| 17 | (sin mozo) | `http://localhost/mesa/d84f46a2-44b1-4c22-aafb-95cea9e20ff7` |
| 18 | (sin mozo) | `http://localhost/mesa/67901aaf-4310-4cef-9610-2cbfca2d5dba` |
| 19 | (sin mozo) | `http://localhost/mesa/0014ae65-b9aa-4e32-814f-a98a22587c8d` |
| 20 | (sin mozo) | `http://localhost/mesa/81322aa7-d238-4a73-a80d-3bc1fd0be691` |

</details>

#### Pizzería La Pala

Una pizzería inventada, con 29 platos.

Login del personal: `http://localhost/staff/d7aa3741-af38-40f2-8bfa-3bba2f2d8e99`

Login de Caja: `http://localhost/staff/d7aa3741-af38-40f2-8bfa-3bba2f2d8e99/caja`

| Rol | Nombre | PIN | Usuario o email |
|---|---|---|---|
| Gerente/Dueño | Dueño La Pala |  | `dueno@lapala.test` |
| Admin | Admin La Pala |  | `admin@lapala.test` |
| Caja | Marta | `3201` |  |
| Caja | Nicolás | `3202` |  |
| Cocina | Oscar | `3301` |  |
| Cocina | Paula | `3302` |  |
| Mozo | Bruno | `3101` | `bruno` |
| Mozo | Camila | `3102` | `camila` |
| Mozo | Diego | `3103` | `diego` |
| Mozo | Florencia | `3104` | `florencia` |
| Mozo | Lucas | `3105` | `lucas` |

<details>
<summary>Las 20 mesas de Pizzería La Pala</summary>

| Mesa | Mozo | URL |
|---|---|---|
| 1 | Bruno | `http://localhost/mesa/89e4bbd7-0b91-4d2c-b96b-2d622d37e8df` |
| 2 | Bruno | `http://localhost/mesa/751646c4-e2b4-4d80-808d-b3bc3236ac8b` |
| 3 | Bruno | `http://localhost/mesa/2bd4faff-7588-47f5-a5af-e625904bfc32` |
| 4 | Bruno | `http://localhost/mesa/77e6cf43-c14b-48bc-8a26-4b9b9d3d6c6c` |
| 5 | Camila | `http://localhost/mesa/d64dc4c0-9b17-42cd-952d-0afd9076bab5` |
| 6 | Camila | `http://localhost/mesa/1b1d1ea0-efcd-4899-a805-97e2cad82d52` |
| 7 | Camila | `http://localhost/mesa/678d6095-0a2a-4340-81dc-a5c11273d274` |
| 8 | Camila | `http://localhost/mesa/3dec99d1-9c0f-405d-a46e-447dc671c7eb` |
| 9 | Diego | `http://localhost/mesa/22b15db4-8992-4e5c-93d1-9a213dbb61db` |
| 10 | Diego | `http://localhost/mesa/3ecbbcd8-fad6-4498-aaeb-c418cb92134e` |
| 11 | Diego | `http://localhost/mesa/2898865a-87a7-4126-8135-7d731666bdee` |
| 12 | Diego | `http://localhost/mesa/c613a06d-8442-4625-b08c-f55575d91998` |
| 13 | Florencia | `http://localhost/mesa/14e8bc42-7a92-49f2-89d4-798ce7512cdc` |
| 14 | Florencia | `http://localhost/mesa/578daaab-d04a-49e2-8a05-204268e68fe7` |
| 15 | Florencia y Lucas | `http://localhost/mesa/59d38688-dfee-49a3-85ab-f04546b868f9` |
| 16 | Florencia y Lucas | `http://localhost/mesa/0dd75686-2041-448f-b5f6-268eeb2b9e34` |
| 17 | Lucas | `http://localhost/mesa/a4d076ed-c33c-4441-8d6d-6c320581f36c` |
| 18 | Lucas | `http://localhost/mesa/a69e28f4-efb8-405d-8c87-7d16390b9a18` |
| 19 | (sin mozo) | `http://localhost/mesa/a380fb10-2b94-4c86-a420-8cb1669e1abe` |
| 20 | (sin mozo) | `http://localhost/mesa/36d7a5dc-3a5b-4ed7-99b2-b624994364a8` |

</details>

#### La Colina

Una parrilla real (@lacolinaresto), con su carta entera: 23 categorías y 189 platos, con sus precios. El
personal y la dirección son inventados.

Login del personal: `http://localhost/staff/4eeea779-7b24-4857-909f-55f3f889efc8`

Login de Caja: `http://localhost/staff/4eeea779-7b24-4857-909f-55f3f889efc8/caja`

| Rol | Nombre | PIN | Usuario o email |
|---|---|---|---|
| Gerente/Dueño | Dueño La Colina |  | `dueno@lacolina.test` |
| Admin | Admin La Colina |  | `admin@lacolina.test` |
| Caja | Inés | `4201` |  |
| Caja | Julián | `4202` |  |
| Cocina | Ramón | `4301` |  |
| Cocina | Sofía | `4302` |  |
| Mozo | Rocío | `4101` | `rocio` |
| Mozo | Sergio | `4102` | `sergio` |
| Mozo | Tomás | `4103` | `tomas` |
| Mozo | Valeria | `4104` | `valeria` |
| Mozo | Gonzalo | `4105` | `gonzalo` |

<details>
<summary>Las 20 mesas de La Colina</summary>

| Mesa | Mozo | URL |
|---|---|---|
| 1 | Rocío | `http://localhost/mesa/993e499c-556a-42c6-bc4e-75eb0a7be17e` |
| 2 | Rocío | `http://localhost/mesa/cb2f5277-c923-412f-8aae-586dd0e99028` |
| 3 | Rocío | `http://localhost/mesa/72841901-b96d-4300-b0f9-6f7c74aecb2e` |
| 4 | Rocío | `http://localhost/mesa/fa2306e2-8fba-4ef5-8007-ddf91a29ad53` |
| 5 | Sergio | `http://localhost/mesa/77475fef-1303-4791-b23d-d6d99054923e` |
| 6 | Sergio | `http://localhost/mesa/ee37858f-d9a7-48a6-bde4-202a793e16c7` |
| 7 | Sergio | `http://localhost/mesa/1925565e-52c6-45cd-b685-baf9dec99138` |
| 8 | Sergio | `http://localhost/mesa/0681b7e1-9e45-4cd9-81c5-93f521fe6290` |
| 9 | Tomás | `http://localhost/mesa/d5b5bf86-3bf2-420d-8f86-1ba870b2b53c` |
| 10 | Tomás | `http://localhost/mesa/09b86c7d-a193-456e-8c10-cc5cf9175df5` |
| 11 | Tomás | `http://localhost/mesa/ff25c0c1-6aed-408a-b737-3ab5234ced16` |
| 12 | Tomás | `http://localhost/mesa/4a95c79c-0bbf-4379-92de-ac4b9fe17f74` |
| 13 | Valeria | `http://localhost/mesa/f7db499d-d79d-4654-8c02-a720c7196945` |
| 14 | Valeria | `http://localhost/mesa/dc713853-b797-4a2d-aa9a-3874540b1eec` |
| 15 | Valeria y Gonzalo | `http://localhost/mesa/0abd4d55-58f1-419a-bf81-3dfc54d1f58c` |
| 16 | Valeria y Gonzalo | `http://localhost/mesa/d341a48f-75c8-439a-9f9c-4058eccd4197` |
| 17 | Gonzalo | `http://localhost/mesa/6dd2be2a-fb35-4a57-b65a-185dd75c1e0f` |
| 18 | Gonzalo | `http://localhost/mesa/7193926d-66e5-431f-a3d8-3cbf0b785e94` |
| 19 | (sin mozo) | `http://localhost/mesa/f97cab94-c658-4227-b6bc-14da84589803` |
| 20 | (sin mozo) | `http://localhost/mesa/9006e0f8-615b-483c-ace3-2e611e64ebc8` |

</details>

### Sin semilla

- El Gerente/Dueño entra por `http://localhost/admin/login` con el `CHITGO_GERENTE_EMAIL` y la
  `CHITGO_GERENTE_CONTRASENA` del `.env`. Es provisoria: lo primero que le pide es elegir la suya. Desde ahí,
  la del `.env` ya no sirve y se puede borrar.
- La URL del login del personal del local sale en el log del back, al crearse el local:

  ```
  docker compose logs back
  ```

  En la línea `Login del personal: http://localhost/staff/...`.
- Si el local ya existe (por ejemplo, después de un `down` y `up`), no se crea otro, y el log lo dice.

## Otro local

En cualquiera de los dos modos, un local más con su Gerente/Dueño se crea con el script del back, adentro de
Docker y sin Maven, contra el MySQL del compose:

```powershell
docker compose run --rm -e SPRING_PROFILES_ACTIVE=script back alta-local --local="La Plaza" --direccion="Av. Siempre Viva 742" --gerente="Dueño La Plaza" --email=dueno@laplaza.test
```

- Pide la contraseña provisoria por la consola, dos veces y sin mostrarla. Al terminar muestra el local, su
  token y la URL del login del personal.
- La dirección es opcional. Si el email ya existe o un dato no sirve, no guarda nada y lo dice.
- El Gerente/Dueño nuevo entra por `http://localhost/admin/login`, y tiene que cambiar la contraseña la
  primera vez.

Un Gerente/Dueño queda bloqueado después de 5 intentos fallidos, y no tiene a nadie por encima que lo destrabe.
Se destraba con:

```powershell
docker compose run --rm -e SPRING_PROFILES_ACTIVE=script back desbloquear-gerente --email=dueno@laplaza.test
```

El detalle de los dos comandos está en "Alta de un local" del [README del back](backend/README.md).

## Bajar y borrar

```
docker compose down
```

Borra los contenedores y la red, pero **no la base ni las fotos de los platos**: cada una vive en su volumen, y
con el próximo `up` siguen. Las fotos que suben los Admin quedan en el volumen y no adentro de la imagen del back,
así que un `up --build` tampoco las pierde.

```
docker compose down -v
```

Borra también la base y las fotos de los platos. El próximo `up` arranca con la base vacía y, con `demo`, vuelve
a cargar la semilla.

## Si el puerto 80 está ocupado

Si el `up` dice que el puerto 80 ya está en uso, cambiar `FRONT_PUERTO` en el `.env`, por ejemplo
`FRONT_PUERTO=8088`, y `URL_FRONT=http://localhost:8088`, volver a correr `docker compose up -d` y entrar a
`http://localhost:8088`. En las URL de este README, cambia `localhost` por `localhost:8088`.

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
