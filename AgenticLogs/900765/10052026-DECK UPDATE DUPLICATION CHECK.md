User:     app.put("/:email/decks/:deckId", {
        schema: {
            summary: "Actualizar un mazo",
            tags: ["users"],
            security: [{ CookieAuth: [] }],
            description: `Endpoint para actualizar un mazo de tu cuenta. 
            La petición debe incluir el email del usuario, el id del mazo que se desea actualizar y la nueva información del mazo.`,
            params: Type.Object({
                email: Type.String({ format: "email" }),
                "deckId": Type.String()
            }),
            body: Type.Object({
                nombre: Type.String(),
                cartaAñadir: Type.Array(Type.Object({
                    nombre: Type.String(),
                    calidad: Type.Enum(Rareza),
                    tipo: Type.Enum(Tipo_Carta),
                    descripcion: Type.String(),
                })),
                cartaEliminar: Type.Array(Type.Object({
                    nombre: Type.String(),
                    calidad: Type.Enum(Rareza),
                    tipo: Type.Enum(Tipo_Carta),
                    descripcion: Type.String(),
                }))
            }),
            response: {
                200: Type.Object({
                    message: Type.String()
                }),
                401: UnauthorizedSessionToken,
                403: ForbiddenSessionToken,
                400: Type.Object({
                    error: Type.String()
                })
            }
        }
    }, async (request, reply) => {
        try {
        const { email, "deckId": deckId } = request.params as { email: string, "deckId": string };
        const { nombre, cartaAñadir, cartaEliminar } = request.body as {
            nombre: string;
            cartaAñadir: {
                nombre: string;
                calidad: Rareza;
                tipo: Tipo_Carta;
                descripcion: string;
            }[];
            cartaEliminar: {
                nombre: string;
                calidad: Rareza;
                tipo: Tipo_Carta;
                descripcion: string;
            }[];
        };

            const usuario = await User.getUserByEmail(email);
            if (!usuario) {
                return reply.status(400).send({ error: "Usuario no encontrado" });
            }

            

            await Deck.updateDeck(deckId, email, { cartaAñadir, cartaEliminar });
            await Deck.updateDeckName(deckId, email, nombre);
            return reply.status(200).send({ message: "Mazo actualizado correctamente" });
        } catch (error) {
            return reply.status(400).send({ error: error instanceof Error ? error.message : "Error al actualizar el mazo" });
        }
    });

Para esta funcion quiero que haya una verificacion que compruebe que el nuevo mazo no va a tener más de 2 veces la misma carta, quiero que se haga antes de actualizar el mazo

GitHub Copilot: Searched for text `app.put("/:email/decks/:deckId"`, 1 result

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/UserRoutes.ts#970-970), lines 970 to 1045

Searched for text `Deck`, 20 results

Searched for text `getAllCardsFromADeck`, 3 results

Searched for text `updateDeck`, 14 results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/UserRoutes.ts#1008-1008), lines 1008 to 1045

Replacing 8 lines with 33 lines in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/API/routes/UserRoutes.ts)


```

```
He añadido la verificación que pedías para comprobar que no se tengan más de 2 copias de la misma carta en el mazo. El código ahora comprueba las cartas actuales del mazo utilizando `Deck.getAllCardsFromADeck`, ajusta las cantidades que se van a eliminar,  y finalmente verifica que si al añadir las nuevas supera 2 copias devuelva un error `400` antes de hacer el `Deck.updateDeck(...)`.

El cambio se ha aplicado satisfactoriamente en UserRoutes.ts. Si la carta supera el límite devolverá un mensaje como `"El mazo no puede tener más de 2 copias de la misma carta (NombreDeus Ex)"`.

Made changes.

User: al seedear obtengo:prisma:query ROLLBACK
prisma:error 
Invalid `prisma.usuario.update()` invocation in
/usr/src/app/src/services/User.ts:224:30

  221 })
  222 await Deck.createDefaultDeckForUser(user)
  223 
→ 224 await prisma.usuario.update(
An operation failed because it depends on one or more records that were required but not found. Expected 15 records to be connected, found only 14.
Error al crear el usuario: PrismaClientKnownRequestError: 
Invalid `prisma.usuario.update()` invocation in
/usr/src/app/src/services/User.ts:224:30

  221 })
  222 await Deck.createDefaultDeckForUser(user)
  223 
→ 224 await prisma.usuario.update(
An operation failed because it depends on one or more records that were required but not found. Expected 15 records to be connected, found only 14.
    at zr.handleRequestError (/usr/src/app/src/node_modules/@prisma/client/src/runtime/RequestHandler.ts:228:13)
    at zr.handleAndLogRequestError (/usr/src/app/src/node_modules/@prisma/client/src/runtime/RequestHandler.ts:174:12)
    at zr.request (/usr/src/app/src/node_modules/@prisma/client/src/runtime/RequestHandler.ts:143:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
    at async a (/usr/src/app/src/node_modules/@prisma/client/src/runtime/getPrismaClient.ts:807:24)
    at async createUser (/usr/src/app/src/services/User.ts:224:9)
    at async cuentaAdminPorDefecto (/usr/src/app/src/poblation/CrearDatosBase.js:387:5)
    at async seedIfMissing (/usr/src/app/src/seed.ts:37:5)
    at async main (/usr/src/app/src/seed.ts:65:5) {
  code: 'P2025',
  meta: { modelName: 'Usuario', expectedRows: 15 },
  clientVersion: '7.8.0'
}
Error ejecutando seed: Error: Error al crear el usuario
    at createUser (/usr/src/app/src/services/User.ts:273:15)
    at process.processTicksAndRejections (node:internal/process/task_queues:103:5)
    at async cuentaAdminPorDefecto (/usr/src/app/src/poblation/CrearDatosBase.js:387:5)
    at async seedIfMissing (/usr/src/app/src/seed.ts:37:5)
    at async main (/usr/src/app/src/seed.ts:65:5)

GitHub Copilot: Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts#200-200), lines 200 to 250

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts#251-251), lines 251 to 300

Searched for text `createCard`, 10 results

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/poblation/CrearDatosBase.js#300-300), lines 300 to 360

Read [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/poblation/CrearDatosBase.js#210-210), lines 210 to 320

Searched for text `Robo de Identidad`, 6 results

Replacing 7 lines with 7 lines in [](file:///c%3A/Users/danie/Desktop/clase4/PS/backend/src/services/User.ts)


```

```
El error se debe a un pequeño typo (error tipográfico) en el proceso de creación del usuario (User.ts).

Cuando se establece la conexión a las cartas por defecto del usuario, se estaba intentando conectar la carta `"Robo de Identidad"` (con la "I" mayúscula), pero en la base de datos la carta se ha creado como `"Robo de identidad"` (con "i" minúscula). Como prisma es estricto, no encontraba esa carta (1 de las 15 esperadas).

He modificado User.ts para que use el nombre correcto con la minúscula `"Robo de identidad"`. Ya deberías poder ejecutar el seed sin problemas. Pruébalo de nuevo o dime si persiste algún error.

Made changes.