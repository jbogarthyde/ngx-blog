# Route Types

Angular comes with a great router. Here I want to show you some different route types and how they work.
I assume that you have some experience with the Angular Router.

## Predefined standard routes

Here's an example of a common, simple route type that associates a component with a predefined constant.

```js
[
  {
    path: 'start',
    component: StartComponent
  }
];
```

The path-matching regular expression for `start` looks like this:

```js
const regex = '/^/start$/';
```

## Defining default routes

There are several ways to define a default route.
 
 * Use a wild card in the path value.
 * Use a null path value with a redirect.
 * Use a null path value with child routes.

### Wild cards for a default route

A wild card is specified with two asterisk characters `**`.
This route will be activated if the user enters a URL that does not match any other route registered.
The following snippet shows that `LumpenSammlerComponent` will be shown when the wild card gets activated.

```js
[
  {
    path: '**',
    component: LumpenSammlerComponent
  }
];
```

> ⚠️ **IMPORTANT**: The Router goes through your router configuration array from the top down,
> checking each path of the config and testing it with a Regular Expression against the current route.
> That means the order of your route configuration matters.
> For child modules or lazy-loaded modules, the Router merges the child routes _before_ the `**` route.
> This means it really comes last.

The regular expression for a wildcard looks like this:

```js
const regex = '^(?:([^/]+))$';
```

### Redirect to a default route

The following snippet defines a default route that is activated if no route is given.

```ts
[
  {
    path: '',
    redirectTo: '/start',
    pathMatch: 'full'
  }
];
```

The `pathMatch` flag specifies the matching strategy. It can be `full` or `prefix`.
The default is `prefix`, meanting that the Router will look at what is left in the URL, and check if it starts with the specified path

```ts
'/blog/11'  => 'blog/:id'
```

You can change the matching strategy to `full` to make sure that the path covers the whole unconsumed URL.

This is particularly important when redirecting empty-path routes.

### Empty path

This type of route does not "consume" any URL segments. It is a perfect fit if you want to use child routing.

```js
[
  {
    path: '',
    component: WelcomeComponent
  }
];
```

See an [example of child routing](https://angular.io/guide/router#child-route-configuration).

## Routes with parameters

This is the most common way to transport data in the route and have a dynamic route.
The string at the segment which is marked with `:id` is stored in the Observable `ActivatedRoute.params`.

```json
{
  "id": "5"
}
```

```js
[
  {
    path: 'blog/:id',
    component: BlogComponent
  }
];
```

The regular expression for `blog/:id` looks like this.

```js
const regex = '/^/blog/(?:([^/]+))$/';
```

## Define a custom route matcher

Definitely a frequently asked question in my workshops is:

- _Q: Can I define a specific regex for a route?_
- **A: Yes!**

Ok, this is not enough so I will explain how you can do this.

A standard route configuration defines a `path` to match against, and a `pathMatch` value that determines which predefined matching strategy to apply.
When the combination of `path` and `pathMatch` isn't expressive enough, you can define a custom URL matcher function that supersedes the default behavior.

Here is a example of a function that matches against a 'numbers-only' regular expression.

```js
const numberRegex = '^[0-9]*$';
const regexMatcher = (url: UrlSegment[]) => {
  return url.length === 1 && url[0].path.match(numberRegex)
    ? { consumed: url }
    : null;
};
[
  {
    matcher: regexMatcher,
    component: BlogComponent
  }
];
```

To use this, we have to define `routeParams`.
We'll define them in the returned object as a `UrlSegment` that can be resolved by the Router. Sounds complicated? It isn't.

```js
const numberRegex = '^[0-9]*$';
const regexMatcher = (url: UrlSegment[]) => {
  return url.length === 1 && url[0].path.match(numberRegex)
    ? {
        consumed: url,
        posParams: {
          id: new UrlSegment(url[0].path, {})
        }
      }
    : null;
};
[
  {
    matcher: regexMatcher,
    component: BlogComponent
  }
];
```

Now we can read the Observable `ActivatedRoute.params` as usual.

```js
this.ActivatedRoute.params.subscribe(p => {
  console.log(p);
  this.id = p.id;
});
```

### Use a custom matcher for i18n

This next example is a great way to have internationalization in the routes.

```js
const i18nRegex = '(needle|nadel)';
const i18nMatcher = (url: UrlSegment[]) => {
  return url.length === 1 && url[0].path.match(i18nRegex)
    ? {
        consumed: url,
        posParams: {
          name: new UrlSegment(url[0].path, {})
        }
      }
    : null;
};
```

#### Credits

Thx to

- <a href="https://twitter.com/brandontroberts"  target="_blank">Brandon Roberts</a> for the inspiration.
- <a href="https://twitter.com/GregOnNet"  target="_blank">Gregor Woiwode</a> for Spelling, style and grammar checks.
- <a href="https://twitter.com/FabianGosebrink" target="_blank">Fabian Gosebrink</a> for Spelling, style and grammar checks.

Thanks, friends! It means a lot!
