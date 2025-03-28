# css-modules-to-scoped-css

This repository serves as a demo to migrate from [ember-css-modules](https://github.com/salsify/ember-css-modules) to [ember-scoped-css](https://github.com/soxhub/ember-scoped-css). It was created as part of the [Ember Initiative](https://mainmatter.com/ember-initiative/) led by Mainmatter. The Ember Initiative aims to secure the Ember ecosystem in the long run with continuous development and improvements. [Get in touch](https://mainmatter.com/contact/) with us to support the initiative and decide on our team's priorities.

## Motivation

New Ember apps from version `3.28` to the latest can be built with Vite. One blocker developers may encounter though relates to the addons they use. If your app depends on a classic addon that is incompatible with Vite, you will be stuck in the classic-build world. To prevent developers from being blocked by classic addons, the Ember Initiative team audited all the top 100 addons on [Ember Observer](https://emberobserver.com/) and started to sort them out: Which are already v2 and fully compatible? Which classic ones should be compatible with Vite? Which ones need a bit of rework? And last but not least, which ones rely too much on classic-build semantics and should be abandoned in favor of a different solution?

[ember-css-modules](https://github.com/salsify/ember-css-modules) is a widely used addon that brings CSS modules to classic Ember apps. However, it was identified as an addon that should be replaced with a different solution before moving to `@embroider/vite`. The problem is that it relies too much on classic-build world semantics, and new options come up to bring CSS modules to modern Ember apps.

This repository introduces one of these options: [ember-scoped-css](https://github.com/soxhub/ember-scoped-css). It gives an example of how to migrate a simple application from ember-css-modules to ember-scoped-css. Though it remains rather minimalist (it doesn't introduce any advanced plugin configuration that you can use with ember-css-modules), it tries to show an overview of ember-css-modules features and how you can achieve the same with ember-scoped-css.

### I want to start using CSS modules in my Ember Vite app, should I use ember-scoped-css?

No, not necessarily; ember-scoped-css is not the only option. You could also follow [Vite documentation](https://vite.dev/guide/features#css-modules) directly or look into what the Ember community came up with. The advantage of ember-scoped-css is that it's quite similar to ember-css-modules in the way the implementation is structured, so it's a migration path we recommend to unblock your upgrade to Vite without drastically changing all your CSS.

### I use ember-css-modules and want to migrate to a Vite-compatible solution, why should I favor ember-scoped-css?

The similarities between ember-css-modules and ember-scoped-css enable the possibility to get them to work at the same time. Let's assume your Ember app contains lots of components and you are using a continuous deployment process: it could be very challenging for you to upgrade to Vite if you need to drastically refactor the CSS as part of the upgrade in a single Pull Request. The path we suggest in this blog post allows you to enable ember-scoped-css file by file, so you can control the pace of the migration until ember-css-modules is no longer used.

## How to use this repo

### A demo inspired by ember-welcome-page

The `6.2` app in this repository is inspired by [ember-welcome-page](https://github.com/ember-cli/ember-welcome-page). A `<WelcomePageCopy />` component has been implemented directly in the application and has been reworked to illustrate ember-css-modules and ember-scoped-css features.

![Screenshot of the demo app, inspired from ember-welcome-page component](./demo-screenshot.png)

### View the before/after

This repository contains several branches that can be compared as a before/after diff views. These branches and views are a resource material for this guide, that illustrates the different steps of the migration: 

1. Installing ember-scoped-css for a file by file migration: [diff](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-css-modules..demo-both-addons-installed)
2. From ember-css-modules to ember-scoped-css: [diff](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-css-modules..demo-ember-scoped-css)
3. From Classic to Vite once ember-scoped-css is used: [diff](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-scoped-css..demo-ember-scoped-css-vite)

You can also have a look at the [Pull Requests](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/pulls) to read additional comments.

### Try it yourself

To better understand the differences between both addons, you might want to see the whole resulting CSS and even experiment in the browser. To help you do this as easily as possible, this repository contains two folders, `dist_ember-css-modules` and `dist_ember-scoped-css`, each corresponding to a dev build of each classic app using one or the other.

- `git clone <this repository URL>`
- In a terminal, run `npx http-server dist_ember-css-modules`
- In another terminal, run `npx http-server dist_ember-scoped-css`

By visiting the the ruturned URLs (http://127.0.0.1:8080 and http://127.0.0.1:8081 by default), you can now view both dev builds side by side and inspect the CSS.

But before digging into the differences, let's see the main idea behind "CSS isolation" that both addons implement more or less the same way.

## CSS isolation by class rename

ember-css-modules and ember-scoped-css both allow you to isolate your component styles by creating a CSS file alongside your component templates. They are different, but they both work following the same general idea: "CSS isolation by class rename". To understand this approach, you can use two resources:

- The document [CSS Isolation](https://github.com/soxhub/ember-scoped-css/blob/main/docs/css-isolation.md) from ember-scoped-css repository.
- The article [Cookbook: migrate an existing Ember app to CSS modules](https://mainmatter.com/blog/2022/08/24/cookbook-ember-app-to-css-modules/) is a walkthrough to install ember-css-modules in an Ember application that used initially one global CSS file. It presents the introduction of CSS modules with a more "practical" angle that allows you to see clearly the "without / with" CSS modules.

To sum it up, ember-css-modules and ember-scoped-css both provide you ways to have your `custom-class` renamed to `custom-class+sha` following an interpolation pattern. This way, this class becomes unique and applies only to this specific component.

ember-css-modules renames the classes to `._custom-class_sha`, and ember-scoped-css renames the classes to `.custom-class_sha`. It's a minor difference that turns to be important in the migration process (because both renaming methods conflict). There are also a few major differences that will force you to rework your code while migrating from one to the other.

## Installing ember-scoped-css for a file by file migration

✨ [**Click here to see the diff**](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-css-modules..demo-both-addons-installed) ✨

To use ember-scoped-css in a classic Ember app, you need to install `ember-scoped-css` and `ember-scoped-css-compat`. As soon as you do this, ember-scoped-css class rename starts to apply to your `.css` files and conflicts with ember-css-modules class rename. As a result, your styles are broken. To solve this problem, the idea is to split the CSS files into two categories: those handled by ember-css-modules, and those handled by ember-scoped-css. To create these categories, we are going to use the `extension` option of ember-css-modules, and a few other things.

### 1. Change the modules extension for ember-css-modules

The `extension` option of ember-css-modules defines what's the file extension of CSS modules (`.css` by default). This option is not properly documented as an individual feature, it's only mentionned in integration with other features like [using sass preprocessor](https://github.com/salsify/ember-css-modules/blob/master/docs/PREPROCESSORS.md#custom-syntax-directly-in-modules), but it actually works as an individual feature.

👉 Change the extension of all the CSS files of your app to `.module.css`, including `app.css` to `app.module.css`.

👉 Add the following configuration to your `ember-cli-build.js`:

```diff
  const app = new EmberApp(defaults, {
+   cssModules: {
+     extension: 'module.css',
+   },
  });
```

ember-css-modules should now work exactly as before.

### 2. Install and configure ember-scoped-css

👉 Install latest `ember-scoped-css` and `ember-scoped-css-compat`.

👉 Add the following configuration to your `ember-cli-build.js`:

```diff
  const app = new EmberApp(defaults, {
    cssModules: {
      extension: 'module.css',
+     intermediateOutputPath: 'css-modules.css',
    },
+   'ember-scoped-css': {
+     passthrough: ['css-modules.css'],
+     passthroughDestination: 'assets',
+   },
  });
```

👉 Create a file `app/styles/app.css` with the following content:

```css
@import 'css-modules';
```

Here, we tell ember-css-modules to emit the modules resulting CSS in a file called `css-modules.css`, then we import this file in the regular `app.css` consumed by ember-scoped-css. The `passthrough` option will tell ember-scoped-css about the existence of this file so it's correctly included in the build. All the files `.module.css` are processed by ember-css-modules, all the files `.css` are processed by ember-scoped-css, and everything gets included in the build. 

At this point, your app styles should be fixed and everything should show exactly as before.

### 3. Do the file by file migration

The migration consists in having more and more CSS modules processed by ember-scoped-css, until you no longer have any module processed by ember-css-modules. To do so, rename a file `.module.css` to `.css`, and eventually refactor ember-css-modules specific features with a different approach using the next section of this guide. 

⚠️ Be aware that ember-css-modules allows you to import styles from another CSS module (see `composes`, `@value`, and `{{local-class}}` later in this guide). If you use this feature, figure out the best migration order for all your files. For instance, you may want to refactor your current CSS to migrate away from the `composes` pattern even while you are still using ember-css-modules.

## From ember-css-modules to ember-scoped-css

✨ [**Click here to see the diff**](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-css-modules..demo-ember-scoped-css) ✨

### 1. `local-class` _versus_ `class`

ember-css-modules introduces the attribute `local-class` as a marker that identifies scoped classes. In a template, it's your responsibility as a developer to assign to `local-class` the scoped classes that you defined in the corresponding CSS module, and to assign to `class` the global classes.

In ember-scoped-css, you only use the attribute `class`. If the class name is defined in the CSS module, it will be transformed, else it just stays what it is and the global styles apply.

### 2. Tag selectors: global _versus_ local

ember-css-modules is designed to rename class selectors. It doesn't rename tag selectors like `div`, `p`, `img`, `a`... even when they are defined in the CSS module of a component. These selectors are bundled as is in the final style and therefore the style applies globally. In that way, using tag selectors in components when using ember-css-modules requires caution.

ember-scoped-css is a bit more intuitive on that front. The tag selectors are no longer global but scoped. When these selectors are used in the component's CSS, a class named after the `sha` is added to the corresponding DOM elements, and the syntax `selector.sha` is used CSS-side to scope the style only to the component. This way, tag selectors behave just like any class selector, which can bring clarity to the CSS module.

```html
<!-- Final DOM -->
<ul class="e9accf110">
  <li class="e9accf110"></li>
</ul>
```

```css
/* Built CSS */
ul.e9accf110 > li.e9accf110 {
  padding-bottom: 0.5em;
  font-size: 1.1em;
}
```

### 3. Id selectors: weird _versus_ global

ember-css-modules behavior with ids is a bit weird. It doesn't rename element ids in the DOM, but it transforms them in the CSS. As a result, if you use HTML ids to style the component, the styles are not applied.

```html
<!-- Final DOM -->
<main id="ember-welcome-page-id-selector"></main>
```

```css
/* Built CSS */
#_ember-welcome-page-id-selector_f42zxn {
  padding: 2em;
  border-style: solid;
  border-width: 1px;
}
```

ember-scoped-css, on the other hand, tries to be a bit smarter on that field. It considers the following HTML principle: the HTML id should be unique in the document. Therefore, ember-scoped-css doesn't transform ids at all and the style remains global.

### 4. Each CSS file is a CSS module _versus_ each component has its CSS module

In ember-css-modules, each CSS file in your `app/styles/` folder is considered a CSS module, and all the classes in there are scoped. This approach allows you to scope the CSS of route controllers. For instance, the local CSS of your `application.hbs` is expected to be defined in `app/styles/application.css`. The class names transform also applies to `app/styles/app.css` (this questionable behaviour is the purpose of [an issue](https://github.com/salsify/ember-css-modules/issues/195), but let's consider the addon as it is here). Each time you want to apply a style globally using a class name, you need to use `:global` pseudo-selector.

ember-scoped-css is more... "scoped". The idea of this addon is to scope your components CSS by creating a `component.css` alongside a `component.hbs`. On the other hand, `app/styles/app.css` contains global CSS, and if you have other global CSS files they should be explicitly imported.

### 5. Importing styles and properties _versus_ parent passes in to child

#### i. ember-css-modules specific features

ember-css-modules has three specific features:

`{{local-class}}` helper imports a local class from another CSS module directly in the template.

```hbs
<p class="{{local-class 'postscript' from='css-modules-to-scoped-css/components/welcome-page-copy.css'}}">
```

`composes` gets a local class inherited from another local class that could be located in another CSS module.

```css
.guide {
  composes: guide from 'css-modules-to-scoped-css/components/welcome-page-copy.css';
  font-weight: bold;
}
```

`@value` imports a CSS variable that could be located in another CSS module.

```css 
/* app/styles/app.css */
@value ember-orange: rgb(255, 92, 68);

/* app/components/welcome-page-copy.css */
@value ember-orange from 'css-modules-to-scoped-css/styles/app';
```

All of these features are based on the same idea: given the path to a CSS module, the content of this CSS module can be imported in another CSS module. Phrased this way, the approach sounds inspired from importing js modules in our code files, but applied to CSS with a very specific semantic.

#### ii. Alternatives in ember-scoped-css

`{{scoped-class}}` helper can be used to pass a local class from the parent component to the child component, which results in something similar to the former `{{local-class}}`. The approach is still very different: the scoped class is passed in from the component that defines it (rather than any component can retrieve the scoped class from anywhere given the path), and since only components have their CSS scoped, we can't use that helper from outside a component (because outside a component there is no CSS module with scoped classes to pass in).

Instead of relying on `@values`, we can try to rework the styles with [CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties) (also called CSS variables) directly.

### 6. Classic cascading _versus_ providing layers

ember-css-modules transforms the class names but doesn't take any initiative such as emitting layers.

By default, ember-scoped-css emits the components CSS in a [CSS layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer) `components`. The name of the layer is [configurable](https://github.com/soxhub/ember-scoped-css/blob/02488a40003923409ffc1ab3fdd1bd6bed7eddaf/README.md#configuration-1).

⚠️ If you were not using layers until now, you can choose disable this behavior with `layerName: false`, but... We recommend to start using them. [Vite manages the CSS](https://vite.dev/guide/features.html#css) a bit differently in dev mode and build mode. Without layers to state the CSS order clearly, you may end up with a different order between what you see in development and your production build.

As long as your app is still a classic app though, `ember-scoped-css` is not very intuitive when it comes to layers. You need to state the layers order first, but `ember-scoped-css` won't let you do this in `app.css`, it will define your components styles first. To work around this issue, you can add a `<style>` tag in your `index.html` before any CSS is imported:

```diff
+ <style>
+   @layer utilities, components;
+ </style>

  <link integrity="" rel="stylesheet" href="{{rootURL}}assets/vendor.css">
  <link integrity="" rel="stylesheet" href="{{rootURL}}assets/css-modules-to-scoped-css.css">
```

## From Classic to Vite once ember-scoped-css is used

✨ [**Click here to see the diff**](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/compare/demo-ember-scoped-css..demo-ember-scoped-css-vite) ✨

Once your classic app uses only ember-scoped-css without style regressions, your app is one step closer to Vite. If you don't have any other blocker to handle, then you can start building with Vite. [ember-vite-codemod](https://github.com/mainmatter/ember-vite-codemod) is here to help!

👉 After running the codemod, remove `ember-scoped-css-compat` from your dependencies. It was here only for the compatibility with the classic app.

👉 [Adapt ember-scoped-css configuration](https://github.com/soxhub/ember-scoped-css?tab=readme-ov-file#configuration). You can also have a look at [the corresponding PR](https://github.com/BlueCutOfficial/css-modules-to-scoped-css/pull/2) on this repository to see additional comments.


