# Implementando el primer escenario

El primer escenario es el siguiente:

```gherkin
Escenario: Ganar puntos al pagar en efectivo
    Dado que he comprado 5 menús del número 1
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago en efectivo con 55 euros
    Entonces la factura está pagada
    Y he obtenido 50 puntos
```

Para implementar esta escenario, que es realmente el primero de nuestro proyecto que vamos a implementar, necesitamos una clase cuenta (_Bill_) que guarde los menús que se han consumido e informe del coste total, de la cantidad que se ha ingresado (en dinero o puntos), de lo que resta por pagar y de los puntos obtenidos.

## Describir la clase _Bill_

Vamos a definir nuestra clase `src/restaurant/bill.ts`:

```typescript
class Bill {}

export default Bill;
```

y la especificación `ssrc/restaurant/bill.spec.ts`:

```typescript
import Bill from "./bill";

describe("Bill", () => {
  let bill = new Bill();

  it("is initializable", () => {
    expect(bill).toBeInstanceOf(Bill);
  });
});
```

Ahora tenemos que describir la API de nuestra clase. Concretamente para este escenario nuestra clase debe proporcionar una API para:

- Añadir un menú a la cuenta
- Obtener el total de la cuenta
- Permitir pagar una cantidad de dinero
- Determinar cuánto resta por pagar
- Determinar cuántos puntos se han ganado

Para esta prueba vamos a suponer que tenemos ya una instancia de _Menu_ que cuesta 10€.

## Añadiendo elementos a la cuenta

Nuestro objetivo es añadir el precio de los menús a la cuenta. Para eso vamos a suponer que tenemos un menú que vale 10€.

Vamos a crear ese menú de ejemplo en nuestra especificación:

```typescript
import { mocked } from "ts-jest/utils";

import Bill from "./bill";
import Menu from "./menu";

jest.mock("./menu");

describe("Bill", () => {
  let bill = new Bill();
  let menu = mocked(Menu);

  beforeEach(() => {
    menu.mockClear();
    menu.prototype.price = jest.fn().mockReturnValue(1000);
  });
});
```

En esta ocasión no estamos usando _beforeEach_ para configurar el constructor de la clase, que por ahora no hemos determinado que vayamos a necesitar, sino para configurar una instancia de la clase _Menu_ y que cuando se llamen a la función _price()_ devolverá 1000. Hay que tener en cuenta que _Menu_ no es algo que hayamos instanciado nosotros. Lo que ha ocurrido es que _jest_ ha creado un _doble_ o _mock_, una clase que simula las respuestas a los métodos con los valores que se le indican con las cláusulas _mockReturnValue_.

Completamos la especificación:

```typescript
import { mocked } from "ts-jest/utils";

import Bill from "./bill";
import Menu from "./menu";

jest.mock("./menu");

describe("Bill", () => {
  let bill = new Bill();
  let menu = mocked(Menu);

  beforeEach(() => {
    menu.mockClear();
    menu.prototype.price = jest.fn().mockReturnValue(1000);
  });

  it("has no items by default", () => {
    expect(bill.total()).toBe(0);
  });

  it("add an item", () => {
    bill.add(menu.prototype);
    expect(bill.total()).toBe(1100);
  });
});
```

Estamos describiendo que nuestra cuenta, cuando se crea, no debe tener ningún elemento, y que los elementos que se añaden incrementan la cuenta (con IVA). Ejecutamos las pruebas, que fallarán, y pasamos a implementar el código. Pasamos a completar el código de nuestra clase _Bill_:

```typescript
import Menu from "./menu";

class Bill {
  private readonly VAT = 1.1;
  private items: Menu[];

  constructor() {
    this.items = [];
  }

  add(menu: Menu): void {
    this.items.push(menu);
  }

  total(): number {
    return (
      this.VAT *
      this.items.reduce((carry: number, item: Menu) => carry + item.price(), 0)
    );
  }
}

export default Bill;
```

Y ejecutamos las pruebas:

```console
$ yarn test
yarn run v1.22.0
$ jest
 PASS  src/restaurant/menu.spec.ts
 PASS  src/restaurant/bill.spec.ts

Test Suites: 2 passed, 2 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        1.701s, estimated 3s
Ran all test suites.
Done in 3.00s.
```

## Refactorizar _Menu_

A la clase Bill, ¿le importa el número del menú? No, en realidad no. En más, ¿podríamos añadir elementos a la cuenta que no fueran menús? Lo lógico es que sí. Entonces, ¿cómo hacemos nuestra clase compatible con cualquier clase que tenga un precio? Pues utilizando interfaces. Vamos a crear una interfaz _Priced_ que obligue a las clases que lo implementen a devolver _price()_. De esa manera, a _Bill_ solo le interesa que elemento que añadimos a la cuenta tenga un método _price_.

Vamos a crear nuestro interfaz `Priced` en `src/restaurant/priced.ts`:

```typescript
interface Priced {
  price(): number;
}

export default Priced;
```

Ahora necesitamos que nuestra clase _Menu_ implemente la interfaz _Priced_:

```typescript hl_lines="3"
import Priced from "./priced";

class Menu implements Priced {
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

Refactorizamos la especificación de Bill:

```typescript
import { mock, mockReset } from "jest-mock-extended";

import Bill from "./bill";
import Priced from "./priced";

describe("Bill", () => {
  let bill = new Bill();
  let menu = mock<Priced>();

  beforeEach(() => {
    mockReset(menu);
    menu.price.mockReturnValue(1000);
  });

  it("has no items by default", () => {
    expect(bill.total()).toBe(0);
  });

  it("add an item", () => {
    bill.add(menu);
    expect(bill.total()).toBe(1100);
  });
});
```

Y refactorizamos la clase _Bill_:

```typescript
import Priced from "./priced";

class Bill {
  private readonly VAT = 1.1;
  private items: Priced[];

  constructor() {
    this.items = [];
  }

  add(item: Priced): void {
    this.items.push(item);
  }

  total(): number {
    return (
      this.VAT *
      this.items.reduce(
        (carry: number, item: Priced) => carry + item.price(),
        0
      )
    );
  }
}

export default Bill;
```

## Implementando los primeros steps

Ahora estamos en posición de implementar los primeros steps:

```gherkin hl_lines="2 3"
Escenario: Ganar puntos al pagar en efectivo
    Dado que he comprado 5 menús del número 1
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago en efectivo con 55 euros
    Entonces la factura está pagada
    Y he obtenido 50 puntos
```

Quedando el código en el archivo `step-definitions/menu.steps.ts` como sigue:

```typescript hl_lines="8 12 31 32 33 34 40"
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

Given("que he comprado {int} menús del número {int}", function(
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

Y comprobamos que, efectivamente, el código funciona:

```sh
$ yarn cucumber features/menu.feature:16
....P--....P--

Warnings:

1) Scenario: Ganar puntos al pagar en efectivo # features/menu.feature:16
   ✔ Before # step-definitions/menu.steps.ts:10
   ✔ Dados los siguientes menús: # step-definitions/menu.steps.ts:15
       | número | precio |
       | 1      | 10     |
       | 2      | 12     |
       | 3      | 8      |
   ✔ Dado que he comprado 5 menús del número 1 # step-definitions/menu.steps.ts:27
   ✔ Cuando pido la cuenta recibo una factura de 55 euros # step-definitions/menu.steps.ts:37
   ? Y pago en efectivo con 55 euros # step-definitions/menu.steps.ts:43
       Pending
   - Entonces la factura está pagada # step-definitions/menu.steps.ts:49
   - Y he obtenido 50 puntos # step-definitions/menu.steps.ts:54

1 scenario (1 pending)
6 steps (1 pending, 2 skipped, 3 passed)
0m00.003s

```

# Implementando el pago

Para implementar el pago debemos ser capaces de indicar una cantidad pagada en metálico, ver cuánto queda por pagar y ver cuántos puntos hemos obtenido.

Esta sería la especificación:

```typescript hl_lines="25 31 37"
import { mock, mockReset } from "jest-mock-extended";

import Bill from "./bill";
import Priced from "./priced";

describe("Bill", () => {
  let bill = new Bill();
  let menu = mock<Priced>();

  beforeEach(() => {
    bill = new Bill();
    mockReset(menu);
    menu.price.mockReturnValue(1000);
  });

  it("has no items by default", () => {
    expect(bill.total()).toBe(0);
  });

  it("add an item", () => {
    bill.add(menu);
    expect(bill.total()).toBe(1100);
  });

  it("can be paid with money", () => {
    bill.add(menu);
    bill.payWithMoney(1100);
    expect(bill.restToPay()).toBe(0);
  });

  it("can give points when is payed with money", () => {
    bill.add(menu);
    bill.payWithMoney(1100);
    expect(bill.points()).toBe(10);
  });

  it("cannot give points when total is not enough", () => {
    const otherMenu = mock<Priced>();
    otherMenu.price.mockReturnValue(99);

    bill.add(otherMenu);
    bill.payWithMoney(109);
    expect(bill.points()).toBe(0);
  });
});
```

Estamos describiendo distintos casos:

- Cuando se paga exacto y no queda nada por pagar
- Cuando se pagan justo 10 euros
- Cuando se pagan menos de 1 euro

Podríamos ser más exhaustivos, como determinar que no se den puntos hasta que no se pague, pero lo dejamos para los casos siguientes.

Completamos el código en la clase _Bill_:

```typescript hl_lines="18 21 22 23 25 26 27 29 30 31 33 34 35 36 37 38"
import Priced from "./priced";

class Bill {
  private readonly VAT = 1.1;
  private items: Priced[];
  private committed: number;

  constructor() {
    this.items = [];
    this.committed = 0;
  }

  add(item: Priced): void {
    this.items.push(item);
  }

  total(): number {
    return this.VAT * this.totalBeforeVAT();
  }

  payWithMoney(amount: number): void {
    this.committed = amount;
  }

  restToPay(): number {
    return this.total() - this.committed;
  }

  points(): number {
    return Math.floor(this.totalBeforeVAT() / 100);
  }

  private totalBeforeVAT(): number {
    return this.items.reduce(
      (carry: number, item: Priced) => carry + item.price(),
      0
    );
  }
}

export default Bill;
```

Y comprobamos que pasamos las pruebas:

```sh
$ yarn test

yarn run v1.22.0
$ jest
 PASS  src/restaurant/menu.spec.ts
 PASS  src/restaurant/bill.spec.ts

Test Suites: 2 passed, 2 total
Tests:       7 passed, 7 total
Snapshots:   0 total
Time:        1.01s
Ran all test suites.
Done in 2.36s.
```

Ya nos resta terminar de implementar la historia de usuario. Nuestra clase `step-definitions/menu.steps.ts` queda así:

```typescript hl_lines="44 48 52"
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

Given("que he comprado {int} menús del número {int}", function(
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

Si ejecutamos este primer escenario debemos comprobar que se ha completado con éxito:

```sh
$ yarn cucumber features/menu.feature:16
..............

1 scenario (1 passed)
6 steps (6 passed)
0m00.003s
Done in 1.85s.
```
