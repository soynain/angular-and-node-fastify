# angular-and-node-fastify
Repository to practice angular and, refresh some knowledge about fastify

Tiempo sin moverle al node JS, más sencillo que Java... no sé a estas alturas, nunca tuve problemas
en lenguajes menos complejos como PHP o Node. 

Producto de otros requerimientos... vamos a darle.

Fastify:

un framework que combina ligeras pre estructuras con una capa de express js a la vez. Es como en java que tienes spring boot y Javalin o
Micronout. Son apis ligeras, pequeñas y de rápido desarrollo inicial.

Yo lo configuré con typescript... nada complejo hasta ahora, más que agarrarle la onda a los hooks.

Así como en api gateway, también usas json schemas para validar body requests, casi que integro zod pero noo, muy batalloso, bendita documentación.

````main.js
/**Response no debe tener wrapper body como jsonschema */
export const generalResponseSchema={
    type: 'object',
    required: ['code', 'message'],
    properties: {
      code: { type: 'string' },
      message: { type: 'string' },
      body: { type: "object" }
    },
}

export const userJsonSchema = {
  body: {
    type: 'object',
    required: ['id', 'name', 'age'],
    properties: {
      id: { type: 'integer' },
      name: { type: 'string' },
      age: { type: "integer" }
    },
  },
  response:{
    200: {...generalResponseSchema}
  },
  attachValidation:true
}

````


Y ya se vinculan a tu controller:

````main.js
import Fastify, { type FastifyInstance, type FastifyReply, type FastifyRequest } from "fastify";
import { userJsonSchema } from "../models/schemas/schemas.js";
import { User } from "../models/user.js";

async function userRoutes(fastify: FastifyInstance, options: Object) {
    fastify.post('/user', { schema: userJsonSchema }, async (request: FastifyRequest, reply: FastifyReply) => {

        /**Lógica de un service */
        let userRequest : User = request.body as User;
        


        reply.send({
            code: "500",
            message: "holi",
        })
    })
}

export default userRoutes;
````

y los registras a tu server:

````main.js
import Fastify, { type FastifyReply, type FastifyRequest } from "fastify";
import userRoutes from "./routers/UserRouter.js";
import fastifyMysql from "@fastify/mysql";

const fastify = Fastify({
  logger: true
});


fastify.register(userRoutes);
fastify.register(fastifyMysql,{ connectionString: 'mysql://blabla', promise: true });


// Run the server!
fastify.listen({ port: 3000 }, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  // Server is now listening on ${address}
})
````

Me falta configurar la bdd, si solo configuro mysql2, solo tendre la posibilidad de usar querys por string, requiero un orm.

Un plugin disponmible es uno de prima... pero ahi entran luego los temas de seguridad de node. Puede ser bueno.
