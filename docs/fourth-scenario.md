# Cuarto escenario

Vamos a implementar el cuarto escenario

```gherkin
Escenario: Intentar pagar el IVA con puntos
    Dado que he comprado 5 menús del número 1
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago con 550 puntos y 0 euros
    Entonces quedan 5 euros por pagar
```

Si ejecutamos la prueba:

```sh
$ yarn cucumber features/menu.feature:37
yarn run v1.22.0
$ ./node_modules/cucumber/bin/cucumber-js features/**/*.feature --require-module ts-node/register --require 'step-definitions/**/*.ts' features/menu.feature:37
.....P.....P

Warnings:

1) Scenario: Intentar pagar el IVA con puntos # features/menu.feature:37
   ✔ Before # step-definitions/menu.steps.ts:10
   ✔ Dados los siguientes menús: # step-definitions/menu.steps.ts:15
       | número | precio |
       | 1      | 10     |
       | 2      | 12     |
       | 3      | 8      |
   ✔ Dado que he comprado 5 menús del número 1 # step-definitions/menu.steps.ts:27
   ✔ Cuando pido la cuenta recibo una factura de 55 euros # step-definitions/menu.steps.ts:37
   ✔ Y pago con 550 puntos y 0 euros # step-definitions/menu.steps.ts:55
   ? Entonces quedan 5 euros por pagar # step-definitions/menu.steps.ts:63
       Pending

1 scenario (1 pending)
5 steps (1 pending, 4 passed)
0m00.002s
```

En esta ocasión, las funcionalidades que queremos comprobar ya las tenemos, solo que no con esas sentencias. Vamos a implementar directamente esas sentencias en el _step-definitions/menu.steps.ts_ y ver si nuestra clase funciona:

```typescript
// ...

Then("quedan {int} euros por pagar", function(amount: number) {
  expect(amount * 100).to.equal(bill.restToPay());
});
```

Ejecutamos de nuevo la prueba:

```sh
$ yarn cucumber features/menu.feature:37
.....F.....F

Failures:

1) Scenario: Intentar pagar el IVA con puntos # features/menu.feature:37
   ✔ Before # step-definitions/menu.steps.ts:10
   ✔ Dados los siguientes menús: # step-definitions/menu.steps.ts:15
       | número | precio |
       | 1      | 10     |
       | 2      | 12     |
       | 3      | 8      |
   ✔ Dado que he comprado 5 menús del número 1 # step-definitions/menu.steps.ts:27
   ✔ Cuando pido la cuenta recibo una factura de 55 euros # step-definitions/menu.steps.ts:37
   ✔ Y pago con 550 puntos y 0 euros # step-definitions/menu.steps.ts:55
   ✖ Entonces quedan 5 euros por pagar # step-definitions/menu.steps.ts:63
       AssertionError
           + expected - actual

           -500
           +0

           at World.<anonymous> (/home/sergio/Developer/aulasoftwarelibre/bdd-by-example-typescript/step-definitions/menu.steps.ts:64:27)

1 scenario (1 failed)
5 steps (1 failed, 4 passed)
0m00.003s
```

El escenario falla porque en la especificación no hemos indicado que el IVA no se puede pagar con puntos. Así que creamos una nueva regla en `src/restaurant/bill.spec.ts` para tener en cuenta este comportamiento.

```typescript
it("cannot pay VAT with points", () => {
  bill.add(menu);
  bill.payWithPoints(110);
  expect(bill.restToPay()).toBe(100);
});
```

Y modificamos nuestra clase _Bill_ para pasar la especificación:

```typescript hl_lines="51 52 53 54"
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
    const maxMoneyPoints = this.totalBeforeVAT();
    const moneyPoints = (100 * this.usedPoints) / 10;

    return moneyPoints > maxMoneyPoints ? maxMoneyPoints : moneyPoints;
  }
}

export default Bill;
```

Y comprobamos que esto consigue que la prueba pase:

```sh
$ yarn cucumber features/menu.feature:37
............

1 scenario (1 passed)
5 steps (5 passed)
0m00.002s
Done in 1.89s.

```
