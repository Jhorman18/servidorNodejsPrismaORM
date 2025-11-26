# Proyecto Node.js + Express + Prisma + MySQL

Este documento describe paso a paso cómo crear y ejecutar un servidor **Node.js** con **Express** usando **Prisma** como ORM y **MySQL** como base de datos.

---

## 1. Requisitos previos

- Tener instalado:
  - [Node.js](https://nodejs.org/) (usaremos la versión 20)
  - [nvm](https://github.com/coreybutler/nvm-windows/releases/tag/1.2.2) (opcional pero recomendado)
  - [MySQL](https://www.mysql.com/) en local o en algún servidor accesible

- Tener creada una base de datos en MySQL, por ejemplo:

```sql
CREATE DATABASE ejemplo_clase;
```
- Tabla de ejemplo:

```sql
create table usuario(
	idUsuario INT auto_increment primary key,
    usuNombres varchar(100) NOT NULL,
    usuCorreo varchar(100) NOT NULL,
    usuTelefono varchar(100) NOT NULL,
    usuPassword varchar(250) NOT NULL
)
```

---

## 2. Inicializar el proyecto

En la carpeta del proyecto:

```bash
nvm use 20              # Usar Node 20 (si tienes nvm instalado)
npm init -y             # Crear package.json
npm install express     # Instalar Express
npm install --save-dev nodemon       # Nodemon para desarrollo
npm install prisma@5.15.0 --save-dev  # Prisma CLI
npm install @prisma/client@5.15.0     # Cliente de Prisma
```

---

## 3. Inicializar Prisma

```bash
npx prisma init
```

Este comando crea:

- archivo `.env`
- carpeta `prisma/`
  - archivo `schema.prisma`

---

## 4. Configurar la conexión a la base de datos

Editar el archivo `.env` en la raíz del proyecto:

```env
DATABASE_URL="mysql://root:@localhost:3306/ejemplo_clase"
```

Ajustar:
- `root` → tu usuario de MySQL
- `@localhost` → host donde corre MySQL
- `3306` → puerto
- `ejemplo_clase` → nombre de tu base de datos

---

## 5. Configurar Prisma para usar MySQL

Editar `prisma/schema.prisma` para asegurarse de que use MySQL:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

### 5.1. Obtener el esquema desde la base de datos (opcional según tu flujo)

Si **ya tienes tablas creadas** en tu base de datos y quieres que Prisma las lea automáticamente:

```bash
npx prisma db pull
```

Esto actualizará `schema.prisma` con los modelos correspondientes a tus tablas.

---

## 6. Generar el cliente de Prisma

Luego de tener el `schema.prisma` (ya sea manual o por `db pull`):

```bash
npx prisma generate
```

Esto genera el cliente de Prisma que usarás en tu código para acceder a la base de datos.

---

## 7. Estructura de carpetas del proyecto

Se recomienda la siguiente estructura:

```txt
.
├── node_modules/
├── prisma/
│   ├── schema.prisma
├── src/
│   ├── prisma/
│   │   └── client.js
│   ├── controllers/
│   ├── routes/
│   └── app.js
├── .env
├── package.json
└── README.md
```

Crear carpetas y archivos base:

```txt
- src/prisma
- src/controllers
- src/routes
- src/app.js
- src/prisma/client.js
```

---

## 8. Cliente de Prisma

Archivo: `src/prisma/client.js`

```js
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

module.exports = prisma;
```

Este archivo exporta una instancia única de Prisma que podrás reutilizar en controladores, rutas, etc.

---

## 9. Servidor Express básico

Archivo: `src/app.js`

```js
const express = require('express');
const app = express();
const prisma = require('./prisma/client'); // Cliente de Prisma

app.use(express.json());

// Ruta de prueba
app.get('/', (req, res) => {
  res.send('Servidor funcionando correctamente');
});

// Ejemplo de ruta usando Prisma (opcional)
// app.get('/users', async (req, res) => {
//   try {
//     const users = await prisma.user.findMany();
//     res.json(users);
//   } catch (error) {
//     console.error(error);
//     res.status(500).json({ error: 'Error al obtener usuarios' });
//   }
// });

app.listen(3000, () => {
  console.log('Servidor corriendo en el puerto 3000');
});
```

---

## 10. Script de desarrollo en `package.json`

En `package.json`, agregar el script `dev`:

```json
{
  "scripts": {
    "dev": "nodemon src/app.js"
  }
}
```

---

## 11. Comandos usados (orden recomendado)

Estos son los comandos en el orden lógico para que todo funcione correctamente:

```bash
nvm use 20
npm init -y
npm install express
npm install --save-dev nodemon
npm install prisma@5.15.0 --save-dev
npm install @prisma/client@5.15.0

npx prisma init          # Crea .env y prisma/schema.prisma
# Editar .env y schema.prisma para MySQL

npx prisma db pull       # Si ya tienes tablas en la base de datos (opcional)
npx prisma generate      # Genera el cliente de Prisma

npm run dev              # Levantar el servidor con nodemon
```

---

## 12. Ejecutar el servidor

Para iniciar el servidor en modo desarrollo:

```bash
npm run dev
```

Luego abrir en el navegador:

- [http://localhost:3000](http://localhost:3000)

Deberías ver:

```text
Servidor funcionando correctamente
```

---

## 13. Notas

- El archivo `prisma.config.js` **no es necesario** para este flujo básico y no se usa en esta guía.
- Si deseas agregar modelos manualmente (en lugar de usar `db pull`), puedes editarlos en `schema.prisma` y luego usar:
  - `npx prisma migrate dev --name init`
  - `npx prisma generate`
- Para nuevas rutas y controladores:
  - Crear archivos dentro de `src/controllers` y `src/routes`
  - Importarlos y usarlos desde `src/app.js`.

---
