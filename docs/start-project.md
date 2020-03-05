# Crear el proyecto

Vamos a implementar el proyecto que implemente las características descritas en el apartado anterior. Vamos a hacer uso de una plantilla ya preparada.

Abrimos la siguiente dirección: [https://github.com/aulasoftwarelibre/bdd-typescript-template](https://github.com/aulasoftwarelibre/bdd-typescript-template). Y pulsamos el botón "Use this template" para crear un nuevo repositorio basado en él.

Clonamos el repositorio en nuestro equipo e instalamos las dependencias con `yarn install`.

## Creación de características

Las características (ficheros `.feature`) deben ir dentro del directorio `features/` de nuestro proyecto.

!!! tip

    Las cajas de ejemplo tienen un icono que, si lo pulsas, permiten copiar el contenido al portapapeles. Úsalo para ir más rápido al copiar el código.

    ![Copiar al portapapeles](images/clipboard.png)

Crearemos dentro de dicho directorio un fichero llamado `menu.feature` con el contenido que describimos en el capítulo anterior.

```gherkin
#language: es
Característica: Pagar un menú
    Reglas:

    - 1 punto por cada euro.
    - 10 puntos equivalen a un descuento de 1 euros.
    - El IVA es del 10%

    Antecedentes:
        Dados los siguientes menús:
        | número | precio |
        | 1      | 10     |
        | 2      | 12     |
        | 3      |  8     |

    Escenario: Ganar puntos al pagar en efectivo
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago en efectivo con 55 euros
        Entonces la factura está pagada
        Y he obtenido 50 puntos

    Escenario: Pagar con dinero y puntos
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago con 10 puntos y 54 euros
        Entonces la factura está pagada
        Y he obtenido 0 puntos

    Escenario: Pagar con puntos
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago con 500 puntos y 5 euros
        Entonces la factura está pagada
        Y he obtenido 0 puntos

    Escenario: Intentar pagar el IVA con puntos
        Dado que he comprado 5 menús del número 1
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago con 550 puntos y 0 euros
        Entonces quedan 5 euros por pagar

    Escenario: Comprar menús de varios tipos
        Dado que he comprado 1 menú del número 1
        Y que he comprado 2 menús del número 2
        Y que he comprado 2 menús del número 3
        Cuando pido la cuenta recibo una factura de 55 euros
        Y pago en efectivo con 55 euros
        Entonces la factura está pagada
        Y he obtenido 50 puntos
```

## Ejecución de cucumber

Ahora que tenemos las pruebas definidas vamos a ejecutar _cucumber_:

```sh
yarn cucumber
```

Obtendremos el resumen de la ejecución de pruebas que contiene la siguiente información:

```sh
5 scenarios (5 undefined)
31 steps (31 undefined)
0m00.000s
```

Lo que significa es que _cucumber_ no reconoce ninguno de los _step_ o pasos de los que se compone cada escenario. Esa parte debemos programarla nosotros. Para ello _cucumber_ nos facilita el trabajo con una serie de _snippets_. Veamos uno:

```typescript
<?php
Given('que he comprado {int} menús del número {int}', function (int, int2) {
// Given('que he comprado {int} menús del número {float}', function (int, float) {
// Given('que he comprado {float} menús del número {int}', function (float, int) {
// Given('que he comprado {float} menús del número {float}', function (float, float2) {
    // Write code here that turns the phrase above into concrete actions
    return 'pending';
});

```

El _step_ se compone de una cabecera con las palabras `Given`, `When` o `Then` y una frase que coincide con la que hayamos determinado en el paso. Los números y las cadenas que se pongan entre comillas se convierten en parámetros del paso. También es posible usar expresiones regulares, pero en esos casos debemos hacerlo a mano. El objetivo es meter todos estos snippets en el archivo de contexto que usa _cucumber_.

Si creamos el archivo _step-definitions/menu.steps.ts_, lo usaremos para introducir todos los snippets que nos ha generado el intérprete.

```typescript
import { Given, Then, When } from "cucumber";

Given("los siguientes menús:", function(dataTable) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Given("que he comprado {int} menús del número {int}", function(int, int2) {
  // Given('que he comprado {int} menús del número {float}', function (int, float) {
  // Given('que he comprado {float} menús del número {int}', function (float, int) {
  // Given('que he comprado {float} menús del número {float}', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pido la cuenta recibo una factura de {int} euros", function(int) {
  // When('pido la cuenta recibo una factura de {float} euros', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pago en efectivo con {int} euros", function(int) {
  // When('pago en efectivo con {float} euros', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("la factura está pagada", function() {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("he obtenido {int} puntos", function(int) {
  // Then('he obtenido {float} puntos', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pago con {int} puntos y {int} euros", function(int, int2) {
  // When('pago con {int} puntos y {float} euros', function (int, float) {
  // When('pago con {float} puntos y {int} euros', function (float, int) {
  // When('pago con {float} puntos y {float} euros', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("quedan {int} euros por pagar", function(int) {
  // Then('quedan {float} euros por pagar', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Given("que he comprado {int} menú del número {int}", function(int, int2) {
  // Given('que he comprado {int} menú del número {float}', function (int, float) {
  // Given('que he comprado {float} menú del número {int}', function (float, int) {
  // Given('que he comprado {float} menú del número {float}', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});
```

Si volvemos a ejecutar behat:

```sh
yarn cucumber
```

Obtenemos algo distinto:

```sh
5 scenarios (5 pending)
31 steps (5 pending, 26 skipped)
0m00.001s
```

Ya los escenarios no están como _undefined_ sino como _pending_.

## Implementando el primer _step_

El primer _step_ es el que corresponde con la parte de antecedentes:

```gherkin hl_lines="10"
#language: es
Característica: Pagar un menú
    Reglas:

    - 1 punto por cada euro.
    - 10 puntos equivalen a un descuento de 1 euros.
    - El IVA es del 10%

    Antecedentes:
        Dados los siguientes menús:
        | número | precio |
        | 1      | 10     |
        | 2      | 12     |
        | 3      |  8     |
```

Que corresponde al siguiente _snippet_

```typescript hl_lines="14 16"
import { Given, Then, When } from "cucumber";

Given("los siguientes menús:", function(dataTable) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});
```

Y aquí nos surge la primera necesidad, necesitamos una clase para almacenar menús.

## Jest

Jest es un framework de pruebas para Javascript.Principalmente lo que vamos a indicar con jest es la API de nuestra clase con el resto del dominio.

Por el momento, para poder pasar la prueba que falla "Dados los siguientes menús", necesitamos una clase que nos ofrezca información del número de menú y del precio. Así que vamos a empezar a describir nuestra clase con la ayuda de _jest_.

Creamos los siguientes ficheros:

```typescript
// src/restaurant/menu.ts
class Menu {}

export default Menu;
```

```typescript
//src/restaurant/menu.spec.ts
import Menu from "./menu";

describe("Menu class", () => {
  let menu = new Menu();

  it("is initializable", () => {
    expect(menu).toBeInstanceOf(Menu);
  });
});
```

Ejecutamos las pruebas y obtenemos lo siguiente:

```console
$ yarn test
yarn run v1.17.3
$ jest
 PASS  src/restaurant/menu.spec.ts
  Menu class
    ✓ it is initializable (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.409s, estimated 2s
Ran all test suites.
Done in 2.01s.
```

Vamos a seguir especificando los requisitos de nuestra clase para pasar la prueba. Concretamente necesitamos que nuestra clase sea capaz de indicar el número de menú y el precio. Vamos a escribir la especificación y la comentamos a continuación. Modificamos nuestro `menu.spec.ts`:

```typescript
import Menu from "./menu";

describe("Menu class", () => {
  const OPTION = 10;
  const PRICE = 2500;

  let menu: Menu;

  beforeEach(() => {
    menu = new Menu(OPTION, PRICE);
  });

  it("has a menu option number", () => {
    expect(menu.option()).toBe(OPTION);
  });

  it("has a price", () => {
    expect(menu.price()).toBe(PRICE);
  });
});
```

Las líneas 13 y 17 especifican los dos comportamientos que esperamos de nuestra clase, devolver el número y devolver el precio. Pero antes de devolver nada esa información debe incorporarse a través del constructor. Para ello usamos la función _beforeEach_ (línea 9), que sirve para configurar la prueba en su comienzo. En este caso, la línea 10 construye la clase con el número y el precio del menú. El uso de constantes es para ser más descriptivo a la hora de leer la prueba. Ya que hemos especificado como se construye la clase, especificamos los otros dos comportamientos.

!!! info

    Estamos usando euros para los ejemplos. En realidad, y dado que TypeScript no tiene un tipo de datos para datos financieros, deberíamos usar alguna clase _Moneda_ o guardar los datos en céntimos para evitar el uso de decimales. Para simplificar el tutorial vamos a usar céntimos. Así que aunque en los test usemos euros en la clase _Menu_ vamos a almacenar el valor en céntimos.

Para indicar el número de menú, indicamos que llamamos a un método number() que debe devolver el mismo valor que se pasó al constructor. Para indicar el precio, lo mismo pero llamando a un método price().

Ejecutamos las pruebas y vemos los errores que obtenemos:

```console
$ yarn test
yarn run v1.17.3
$ jest
 FAIL  src/restaurant/menu.spec.ts
  ● Test suite failed to run

    TypeScript diagnostics (customize using `[jest-config].globals.ts-jest.diagnostics` option):
    src/restaurant/menu.spec.ts:10:21 - error TS2554: Expected 0 arguments, but got 2.

    10     menu = new Menu(OPTION, PRICE);
                           ~~~~~~~~~~~~~
    src/restaurant/menu.spec.ts:14:17 - error TS2339: Property 'option' does not exist on type 'Menu'.

    14     expect(menu.option()).toBe(OPTION);
                       ~~~~~~
    src/restaurant/menu.spec.ts:18:17 - error TS2339: Property 'price' does not exist on type 'Menu'.

    18     expect(menu.price()).toBe(PRICE);
                       ~~~~~

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        2.677s
```

Ahora solo queda implementar la funcionalidad para pasar la especificación:

```typescript
class Menu {
  constructor(private _option: number, private _price: number) {}

  option(): number {
    return this._option;
  }

  price(): number {
    return this._price;
  }
}

export default Menu;
```

Ejecutamos las pruebas y comprobamos que pasan:

```console
$ yarn test
yarn run v1.17.3
$ jest
 PASS  src/restaurant/menu.spec.ts
  Menu class
    ✓ has a menu option number (5ms)
    ✓ has a price (1ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        2.737s
Ran all test suites.
Done in 3.45s.
```

## Terminar de implementar los antecedentes

Ahora podemos programar el paso para ir progresando en nuestros casos de uso. Editamos el fichero `step-definitions/menu.steps.ts`.

```typescript
import { Before, Given, TableDefinition, Then, When } from "cucumber";

import Menu from "../src/restaurant/menu";

let menus: Menu[];

Before(() => {
  menus = [];
});

Given("los siguientes menús:", function(dataTable: TableDefinition) {
  dataTable
    .rows()
    .forEach(
      values =>
        (menus[values[0]] = new Menu(
          Number(values[0]),
          100 * Number(values[1])
        ))
    );
});

Given("que he comprado {int} menús del número {int}", function(int, int2) {
  // Given('que he comprado {int} menús del número {float}', function (int, float) {
  // Given('que he comprado {float} menús del número {int}', function (float, int) {
  // Given('que he comprado {float} menús del número {float}', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pido la cuenta recibo una factura de {int} euros", function(int) {
  // When('pido la cuenta recibo una factura de {float} euros', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pago en efectivo con {int} euros", function(int) {
  // When('pago en efectivo con {float} euros', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("la factura está pagada", function() {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("he obtenido {int} puntos", function(int) {
  // Then('he obtenido {float} puntos', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

When("pago con {int} puntos y {int} euros", function(int, int2) {
  // When('pago con {int} puntos y {float} euros', function (int, float) {
  // When('pago con {float} puntos y {int} euros', function (float, int) {
  // When('pago con {float} puntos y {float} euros', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Then("quedan {int} euros por pagar", function(int) {
  // Then('quedan {float} euros por pagar', function (float) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});

Given("que he comprado {int} menú del número {int}", function(int, int2) {
  // Given('que he comprado {int} menú del número {float}', function (int, float) {
  // Given('que he comprado {float} menú del número {int}', function (float, int) {
  // Given('que he comprado {float} menú del número {float}', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});
```

Ejecutamos _cucumber_ y vemos que ya hay pruebas que pasan:

```console
$ yarn cucumber
5 scenarios (5 pending)
31 steps (5 pending, 21 skipped, 5 passed)
0m00.001s
```
