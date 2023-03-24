# Repository Overview (CDapp)

## Components

Inside the "Components" folder, we have three subfolders:

#### Common: More complex pieces of UI, often piecing together components from the “shared” directory

#### Frame: Layout and app structure

#### Shared: “Core” / generic components

Please ensure your components end up in the correct one.

### Structure

When developing a new component, or extracting reusable components, the folder structure should look something like this:

└── common/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── NewComponent/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── index.ts
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── NewComponent.tsx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── NewComponent.css
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── NewComponent.css.d.ts
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── (optional) types.ts
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── (optional) helpers.ts

**index.ts**: the file that exports the component, and anything else from within the sub-directory
**NewComponent.tsx**: the file the actual component lives in
**NewComponent.css**: where component styles live
**NewComponent.css.d.ts**: Automatically-generated file so that the style object (a post-css enhacement) is typed correctly

Optional:
**types.ts**: where component-specific types live
**helpers.ts**: where you put functions that help make business logic more concise and reusable

### Imports

Imports belong at the top of the file, and in the following order:

1. External library imports
2. Imports from other folders (typically accessed via '~', e.g. '~common/SomeComponent')
3. Imports from this folder or parent folder
4. The style import

Note that we use webpack to transform paths, so if a path is registered in the webpack config, you can access via the ~ prefix.

### DisplayName and Component naming

Ensure components names are PascalCase, named logically given the component they represent, and have a displayName property (which makes debugging easier). The displayName should be assigned to a `const` at the top of the file, so it can be reused in the `MSG` object, and should more or less mirror the file structure (e.g. `common.NewComponent.SubComponent`)

### The MSG object

We use [React-Intl](https://formatjs.io/docs/react-intl/) to format strings for an international audience. Often, you will find a `MSG` object at the top of a component file. All user-facing strings belong inside it.

### File length

Generally, we want to keep files as small as possible and break code up into its constituent parts where possible. This makes code easier to debug, reuse, and review in the future. This applies to both React components as well as ordinary javascript.

Components over 100-150 lines could probably be refactored either using hooks or encapsulating parts of the view into their own components. This is not a strict rule, but it's important to keep in mind as a component grows. Even if you weren't initially responsible for the component, once you start adding code that takes the file length beyond this limit, it's **your responsibility** to refactor it.

### Styles

Consult our `variables.css` folder for a list of frequently used styles. If your style already exists in here, please access it via a css variable, and generally avoid adding new variables (especially if it's very similar to an existing one)
