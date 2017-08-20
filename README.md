# Sharing ReactJs Code in a Monorepo

This presentation was powered by [reveal-js](https://github.com/hakimel/reveal.js#online-editor).

## Running

`yarn`
`yarn start`

Presentation will open in `localhost:8000`. In the next section, you'll find a rough draft of the presenter notes. I will most certainly forget to touch most of those point in the actual presentation.

## Presenter notes

**0. Intro**

Tonight I want to walk you through our experiences at simplesurance in regards to sharing javascript code between multiple frontend apps in a monorepo.


**1. Monorepo? What is that? Why would you do that?**

First of all, what is a monorepo? A monorepo is a git repository organization strategy where all the code for the different parts of a project are in the same place. So all your backend, your APIs, microservices, frontend apps and everything else lives in harmony in single git repo.

If you transition to a monorepo, the different parts of your code are now always in sync. There's no more code reviewing across multiple repositories. There's no more "this pull request depends on that pull request in the the other repo to be merged first". Refactoring pieces of one app into a separate microservice becomes faster and cheaper.

**2. Our case. Sisu's stack**

At simplesurance, we have different kinds of app co-existing in our monorepo. We have symfony/twig MVC apps, PHP and Go APIs, PHP and Go microservices, React and Angular frontend apps, and others.

**3. The case for splitting frontend apps**

When we initially started to port parts of our frontend to ReactJS, we saw that as an opportunity to have separate self-contained single-page. From an UX prespective, our single-page applications provide a much better user experience by keeping the user more focused and informed about the state of the whole process of purchase or claim. On the developer side, we have modular apps that have a clear purpose and specific target actors with different needs. It also favours developers working in parallel on different modules.

**4. Advantages of sharing frontend code**

Rethinking our frontend was also a good chance to finally work with the design team to finally create corporate identity guidelines for all apps. Sharing code between our apps allows us to keep a consistent deign as well as sharing settings like colors and icons, and assets like images and fonts.

**5. Which code to share**

Components that are useful to share are things like form fields - inputs, check boxes, radio buttons, signature fields - common elements like buttons, lists, cards, and other containers and layouts like headers, navigation bars and the whole page skeleton.
It's also useful to share your translations if you have them as static files.

**6. The problem**

The problem was that, as convenient as it was to have all the shared components in a single place, using them was painful.
Say we had a component in `our-app/src/components/page/view.js` that needed to import the component for the site header. We had to write something like `import SiteHeader from '../../../../shared-code/src/components/common/SiteHeader/view.js'`. Which is an eyesore, poorly legible and hard to maintain. Also, depending on how deep we were in our code, we would need a different import statement. So we wanted to avoid this.

**6. Strategy 1: use symbolic links**

The first idea before we even started was that we could use symbolic links. With symbolic links, we could fool the app into thinking the share code was part of its code base and everything would be very zen and work.
Unfortunately, webpack saw through our plans and resolved the symbolic links. So the `import` paths did not work as expected because the components were not defined in the correct location.

**7. Strategy 2: why not a private npm package?**

At this point one might: why not just create a private npm package containing the shared code and include it as a dependency of our project?
Well, that would defeat the purpose of working on a monorepo. It would require that we manage the package, update it accordingly, and release regular updates. Also, it means that new features that change a share component need to push code to two different repositories and the app update becomes blocked by the update of the private npm package.
Furthermore, in this early stage of the app development it is critical that development is agile, so being able to push changes to the app and to the share component at the same time is very convenient.

**8. Strategy 3: webpack aliases**

Later we found about webpack aliases. This would surely get rid of those horribly long import statements. Webpack lets us define an alias for any directory

```
resolve: {
  alias: {
    components: path.resolve(__dirname, '../shared-code')
  }
}
```

Then, we export all our shared components from an `index.js` at the root of the directory, like this:

```
export * from "./common"
export * from "./config/colors"
export * from "./config/dimensions"
export * from "./config/icons"
export * from "./inputs"
export * from "./layout"
...
```

So our old import statement can be turned into just `import SiteHeader from 'components'`.
This seemed great but there was still one issue: our shared components, written in ES6 were not being transpiled to ES5 anymore. Well, that's natural because it's not inside our `src` directory.

**9. Strategy 4: pre-built "package" and webpack aliases**

The final step was to tell webpack that the shared code had to be transpiled by babel.

```
rules: [{
  test: /.jsx?$/,
  include: [
    path.resolve(__dirname, '../components'),
    path.resolve(__dirname, 'src')
  ],
  loader: 'babel-loader'
}]
```

Or at least we thought this was the final step. Because webpack cannot find babel or any other node modules from within the shared directory. The solution was to, ironically, making it a node package. We ran `npm init` and gave it some minimum settings and all the dependencies it needed. We just had to make sure that when we build our app we also build the shared code.

**10. Final notes**

We're not absolutely sure this was the ideal solution in the long term. Maybe releasing our component palette as a private npm package would be the best way to use it in the future. It's very hard to create something future-proof in the frontend world when everything changes so fast. This was our solution, I hope you learned something with it, and let me know if you have any questions.
