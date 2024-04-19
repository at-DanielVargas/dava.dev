---
title: "Cómo Integrar Redux con Angular "
author: Daniel Vargas
pubDatetime: 2024-04-19T17:00
slug: como-integrar-redux-con-angular
featured: false
draft: false
tags:
  - Angular
  - Redux
description: "En el desarrollo de aplicaciones web modernas, mantener un estado
  predecible es fundamental para la creación de experiencias de usuario fluidas
  y confiables. "
---
En el desarrollo de aplicaciones web modernas, mantener un estado predecible es fundamental para la creación de experiencias de usuario fluidas y confiables.

Angular, uno de los frameworks más populares para el desarrollo de aplicaciones web, brinda una estructura sólida y eficiente para construir aplicaciones robustas. Sin embargo, cuando se trata de gestionar estados complejos y dinámicos, la integración con Redux se convierte en una solución poderosa.

Redux, un contenedor de estado predecible para aplicaciones JavaScript, ofrece una manera excelente de manejar el flujo de datos y el estado en aplicaciones complejas.

Comenzando por el principio, antes de sumergirnos en la integración de Redux con Angular, es esencial comprender los principios fundamentales de Redux y sus tres principios clave:

1.  **Un único origen de verdad**: El estado de toda tu aplicación se almacena en un árbol de estado único, lo que facilita el seguimiento de cambios y la depuración.
    
2.  **El estado es de solo lectura**: La única forma de cambiar el estado es emitiendo acciones, objetos que describen qué debe suceder.
    
3.  **Los cambios se realizan con funciones puras**: Para especificar cómo el árbol de estado se transforma por las acciones, se utilizan reducers, que son funciones puras.
    

Comprende el Flujo de Datos Unidireccional

Redux se basa en un flujo de datos unidireccional, lo que significa que los datos en tu aplicación fluyen en una sola dirección. Este concepto puede ser diferente a lo que algunos desarrolladores están acostumbrados. Familiarizarse con este flujo es esencial para utilizar Redux efectivamente.

Piensa en Acciones como Eventos, No Comandos

En Redux, las acciones son mejor entendidas como eventos que describen algo que ocurrió, en lugar de comandos que dictan lo que debe suceder. Esta mentalidad ayuda a diseñar una arquitectura más flexible y escalable.

Acepta la Verbosidad por Claridad

A primera vista, Redux puede parecer verboso debido a su estructura de acciones, reducers y el store. Sin embargo, esta verbosidad contribuye a una mayor claridad y previsibilidad del estado de la aplicación. Aceptar y abrazar esta característica desde el principio facilitará la implementación.

Ahora si pongamos manos a la obra y vamos a integrar Redux con Angular de manera efectiva. Generalmente se realiza a través de la biblioteca NgRx, una implementación de Redux para Angular que aprovecha RxJS, ofreciendo una gestión de estado reactiva y poderosa.

A continuación, se describe un enfoque paso a paso para integrar Redux con Angular utilizando NgRx:

Para empezar, instala NgRx en tu proyecto Angular ejecutando el siguiente comando en tu terminal. Este comando instalará las dependencias necesarias para trabajar con NgRx

```bash
npm install @ngrx/store @ngrx/effects @ngrx/store-devtools
```

Configuración del Store
Después de instalar NgRx, el siguiente paso es configurar el store. En tu módulo principal de Angular `(usualmente app.module.ts)`, importa `StoreModule` de `@ngrx/store` y `EffectsModule` de `@ngrx/effects`, y añádelos al array de imports de tu módulo con la configuración inicial del estado y los reducers.

```typescript
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { reducers, metaReducers } from './reducers';

@NgModule({
  declarations: [
    AppComponent,
    // otros componentes
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    StoreModule.forRoot(reducers, {
      metaReducers,
      runtimeChecks: {
        strictStateImmutability: true,
        strictActionImmutability: true,
      }
    }),
    EffectsModule.forRoot([]),
    // otros módulos
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
En este ejemplo, reducers es un objeto que mapea los estados a sus respectivos reducers. La configuración `strictStateImmutability` y `strictActionImmutability` asegura que el estado y las acciones sean inmutables, lo cual es una buena práctica en Redux.

Definición de Acciones y Reducers
Define acciones y reducers para manejar los cambios en el estado de tu aplicación. Primero, crea las acciones utilizando `createAction` de `@ngrx/store`.

```typescript
import { createAction, props } from '@ngrx/store';

export const loadItems = createAction('[Item List] Load Items');
export const addItem = createAction('[Item List] Add Item', props<{ item: string }>());
```

Luego, define un reducer utilizando `createReducer` y on de `@ngrx/store`, especificando cómo el estado cambia en respuesta a las acciones.

```typescript
import { createReducer, on } from '@ngrx/store';
import * as ItemActions from './item.actions';

export const initialState: ReadonlyArray<string> = [];

export const itemReducer = createReducer(
  initialState,
  on(ItemActions.addItem, (state, { item }) => [...state, item]),
);

```

Inyectando el Store en Componentes
Para utilizar el store en tus componentes Angular, inyéctalo en el constructor del componente y selecciona datos del estado o despacha acciones según sea necesario.

```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import * as fromRoot from './reducers';
import * as itemActions from './item.actions';

@Component({
  selector: 'app-item-list',
  templateUrl: './item-list.component.html',
  styleUrls: ['./item-list.component.css']
})
export class ItemListComponent {
  items$: Observable<string[]>;

  constructor(private store: Store<fromRoot.State>) {
    this.items$ = store.select(state => state.items);
  }

  addItem(item: string) {
    this.store.dispatch(itemActions.addItem({ item }));
  }
}
```

En este ejemplo, `items$` es un `Observable` que representa la lista de ítems del estado. La función addItem despacha una acción para agregar un nuevo ítem al estado.