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


Le dí una leída a la docu de angular, ehm sobre el orm se cancela, puesto que los orms no se pueden configurar con
async. 

````main.js
import { MySQLPromisePool, MySQLResultSetHeader } from "@fastify/mysql";
import { User } from "../models/user.js";
import Fastify from "fastify";
import { GeneralResponse } from "../models/generalresponse.js";
export async function saveUserDB(user: User,query: MySQLPromisePool){
    try {
        const connectionInstance = await query.getConnection();
        const result = await connectionInstance.execute<MySQLResultSetHeader>("INSERT INTO exampletable(id,name,age) values (?,?,?)",[user.id,user.name,user.age]);

        let response: GeneralResponse = {
            code:"SQL_200",
            message:"Registro guardado correctamente",
            body:{
                "rowUpdated":result[0].affectedRows
            }
        };

        return response;
    } catch (error:unknown) {
      //  console.log((error as Error).message)
        let except = (error as Error).message;
        console.log(except)
        let response: GeneralResponse = {
            code:"SQL_500",
            message:"Error al guardar",
            body:{
                "error":except
            }
        };

        return response;

    }    
}

````

Así puede quedar, para objetos más grandes eso si, puede complicarse :/

Pero así lo dejamos por ahora y después indago con el ORM porque si sería más fácil así.

Por ahora, lo dejaremos así, hice dos endpoints más para fastify, que quedan así:

````main.ts
import { MySQLPromisePool, MySQLResultSetHeader, MySQLRowDataPacket } from "@fastify/mysql";
import { User } from "../models/user.js";
import { toGeneralResponse } from "../helpers/helpers.js";
import { GeneralResponse } from "../models/generalresponse.js";
export async function saveUserDB(user: User,query: MySQLPromisePool) : Promise<GeneralResponse>{
    try {
        const connectionInstance = await query.getConnection();
        const result = await connectionInstance.execute<MySQLResultSetHeader>("INSERT INTO exampletable(id,name,age) values (?,?,?)",[user.id,user.name,user.age]);
        
        connectionInstance.release();

        let bodyRes ={"rowUpdated":result[0].affectedRows};

        return toGeneralResponse("SQL_200","Registro guardado correctamente",bodyRes);
    } catch (error:unknown) {
      //  console.log((error as Error).message)
        let except = (error as Error).message;
        console.log(except)
        return toGeneralResponse("SQL_500","Error al guardar",{"error":except});
    }    
}

export async function getAllUsers(query: MySQLPromisePool) : Promise<GeneralResponse>{
    try {
        const connectionInstance = await query.getConnection();
        const rows = await connectionInstance.query<MySQLRowDataPacket[]>("SELECT * FROM exampletable limit 1000;");
        connectionInstance.release();
        
        let bodyRes ={"rows":rows[0]};

        return toGeneralResponse("SQL_200","Usuarios recuperados correctamente",bodyRes);
    } catch (error) {
        let except = (error as Error).message;
        console.log(except)
       
        return toGeneralResponse("SQL_500","Error al recuperar",{"error":except});
    }
}

````

A partir de esto ya podremos codear con angular.

¿Qué es angular? un SPA con pre estructura, más robusto.... como odio el front, ya no me acuerdo de muchas cosas.

Ay dios, pero bueno, iremos construyendo algo sencillo.
