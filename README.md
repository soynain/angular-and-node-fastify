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

¿Qué es angular? un SPA con pre estructura, más robusto. Empezemos, empezaremos por los básicos. Se pueden injertar variavles directamente al html, cosa que con vue no como tal, declarabas 
componentes vue.

<img width="1307" height="761" alt="image" src="https://github.com/user-attachments/assets/89334522-1969-4384-ba81-4314ae6f67b3" />


La estructura del proyecto consiste en el css de tu aplicativo, el routes.ts para que hagas tus rutas al spa,
spec.ts para pruebas unitarias y el ts para lo demás

<img width="371" height="288" alt="image" src="https://github.com/user-attachments/assets/bc7c8fcb-8b93-493b-baa5-15946339e20c" />

Ya medio voy recordando el pex, las spa eran pa los componentes, y podias reutilizar footer y header, y dentro del bloque body cambiar el contenido
me parece de acuerdo a las directivas que quisieras usar:

Aquí nomás ando recordando, porque aunque sea fullstack el front es lo que menos disfruto, aunque he de presumir que cada maqueta que me han asignado,
lo he podido replicar en responsive sin problemas... así que tampoco soy de subestimar

Pero asi era el tema, y aquí:

<img width="1823" height="877" alt="image" src="https://github.com/user-attachments/assets/15b40585-eea1-4ba9-afe7-faf0b3a96efa" />

Ya me ando involucrando con esas reactividades. Lo más importante al manipular los fronts son los forms, secciones,
componentes y las directivas ifs, los renders sobre html y esas madre, y los keys para recgargar los componentes.

Son cosas que medio recuerdo de vue js, y que ya tengo códigos.

Para forms es de esta manera:

````main.ts
import { Component, signal } from '@angular/core';
import { FormControl, FormGroup, FormsModule, ReactiveFormsModule, Validators } from '@angular/forms';
import { form, FormField } from '@angular/forms/signals';
import { RouterOutlet } from '@angular/router';

@Component({
    selector: 'form-root',
    imports: [ReactiveFormsModule],
    templateUrl: '/form.html',
    styleUrl: '/form.css'
})


export class FormRoot {

    userModel = new FormGroup({
        id:new FormControl(0,[Validators.min(1)]),
        name:new FormControl("",[Validators.required]),
        age:new FormControl(0,[Validators.min(1)])
    })


    onSubmit(){
        console.log(this.userModel.value)
    }
}
````

Estructurita del form:
````form.html
<form [formGroup]="userModel" (ngSubmit)="onSubmit()">
    <label for="id">ID:</label>
    <input type="number" id="id" formControlName = "id">

    <label for="name">Name:</label>
    <input type="text" id="name"  formControlName="name">

    <label for="age">Age:</label>
    <input type="number" id="age" formControlName="age">

    <button type="submit" [disabled]="!userModel.valid">Submit</button>
</form>
````

Integración al main app:

````main.ts
import { Component, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { FormRoot } from './modules/module';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet,FormRoot],
  templateUrl: './app.html',
  styleUrl: './app.css'
})


export class App {
  protected readonly title = signal('angular-basics');
  stringEx = "Hola mundo";
}
````

Integración a tu html raiz:
````root.html
<style>
  
</style>

<main class="main">
  <div class="content">
    {{ stringEx}}

    <form-root></form-root>
  </div>
</main>

<router-outlet />

````

Y la estructura primeriza, aun no visualizo porque dicen que angular tiene un marco de arquitectura por default. A lo mejor por la anotación 
component

<img width="507" height="351" alt="image" src="https://github.com/user-attachments/assets/311fe9c0-5685-45d7-a9d4-78508baa04b1" />


Iré probando otros conceptos para volverme más familiar y, también veremos rooteo para probar lo del footer y header.

Por lo que veo de forms, si tuvieras un form catcheado y solo quisieras modificar partes de un form (PATCH), usas esto:

<img width="1528" height="681" alt="image" src="https://github.com/user-attachments/assets/c4503e4e-457f-48d1-b4bc-37f8ccd59a75" />

Tienes dos formas de hacer un form, por builder y por formControl:

<img width="1603" height="944" alt="image" src="https://github.com/user-attachments/assets/372c119e-8fca-4996-9958-b073b2d32af4" />


Otro concepto son los pipes y suscribes de RxJS. Puedes tener listeners sobre un form, los cuales tienen métodos de eventos por default. En un form puedes suscribirte a los eventos del form mismo por field o el form mismo:

<img width="1990" height="844" alt="image" src="https://github.com/user-attachments/assets/04af9bef-6178-4cb9-8324-9dd66ffca01d" />

En base a ciertas condicionales, podrias crear un label de validación como se muestra ahí, o desactivar secciones de tu form para
que no proceda hasta corregir secciones o inputs correctamente.

````main.ts
export class FormRoot implements OnInit,OnDestroy{

    private eventSubscription!: Subscription;
    warningInvalidCharsCustom = "";

    userModel = new FormGroup({
        id:new FormControl(0,[Validators.min(1)]),
        name:new FormControl("",[Validators.required]),
        age:new FormControl(0,[Validators.min(1)])
    })

    ngOnDestroy(): void {
       //  this.userModel.events.pipe().
       this.eventSubscription.unsubscribe()
    }
    ngOnInit(): void {
        this.userModel.get('name')?.valueChanges.subscribe(value=>{
            if(value?.match(/[ `!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?~]/)){
                this.warningInvalidCharsCustom = "Tu nombre no debe contener caracteres especiales"
            }else{
                this.warningInvalidCharsCustom = ""
            }
            console.log("Una forma de escuchar los cambios de un input "+value)
        })

        this.eventSubscription = this.userModel.events.pipe(filter(e=>e instanceof StatusChangeEvent))
        .subscribe((st)=>console.log('Status: '+st.status))
    }

    onSubmit(){
        console.log(this.userModel.value)
    }
}
````

También parece ser los componentes tienen un ciclo de vida, y los pipes también deben tener una desuscripción
para no tener memory leaks.

Nos adentramos en otros conceptos implicitos pero ya acabamos con esto la sección de forms que es la más importante.

Tópicos checados:

Inyección de dependencias:

Es posible crear inyección de dependencias por constructor o por inject (lo medio equivalente a un @Autowired en spring boot)

<img width="1553" height="611" alt="image" src="https://github.com/user-attachments/assets/c7e310b5-c052-4a36-8e64-efe624714796" />

````main.ts
export class SignalExample implements OnInit{
    ngOnInit(): void {
        this.signalComputed.initComputedProperty(this.valor)
    }
    protected readonly title = signal('angular-basics');
    protected valor = signal('gooks')
    protected signalComputed: UpperCaseSignal = inject(UpperCaseSignal);


    

    changeVal() {
        this.valor.set('reactividad probada')
    }
}
````
````main.ts
@Injectable({providedIn:'root'})
export class UpperCaseSignal{

    protected computedProperty!:Signal<string>

    initComputedProperty(value: Signal<string>){
        this.computedProperty = computed(()=>value().toUpperCase());
    }

    getComputedProperty(){
        return this.computedProperty()
    }
}
````
Y en html:

````main.html
<section>
    <p>Ejemplos de reactividad e injección de dependencias</p>
    
    <p>Valor signal reactivo: {{valor()}}</p>

    <p>Valor reactivo con mayúsculas: {{signalComputed.getComputedProperty()}}</p>

    <button (click)="changeVal()">Enviar signal a variable para reactividad</button>
</section>
````

Y al clickear el botón, cambias el contenido de la variable reactiva y también el computed cambia en base al contenido.

Los computed son variables que te hacen calculos en automático, imagina una calculadora de simulación
de presupuesto de hipoteca por año con intereses, eso te lo calcula.

También el anhidar booleanos a los estilos por elemento html, lo básico:

````main.html
<section>
        <p [style.visibility]="!hideBlock() ? 'visible' : 'hidden'">Este elemento debería esconderse si le doy click a mi botón</p>

        <button (click)="hideBlockAction()">Esconder bloque con signal asignado</button>
</section>
````

````main.ts
protected hideBlock = signal(false)
    hideBlockAction(){
        !this.hideBlock() ? this.hideBlock.set(true) : this.hideBlock.set(false);
    }
````

Otro punto son los eventos por tecla en input, vienen útiles y también puedes añadir el truco de timeouts al hacer invocaciones ^^

<img width="776" height="138" alt="image" src="https://github.com/user-attachments/assets/ba9b50bd-50d9-48aa-b3ed-4b96a49bffd5" />

````main.html
<section>
        <input type="text" (keyup.control.c)="ctrlCInputAction($event)">

        @if(tempInterval()){
            <p>Texto copiado al portapapeles (simulación de timeout con teclas)</p>
        }
    </section>
````

````main.ts
protected tempInterval = signal(false)
    ctrlCInputAction(event: Event){
        this.tempInterval.set(true);

        setTimeout(()=>{
            this.tempInterval.set(false);
        },5000)
    }
````

Entonces para tus alerts customizados con teclas viene perfecto, se oculta en 5 segundos.

Volviendo a la práctica del angulas basics, veremos dos nuevos conceptos: 

*Inputs*

Hay unos inputs de anotación que te permiten crear tus componentes customizados
con tus propios parametros, muy buenos para lógica explicita:

````component.html
<section>
    <header>Aquí en este componente haremos el uso de los inputs:</header>

    <p>Los inputs son componentes donde puedes definir sus propiedades customizadas</p>

    <moyi-componente userName="Moisexy" userAge="26" credentialPrice="1393546.98"></moyi-componente>
</section>
````

Quiero un componente que me despliegue datos formateados, en código lo creas asi:

````main.ts
import { CurrencyPipe } from "@angular/common";
import { Component, Input, numberAttribute } from "@angular/core";

@Component({
    selector:'moyi-componente',
    imports:[CurrencyPipe],
    template:`
        <section>
        <p>Nombre del usuario: {{userName}}</p>
        <p>Edad del usuario: {{userAge}}</p>
        <p>Deuda: {{credentialPrice | currency}}</p>
    </section>
    `
})

export class MoyiComponente{
    @Input({required: true}) userName!: string;
    @Input({required: true,transform:numberAttribute}) userAge!:string;
    @Input({required:false}) credentialPrice!:string;
}
````

Lo declaras en un componente main vacio, o con otra lógica:

````main.ts
import { Component } from "@angular/core";
import { MoyiComponente } from "./moyicomponente";

@Component({
    imports:[MoyiComponente],
    templateUrl:'./component.html',
    selector: 'input-example-section'
})

export class InputComponentExample{}
````

Y lo adaptas a tu app.ts. Al final este será el resultado:

<img width="924" height="342" alt="image" src="https://github.com/user-attachments/assets/45fa2f4a-ff45-40e7-b5ce-2f7641b930ad" />

Un componente que usa pipes, los pipes es otro concepto, componentes que sirven para transformar datos, angular trae unos por default pero tu puedes crear
tus propios datos.

Aquí aunque no es necesario, por temas de demostración hicimos un pipe para, manualmente parsear todo el array con formato currency:

````main.ts
import { CurrencyPipe, formatCurrency, formatNumber } from "@angular/common";
import { Inject, LOCALE_ID, numberAttribute, Pipe, PipeTransform } from "@angular/core";

@Pipe({
    name:'moyiPipe',
    standalone: true
})

export class MoyiPipe implements PipeTransform{
    constructor(
    @Inject(LOCALE_ID) public locale: string,){}
    transform(value: string[]) {
       return value.map((e:string,index:number)=>formatCurrency(Number.parseFloat(e),this.locale,"$"));
    }

}
````

Con standalone no lo anhidas como ngModel. Y lo implementas con la anotación pipe | :

````main.html
<header>Gastos del mes actual desglosados</header>

    @for(price of getBillsActualMonth() | moyiPipe ; track price){
        <p>{{price}}</p>
    }

    <p>Total de gasto: {{totalArr() | currency}}</p>
````

Y para el total, se crea con un computed property:

````main.ts
export class InputComponentExample{
    protected spendsActualMonth = ["1345623.97","295.56","567.90","4567.88"]

    getBillsActualMonth(){
        return this.spendsActualMonth
    }

    protected totalArr = computed(()=>{return this.spendsActualMonth.map((elem)=>Number.parseFloat(elem)).reduce((prev,curr)=>prev+=curr).toFixed(2) });
    
}
````

<img width="936" height="632" alt="image" src="https://github.com/user-attachments/assets/0359a6ee-7e29-4280-a66b-a64cc871ed76" />

Y listo, puedes crear tus calculadoras dinámicas.

También hay unos input por campo que se usa con doble anhidado [()], pero tienes el formgrouo para muchas cuestiones.

Otro tema a ver por último, serán los stores, routing, testing y, Lit-Element que es una librería que ocupan en BBVA.

No confundan en estos ejemplos la falta de css con nulo conocimiento en front, es probar comportamiento nadamas del framework.
