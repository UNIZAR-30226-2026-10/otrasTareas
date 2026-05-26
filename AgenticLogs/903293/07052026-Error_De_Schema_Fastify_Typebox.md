**Source:** https://aistudio.google.com/prompts/1T5dXEtc_IMOirw3thz85Uxo6HN_omh8C

## System Instructions

Optional tone and style instructions for the model

---

### User

Error starting server: FastifyError: Failed building the serialization schema for GET: /api/users/:email/SEP, due to error schema is invalid: data must be object,boolean
at Boot.<anonymous> (/usr/src/app/node_modules/fastify/lib/route.js:440:21)
at Object.onceWrapper (node:events:630:28)
at Boot.emit (node:events:521:24)
at /usr/src/app/node_modules/avvio/boot.js:102:12
at /usr/src/app/node_modules/avvio/boot.js:451:7
at done (/usr/src/app/node_modules/avvio/lib/plugin.js:228:5)
at check (/usr/src/app/node_modules/avvio/lib/plugin.js:252:9)
at process.processTicksAndRejections (node:internal/process/task_queues:104:5) {
code: 'FST_ERR_SCH_SERIALIZATION_BUILD',
statusCode: 500
}

app.get("/:email/SEP", {
schema: {
summary: "Obtener el SEP de un usuario",
tags: ["users"],
security: [{ CookieAuth: [] }],
description: Endpoint para obtener los SEP de tu cuenta.  La petición debe incluir el email del usuario para el cual se quiere obtener los SEP.,
params: Type.Object({
email: Type.String({ format: "email" })
}),
response: {
200: Type.Number,
401: UnauthorizedSessionToken,
403: ForbiddenSessionToken,
404: Type.Object({
error: Type.String()
})
}
}
}, async (request, reply) => {
const { email } = request.params as { email: string };
const user = await User.getUserByEmail(email);

code
Code
download
content_copy
expand_less
if (!user) {
        return reply.status(404).send({ error: "usuario no encontrado" });
    }

    return reply.status(200).send(user.SEP);
});

---

### Model

El error se debe a que estás pasando la referencia a la función Type.Number en lugar de ejecutarla para generar el esquema. En TypeBox (y en la mayoría de las librerías de esquemas para Fastify), los tipos se definen llamando a funciones.

El Problema

En tu código tienes:

code
TypeScript
download
content_copy
expand_less
response: {
    200: Type.Number, // <--- Aquí falta el "()"
    // ...
}

Fastify intenta procesar la función Type.Number como si fuera un objeto de esquema JSON, no lo reconoce y lanza el error schema is invalid.

La Solución

Simplemente añade los paréntesis para ejecutar la función:

code
TypeScript
download
content_copy
expand_less
response: {
    200: Type.Number(), // <--- Corrección: Añade paréntesis
    401: UnauthorizedSessionToken,
    403: ForbiddenSessionToken,
    404: Type.Object({
        error: Type.String()
    })
}
Recomendación de Diseño (Buenas Prácticas)

Aunque devolver un número directamente es válido en JSON, es una mejor práctica devolver un objeto. Esto te permite ampliar la respuesta en el futuro sin romper la compatibilidad con los clientes que consumen tu API.

Código corregido y mejorado:

code
TypeScript
download
content_copy
expand_less
app.get("/:email/SEP", {
    schema: {
        summary: "Obtener el SEP de un usuario",
        tags: ["users"],
        security: [{ CookieAuth: [] }],
        params: Type.Object({
            email: Type.String({ format: "email" })
        }),
        response: {
            200: Type.Object({
                sep: Type.Number() // Envoltura en un objeto
            }),
            401: UnauthorizedSessionToken,
            403: ForbiddenSessionToken,
            404: Type.Object({
                error: Type.String()
            })
        }
    }
}, async (request, reply) => {
    const { email } = request.params as { email: string };
    const user = await User.getUserByEmail(email);

    if (!user) {
        return reply.status(404).send({ error: "usuario no encontrado" });
    }

    // Si usas la recomendación del objeto:
    return reply.status(200).send({ sep: user.SEP });
    
    // Si decides seguir devolviendo solo el número (con Type.Number()):
    // return reply.status(200).send(user.SEP);
});
Otros puntos a revisar:

Variables externas: Asegúrate de que UnauthorizedSessionToken y ForbiddenSessionToken también sean esquemas válidos (objetos resultantes de llamar a Type.Object(...)) y no solo definiciones de clase o funciones sin ejecutar.

Formato Email: Para que format: "email" funcione en la validación, asegúrate de tener instalada la librería ajv-formats y configurada en tu instancia de Fastify, de lo contrario Fastify ignorará ese formato o podría dar una advertencia.

---

