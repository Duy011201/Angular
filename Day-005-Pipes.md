# Pipes

Dùng pipes to biến đổi dũ liệu strings, currency amounts, dates, and other data để hiển thị trên views.

## Syntax

views `{{ data | datatype }}`

`a ? b : c | x`

`a ? b : (c | x)`

`(a ? b : c) | x`

## Parameters

views `{{ birthday | date:"MM/dd/yy" }}`

## String Pipes

views `{{ birthday | date:"MM/dd/yy" | uppercase }}`

## Formatter Data

- `DatePipe`: Formats a date value according to locale rules.

-  `UpperCasePipe`: Transforms text to all upper case.

- `LowerCasePipe`: Transforms text to all lower case.

- `CurrencyPipe`: Transforms a number to a currency string, formatted according to locale rules.

- `DecimalPipe`: Transforms a number into a string with a decimal point, formatted according to locale rules.

- `PercentPipe`: Transforms a number to a percentage string, formatted according to locale rules.

## Custom Pipes

```ts
#src/app/exponential-strength.pipe.ts

import { Pipe, PipeTransform } from '@angular/core';
/*
 * Raise the value exponentially
 * Takes an exponent argument that defaults to 1.
 * Usage:
 *   value | exponentialStrength:exponent
 * Example:
 *   {{ 2 | exponentialStrength:10 }}
 *   formats to: 1024
*/
@Pipe({name: 'exponentialStrength'})
export class ExponentialStrengthPipe implements PipeTransform {
  transform(value: number, exponent?: number): number {
    return Math.pow(value, isNaN(exponent) ? 1 : exponent);
  }
}
```

```ts
#src/app/power-booster.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'app-power-booster',
  template: `
    <h2>Power Booster</h2>
    <p>Super power boost: {{2 | exponentialStrength: 10}}</p>
  `
})
export class PowerBoosterComponent { }
```