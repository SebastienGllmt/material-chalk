<p align="center">
  <a href="https://www.npmjs.com/package/material-chalk">
    <picture>
      <img src="https://img.shields.io/npm/v/material-chalk" alt="NPM">
    </picture>
  </a>
  <a href="https://jsr.io/@sebastiengllmt/material-chalk">
    <img src="https://jsr.io/badges/@sebastiengllmt/material-chalk" alt="JSR" />
  </a>
</p>

# Overview

`material-chalk` is a library for generating beautiful colors for namespaces based on color theory in a way that is deterministic and extensible.

It is built using the latest Typescript and ESM standards, while being compatible with older standards when needed.

It is focused on being an opinionated standard to encourage consistent colors for the same namespace across different programming languages and tools, while still being composable with any frontend from [terminal-focused](https://www.npmjs.com/package/chalk) libraries to full GUI applications.

<p align="center">
  <img src="https://github.com/user-attachments/assets/897173dc-0cfa-4c27-9443-d19b85ebf006" alt="lib-preview" />
</p>

# How to Use

## Simple color

> :warning: your terminal must support `truecolor` to see all possible colors. Although tools like `chalk` attempt to detect this automatically, you may have to set [chalk.level](https://github.com/chalk/chalk?tab=readme-ov-file#chalklevel) manually or set `export COLORTERM=truecolor` in your shell (many shells use `xterm-256color` by default which limits the color range).


```typescript
import chalk from 'chalk';
import { createMaterial, Format } from 'material-chalk'

const chalkFormat = Format.Chalk(chalk)
const namespaceFoo = createMaterial('foo').formatAs(chalkFormat);
console.log(
  namespaceFoo(`Hello, world!`)
);
```

There are multiple format options available, notably:
- `Format.Htc` for the direct HCT (hue, chroma, tone) representation from Material Design
- `Format.Hex` for a hex color representation (ex: `#ff0000`)
- `Format.Custom` for a custom format function that you provide
- `Format.Chalk` for to use Chalk to print with color
- `Format.Scheme` for generating palettes (see [below](#building-a-full-palette))

## Creating nested namespaces

`material-chalk` allows you to define namespaces both statically and dynamically.

Static nested namespaces
```typescript
const nestedNamespace =
  createMaterial(['parent', 'child'])
  .formatAs(chalkFormat);
```
Dynamic nested namespaces
```typescript
const parentNamespace = createMaterial('parent');
const nestedNamespace = parentNamespace
  .subMaterial('child')
  .formatAs(chalkFormat);
```

Namespaces leverage the `blend` concept in Material Design which shifts the hue of the children towards that of the parent.

![blend-example](https://github.com/user-attachments/assets/b21cd532-0ad6-452a-b3b9-d2e6a3313f50)

## Building a full palette

You can easily build full UIs around the color generated for a namespace using Material Design v3 [dynamic color schemes](https://m3.material.io/styles/color/choosing-a-scheme).

Material Design allows you to create your own color scheme [manually](https://github.com/material-foundation/material-color-utilities/blob/main/dev_guide/creating_color_scheme.md#typescript-1), but it also comes with many [built-in schemes](https://github.com/material-foundation/material-color-utilities/tree/main/typescript/scheme).

Material Design will modify the hue/chroma/tone of your colors as needed for (this is by design), but we provide a helper function to convert a scheme to one that more strongly retains the color generated for the namespace: `sourceAsPrimary`.

```typescript
import chalk from 'chalk';
import { createMaterial, sourceAsPrimary, Format } from 'material-chalk'
import { hexFromArgb, SchemeVibrant } from "@material/material-color-utilities";

// convert one of the many built-in schemes
// to one that more accurately uses the namespace's color
const SchemeVibrantNamespace = Format.Scheme(sourceAsPrimary(SchemeVibrant))(
  true, // isDark
  0 // contrast level (for accessibility). 0 is normal contrast
)

// instantiate our new color scheme on a specific namespace
const namespaceFoo = createMaterial('foo').formatAs(SchemeVibrantNamespace);

// utility function just to demo functionality
function printWithChalk(color: number, text: string) {
  console.log(chalk.hex(hexFromArgb(color)).bold(text));
}

printWithChalk(namespaceFoo.primaryPaletteKeyColor, 'Primary');
printWithChalk(namespaceFoo.secondaryPaletteKeyColor, 'Secondary');
printWithChalk(namespaceFoo.tertiaryPaletteKeyColor, 'Tertiary');
printWithChalk(namespaceFoo.neutralPaletteKeyColor, 'Neutral');
printWithChalk(namespaceFoo.neutralVariantPaletteKeyColor, 'Neutral Variant');
```

Here are some examples of how a namespace looks like under different schemes:

![scheme-preview](https://github.com/user-attachments/assets/1c3b13f8-75a4-4592-a3aa-c650f6444bbd)

### Material Design v2

If you're still using Material Design v2, you can still use this library to generate your palette

```typescript
import chalk from 'chalk';
import { createMaterial, Format } from "@material/material-color-utilities";
import { hctForNamespace } from 'material-chalk'

const namespaceFoo = createMaterial('foo').formatAs(Format.Hct);

// utility function just to demo functionality
function printWithChalk(color: number, text: string) {
  console.log(chalk.hex(hexFromArgb(color)).bold(text));
}

const corePalette = CorePalette.of(namespaceFoo.toInt());
printWithChalk(corePalette.a1.keyColor.toInt(), "Primary");
printWithChalk(corePalette.a2.keyColor.toInt(), "Secondary");
printWithChalk(corePalette.a3.keyColor.toInt(), "Tertiary");
printWithChalk(corePalette.n1.keyColor.toInt(), "Neutral 1");
printWithChalk(corePalette.n2.keyColor.toInt(), "Neutral 2");
```

## Utility functions

`material-chalk` provides a few utility functions to make it easier to work with colors.

### Brand colors with `registerBrand` and `matchColor` 

Sometimes, for business reasons, you need to enforce that a namespace is associated with a specific brand color. In those cases, you can `registerBrand` to override cache the color that should be generated for a specific namespace.

If you want to make sure your color matches the same brightness as other colors in `material-chalk`, you can use the `matchColor` function to adjust the brightness (internally, this will try its best to maintain your brand color)

```typescript
import { createMaterial, matchColor, registerBrand, Format } from 'material-chalk'

const brandColor = matchColor("#ff0000");
registerBrand("my-brand", brandColor.formatAs(Format.Hct));

// this will retrieve the cached color
const namespace = createMaterial("my-brand").formatAs(Format.Hct);
```

### Printing chains of namespaces with `chainedMessage`

Often times you want to print a chain of all the namespaces leading up to a specific message. You can use `chainedMessage` to do this:

```typescript
import chalk from 'chalk';
import { createMaterial, chainedMessage, Format } from 'material-chalk'

const chalkFormat = Format.Chalk(chalk)
const namespaceFoo = createMaterial('foo').formatAs(chalkFormat);
console.log(
  chainedMessage(chalk, ['root-namespace', 'middle', 'child'], 'Hello, world!')
);
```

Here is a preview of what it looks like for different chain lengths:

![chained-message-preview](https://github.com/user-attachments/assets/910bb26c-c213-40d8-bac4-fba10806d711)

## Performance

`material-chalk` caches namespaces generated by default, meaning the performance hit is negligible for the majority of use-cases.

Performance does matter in the not-so-realistic scenarios that you are generating millions of random strings as namespaces. In this case:
1. It takes `0.01` milliseconds per namespace color generation *(cached)*
2. It takes some memory to cache the namespaces

If you need to save memory, you can disable the cache:

```typescript
import { createMaterial } from 'material-chalk'

const namespaceFoo = createMaterial(
  'foo',
  { cache: false }
);
```

# Problems with existing implementations

There are other ad-hoc implementations of generating colors from strings, but they generally have at least one of the following issues:

| Issue                                       | Other libraries                                                                                                | `material-chalk`                                                  |
|---------------------------------------------|----------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------|
| **Legacy code**                             | old and do not have Typescript/ESM support                                                                     | built on the latest best practices                                |
| **Poor color choices**                      | no justification based on color theory for why a specific choice was made, leading to poor color choices       | full justification for choices [here](https://github.com/SebastienGllmt/material-chalk/blob/master/Justifications.md)                       |
| **Not extensible** ex: color sub-namespaces | provide no opinionated way on how to extend it for a given namespace                                           | leverage Material Design to build for palettes for your namespace |
| **Lack of standardization**                 | provide too many configurations leading to inconsistent colors across tools & languages for the same namespace | opinionated deterministic choices based on color theory           |
| **Limited to RGB**                          | only support the RGB range                                                                                     | supports colors outside the standard RGB range                    |
| **Poor randomness**                         | depend on non-deterministic poor sources of randomness like Math.random() internally                           | uses fnv-1a for good deterministic seeding of randomness          |
