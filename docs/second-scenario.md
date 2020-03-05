# Segundo escenario

Vamos a implementar ahora el segundo escenario

```gherkin
Escenario: Pagar con dinero y puntos
Dado que he comprado 5 menús del número 1
Cuando pido la cuenta recibo una factura de 55 euros
Y pago con 10 puntos y 54 euros
Entonces la factura está pagada
Y he obtenido 0 puntos
```

Si intentamos ejecutar la prueba directamente vemos que hay ya muchos pasos que pasan:

```sh
$ yarn cucumber features/menu.feature:23
yarn run v1.22.0
$ ./node_modules/cucumber/bin/cucumber-js features/**/*.feature --require-module ts-node/register --require 'step-definitions/**/*.ts' features/menu.feature:23
....P--....P--

Warnings:

1) Scenario: Pagar con dinero y puntos # features/menu.feature:23
   ✔ Before # step-definitions/menu.steps.ts:10
   ✔ Dados los siguientes menús: # step-definitions/menu.steps.ts:15
       | número | precio |
       | 1      | 10     |
       | 2      | 12     |
       | 3      | 8      |
   ✔ Dado que he comprado 5 menús del número 1 # step-definitions/menu.steps.ts:27
   ✔ Cuando pido la cuenta recibo una factura de 55 euros # step-definitions/menu.steps.ts:37
   ? Y pago con 10 puntos y 54 euros # step-definitions/menu.steps.ts:55
       Pending
   - Entonces la factura está pagada # step-definitions/menu.steps.ts:47
   - Y he obtenido 0 puntos # step-definitions/menu.steps.ts:51

1 scenario (1 pending)
6 steps (1 pending, 2 skipped, 3 passed)
0m00.001s
```

Solo tenemos que implementar el pago con puntos.

## Describir la funcionalidad

El primer paso, es escribir la descripción en nuestra especificación de la clase _Bill_:

```typescript
// ...
it("can be paid with money and points and get no points", () => {
  bill.add(menu);
  bill.payWithMoney(1000);
  bill.payWithPoints(10);
  expect(bill.restToPay()).toBe(0);
  expect(bill.points()).toBe(0);
});
```

Refactorizamos nuestro código para pasar la prueba:

```typescript hl_lines="7 12 27 28 29 32 36 37 38 50 51 52"
import Priced from "./priced";

class Bill {
  private readonly VAT = 1.1;
  private items: Priced[];
  private committed: number;
  private usedPoints: number;

  constructor() {
    this.items = [];
    this.committed = 0;
    this.usedPoints = 0;
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

  payWithPoints(amount: number): void {
    this.usedPoints = amount;
  }

  restToPay(): number {
    return this.total() - this.committed - this.moneyPoints();
  }

  points(): number {
    if (this.usedPoints > 0) {
      return 0;
    }

    return Math.floor(this.totalBeforeVAT() / 100);
  }

  private totalBeforeVAT(): number {
    return this.items.reduce(
      (carry: number, item: Priced) => carry + item.price(),
      0
    );
  }

  private moneyPoints(): number {
    return 100 * (this.usedPoints / 10);
  }
}

export default Bill;
```

Comprobamos que pasamos las pruebas y pasamos a implementar la historia de usuario.

## Completando la historia de usuario

Creamos el código que implementa el paso que nos falta:

```typescript
//...

When("pago con {int} puntos y {int} euros", function(
  points: number,
  money: number
) {
  bill.payWithMoney(money * 100);
  bill.payWithPoints(points);
});
```

Y ya hemos conseguido terminar otro escenario:

```sh
$ yarn cucumber features/menu.feature:23
..............

1 scenario (1 passed)
6 steps (6 passed)
0m00.004s
Done in 1.89s.
```

## Conclusiones

Como vamos viendo, el código que se genera en _jest_ es realmente el código que implementa nuestras reglas de negocio. En _cucumber_ solo implementamos código para poder usar el dominio en las pruebas y comprobar que nuestras dos clases (_Menu_ y _Bill_) trabajan bien juntas.
