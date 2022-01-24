---
layout: post
title: "How to handle errors in SwaggerUI"
description: "Error handling is an essential aspect of today's modern Single Page Applications. Error handling refers to the anticipation, detection, and resolution of different kinds of errors. As of version v4.3.0, SwaggerUIs error handling capabilities have considerably improved and allowed SwaggerUI integrators to easily integrate their custom errors handlers."
date: 2022-01-24 10:00:00 +0200
image:
  path: assets/img/blog/swagger-ui-error-handling.webp
  width: 300
  height: 299
  caption: SmartBear Software, Inc.
  object-fit: scale-down
---

<p class="lead">
  Error handling is an essential aspect of today's modern Single Page Applications.
  Error handling refers to the anticipation, detection, and resolution of different kinds of errors.
  As of version <a href="https://www.npmjs.com/package/swagger-ui/v/4.3.0">v4.3.0</a>, <a href="https://github.com/swagger-api/swagger-ui">SwaggerUIs</a> error handling capabilities have considerably improved
  and allowed SwaggerUI integrators to easily integrate their custom errors handlers. 
</p>

Original SwaggerUI error handling consisted of wrapping every render method of every class component
with an imperative `try/catch` statement and displaying the `Fallback` component if the error was thrown.
Along with displaying the Fallback component, `console.error` was used to display the caught error in the browser console.
This solution required wrapping every function component into class component to make sure that
the component has the `render` method.

React 16 introduces a new concept of an [error boundary](https://reactjs.org/docs/error-boundaries.html).
It's no longer necessary to create a custom solution for catching errors in React apps, as we have a standardized way of doing it.

In order to fully utilize new error boundaries, I had to rewrite SwaggerUIs [view plugin](https://github.com/swagger-api/swagger-ui/tree/master/src/core/plugins/view), which all other 
plugins build on. An additional new plugin called `safe-render` was introduced to allow configurable
application of error boundaries to SwaggerUI components.

### View plugin

View plugins **public API didn't change**, I've just added one additional util function called `getDisplayName`
which is used for accessing React component names.

{% highlight js linenos %}
{
  rootInjects: {
    getComponent: memGetComponent,
    makeMappedContainer: memMakeMappedContainer,
    render: render(getSystem, getStore, getComponent, getComponents),
  },
  fn: {
    getDisplayName,
  },
}
{% endhighlight %}

`view` plugin is no longer responsible for error handling logic. Rewriting the plugin had several
positive side effects like making the SwaggerUI rendering faster and simplifying the React component
tree in React developer tools:


<figure class="figure d-block border">
  <img src="{% link assets/img/blog/swagger-ui-error-handling-rdtd-before.webp %}" width="539" height="466" class="figure-img rounded mx-auto d-block img-fluid" alt="todoList plugin" />
  <figcaption class="figure-caption text-center">Before refactor</figcaption>
</figure>

<figure class="figure d-block border">
  <img src="{% link assets/img/blog/swagger-ui-error-handling-rdtd-after.webp %}" width="551" height="305" class="figure-img rounded mx-auto d-block img-fluid" alt="todoList plugin" />
  <figcaption class="figure-caption text-center">After refactor</figcaption>
</figure>

### Safe-render plugin

This plugin is solely responsible for error handling logic in SwaggerUI. Accepts a list of component
names that should be protected by error boundaries.  

It's public API looks like this:

{% highlight js linenos %}
{
  fn: {
    componentDidCatch,
    withErrorBoundary: withErrorBoundary(getSystem),
  },
  components: {
    ErrorBoundary,
    Fallback,
  },
}
{% endhighlight %}

safe-render plugin is automatically utilized by [base](https://github.com/swagger-api/swagger-ui/blob/78f62c300a6d137e65fd027d850981b010009970/src/core/presets/base.js) and [standalone](https://github.com/swagger-api/swagger-ui/tree/78f62c300a6d137e65fd027d850981b010009970/src/standalone) SwaggerUI presets and
should always be used as the last plugin, after all the components are already known to the SwaggerUI.
The plugin defines a default list of components that should be protected by error boundaries:

{% highlight js %}
[
  "App",
  "BaseLayout",
  "VersionPragmaFilter",
  "InfoContainer",
  "ServersContainer",
  "SchemesContainer",
  "AuthorizeBtnContainer",
  "FilterContainer",
  "Operations",
  "OperationContainer",
  "parameters",
  "responses",
  "OperationServers",
  "Models",
  "ModelWrapper",
  "Topbar",
  "StandaloneLayout",
  "onlineValidatorBadge"
]
{% endhighlight %}

As demonstrated below, additional components can be protected by utilizing the safe-render plugin 
with configuration options. This gets really handy if you are a SwaggerUI integrator and you maintain a number of
plugins with additional custom components.

{% highlight js linenos %}
const swaggerUI = SwaggerUI({
  url: "https://petstore.swagger.io/v2/swagger.json",
  dom_id: '#swagger-ui',
  plugins: [
    () => ({
      components: {
        MyCustomComponent1: () => 'my custom component',
      },
    }),
    SwaggerUI.plugins.SafeRender({
      fullOverride: true, // only the component list defined here will apply (not the default list)
      componentList: [
        "MyCustomComponent1",
      ],
    }),
  ],
});
{% endhighlight %}

##### componentDidCatch

This static function is invoked after a component has thrown an error.  
It receives two parameters:

1. `error` - The error that was thrown.
2. `info` - An object with a componentStack key containing [information about which component threw the error](https://reactjs.org/docs/error-boundaries.html#component-stack-traces).

It has precisely the same signature as error boundaries [componentDidCatch lifecycle method](https://reactjs.org/docs/react-component.html#componentdidcatch),
except it's a static function and not a class method.

Default implement of componentDidCatch uses `console.error` to display the received error:

{% highlight js %}
export const componentDidCatch = console.error;
{% endhighlight %}

To utilize your own error handling logic (e.g. [bugsnag](https://www.bugsnag.com/)), create new SwaggerUI plugin that overrides componentDidCatch:

{% highlight js linenos %}
const BugsnagErrorHandlerPlugin = () => {
  // init bugsnag

  return {
    fn: {
      componentDidCatch = (error, info) => {
        Bugsnag.notify(error);
        Bugsnag.notify(info);
      },
    },
  };
};
{% endhighlight %}

##### withErrorBoundary

This function is HOC (Higher Order Component). It wraps a particular component into the `ErrorBoundary` component.
It can be overridden via a plugin system to control how components are wrapped by the ErrorBoundary component.
In 99.9% of situations, you won't need to override this function, but if you do, please read the source code of this function first.

##### Fallback

The component is displayed when the error boundary catches an error. It can be overridden via a plugin system.
Its default implementation is trivial:

{% highlight js linenos %}
import React from "react"
import PropTypes from "prop-types"

const Fallback = ({ name }) => (
  <div className="fallback">
    😱 <i>Could not render { name === "t" ? "this component" : name }, see the console.</i>
  </div>
)
Fallback.propTypes = {
  name: PropTypes.string.isRequired,
}
export default Fallback
{% endhighlight %}

Feel free to override it to match your look & feel:

{% highlight js linenos %}
const CustomFallbackPlugin = () => ({
  components: {
    Fallback: ({ name } ) => `This is my custom fallback. ${name} failed to render`,
  },
});

const swaggerUI = SwaggerUI({
  url: "https://petstore.swagger.io/v2/swagger.json",
  dom_id: '#swagger-ui',
  plugins: [
    CustomFallbackPlugin,
  ]  
});
{% endhighlight %}

##### ErrorBoundary

This is the component that implements React error boundaries. Uses `componentDidCatch` and `Fallback`
under the hood. In 99.9% of situations, you won't need to override this component, but if you do, 
please read the source code of this component first.


### Change in behavior

In prior releases of SwaggerUI, almost all components have been protected, and when thrown error,
`Fallback` component was displayed. This changes with SwaggerUI v4.3.0. Only components defined
by the `safe-render` plugin are now protected and display fallback. If a small component somewhere within
SwaggerUI React component tree fails to render and throws an error. The error bubbles up to the closest
error boundary, and that error boundary displays the `Fallback` component and invokes `componentDidCatch`.

If you're interested in more technical details, here is the [PR](https://github.com/swagger-api/swagger-ui/pull/7761/files)
that introduced the new error handling into SwaggerUI.