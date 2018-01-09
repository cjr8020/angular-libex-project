# LibexProject

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.6.3.

# Publish Angular components/modules as a library

## what is a good candidate for a library?

Here are some questions you should ask yourself before deciding if a component or service is worthy of its own module:

* Does this solve a common problem you encounter?
* Have you written code like this before and find yourself writing it repeatedly?
* Do you see other projects copy your application to use some of your components, and then modify it to remove the components they don't need?
* Is this component something you or someone else will re-use on another project? (not `might` - **will** )
* Could this be a simple building block to solving a larger issue?
* Are these collections of components unique enough that they are almost their own project?

## can this be done with angular/cli?

At this point, angular/cli does not provide a straight forward way to build and test your Angular library

## what options exist

Boils down to two options:

1. A complicated option, that inlines CSS and HTML, compiles the sources, runs Rollup.js to build [UMD](https://github.com/umdjs/umd) and ES5 modules and does some other magic.
2. A relatively simple setup that publishes TypeScript source files to npm.

When should you use which method? If you're writing a module for the community, then you'll have to go with the complicated setup. 
you will need to go and get a "library-generator". The best one I've found is https://github.com/jvandemo/generator-angular2-library, It sets up the complicated build pipeline for you and you don't have to understand exactly what it does, even though it helps, when need to troubleshoot. An example of a library build with that library generator would be http://spinner.tsmean.com. 

You should use the simple(r) option if you want to use the library for your compoany internal purposes or you want to wait until there is an "official library generator".


### Step 1: Create a new project with the angular/cli

This will be a wrapper and test consumer for your library module.

```
ng new libex-project --prefix libex
```

`libex` for "lib example".   "Prefix" is what you'll write in front of your components.

### Step 2: create a new module

Your library will reside in it's own module.

```
ng g module libex
```

or you can choose to generate your modules in `modules` folder:

```
ng g module modules/libex
```

### Step 3: build your library module

#### create `Header` component

```
ng g component libex/header
```

#### export your `Header` component

```
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [HeaderComponent],
  exports: [HeaderComponent] // <-- this
})
export class LibexModule { 
  static forRoot() {
    return {
      ngModule: LibexModule,
      providers: []
    };
  }
}
```

#### if you want to add singleton services

```
@NgModule({
  providers: [ /* Don't add the services here */ ],
  imports: [
    CommonModule
  ],
  declarations: [HeaderComponent],
  exports: [HeaderComponent]  
})
export class LibexModule {
  static forRoot() {
    return {
      ngModule: LibexModule,
      providers: [ SomeService ]
    }
  }
}
```

and change the imports in AppModule to 

```
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    LibexModule.forRoot() // <-- this
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

#### header template

```
<h1>
  <ng-content></ng-content>
</h1>
```

use content injection to inject content from parent component (app.component) just for 
demo purposes

make `header.component.css` pretty: 

```
h1 {
  color: red;
}
```
  
now add header to `app.component.html`

```
<libex-header>My Header</libex-header>  
```

We're `projecting` "My Header" into the `<ng-content></ng-content>` block.  


### Step 4: Publish

In your module folder, create a new package.json.
You can do this with `npm init` command.
It should look like:

```
{
  "name": "libex",
  "version": "1.0.16",
  "description": "An Example Library.",
      ...
}
```

For the most part, only the library name matters.
**Also rename `libex.module.ts` to `index.ts` since that's the standard name for a main file.**

Since we're publishing the typescript sources, just run

```
npm publish
```

`.npmingore`

you can also add this file so you publish only exactly what's needed.

#### subsequent releases

```
npm version patch | minor | major
npm publish
```


