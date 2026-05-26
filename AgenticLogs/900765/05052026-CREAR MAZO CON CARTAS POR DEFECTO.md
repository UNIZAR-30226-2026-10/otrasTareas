User: Como seria mi funcion para crear un mazo con las cartas por defecto teniendo en cuenta que existe este archivo

GitHub Copilot: Optimizing tool selection...Voy a ubicar cómo está modelada la relación de barajas y cartas para darte una función que encaje con tu Prisma actual, y de paso verificar si ya tienes una función parecida en el servicio.

Searched for regex `createDefaultDeckForUser|mazosPorDefecto|BarajaCarta` (`**/backend/src/**/*.ts`), 50 results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/prisma/schema.prisma)

Ya vi el esquema: la baraja se identifica por el compuesto nombre + usuarioEmail, y la relación BarajaCarta se crea con ese mismo compuesto más cartaNombre. Ahora voy a mirar cómo se invoca desde usuarios para ajustar la función a ese flujo real.

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts#180-180), lines 180 to 280

Searched for regex `Baraja por defecto|MazoPorDefecto|createDefaultDeckForUser` (`**/backend/src/**/*.ts`), 8 results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/ReturnTypes.ts#1-1), lines 1 to 160

Tengo ya el punto crítico: tu función actual mezcla `connect` donde Prisma suele esperar `create`/`connectOrCreate` para relaciones nuevas. Voy a comparar con otro sitio del código donde sí se crean relaciones anidadas para darte la versión correcta y mínima.

Searched for regex `connect: \{ carta: \{|barajaCartas: \{\s*create|connectOrCreate` (`**/backend/src/**/*.ts`), no results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Matches.ts#1-1), lines 1 to 120

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Matches.ts#120-120), lines 120 to 220

La hipótesis concreta ya quedó bastante clara: esa creación por defecto está intentando conectar una relación que todavía no existe. Voy a usar la validación más barata posible sobre ese archivo para confirmar si además el tipado ya lo delata.

Checked [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts), 2 problems found

Apareció un segundo problema importante: además del `connect` inválido, la función está devolviendo una propiedad que TypeScript no ve en ese `update`. También hay una inconsistencia de nombre entre `Baraja por defecto`, `MazoPorDefecto` y `mazoPorDefecto` que puede romper el uso real en partidas. Voy a comprobar ese nombre exacto antes de tocar el servicio.

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/managers/lobbyManager.ts#90-90), lines 90 to 130

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/seed.ts)

Voy a cerrar la ambigüedad de nombres con una búsqueda rápida; si solo aparece en el seed y en la creación inicial, lo ajusto junto con la función para que el mazo por defecto sea el mismo en todo el flujo.

Searched for regex `MazoPorDefecto|mazoPorDefecto|Baraja por defecto` (`**/backend/src/**/*.ts,backend/src/**/*.js`), no results

Searched for regex `MazoPorDefecto|mazoPorDefecto|Baraja por defecto` (`**/backend/src/**/*`), 8 results

La mejor solución parece reutilizar `createDeck` con una lista fija de cartas, pero antes de tocar imports quiero evitar un ciclo accidental con `Cards.ts`. Voy a verificar solo el encabezado de ese servicio.

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Cards.ts)

User: En mis tests obtengo estos errores, arreglalos:
PARA LANZARLO HE USADO NPM RUN TESTSEED ASI QUE TEN EN CUENTA QUE SE SEEDEA LA BD ANTES DE LOS TESTS
ℹ tests 78
ℹ suites 5
ℹ pass 58
ℹ fail 20
ℹ cancelled 0
ℹ skipped 0
ℹ todo 0
ℹ duration_ms 14383.03294

✖ failing tests:

test at test/TestDeckCompleto.js:1:1
✖ test/TestDeckCompleto.js (1788.23891ms)
  'test failed'

test at test/TestLobbyManager.js:11:5
✖ Crear Lobby (6.936316ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly equal:
  + actual - expected
  
  + undefined
  - 'ag@gmail.com'
  
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:13:16)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Promise.all (index 0)
      at async Suite.run (node:internal/test_runner/test:1515:7)
      at async startSubtestAfterBootstrap (node:internal/test_runner/harness:358:3) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: undefined,
    expected: 'ag@gmail.com',
    operator: 'strictEqual',
    diff: 'simple'
  }

test at test/TestLobbyManager.js:43:5
✖ Añadir un bot a un lobby (1.172518ms)
  Error: NOT_IN_A_LOBBY
      at LobbyManager.addBot (/usr/src/app/src/managers/lobbyManager.ts:103:57)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:45:17)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:48:5
✖ Añadir un bot cuando no eres el líder (2.205172ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly deep-equal:
  + actual - expected
  
    Comparison {
  +   message: 'NOT_IN_A_LOBBY',
  -   message: 'CANT_ADD',
      name: 'Error'
    }
  
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:51:16)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: Error: NOT_IN_A_LOBBY
        at LobbyManager.addBot (/usr/src/app/src/managers/lobbyManager.ts:103:57)
        at file:///usr/src/app/src/test/TestLobbyManager.js:52:21
        at getActual (node:assert:611:5)
        at strict.throws (node:assert:759:24)
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:51:16)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    expected: Error: CANT_ADD
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:53:12)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    operator: 'throws',
    diff: 'simple'
  }

test at test/TestLobbyManager.js:64:5
✖ Añadir bot a un lobby lleno (2.216298ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly deep-equal:
  + actual - expected
  
    Comparison {
  +   message: 'NOT_IN_A_LOBBY',
  -   message: 'LOBBY_IS_FULL',
      name: 'Error'
    }
  
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:69:16)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: Error: NOT_IN_A_LOBBY
        at LobbyManager.addBot (/usr/src/app/src/managers/lobbyManager.ts:103:57)
        at file:///usr/src/app/src/test/TestLobbyManager.js:70:21
        at getActual (node:assert:611:5)
        at strict.throws (node:assert:759:24)
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:69:16)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    expected: Error: LOBBY_IS_FULL
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:71:12)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    operator: 'throws',
    diff: 'simple'
  }

test at test/TestLobbyManager.js:73:5
✖ Expulsar jugador de un lobby (1.060979ms)
  Error: CANT_KICK
      at LobbyManager.deletePlayer (/usr/src/app/src/managers/lobbyManager.ts:120:23)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:76:32)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:80:5
✖ Salir de un Lobby siendo el líder (0.340124ms)
  Error: WRONG_LOBBY
      at LobbyManager.deletePlayer (/usr/src/app/src/managers/lobbyManager.ts:127:23)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:82:32)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:88:5
✖ Salir de un Lobby (0.362545ms)
  Error: WRONG_LOBBY
      at LobbyManager.deletePlayer (/usr/src/app/src/managers/lobbyManager.ts:127:23)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:91:32)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:107:5
✖ Expulsar a un jugador que no está en el lobby (4.094114ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly deep-equal:
  + actual - expected
  
    Comparison {
  +   message: 'CANT_KICK',
  -   message: 'NOT_IN_LOBBY',
      name: 'Error'
    }
  
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:109:16)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: Error: CANT_KICK
        at LobbyManager.deletePlayer (/usr/src/app/src/managers/lobbyManager.ts:120:23)
        at file:///usr/src/app/src/test/TestLobbyManager.js:110:21
        at getActual (node:assert:611:5)
        at strict.throws (node:assert:759:24)
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:109:16)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    expected: Error: NOT_IN_LOBBY
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:111:12)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    operator: 'throws',
    diff: 'simple'
  }

test at test/TestLobbyManager.js:113:5
✖ Expulsar a un bot del lobby (0.425365ms)
  Error: NOT_IN_A_LOBBY
      at LobbyManager.addBot (/usr/src/app/src/managers/lobbyManager.ts:103:57)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:115:25)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:121:5
✖ Seleccionar mazo (0.910613ms)
  Error: WRONG_LOBBY
      at LobbyManager.selectDeck (/usr/src/app/src/managers/lobbyManager.ts:159:68)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:124:17)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:158:5
✖ Poner jugador listo (0.410896ms)
  Error: WRONG_LOBBY
      at LobbyManager.setReady (/usr/src/app/src/managers/lobbyManager.ts:195:68)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:160:17)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:174:5
✖ Cambiar tablero siendo el lider del lobby (0.337453ms)
  Error: CANT_CHANGE_BOARD
      at LobbyManager.changeBoard (/usr/src/app/src/managers/lobbyManager.ts:169:56)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:176:17)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:207:5
✖ Aceptar invitación (0.694647ms)
  Error: INVITES_NOT_FOUND
      at LobbyManager.manageInvite (/usr/src/app/src/managers/lobbyManager.ts:68:53)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:211:32)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:217:5
✖ Rechazar invitación (0.318079ms)
  Error: INVITES_NOT_FOUND
      at LobbyManager.manageInvite (/usr/src/app/src/managers/lobbyManager.ts:68:53)
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:221:32)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestLobbyManager.js:226:5
✖ Aceptar o rechazar invitación en un lobby que no existe (1.883107ms)
  AssertionError [ERR_ASSERTION]: Expected values to be strictly deep-equal:
  + actual - expected
  
    Comparison {
  +   message: 'INVITES_NOT_FOUND',
  -   message: 'LOBBY_NOT_FOUND',
      name: 'Error'
    }
  
      at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:231:16)
      at Test.runInAsyncScope (node:async_hooks:214:14)
      at Test.run (node:internal/test_runner/test:1103:25)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7) {
    generatedMessage: true,
    code: 'ERR_ASSERTION',
    actual: Error: INVITES_NOT_FOUND
        at LobbyManager.manageInvite (/usr/src/app/src/managers/lobbyManager.ts:68:53)
        at file:///usr/src/app/src/test/TestLobbyManager.js:232:21
        at getActual (node:assert:611:5)
        at strict.throws (node:assert:759:24)
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:231:16)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    expected: Error: LOBBY_NOT_FOUND
        at TestContext.<anonymous> (file:///usr/src/app/src/test/TestLobbyManager.js:233:12)
        at Test.runInAsyncScope (node:async_hooks:214:14)
        at Test.run (node:internal/test_runner/test:1103:25)
        at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7),
    operator: 'throws',
    diff: 'simple'
  }

test at test/TestUsers_001.js:39:3
✖ Crear un usuario (348.65799ms)
  Error: Error al crear el usuario
      at Object.createUser (/usr/src/app/src/services/User.ts:225:15)
      at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
      at async TestContext.<anonymous> (file:///usr/src/app/src/test/TestUsers_001.js:40:18)
      at async Test.run (node:internal/test_runner/test:1110:7)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestUsers_001.js:105:3
✖ Email ya registrado (150.474203ms)
  Error: Error al crear el usuario
      at Object.createUser (/usr/src/app/src/services/User.ts:225:15)
      at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
      at async TestContext.<anonymous> (file:///usr/src/app/src/test/TestUsers_001.js:106:5)
      at async Test.run (node:internal/test_runner/test:1110:7)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestUsers_001.js:122:3
✖ Ñ y acentos en el nombre (156.486843ms)
  Error: Error al crear el usuario
      at Object.createUser (/usr/src/app/src/services/User.ts:225:15)
      at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
      at async TestContext.<anonymous> (file:///usr/src/app/src/test/TestUsers_001.js:123:20)
      at async Test.run (node:internal/test_runner/test:1110:7)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

test at test/TestUsers_001.js:132:3
✖ Modificar el cosmetico de un usuario (154.93672ms)
  Error: Error al crear el usuario
      at Object.createUser (/usr/src/app/src/services/User.ts:225:15)
      at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
      at async TestContext.<anonymous> (file:///usr/src/app/src/test/TestUsers_001.js:133:18)
      at async Test.run (node:internal/test_runner/test:1110:7)
      at async Suite.processPendingSubtests (node:internal/test_runner/test:785:7)

GitHub Copilot: Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/test/TestLobbyManager.js)

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/managers/lobbyManager.ts)

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/test/TestUsers_001.js)

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/test/TestDeckCompleto.js)

Searched for regex `usernameCreador|idCreador` (`**/backend/src/**/*`), 43 results

Searched for regex `jugadoresEnCola|getInvitesOfPlayer|sendInvite|manageInvite|selectDeck|setReady|changeBoard` (`**/backend/src/**/*.ts`), 27 results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts#1-1), lines 1 to 140

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts#140-140), lines 140 to 340

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts#340-340), lines 340 to 607

Searched for files matching `**/CrearDatosBase.js`, 1 match

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts#1-1), lines 1 to 200

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts#200-200), lines 200 to 320

Generating patch (193 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/managers/lobbyManager.ts)


```

```


Generating patch (87 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts)


```

```


Searched for text `usernameCreador` (`**/backend/src/**/*.ts`), 1 result

Generating patch (13 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts)


```

```


Generating patch (78 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts)


```

```


Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts#40-40), lines 40 to 140

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts#120-120), lines 120 to 260

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts#1-1), lines 1 to 40

Generating patch (147 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts)


```

```


Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts#60-60), lines 60 to 180

Generating patch (7 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/poblation/CrearDatosBase.js)


```

```


Generating patch (6 lines) in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/test/TestDeckCompleto.js)


```

```


Searched for text `usernameCreador` (`**/backend/src/**/*.ts`), no results

Checked [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/managers/lobbyManager.ts), [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/LobbiesRoutes.ts), [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/Deck.ts), [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/test/TestDeckCompleto.js), no problems found

Searched for regex `mazoPorDefecto|MazoPorDefecto` (`**/backend/src/**/*`), 6 results

Listo: corregi los errores que estaban rompiendo LobbyManager y la creacion del mazo por defecto, y ajuste el import del test de mazos.

- Ajuste el uso de `idJugador` y `idCreador` para que los mapas y validaciones trabajen con el email (como esperan los tests), y actualice las respuestas del API para usar `idCreador` de forma consistente en lobbyManager.ts y LobbiesRoutes.ts.
- Evite la colision por `@unique` en `Baraja.nombre` creando el mazo por defecto con un nombre por usuario y validando si ya existe en Deck.ts.
- Unifique el nombre del mazo de bots/seed a `mazoPorDefecto` en CrearDatosBase.js.
- Corrigi el import de `cosmeticosPorDefecto` en TestDeckCompleto.js.

Si quieres verificar:
1. `npm run testseed`
2. `npm test`

Si prefieres que el mazo por defecto tenga un nombre fijo para todos los usuarios, dime y te propongo un cambio de esquema o de estrategia.

Made changes.