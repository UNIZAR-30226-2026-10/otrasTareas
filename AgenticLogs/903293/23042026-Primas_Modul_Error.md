**Source:** https://aistudio.google.com/prompts/12qiR-9-2YwFrN5pxbN-AnQT-dcMPnGyM

## System Instructions

Optional tone and style instructions for the model

---

### User

Uso Nodejs 24 y prisma v7. En los logs al lanzar el docker me sale este error:

Node.js v24.15.0
Loaded Prisma config from prisma.config.ts.

Prisma schema loaded from prisma/schema.prisma.
Datasource "db": PostgreSQL database "postgres", schema "public" at "postgres_db:5432"

5 migrations found in prisma/migrations

No pending migrations to apply.
node:internal/modules/esm/resolve:271
throw new ERR_MODULE_NOT_FOUND(
^

Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/usr/src/app/dist/generated/prisma/internal/class.ts' imported from /usr/src/app/dist/generated/prisma/client.js
at finalizeResolution (node:internal/modules/esm/resolve:271:11)
at moduleResolve (node:internal/modules/esm/resolve:861:10)
at defaultResolve (node:internal/modules/esm/resolve:988:11)
at #cachedDefaultResolve (node:internal/modules/esm/loader:697:20)
at #resolveAndMaybeBlockOnLoaderThread (node:internal/modules/esm/loader:714:38)
at ModuleLoader.resolveSync (node:internal/modules/esm/loader:746:52)
at #resolve (node:internal/modules/esm/loader:679:17)
at ModuleLoader.getOrCreateModuleJob (node:internal/modules/esm/loader:599:35)
at ModuleJob.syncLink (node:internal/modules/esm/module_job:162:33)
at ModuleJob.link (node:internal/modules/esm/module_job:252:17) {
code: 'ERR_MODULE_NOT_FOUND',
url: 'file:///usr/src/app/dist/generated/prisma/internal/class.ts'
}

---

### Model

Este error es común en Prisma v6 y v7 cuando se utiliza una ruta de salida personalizada (output) dentro de la carpeta de compilación (dist) y se trabaja con ESM (EcmaScript Modules) en Node.js.

El problema principal es que el cliente de Prisma generado intenta importar archivos internos con extensión .ts que Node.js no puede resolver directamente en tiempo de ejecución desde la carpeta dist, o que fueron borrados/no incluidos durante el proceso de build.

Aquí tienes las soluciones ordenadas de la más recomendada a la menos:

1. Usar la ruta por defecto (Solución recomendada)

La mejor práctica con Prisma es no definir un output personalizado en el schema.prisma. Por defecto, Prisma genera el cliente en node_modules/.prisma/client. Esto evita problemas de rutas, de compilación y de tipos.

En tu prisma/schema.prisma:

code
Prisma
download
content_copy
expand_less
generator client {
  provider = "prisma-client-js"
  // Borra la línea de 'output'
}

Luego actualiza tu código donde importas el cliente:

code
TypeScript
download
content_copy
expand_less
import { PrismaClient } from '@prisma/client';
2. Si necesitas una ruta personalizada (Fuera de dist)

Si por alguna razón técnica necesitas que el cliente no esté en node_modules, nunca lo generes dentro de dist, ya que dist es el resultado de tu compilación y Prisma genera código que a su vez necesita ser procesado o tratado como módulo.

Cámbialo a una carpeta como src/generated:

En schema.prisma:

code
Prisma
download
content_copy
expand_less
generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"
}

Importante: Si haces esto, asegúrate de que tu tsconfig.json incluya esa carpeta y que tu proceso de build (tsc, esbuild, etc.) la procese correctamente.

3. Ajuste en el Dockerfile (Orden de ejecución)

El error ERR_MODULE_NOT_FOUND buscando un .ts dentro de dist sugiere que estás ejecutando prisma generate en un momento que ensucia o confunde al cargador de Node.js 24.

Asegúrate de que tu Dockerfile siga este orden:

Copiar archivos de dependencias.

Instalar dependencias.

Copiar el schema de Prisma.

Ejecutar npx prisma generate.

Copiar el resto del código fuente.

Ejecutar el build (tsc).

Si generas Prisma después del build dentro de la carpeta dist, Node.js intentará buscar archivos TypeScript que no han sido compilados a JavaScript.

4. Configuración de tsconfig.json (Node 24 + ESM)

Dado que usas Node 24, asegúrate de tener configurado el soporte para módulos modernos. En tu tsconfig.json verifica:

code
JSON
download
content_copy
expand_less
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "allowImportingTsExtensions": true, // A veces necesario en Node 24 para imports internos de Prisma
    "rewriteRelativeImportExtensions": true
  }
}
5. ¿Por qué sale .ts en el error?

En Prisma 7, el motor interno a veces utiliza archivos auxiliares. Si el entorno de Node.js no tiene activado el soporte experimental para TypeScript (que Node 24 ya trae), fallará.

Si decides seguir usando Node 24 para ejecutar archivos .ts directamente, intenta añadir este flag al lanzar tu aplicación en el Docker:

code
Bash
download
content_copy
expand_less
node --experimental-strip-types dist/main.js

Nota: Aunque Node 24 soporta TS, el cliente de Prisma generado suele esperar una estructura de paquetes estándar.

Resumen de acción rápida:

Quita la línea output de tu schema.prisma.

Borra tu carpeta dist.

Ejecuta npm install y npx prisma generate.

Cambia tus imports a import { PrismaClient } from '@prisma/client'.

Re-construye tu imagen de Docker.

Esto solucionará el 99% de los problemas de "Module Not Found" con Prisma v7.

---

