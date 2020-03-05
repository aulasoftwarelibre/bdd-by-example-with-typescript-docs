# Tercer escenario

Vamos a implementar el tercer escenario

```gherkin
Escenario: Pagar con puntos
    Dado que he comprado 5 menús del número 1
    Cuando pido la cuenta recibo una factura de 55 euros
    Y pago con 500 puntos y 5 euros
    Entonces la factura está pagada
    Y he obtenido 0 puntos
```

Si ejecutamos la prueba:

```sh
$ yarn cucumber features/menu.feature:30
..............

1 scenario (1 passed)
6 steps (6 passed)
0m00.003s
Done in 1.88s.
```

¡La prueba pasa! No tenemos que implementar nada nuevo. Una cuestión importante a la hora de escribir los pasos (_steps_), es intentarlos escribir siempre de la misma manera, de tal manera que podamos reutilizarlos en sucesivas pruebas. De esta manera, en muchas ocasiones comprobaremos que no tenemos que implementar escenarios porque ya se ejecutan con los pasos definidos en los anteriores.
