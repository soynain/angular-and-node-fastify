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

*Inyección de dependencias*

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

*Pipes*

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

Iremos tocando los últimos conceptos a validar en Angular, también hay que incluir los shared resources y debounce.

*Routing*

También básico, yo recuerdo en Vue que podías aplicar routing solo sobre un componente y mantener header y footer intactos.

Si quieres cambiar simplemente el prefijo main de la url, lo declaras así:

````main.ts
import { Routes } from '@angular/router';
import { App } from './app';

export const routes: Routes = [
    {
        path:'',
        redirectTo:'mainPage/home',
        pathMatch:'full'
    },
    {
        path:'mainPage/home',
        title:'MainApp',
        component:App
    }
];

````

Cada vez que vayas a localhost taiz te va a redirigir a tu prefijo declarado, también no puedes empezar los paths con una diagonal al inicio
aunque la documentación te indique lo contrario.

<img width="2056" height="480" alt="image" src="https://github.com/user-attachments/assets/da9bf166-553b-453a-b483-e176ce5d30b6" />

Con un pequeño twist, ya supe como hacerle, en tu app html importas el router-outlet dentro de tu section:

````main.html
<style>
  
</style>
<header style="border: 2px solid black;">
  HEADER DE TU PÁGINA PRINCIPAL
</header>
<main class="main">
  <div class="content">
    <p>{{ stringEx}}</p>

    <router-outlet/>
  </div>
</main>
<footer style="border: 2px solid black;">Footer de tu pagina principal, disponible en todos tus components</footer>
````
El app ts lo dejas vacio solo con RouterOutlet y en routes importas otro componente encapsulando nuestras prácticas:

````main.ts
@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css'
})


export class App {
  stringEx = "Bienvenido a está website para prácticas sencillitas de angular";
}

/*Y en otro componente agrupas tu section con tus funcionalidades*/

@Component({
    selector:'homepage',
    imports:[FormRoot,SignalExample,InputComponentExample],
    templateUrl:'./homepage.html'
})

export class HomePage{}

/**Y en routes declaras el principal, tu app solo se vuelve un gateway que usas con routeroutler**/
export const routes: Routes = [
    {
        path:'',
        redirectTo:'mainPage/home',
        pathMatch:'full'
    },
    {
        path:'mainPage/home',
        title:'MainApp',
        component:HomePage
    }
];
````

Para simular un login con state, use NgRX, ya que implementan varias estrategias de store. La más recientes es signal store.

En vue js los stores más modernos se manejan por pinia que es sencillo, en versiones antiguas de vue se usaba vuex.

En angular es el store común que requiere varios archivos, signal store te simplifica eso. La neta no le entendí a la docu, pero este blog
me sirvió para entenderlo bien, se los recomiendo: https://www.codeabien.com/blog/ngrx-signal-store

La estructura de mi store queda así:

````main.ts
import { patchState, signalStore, withMethods, withState } from '@ngrx/signals';
export interface UserState {
    isLoggedIn: boolean;
    userInfo: {
        name: string;
        email: string;
    } | null;
}

/**De acuerdo a la docu, mockear un fetch */
const initialState: UserState = {
    isLoggedIn: false,
    userInfo: {name: 'n/a',email: 'n/a'}
};

/**La misma simplicidad que pinia */
export const UserStore = signalStore({ providedIn: 'root' },
    withState<UserState>(initialState),
    withMethods((store)=>({
        saveUserSession(user: UserDetails){
            patchState(store,{userInfo:{'name':user.name,'email':user.email}}) //con patch seteas las propiedades
            patchState(store,{isLoggedIn:true});
            console.log("AUTH STORE INICIALIZADO")
        }
    })),
    withHooks({
        onInit(store){
            const saved = localStorage.getItem('session_data');
            if (saved) {
                patchState(store, JSON.parse(saved));
            }

            // 2. Efecto reactivo: cada vez que el store cambie, guardamos en localStorage
            effect(() => {
                const state = {
                    userInfo: store.userInfo(),
                    isLoggedIn: store.isLoggedIn()
                };
                localStorage.setItem('session_data', JSON.stringify(state));
            });
        }
    })
);
````
El  withHooks sirve para mantener las cosas en tu localStorage, porque no importa que uses stores, todos
dependen del local storage.

En pinia la sintaxis es así, esto es fragmento de un proyecto x... donde implementé pinia para un login hace ya años:

<img width="1012" height="874" alt="image" src="https://github.com/user-attachments/assets/803dde02-15a1-4f55-99cc-c6af39b20a58" />

Siguen la misma lógica si te das cuenta: estado inicial, acciones e instancia, y los puedes instanciar en cualquier parte de tu aplicativo.

Creamos un botton mock y programamos su comportamiento, instanciando el método del store que podemos inyectar como Provider:

````main.ts
`<button (click)="testStore()">Login solo con boleano</button>`

@Component({
   /***/
    providers:[UserStore]
})

export class InputComponentExample{
 /*.........*/

    testStore(){
        console.log(this.userStory.userInfo())

        let mockLoginByButton:UserDetails = {name:'Moisexy',email:'moisexy@godly.mx'}

        this.userStory.saveUserSession(mockLoginByButton);
    }
    
    readonly userStory = inject(UserStore);
````

Y así podemos cambiar un booleano guardado como estado a true, util para nuestros routings protegidos.

Ahora para simular rutas protegidas, hacemos lo sig:

declara tu ruta con el tag "canActicate", después

````main.ts
 {
        path:'logged',
        title:'Felicidades',
        component:LoggedIn,
        canActivate:[basicGuard],
}

export const basicGuard: CanActivateFn= (route: ActivatedRouteSnapshot,state: RouterStateSnapshot)=>{
    const userStore = inject(UserStore);
    const router = inject(Router);

    if(!userStore.isLoggedIn()){
        const loginPath = router.parseUrl("/mainPage/home");
        return new RedirectCommand(loginPath, {
          skipLocationChange: true,
        });
    }
    
    console.log('pre guard '+userStore.isLoggedIn())
    return userStore.isLoggedIn();
}
````

Y crear una función canActivate, hay varios tipos de función de rooteo que puedes encadenar en paths.

Con esto si no estás logueado te redirife a tu homePage, y si lo estás, te permitirá acceder a tu componente:

<img width="2546" height="1439" alt="image" src="https://github.com/user-attachments/assets/d6ffa143-8859-472d-a8f0-2a43fe62714d" />


Si visito manualmente el path

<img width="2442" height="1332" alt="image" src="https://github.com/user-attachments/assets/685fa5db-64f1-4efa-a910-a53b6bba42a7" />

Ya me permitirá ver mi componente, de lo contrario no.

Con esto ya dominamos los básicos de angular ciertamente, al menos lo moderno.

Dejaré el tópico de testing a lo último, pero no creo que haya tanto problema en eso.

Por último, este parametro
````main.ts
return new RedirectCommand(loginPath, {
          skipLocationChange: false,
        });
````
Más info aquí:

https://angular.dev/api/router/RedirectCommand



Dejalo en false, es para que cuando te haga el redirect, cambie tu url al redirigir!

Ahora solo faltaria ver testing, y lit element, aunque tal vez el último framework me lo salte. Sería testing pero
yo creo pospondré tantito ese tópico para rotar a algun tema de back.

Y así de simple, hemos dominado la versión 20 de Angular ^^.

El resto de detalles como debounce y esas cosillas las checaré ya en un trabajo. La verdad es que... para eso existe la documentación por dios.

Para angular se ha tratado de ocupar chatgpt pocas veces, por lo tanto puedo constatar que su documentación es muy fabulosa y buena, y enseña bien las bases.

Si vienes de Vue js, no te va a costar trabajar. 

Con esto ya hemos cubierto las básicas sobre este tópico... tal vez añadamos pruebas....

Por último, pruebas de cobertura básicas:

El reporte te lo muestra así:

<img width="955" height="649" alt="image" src="https://github.com/user-attachments/assets/1c1778b7-4dfe-4ffd-8a44-35840fd26c9d" />

Te indica las líneas a probar en rojo.

````main.ts
import { TestBed } from '@angular/core/testing';
import { App } from './app';
import { FormRoot } from './modules/UserForm/module';
import { HomePage } from './modules/HomePageComponent/module';
import { SignalExample } from './modules/ComputedExample/module';
import jasmine from 'jasmine';
import { InputComponentExample } from './modules/AnotherForm/module';


describe('App', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [App],
    }).compileComponents();
  });

  it('should create the app', () => {
    const fixture = TestBed.createComponent(App);
    const app = fixture.componentInstance;
    expect(app).toBeTruthy();
  });

  it('should render title', async () => {
    const fixture = TestBed.createComponent(App);
    await fixture.whenStable();
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('p')?.textContent).toContain('Bienvenido a está website para prácticas sencillitas de angular');
  });


});


describe('FormRoot', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [HomePage],
    }).compileComponents();
  });

  it('Deberia crear el formulario', async() => {
    const fixture = TestBed.createComponent(HomePage);
    const app = fixture.componentInstance;
    await fixture.detectChanges();
    expect(app).toBeTruthy();
  });

  it('El init lifecycle debe bloquear caracteres especiales en el campo nombre y mostrar un texto indicando el error',async()=>{
    //based on formRoot.ts, create a test to validate the formgroup and onSubmit event
    const fixture = TestBed.createComponent(FormRoot);
    const app = fixture.componentInstance;

    await fixture.whenStable();
    fixture.detectChanges();

    const compiled = fixture.nativeElement as HTMLElement;

    const nameControl = app.userModel.get('name');
    nameControl?.setValue('John Doe@');

    expect(app.warningInvalidCharsCustom()).toEqual(true);
    await fixture.whenStable();
    fixture.detectChanges();
    expect(compiled.querySelector('p')?.textContent).toContain('El nombre no puede tener caracteres especiales')
    
  });


});

describe('SignalExample', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [SignalExample],
    }).compileComponents();
  });

  
  it('Deberá cambiar el valor signal del string a otro valor con la función changeVal()',async()=>{
    const fixture = TestBed.createComponent(SignalExample);
    const app = fixture.componentInstance;

    await fixture.detectChanges();

    expect(app.getValor()).toEqual('gooks');

    app.changeVal();

    await fixture.detectChanges();

    expect(app.getValor()).toEqual('reactividad probada');
  });

  it('Deberá esconder bloques de ejemplo con hideBlock() en el html y cambiar el valor de esa variable signal',async()=>{
    const fixture = TestBed.createComponent(SignalExample);
    const app = fixture.componentInstance;

    await fixture.detectChanges();
    let compiled = fixture.nativeElement as HTMLElement;

    expect(app.getHideBlock()).toEqual(false)

    let blockElement = compiled.querySelector('#blockHidden');
    expect(blockElement).toBeTruthy();
    expect(window.getComputedStyle(blockElement!).visibility).toEqual('visible');

    app.hideBlockAction();

    await fixture.detectChanges();
    compiled = fixture.nativeElement as HTMLElement;

    expect(app.getHideBlock()).toEqual(true)

    blockElement = compiled.querySelector('#blockHidden');
    expect(blockElement).toBeTruthy();
    expect(window.getComputedStyle(blockElement!).visibility).toEqual('hidden');
  });
  it('Al activar el intervalo se mostrará un elemento por 5 segundos, pasado los 5 segundos, deberá ocultar otra vez un elemento html',async()=>{
    const fixture = TestBed.createComponent(SignalExample);
    const app = fixture.componentInstance;

    await fixture.detectChanges();

    expect(app.getTempInterval()).toEqual(false);

    app.ctrlCInputAction(new Event('input.keyup.control.c'));
  //  expect(app.getTempInterval()).toEqual(true);
    await fixture.whenStable(); 
    fixture.detectChanges();

    expect(app.getTempInterval()).toEqual(true);
    
    await new Promise((resolve)=>setTimeout(resolve,5000))

    expect(app.getTempInterval()).toEqual(false);
    
  },7000);

}); 


describe('InputComponentExample', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [InputComponentExample],
    }).compileComponents();
  });

  it('Debe iniciar sesión correctamente',async()=>{
    const fixture = TestBed.createComponent(InputComponentExample);
    const app = fixture.componentInstance;

    await fixture.detectChanges();
    expect(app.userStory.isLoggedIn()).toEqual(false);
    app.testStore();

    await fixture.detectChanges();
    expect(app.userStory.isLoggedIn()).toEqual(true);

  })

});

````
Al final me faltó unos cuantos tests, no pude replicarlo por un tema con la implementación del suscribe, entonces
la cobertura te obliga a validar otra manera de testear:

<img width="789" height="573" alt="image" src="https://github.com/user-attachments/assets/6c4999db-6226-4286-aca1-eb313e100395" />

Con esto ya finalizamos el tema de angular y fastify en basics.
