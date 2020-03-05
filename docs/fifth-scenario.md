# Quinto escenario

Vamos a implementar el último escenario

```gherkin
Escenario: Comprar menús de varios tipos
    Dado que he comprado 1 menú del número 1
    Y que he comprado 2 menús del número 2
    Y que he comprado 2 menús del número 3
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago en efectivo con 55 euros
    Entonces la factura está pagada
    Y he obtenido 50 puntos
```

Si ejecutamos la prueba:

```sh
$ yarn cucumber features/menu.feature:43
..P------..P------

Warnings:

1) Scenario: Comprar menús de varios tipos # features/menu.feature:43
   ✔ Before # step-definitions/menu.steps.ts:10
   ✔ Dados los siguientes menús: # step-definitions/menu.steps.ts:15
       | número | precio |
       | 1      | 10     |
       | 2      | 12     |
       | 3      | 8      |
   ? Dado que he comprado 1 menú del número 1 # step-definitions/menu.steps.ts:67
       Pending
   - Y que he comprado 2 menús del número 2 # step-definitions/menu.steps.ts:27
   - Y que he comprado 2 menús del número 3 # step-definitions/menu.steps.ts:27
   - Cuando pido la cuenta recibo una factura de 55 euros # step-definitions/menu.steps.ts:37
   - Y pago en efectivo con 55 euros # step-definitions/menu.steps.ts:43
   - Entonces la factura está pagada # step-definitions/menu.steps.ts:47
   - Y he obtenido 50 puntos # step-definitions/menu.steps.ts:51

1 scenario (1 pending)
8 steps (1 pending, 6 skipped, 1 passed)
0m00.001s
```

Falla en el primer paso. Realmente la frase "que he comprado 1 menú del número 1" difiere de "que he comprado 2 menús del número 2" en el plural de _menús_. Al ser frases distintas behat las interpreta como pasos distintos. Necesitamos conseguir que un mismo paso se ejecute con sentencias distintas.

Por lo pronto el código siguiente:

```typescript
Given("que he comprado {int} menú del número {int}", function(int, int2) {
  // Given('que he comprado {int} menú del número {float}', function (int, float) {
  // Given('que he comprado {float} menú del número {int}', function (float, int) {
  // Given('que he comprado {float} menú del número {float}', function (float, float2) {
  // Write code here that turns the phrase above into concrete actions
  return "pending";
});
```

Que pertenece a nuestro _step-definitions/menu.steps.ts_ sobra y lo eliminamos.

## Escribir las reglas como expresiones regulares

Cambiamos el _snippet_ que nos generó _cucumber_ para crear una expresión regular que soporte la frase en plural y en singular:

```typescript
Given(/^que he comprado (\d+) menús? del número (\d+)$/, function(
  units: number,
  option: number
) {
  const menu = menus[option];
  for (let i = 0; i < units; i++) {
    bill.add(menu);
  }
});
```

En el caso de usar expresiones regulares, debemos usar grupos de captura (el contenido entre paréntesis) de aquellos valores que queramos pasar a la función.

## Estado final

Vamos a poner el código del _step-definitions/menu.steps.ts_:

```typescript
import { expect } from "chai";
import { Before, Given, TableDefinition, Then, When } from "cucumber";

import Bill from "../src/restaurant/bill";
import Menu from "../src/restaurant/menu";

let menus: Menu[];
let bill: Bill;

Before(() => {
  menus = [];
  bill = new Bill();
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

Given(/^que he comprado (\d+) menús? del número (\d+)$/, function(
  units: number,
  option: number
) {
  const menu = menus[option];
  for (let i = 0; i < units; i++) {
    bill.add(menu);
  }
});

When("pido la cuenta recibo una factura de {int} euros", function(
  total: number
) {
  expect(bill.total()).to.equal(total * 100);
});

When("pago en efectivo con {int} euros", function(ammount: number) {
  bill.payWithMoney(ammount * 100);
});

Then("la factura está pagada", function() {
  expect(bill.restToPay()).to.equal(0);
});

Then("he obtenido {int} puntos", function(points: number) {
  expect(bill.points()).to.equal(points);
});

When("pago con {int} puntos y {int} euros", function(
  points: number,
  money: number
) {
  bill.payWithMoney(money * 100);
  bill.payWithPoints(points);
});

Then("quedan {int} euros por pagar", function(amount: number) {
  expect(amount * 100).to.equal(bill.restToPay());
});
```

Y ejecutamos, ahora sí, todas las pruebas:

```sh
$ yarn cucumber features/menu.feature
........................................................................

5 scenarios (5 passed)
31 steps (31 passed)
0m00.007s
Done in 1.94s.
```

Podemos comprobar que hemos conseguido pasar todas las pruebas y nuestra aplicación cumpliría todas las especificaciones.
