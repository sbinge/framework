- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
- [Styling & Theming in Dojo](#styling--theming-in-dojo)
  - [Structural widget styling](#structural-widget-styling)
    - [Example](#example)
  - [Abstracting and extending stylesheets](#abstracting-and-extending-stylesheets)
    - [CSS custom properties](#css-custom-properties)
    - [CSS module composition](#css-module-composition)
- [Working with themes](#working-with-themes)
  - [Widget theme keys](#widget-theme-keys)
    - [Theme key example](#theme-key-example)
  - [Writing a theme](#writing-a-theme)
  - [Scaffolding themes for third-party widgets](#scaffolding-themes-for-third-party-widgets)
    - [Compatible packages](#compatible-packages)
  - [Distributing themes](#distributing-themes)
    - [Usage](#usage)
  - [Using Dojo-provided themes](#using-dojo-provided-themes)
    - [Composing off Dojo themes](#composing-off-dojo-themes)
- [Theming a Dojo application](#theming-a-dojo-application)
  - [Making themeable widgets](#making-themeable-widgets)
    - [`ThemedMixin` `theme()` method](#themedmixin-theme-method)
    - [`ThemedMixin` Widget Properties](#themedmixin-widget-properties)
      - [Passing extra classes to widgets](#passing-extra-classes-to-widgets)
    - [Themeable Widget Example](#themeable-widget-example)
  - [Making themeable applications](#making-themeable-applications)
    - [Changing the currently active theme](#changing-the-currently-active-theme)
      - [`ThemeSwitcher` Properties](#themeswitcher-properties)
      - [`ThemeSwitcher` Example Usage](#themeswitcher-example-usage)

# Introduction

As a HTML-based technology, Dojo makes use of CSS for styling elements across the framework and for applications developed with it.

Dojo promotes encapsulated structural styling of individual widgets for maximum reuse, as well as simplified presentational theming across all widgets within an application, regardless of whether the widgets are from Dojo's native widget library, a third-party provider, or developed in-house for a particular application.

Feature | Description
--- | ---
A | B

# Basic Usage

# Styling & Theming in Dojo

Dojo widgets function best as simple components that each handle a single responsibility. They should be as encapsulated and modular as possible to promote reusability while avoiding conflicts with other widgets the application may also be using.

Widgets can be styled via regular CSS, but to support encapsulation and reuse goals, each widget should maintain its own independent CSS module that lives parallel to the widget's source code. This allows widgets to be styled independently, without clashing on similar class names used elsewhere in an application.

Dojo differentiates between two types of styling, each representing a different granularity of styling concerns within an enterprise web application:

* [Structural styles](#structural-widget-styling) (_granularity:_ per-widget)
  * The minimum styles necessary for a widget to function. These are localized to the particular widget they are needed for.
* [Presentational styles](#theming-a-dojo-application) (_granularity:_ appliation-wide)
  * Styles - usually visual - that may be changed without impacting widget functionality. These can cut across several widgets and usually provide a consistent theme for all widgets used within an application.

## Structural widget styling

Dojo leverages [CSS Modules](https://github.com/css-modules/css-modules) to provide all of the flexibility of CSS, but with the additional benefit of localized classes to help prevent inadvertent styling collisions across a large application. Dojo also makes use of [typed CSS modules](https://github.com/Quramy/typed-css-modules), allowing widgets to import their CSS modules similar to any other TypeScript module, and refer to specific CSS class names at design-time via IDE autocompletion.

The CSS module file for a widget should have a `.m.css` extension, and by convention is usually named the same as the widget it is associated with.

### Example

Given the following CSS module file for a widget:

>src/styles/MyWidget.m.css

```css
.myWidgetClass {
    font-variant: small-caps;
}

.myWidgetExtraClass {
    font-style: italic;
}
```

This stylesheet can be used within a corresponding widget as follows:

>src/widgets/MyWidget.ts

```ts
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import { v } from '@dojo/framework/widget-core/d';

import * as css from '../styles/MyWidget.m.css';

export default class MyWidget extends WidgetBase {
	protected render() {
		return v('div', { classes: [css.myWidgetClass, css.myWidgetExtraClass] }, ['Hello from a Dojo widget!']);
	}
}
```

Similarly, if using TSX widget syntax:

>src/widgets/MyTsxWidget.tsx
```ts
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import { tsx } from '@dojo/framework/widget-core/tsx';

import * as css from '../styles/MyWidget.m.css';

export default class MyTsxWidget extends WidgetBase {
	protected render() {
		return <div classes={[css.myWidgetClass, css.myWidgetExtraClass]}>Hello from a Dojo TSX widget!</div>;
	}
}
```

When inspecting the CSS classes of these sample widgets in a built application, they will not contain `myWidgetClass` and `myWidgetExtraClass`, but rather obfuscated CSS class names similar to `MyWidget-m__myWidgetClass__33zN8` and `MyWidget-m__myWidgetExtraClass___g3St`.

These obfuscated class names are localized to `MyWidget` elements, and are determined by Dojo's CSS modules build process. With this mechanism it is possible for other widgets in the same application to also use the `myWidgetClass` class name with different styling rules, and not encounter any conflicts between each set of styles.

**Warning:** The obfuscated CSS class names should be considered unreliable and may change with a new build of an application, so developers should not explicitly reference them (for examples if attempting to target an element from elsewhere in an application).

## Abstracting and extending stylesheets

### CSS custom properties

Dojo allows use of CSSnext features such as [custom properties and `var()`](http://cssnext.io/features/#custom-properties-var) to help abstract and centralize common styling properties within an application.

Rather than having to specify the same values for colors or fonts in every widget's CSS module, abstract custom properties can instead be referenced by name, with values then provided in a centralized CSS `:root` pseudo-class. This separation allows for much simpler maintenance of common styling concerns across an entire application.

Note that the `:root` pseudo-class is global within a webpage, but through Dojo's use of CSS modules, `:root` properties could potentially be specified in many locations across an application. However, Dojo does not guarantee the order in which CSS modules are processed, so to ensure consistency of which properties appear in `:root`, it is recommended that applications use a single `:root` definition within a centralized `variables.css` file in the application's codebase. This centralized variables file is a regular CSS file (not a CSS module) and can be `@import`ed as such in any CSS modules that require custom property values.

For example:

> src/themes/variables.css

```css
:root {
	/* different sets of custom properties can be used if an application supports more than one possible theme */
	--light-background: lightgray;
	--light-foreground: black;

	--dark-background: black;
	--dark-foreground: lightgray;

	--font: monospace;
	--padding: 32px;
}
```

> src/themes/myTheme/MyWidget.m.css

```css
@import '../../variables.css';

.root {
	font-family: var(--font);
	margin: var(--padding);

	color: var(--dark-foreground);
	background: var(--dark-background);
}
```

### CSS module composition

Applying a theme to a Dojo widget results in the widget's default styling classes being entirely overridden by those provided in the theme (see [Note 3 of `ThemedMixin`'s `this.theme()` method](#themedmixin-theme-method) for details). This can be problematic when only a subset of properties in a given styling class need to be modified through a theme, while the remainder can stay as default.

[CSS module files](https://github.com/css-modules/css-modules) in Dojo applications can leverage `composes:` functionality to apply sets of styles from one class selector to another. This can be useful when creating a new theme that tweaks an existing one, as well as for general abstraction of common sets of styling properties within a single theme (note that [CSS custom properties](#css-custom-properties) is a more standardized way of abstracting values for individual style properties).

**Warning:** Use of `composes:` can prove brittle, for example when extending a third-party theme that is not under direct control of the current application. Any change made by a third-party could break an application theme that `composes` the underlying theme, and such breakages can be problematic to pin down and resolve.

However, careful use of this feature can be helpful in large applications. For example, centralizing a common set of properties:

> src/themes/common/ButtonArchetype.m.css

```css
.buttonArchetype {
	margin-right: 10px;
	display: inline-block;
	font-size: 14px;
	text-align: left;
	background-color: white;
}
```

> src/themes/myBlueTheme/MyButton.m.css

```css
.root {
	composes: buttonArchetype from '../common/ButtonArchetype.m.css';
	background-color: blue;
}
```

# Working with themes

## Widget theme keys

Dojo's theming framework uses the concept of a 'widget theme key' to connect style overrides to the corresponding widget that the styles are intended for. Style overrides are usually [specified in a theme](#writing-a-theme), but can also be passed [directly via `ThemedMixin`'s `classes` override property](#passing-extra-classes-to-widgets), if required.

The theme key for a given widget is determined as:

    {package-name}/{widget-css-module-name}

where `package-name` is the value of the `name` property within the project's `package.json`, and `widget-css-module-name` is the filename of the primary CSS module used for the widget (_without_ the `.m.css` extension).

### Theme key example

For a given project:

>package.json

```json
{
	"name": "my-app"
}
```

When [following widget CSS module naming conventions](#structural-widget-styling), a given widget such as `src/widgets/MyWidget.ts` will use a corresponding CSS module name similar to `src/styles/MyWidget.m.css`. The theme key for `MyWidget` is therefore:

    my-app/MyWidget

Here, the name of the widget is the same as the the name of its CSS module file, but developers should be careful not to mistake the widget's theme key as representing the widget's TypeScript class name.

For a second widget that does not follow CSS module naming conventions, such as `src/widgets/BespokeWidget.ts` that uses a corresponding CSS module such as `src/styles/BespokeStyleSheet.m.css`, its widget theme key would instead be:

    my-app/BespokeStyleSheet

## Writing a theme

Themes are TypeScript modules that export an unnamed default object which maps [widget theme keys](#widget-theme-keys) to typed CSS module imports. CSS modules in a theme are the same as regular modules used directly in widgets. Once a theme is applied in an application, each widget identified via its theme key in the theme's definition object will have its styles overridden with those specified in the CSS module associated with that widget's theme key.

The following is a simple illustration of a complete theme for a single `MyWidget` widget (using a default CSS module of `MyWidget.m.css`), contained in a project named `my-app`:

> src/themes/myTheme/styles/MyWidget.m.css

```css
.root {
	color: blue;
}
```

> src/themes/myTheme/theme.ts

```ts
import * as myThemedWidgetCss from './styles/MyWidget.m.css';

export default {
	'my-app/MyWidget': myThemedWidgetCss
};
```

Here, `MyWidget` is following naming conventions with its [primary style class being named `root`](#making-themeable-widgets), allowing `myTheme` to easily override it via the `root` class in its `src/themes/myTheme/styles/MyWidget.m.css` CSS module.

The theme associates the new `root` styling class to `MyWidget` via its [theme key of `my-app/MyWidget`](#widget-theme-keys). When `myTheme` is applied, `MyWidget` will have its color set to blue and will no longer receive any other styles defined in the `root` class in its original CSS module (see [Note 3 of `ThemedMixin`'s `this.theme()` method](#themedmixin-theme-method) for details).

## Scaffolding themes for third-party widgets

It is likely that applications will need to extend their existing themes to include styling of any third-party widgets that may be used, such as those provided by [Dojo's native widget library](https://github.com/dojo/widgets).

The [`@dojo/cli-create-theme`](https://github.com/dojo/cli-create-theme) package provides tooling support to quickly generate theme scaffolding for third party widgets, via its `dojo create theme` CLI command. It can be installed locally within an application via:

```sh
npm install --save-dev @dojo/cli-create-theme
```

and can be used as follows from a project's root directory:

```sh
dojo create theme -n {myThemeName}
```

Running this command will begin to create the specified `myThemeName` theme by asking two questions:

- **What Package to do you want to theme?** 
  - The answer to this should be all the packages that contain the third-party widgets intended for theming, for example `@dojo/widgets`.
- **Which of the _{third-party-package}_ theme files would you like to scaffold?**
  - A list will be shown of all themeable widgets in the third-party packages that were specified when answering the first question. Users can then pick the subset of all compatible widgets that should be included in the resulting theme - usually only the widgets that are actually used in the current application will be selected, to help keep the theme's size to a minimum.

Several files will be created in the current project upon successful execution of the command:

* `src/themes/{myThemeName}/theme.ts`
* `src/themes/{myThemeName}/{third-party-package}/path/to/{selectedWidget}.m.css`

The theme's CSS modules created for all `{selectedWidget}`s come ready with themeable CSS selectors which can then be filled in with the appropriate stylings for `{myThemeName}`.

### Compatible packages

Any third-party package that has a `theme` directory containing widget CSS module files (`*.m.css`) and their corresponding compiled definition files (`*.m.css.js`) is compatible.

For example:

    node_modules
    └── {third-party-package}
        └── theme
            │   {widget}.m.css
            │   {widget}.m.css.js

## Distributing themes

Dojo's [`cli-build-theme`](https://github.com/dojo/cli-build-theme) package provides a CLI command to help build themes that are intended for distribution across multiple applications.

The command outputs the processed CSS modules, CSS source maps, an `index.js` theme module with a corresponding `.d.ts`, as well as any associated assets. The ouput also includes versioned `index.css` and `index.js` equivalents that are compatible with Dojo custom elements.

>Note: if you are using [ `dojo create theme`](https://github.com/dojo/cli-create-theme) within an existing application or Dojo custom element, then there is no need to use this package. As long as the theme files are within the existing build pipeline, they will be included in the build generated with [`@dojo/cli-build-app`](https://github.com/dojo/cli-build-app) or [`@dojo/cli-build-widget`](https://github.com/dojo/cli-build-widget);

### Usage

To use `@dojo/cli-build-theme` in a themes project, first install `@dojo/cli` globally (if you have not already done so), and then install the package:

```bash
npm install --global @dojo/cli
npm install --save-dev @dojo/cli-build-theme
```

To build a theme, run `dojo build theme` from the command line, specifying the theme `name` as well as an optional `release` version.

```bash
dojo build theme --name=my-theme --release=1.2.3
```

If no `release` is specified, then the version from `package.json` will be used. Both `name` and `release` are aliased as `n` and `r`, respectively, so the above command can be shortened to:

```bash
dojo build theme -n my-theme -r 1.2.3
```

The above will create a new `dist/src/my-theme` directory at the project root that contains:

- All raw `.m.css` files. Copying these files as-is enables composition (i.e., `composes: root from 'node_modules/my-theme/my-widget'`)
- An `assets` directory containing all fonts and images included within the theme's directory
- An `index.js` file that can be imported into Dojo widgets and passed to the [`@theme` decorator](https://github.com/dojo/framework/blob/master/src/widget-core/README.md#styling--theming)
- An `index.css` file that is imported into an application's `main.css`
- A `{name}-{release}.js` file for use with custom elements that registers the theme with a global registry and is added via a `<script>` tag
- A `{name}-{release}.css` file for use with custom elements that is added via a `<link rel="stylesheet">` tag

## Using Dojo-provided themes

The [`@dojo/themes`](https://github.com/dojo/themes) package provides a collection of ready-to-use themes that cover all widgets in Dojo's [native widget library](https://github.com/dojo/widgets). The themes can be used as-is, or [composed as the basis](#composing-off-dojo-themes) for a full application theme.

1. To use the themes, install `@dojo/themes` into your project, for example through `npm i @dojo/themes`. Then, for regular Dojo applications:

2. Import the theme CSS into your project's `main.css`:

   ```css
   @import '~@dojo/themes/dojo/index.css
   ```

3. Import the theme typescript module and use it [as per any other theme](#making-themeable-applications):

   ```ts
   import theme from '@dojo/themes/dojo';

   render() {
   	return w(Button, { theme }, [ 'Hello World' ]);
   }
   ```

If attempting to use the themes in custom elements, after installing `@dojo/themes`:

1. Add the custom element-specific theme CSS to `index.html`:

   ```html
   <link rel="stylesheet" href="node_modules/@dojo/themes/dojo/dojo-{version}.css">
   ```

2. Add the custom element-specific theme JS to `index.html`:
   ```html
   <script src="node_modules/@dojo/themes/dojo/dojo-{version}.js"></script>`
   ```

### Composing off Dojo themes

Once `@dojo/themes` is installed in a project, it can be used as the basis for an extended application theme by including relevant components with [CSS modules' composes functionality](#css-module-composition) in the new theme.

`@dojo/themes` also includes its own [`:root` `variables.css` file](#css-custom-properties) which can be imported if the extended application theme would like to reference Dojo-specified properties elsewhere in the new theme.

The following is an example of a new theme for the `@dojo/widgets/button` widget that extends off `@dojo/themes`, and changes the button's background to green while retaining all other button theme style properties:


> src/themes/myTheme/theme.ts

```ts
import * as myButton from './myButton.m.css';

export default {
	'@dojo/widgets/button': myButton
};
```

> src/themes/myTheme/myButton.m.css

``` css
@import '@dojo/themes/dojo/variables.css';

.root {
	composes: root from '@dojo/themes/dojo/button.m.css';
	background-color: var(--dojo-green);
}
```

# Theming a Dojo application

Dojo applications need a way to present all the widgets they use in a consistent manner, so that users perceive and interact with application features holistically, rather than as a mashup of disjointed elements on a webpage. This is usually implemented via a corporate or product marketing theme that specifies colors, layout, font families, and more.

## Making themeable widgets

There are three requirements for widgets to be considered themeable:

1. The widget's class should have the `@theme()` decorator applied, passing in the widget's imported CSS module as a decorator argument.
2. The widget's parent class should be wrapped with `ThemedMixin`.
3. One or more of the widget's styling classes should be wrapped in a call to `this.theme()` when rendering the widget.

By convention, there is a fourth requirement that is useful when developing widgets intended for distribution (this is a convention that widgets in Dojo's widget library follow):

4. The widget's main styling class should be named `root`. Doing so provides a predictable way to target the main styling class of any third-party themeable widgets when overriding styles in a custom theme.

`ThemedMixin` and the `@theme()` decorator can be imported from the `@dojo/framework/widget-core/mixins/Themed` module. Using `ThemedMixin` on a widget provides access to the `this.theme()` function.

### `ThemedMixin` `theme()` method

```ts
type SupportedClassName = string | null | undefined | boolean;
```

- `this.theme(classes: SupportedClassName): SupportedClassName`
- `this.theme(classes: SupportedClassName[]): SupportedClassName[]`
  - Widgets can pass in one or multiple CSS class names and will receive back the corresponding amount of themed class names, suitable for passing on to the `classes` property when rendering the widget's WNodes or VNodes.

    - **Note 1:** Theme overrides are at the level of CSS classes only, not individual style properties within a class.

    - **Note 2:** If the currently active theme does **not** provide an override for a given styling class, the widget will fall back to using its default style properties for that class.

    - **Note 3:** If the currently active theme does provide an override for a given styling class, the widget will _only_ have the set of CSS properties specified in the theme applied to it. For example, if a widget's default styling class contains ten CSS properties but the current theme only specifies one, the widget will render with a single CSS property and lose the other nine that were not specified in the theme override.

### `ThemedMixin` Widget Properties

- `theme` (optional)
  - If specified, [the provided theme](#writing-a-theme) will act as an override for any theme that the widget may use, and will take precedence over [the application's default theme](#making-themeable-applications) as well as [any other theme changes made in the application](#changing-the-currently-active-theme).
- `classes` (optional)
  - described in the following section:

#### Passing extra classes to widgets

The theming mechanism provides a simple way to consistently apply custom styles across every widget in an application, but isn't flexible enough for scenarios where a user wants to apply additional styles to specific instances of a given widget.

Extra styling classes can be passed in through a themeable widget instance's `classes` property. Each set of classes is keyed by the appropriate [widget theme key](#widget-theme-keys) to specify the widget that the classes should be applied to, including any that may be used in child widgets that the widget receiving the `classes` property may utilize.

These extra classes are additive and do not override the underlying widget styling classes, 

Note that it is a widget author's responsibility to explicitly pass the `classes` property to all child widgets that are leveraged, as the property will not be injected nor automatically passed to children by Dojo itself.

For example, if attempting to add extra styling classes to an instance of a Dojo button, as well as the icon child widget it contains:

```ts
interface Classes {
	[widgetKey: string]: {
		[className: string]: string | null | undefined;
	}
}

const myExtraClasses = {
	'@dojo/widgets/icon': {
		root: [css.extraRoot]
	},
	'@dojo/widgets/button': {
		root: [css.extraRoot]
	}
}

// example render method
render() {
	return w(Button, { classes: myExtraClasses });
}
```

### Themeable Widget Example

Given the following CSS module file for a themeable widget:

>src/styles/MyThemeAwareWidget.m.css

```css
/* requirement 4, i.e. this widget is intended for wider distribution and therefore the main presentational styles for
the widget that could be overridden with a custom theme are contained in this class : */
.root {
	font-family: sans-serif;
}

/* widgets can use any variety of ancillary CSS classes that are also themeable */
.myWidgetExtraThemeableClass {
	font-variant: small-caps;
}

/* extra 'fixed' classes can also be used to specify a widget's structural styling, which is not intended to be
overridden via a theme */
.myWidgetStructuralClass {
	font-style: italic;
}
```

This stylesheet can be used within a corresponding themeable widget as follows:

>src/widgets/MyThemeAwareWidget.ts

```ts
import { v } from '@dojo/framework/widget-core/d';
import { theme, ThemedMixin } from '@dojo/framework/widget-core/mixins/Themed';
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import * as css from '../styles/MyThemeAwareWidget.m.css';

/* requirement 1: */ @theme(css)
export default class MyThemeAwareWidget /* requirement 2: */ extends ThemedMixin(WidgetBase) {
	protected render() {
		return v(
			'div',
			{
				classes: [
					/* requirement 3: */ ...this.theme([
						/* requirement 4: */ css.root,
						css.myWidgetExtraThemeableClass
					]),
					css.myWidgetStructuralClass
				]
			},
			['Hello from a themed Dojo widget!']
		);
	}
}
```

Similarly, if using TSX widget syntax:

>src/widgets/MyThemeAwareTsxWidget.tsx
```ts
import { WidgetBase } from '@dojo/framework/widget-core/WidgetBase';
import { ThemedMixin, theme } from '@dojo/framework/widget-core/mixins/Themed';
import { tsx } from '@dojo/framework/widget-core/tsx';

import * as css from '../styles/MyThemeAwareWidget.m.css';

/* requirement 1: */ @theme(css)
export default class MyThemeAwareTsxWidget /* requirement 2: */ extends ThemedMixin(WidgetBase) {
	protected render() {
		return (
			<div
				classes={[
					/* requirement 3: */ ...this.theme([
						/* requirement 4: */ css.root,
						css.myWidgetExtraThemeableClass
					]),
					css.myWidgetStructuralClass
				]}
			>
				Hello from a themed Dojo TSX widget!
			</div>
		);
	}
}
```

## Making themeable applications

In order to specify a theme for all themeable widgets in an application, `registerThemeInjector()` can be used in conjunction with the application's global registry. This injector utility function is available from the `@dojo/framework/widget-core/mixins/Themed` module.

For example, specifying a primary application theme:

> src/main.ts

```ts
import renderer from '@dojo/framework/widget-core/vdom';
import { w } from '@dojo/framework/widget-core/d';
import Registry from '@dojo/framework/widget-core/Registry';
import { registerThemeInjector } from '@dojo/framework/widget-core/mixins/Themed';

import myTheme from './src/themes/myTheme/theme';
import App from './App';

const registry = new Registry();
registerThemeInjector(myTheme, registry);

const r = renderer(() => w(App, {}));
r.mount({ registry });
```

See [Writing a theme](#writing-a-theme) for a description of how the `myTheme` import should be structured.

Note that using themeable widgets without specifying an explicit theme (for example, passing an empty theme object to `registerThemeInjector` and not [explicitly overriding a widget instance's theme or styling classes](#themedmixin-widget-properties)) will result in each widget using its default style rules.

### Changing the currently active theme

`registerThemeInjector()` will return a handle to the `themeInjector` that can be used to change the active theme. Applications can call `themeInjector.set()`, passing in a new theme object, which will invalidate all themed widgets in the application tree and re-render them using the new theme.

To simplify the process of changing themes and to allow for easier dynamic switching in a running application, a `ThemeSwitcher` utility widget is also available from `@dojo/framework/widget-core/mixins/Themed`.

#### `ThemeSwitcher` Properties

- `renderer: (updateTheme(theme: Theme) => void): DNode | DNode[]`
  - The `renderer` that is called with an `updateTheme` function that can be used to switch themes based on user selection, for example. The `renderer` implementation should return `DNode | DNode[]`, which are the elements that will be rendered to allow theme selection within the application.
- `registryLabel ?: string`
  - (Optional) The registry label used to register the theme injector. When using the `registerThemeInjector` utility function, this property does not need to be set.

#### `ThemeSwitcher` Example Usage

The following example shows a themeable widget that uses `ThemeSwitcher` to render two buttons that allow users to switch between a `light` and `dark` theme.

```ts
import WidgetBase from '@dojo/framework/widget-core/WidgetBase';
import ThemedMixin, { ThemeSwitcher, theme, UpdateTheme } from '@dojo/framework/widget-core/mixins/Themed';

import lightTheme from '../light-theme';
import darkTheme from '../dark-theme';
import * as css from './style/MyApp.m.css';

@theme(css)
class MyApp extends ThemedMixin(WidgetBase) {
	protected render() {
		return v('div', [
			w(ThemeSwitcher, {
				renderer: (updateTheme: UpdateTheme) => {
					return v('div', [
						v(
							'button',
							{
								onclick: () => {
									updateTheme(lightTheme);
								}
							},
							['light']
						),
						v(
							'button',
							{
								onclick: () => {
									updateTheme(darkTheme);
								}
							},
							['dark']
						)
					]);
				}
			}),
			v('div', { classes: this.theme(css.container) })
		]);
	}
}
```
