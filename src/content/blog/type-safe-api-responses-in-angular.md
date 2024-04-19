---
title: Respuestas de api con seguridad de tipos en angular con zod
author: Daniel Vargas
pubDatetime: 2024-04-19T15:27:52Z
slug: type-safe-api-responses-in-angular
featured: true
draft: false
tags:
  - TypeScript
  - Angular
description: Como tener respuestas de api con seguridad de tipos en angular.
---

<img src="/assets/ts-hero.jpeg" alt="Typescript es un Heroe" />

#### El Problema

Imagina que eres un desarrollador front-end. Hay una nueva característica en su proyecto actual que debe implementarse. hablas con el equipo de back-end, se acuerda el contrato de API y comienza la implementación.
Incluso se trabajan pruebas para asegurarse de que el front-end funcione en muchos escenarios diferentes que pueden suceder. Y todo es genial, la función está terminada y todos están contentos.

# !!!Pero

el maldito pero.

Pasa un tiempo y luego la función deja de funcionar valga la redundancia. empiezas a investigar lo que acaba de suceder, pero el código de interfaz parece estar bien. Incluso tus pruebas no fallan.

<iframe src="https://giphy.com/embed/KhliiAkDFP9YY" width="300px" height="100%" frameBorder="0" class="giphy-embed float-right" allowFullScreen></iframe>

pero luego, le hechas un vistazo a la pestaña Red en DevTools de tu navegador. y... te das cuenta que la forma en la que ingresó la respuesta API ya no es es igual. El contrato ha cambiado. Y lo que es peor, los cambios se propagan por toda la aplicación. Por que diablos nadie lo notifico un correo o paloma mensajera por lo menos.

Pero bueno luego de solucionar el problema, empiezas a preguntarte: ¿qué diablos puedo hacer para evitar este tipo de problemas en el futuro?

Aqui entra ZOD.

Una de las soluciones es validar datos externos (por ejemplo, respuestas API, almacenamiento local) antes de usarlos, Eso es algo que podemos lograr usando Zod.

Zod es una pequeña biblioteca que permite definir un esquema y analizar fácilmente los datos, ya sea arrojando un error o no, según el caso de uso.

Veamos algunos ejemplos y cómo solucionar el problema paso a paso.

Hay muchos tipos de datos que Zod admite, pero usemos algo simple: analizar valores de cadenas. Primero, creamos una definición de tipo Zod.

```typescript
import { z } from "zod";

const ZodString = z.string();
const ZodStringEmail = z.string().email();
const ZodStringLength = z.string().min(1).max(10);
```

Observa cómo podemos limitar lo que consideramos válido. Zod puede comprobar no sólo el tipo, sino también si el valor se ajusta a nuestras necesidades (por ejemplo, es una dirección de correo electrónico). Luego podemos usar objetos creados para analizar nuestros datos:

```typescript
ZodString.parse(123); // arroja un error runtime
ZodString.parse("123"); // la validacion pasa y se devuelve el valor seguro

ZodString.safeParse(123); // devuelve un objeto con las propiedades success: false y error  con los detalles
ZodString.safeParse("123"); // devuelve un objeto con las propiedades success: true y el valor verificado
```

Puedes definir mensajes de error personalizados:

```typescript
const ZodStringCustomized = z.string({
  invalid_type_error: "Oh no!",
});
```

Volviendo a nuestro ejemplo con el contrato API, digamos que antes de los cambios, el código responsable de una llamada API tenía este aspecto:

```typescript
interface ApiResponse {
  foo: number;
  email: string;
  bar: ApiResponseBar[];
}

interface ApiResponseBar {
  id: string;
  baz: string;
}

@Injectable({ providedIn: "root" })
export class ExampleHttpService {
  private readonly http = inject(HttpClient);

  fetchData(): Observable<ApiResponse> {
    return this.http.get<ApiResponse>("/api/example");
  }
}
```

En este ejemplo estoy usando Angular 14 y su nueva función inject() para obtener una instancia de HttpClient. También puedes hacerlo usando el constructor.
Con Zod, sería un poco diferente:

```typescript
import { z } from 'zod';

const ZodApiResponse = z.object({
  foo: z.number(),
  email: z.string().email(),
  bar: z.array(
    z.object({
      baz: z.string(),
      id: z.string(),
    })
  ),
});
type ZodApiResponse = z.infer<typeof ZodApiResponse>;

@Injectable({ providedIn: 'root' })
export class ZodHttpService {
  private readonly http = inject(HttpClient);

  fetchData(): Observable<ZodApiResponse> {
    return this.http
      .get('/api/example’)
      .pipe(map((response) => ZodApiResponse.parse(response)));
  }
}
```

Primero hemos creado un tipo de objeto Zod que representa nuestra estructura de datos deseada. Luego, usando z.infer, creamos un tipo TypeScript a partir de él.
La llamada a la API tiene el mismo aspecto, pero después de obtener la respuesta, llamamos a la función parse() para asegurarnos de que el esquema sea correcto. Si hay algún problema, lo sabremos.
Podríamos ir aún más lejos y crear un operador RxJs personalizado para facilitar la validación:

```typescript
export function verifyResponseType<T extends z.ZodTypeAny>(zodObj: T) {
  return pipe(map((response) => zodObj.parse(response)));
}

@Injectable({ providedIn: 'root' })
export class ZodHttpService {
  private readonly http = inject(HttpClient);


  fetchData(): Observable<ZodApiResponse> {
    return this.http
      .get('/api/example’)
      .pipe(verifyResponseType(ZodApiResponse));
  }
}
```

Ahora si papá, si recibimos datos con esquema diferente, por ejemplo:

```json
{
  "differentFoo": 21, // front-end espera “foo”!
  "email": "example@test.com",
  "bar": [
    { "baz": "abc", "id": "id1" },
    { "baz": "def", "id": "id2" }
  ]
}
```

Si esto sucede vermos un mensaje en la consola del navegador como este:

<img src="/assets/zod-error.png" alt="Zod error" />

En la mayoría de las aplicaciones que escribimos como desarrolladores front-end, confiamos en información externa, como respuestas API. Desafortunadamente, dichos datos están fuera del alcance de TypeScript, lo que significa que no podemos estar seguros de si coinciden con nuestras interfaces o no. Podemos solucionar este problema de (al menos) dos formas: generar tipos basados en la API o validarlos en tiempo de ejecución. Si bien lo primero no siempre es posible, ya que requiere cierto esfuerzo por parte del equipo de back-end, lo segundo se puede lograr utilizando una herramienta como Zod.
