# Mastering TypeScript

## Highcharts TypeScript Declarations (beta)

Highcharts 7 provides TypeScript declarations for describing functionality and possibilities of the Highcharts libraries, similar to a header file in C-based languages. TypeScript-compatible editors can give you now tooltips and suggestions for Highcharts as you type. TypeScript compilers can watch and point out potential problems in your use-case of Highcharts. The result is a faster and better fail-proofed development of Highcharts-based solutions.

## Installation
You only need a TypeScript-capable Editor, like Microsoft's Visual Studio Code, to get autocompletion and hints for Highcharts.

## Configuration
Because Highcharts is a charting library most often used in the web browser, you probably need to modify the TypeScript configuration for the target platform. The `tsconfig.json` below, covers a typical use case of Highcharts:
```json
{
    "compilerOptions": {
        "strict": true,
        "target": "es6",
        "module": "umd",
        "moduleResolution": "node",
        "outDir": "mychart/",
    },
    "exclude": [
        "node_modules"
    ]
}
```

If you place your TypeScript source code (`*.ts`) in one of your project folders, the TypeScript compiler will automatically find and compile it and outputting JavaScript compiled code (`*.js`) to the `outDir` folder. Prevent with the `exclude` property that TypeScript source code is compiled from the `node_modules` folder.

## Migration

### Migration from Definitely Typed
If you use existing TypeScript declarations for Highcharts like `@types/highcharts`, you should uninstall them to prevent any mismatch:
```sh
npm uninstall @types/highcharts
```

### Migration from JavaScript
For full fail-proofed development you may convert your source code to TypeScript. The initial step is:
```sh
npm install typescript &amp;&amp; npx tsc --init
```

More information about migrating JavaScript projects to TypeScript can be found in the official [TypeScript handbook](http://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html).

## Solving Problems: Debugging
If TypeScript is falling over your Highcharts options, it might be that it is not finding the correct series as a reference. In that case you can cast series options to the correct type.
```ts
series: [{
    type: "line",
    data: [1, 2, "3", 4, 5] as Highcharts.SeriesLineDataOptions
}]
```

## Solving Problems: Missing modules
In some cases there are not yet declarations for a Highcharts module available. In that case, you have to cast modules and unknown functions to the `any` type. Create first a `global.d.ts` file in your root folder with the following file content:
```ts
declare module '*';
```

Now you can use all JavaScript modules and cast the Highcharts namespace as required. For example:
```ts
import * as Highcharts from 'highcharts';

// Module with declaration:
import AccessibilityModule from 'highcharts/modules/accessibility';

// Module with any type:
import NewModule from 'highcharts/modules/new';
    
NewModule(Highcharts);
AccessibilityModule(Highcharts);
    
// Initiate the chart
(Highcharts as any).newChart('container', {
    series: [{
        type: 'new',
        data: [1, 2, 3, 4, 5]
    }]
});
```

## Solving Problems: Reporting Bugs
The TypeScript declarations for Highcharts are in a beta state, where they can be used in production, but might break existing code with future type updates. In these cases only small modifications to types will be necessary. If you found something in TypeScript that is not working like in our [documentation](https://api.highcharts.com/) or [Demos](https://www.highcharts.com/demo), you should take a look at our 

## Advanced: Extending Highcharts in TypeScript
The Highcharts libraries come with a huge amount of possibilities right out of the box, but sometimes you need something special and like to customize you chart with more than some options. Extending Highcharts is easy as well and teaching TypeScript about it, too.

### To Do
- Set proper types for your extensions
- Declare additional interfaces with your extensions to the Highcharts namespace
- Make use of existing [Highcharts types](https://api.highcharts.com/class-reference/Highcharts) and [interfaces](https://api.highcharts.com/class-reference/Highcharts.Dictionary_T_)

### Example
If you like to have a custom highlight function for you data points, you could come up with the following code. You might notice, that the beginning code is actually not real TypeScript and instead is just common ECMAScript, widely known as JavaScript:
```js
import * as Highchart from 'highcharts';

Highcharts.Point.prototype.highlight = function (event) {
    event = this.series.chart.pointer.normalize(event);
    this.onMouseOver(event);
    this.series.chart.tooltip.refresh(this);
    this.series.chart.xAxis[0].drawCrosshair(event, this);
};
```

First you have to make TypeScript aware of the types in your function, so you have to specify the type of the `event` parameter in this example:
```ts
[...]
Highcharts.Point.prototype.highlight = function (
    event: Highcharts.PointerEvent
) {
[...]
```

Secondly you have to assure TypeScript, that creating a new function on `Highcharts.Point` is really made by intention. You can do this with declarations for the Highcharts namespace - in a separate file like `./global.d.ts` or directly in your code:
```ts
[...]
declare module 'highcharts' {
    interface Point {
        highlight: (event: Highcharts.PointerEventObject) => void;
    }
}
[...]
```

In the end the source code of the example would look like this:
```ts
import * as Highchart from 'highcharts';

declare module 'highcharts' {
    interface Point {
        highlight: (event: Highcharts.PointerEventObject) => void;
    }
}

Highcharts.Point.prototype.highlight = function (
    event: Highcharts.PointerEvent
) {
    event = this.series.chart.pointer.normalize(event);
    this.onMouseOver(event);
    this.series.chart.tooltip.refresh(this);
    this.series.chart.xAxis[0].drawCrosshair(event, this);
};
```