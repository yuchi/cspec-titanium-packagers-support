
# White/black/gray listing files

One of the biggest problem in *distributed clients* and therefore in Mobile Apps is the size of the final product. Different tools and technics are available in order to ensure that only those resources the client really needs are shipped.

Regardless the final choice for directory structure and *package manager*, packages will always come with unwanted stuff in them, and without a clear list of files to be included.

**This sub-proposal tries to define such a tool** to reduce the number of resources that are extracted from the installed packages, with or without action from the package author.

The main philosophy is that the developer (the package consumer) will always win when conflicts arise.

In this sub-proposal we will use npm directory structure, but it should apply to other scenarios as well. If not, please consider it an issue and report it.

---

## Gray area

Every file installed from packages is by default in a **gray area**.

A grey area is a special condition where the building process simply doesn’t know if the file should be included or not.

By using static analysis tools, most probably `substack/module-deps`, and starting from the application entry points, a *module dependency graph* will be derived and used to identify resources to include.

Let’s break this down with examples.

>     • MyApp/
>       ├─ index.js
>       ├─ download-files.js
>       ├─ package.json
>       └─ node_modules/
>          ├─ backbone/
>          │  ├─ index.js
>          │  ├─ package.json
>          │  └─ node_modules/
>          │     └─ underscore/
>          │        ├─ index.js
>          │        └─ package.json
>          └─ lodash/
>             ├─ index.js
>             └─ package.json
>
> Fig. 1 — A sample app

Given the directory structure of Fig. 1 and the following *package.json*:

> ```json
> {
>   "name": "myapp",
>   "main": "index.js"
> }
> ```
>
> Fig. 2 — A sample *package.json*

And *index.js*:

> ```js
> require('backbone');
> ```
>
> Fig. 3 — A sample *index.js*

We can therefore state that:

- Because `MyApp/index.js` is defined as `main` in the *package.json* it will be
  - **included** in the app and
  - used as an **entry point**.
- By statically analyzing the code **starting from every entry point** we know that
  - `…/backbone/index.js` is required by `MyApp/index.js`
  - `…/underscore/index.js` is required by `…/backbone/index.js`
  - therefore they are **included** in the app.
- The files inside `…/lodash` are not used from any point so they are **not included**
- The file `MyApp/download-files.js` is neither required nor specified as an entry point therefore it is **not included**.

## Multiple entry points

Because a mobile app could have more than one entry points, such as those files that represent Background Services in iOS and Android, a developer can specify those as `titanium.files.entries` in the *package.json*:

> ```json
> {
>   "name": "myapp",
>   "main": "index.js",
>   "titanium": {
>     "files": {
>       "entries": [ "./download-files.js" ]
>     }
>   }
> }
> ```
>
> Fig. 4 — A *package.json* file with a background service specified as an entry.

## Static file inclusion by `require.resolve()`

Files are automatically included also when they are resolved by the code:

> ```js
> module.exports = require.resolve('./fonts/Titillium.ttf');
> ```
>
> Fig.5 — Example code that resolves the position of a static resource.

In the case at Fig. 5 the file `fonts/Titillium.ttf` is included, regardless if this code comes from the app source or a package.

## White/black listing

Because every magic process could break somewhere and produce unexpected results both package authors and developers should be able to override the default behavior by both including files otherwise ignored, and excluding files that should not be present in the final build.

### White/black listing for package authors

Package authors can include files by specifying them in `titanium.files.include` in the *package.json*.

Additional exclusions can be placed in `titanium.files.exclude`.

Standard *globs* (`**`, `*`, `?`) can be used in the name.

> ```json
> {
>   "name": "my-awesome-titanium-package",
>   "main": "lib/index.js",
>   "titanium": {
>     "files": {
>       "include": [ "./fonts/*" ],
>       "exclude": [ "./fonts/*.eof" ]
>     }
>   }
> }
> ```
>
> Fig. 6 — Example of a package with a set of included files by pattern and an excluded sub-set of it.

When inclusions and exclusions collide the `exclude` clause wins, in fact ignoring the colliding files.

### White/black listing for package consumers

When a developer wants to be sure that some files that are not under their own control but in another package are included or excluded a completely different syntax is required. The naïve approach of using `node_modules` in the path breaks very fast when de-duplication occurs.

To solve this this proposal uses the following syntax:

> ```json
> {
>   "name": "my-awesome-titanium-package",
>   "main": "lib/index.js",
>   "titanium": {
>     "packages": {
>       "titillium": {
>         "include": [ "**/*.ttf" ],
>         "exclude": [ "**/*.eof" ]
>       }
>     }
>   },
>   "dependencies": {
>     "titillium": "^1.0.3"
>   }
> }
> ```
>
> Fig. 7 — A package that explicitly includes files from a dependency package.

A package author or a developer can use the `titanium.packages` object in the *package.json* to define included and excluded files, where keys are the dependencies’ names.

Because dependencies should be considered in a global space (in order for de-dupe semantics to work) than also this package-specific configuration will be considered globally. If the package in Fig. 7 is used inside an app that directly or indirectly requires `titillium` too then the configuration will apply too.

When collisions occur of two different configuration in different packages for the same package, the `include`s and `exclude`s will be merged together.

When the collision occurs between a configuration in a package and in a final app’s *package.json* then the app configuration wins.

## Device specific configuration

The proposed syntax supports per-platform specific configuration:

> ```js
> {
>   "name": "myapp",
>   "main": "index.js",
>   "titanium": {
>     "files": {
>       "platforms": {
>         "android": {
>           "entries": [ "./download-files.js "]
>         }
>       }
>     },
>     "packages": {
>       "titllium": {
>         "include": [ "**/*.ttf" ],
>         "exclude": [ "**/*.eof" ]
>         "platforms": {
>           "windows": {
>             "include": "**/*.eof"
>           }
>         }
>       }
>     }
>   }
> }
> ```
>
> Fig. 8 — Platform specific examples

When a platform specific configuration is specified it is merged with the default one and replaces the content key-by-key.
