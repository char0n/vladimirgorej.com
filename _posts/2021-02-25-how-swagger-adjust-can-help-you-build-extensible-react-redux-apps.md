---
layout: post
title: "How Swagger Adjust can help you build extensible React+Redux apps"
description: "How Swagger Adjust can help you build extensible React+Redux apps"
date: 2021-02-25 10:39:04 +0200
image:
  path: assets/img/blog/swagger-adjust.png
  width: 496
  height: 288
  caption: Swagger Adjust
---

More than four years ago, I was working on a backend project where I needed to render HTML
documentation from [OpenApi 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) JSON document defining our REST API written in hapi.js. 
The obvious choice was of course SwaggerUI. [hapi.js](https://hapi.dev/) ecosystem came with already written
integrations for OpenApi 2.0 and [SwaggerUI](https://swagger.io/tools/swagger-ui/). We just needed to use them; and we did. 
It all worked like a charm. At that time I was successfully getting out from complex
[Angular 1.2/2](https://devdocs.io/angularjs~1.2/) world and getting into simple and more predictable world of [React](https://reactjs.org/)+[Redux](https://redux.js.org/). 
I figured out that SwaggerUI was built in React so I navigated to its [GitHub repo](https://github.com/swagger-api/swagger-ui) and 
started to read the code to learn how large scale React apps are written. The code was
different then the code I was used to write. I built React applications using classical
patterns like [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) (smart & dump components) and [ducks pattern](https://github.com/erikras/ducks-modular-redux).
It took me some time to understand that SwaggerUI was primarily built for extensibility in mind. 
Everything inside the code was revolving around this primary idea. Codebase was composed from
number of plugins. These plugins were either adding new behaviors to SwaggerUI or enhancing
other behaviors defined in different plugins. There were almost none static imports involved 
because it had its own flavor of DI-container called **System**. Although the architecture looked 
interesting it seemed like an overkill to use something like that in my current project, 
where requirement for extensibility didn’t exist (well..it actually did but the extensibility
needed to be internal and not external).

In 2017, I was employed by a company called [Apiary](https://apiary.io/). I joined it a couple of months after 
the acquisition by [Oracle](https://www.oracle.com/) and my primary task was to architect and develop a new version
of Apiary Documentation Renderer named: **ApiaryUI**. When I started to work on it, I remembered
my study on SwaggerUI and it’s unique architecture. I built ApiaryUI around the same ideas
of external extensibility. Of course the code was completely different but the architecture
was very similar to SwaggerUI. The core of ApiaryUI was based on a DI-container called **Kernel**.
I managed to finish the new [ApiaryUI](https://blogs.oracle.com/developers/apiaryui) on time. Unfortunately it has never been released as OpenSource,
due to different political and organizational reasons but it’s been utilized by Apiary
in their [web app](https://apiary.io/) and used throughout different Oracle organizations.

Today I work for [SmartBear](https://smartbear.com/), the company that’s behind SwaggerUI and the entire Swagger ecosystem.
My primary focus in SmartBear is not on SwaggerUI, but from the moment I joined the company
I started to think how to improve and modernize it’s DI-container (System). I’ve been actually
thinking about this for a long time. I started writing this article almost 3 years ago, 
when I decoupled it’s DI-container for the first time as I needed it for a small project
with external extensibility requirements. Unfortunately I never finished the article and 
never published the decoupled SwaggerUI DI-container as an OpenSource.

### Swagger Adjust

But it’s all changing now. Let me introduce [Swagger Adjust](https://github.com/char0n/swagger-adjust)  - a plugin-able framework for
building extensible React+Redux applications. This framework was born by decoupling ideas of 
the plugin-able core of SwaggerUI into separate reusable component. It allows you to build
React+Redux applications in a way, where every component, being it a React Component, 
Redux state slice composed of actions, selector, reducers or general business logic is
override-able and extensible.

Swagger Adjust is a completely rewritten SwaggerUI’s DI-container (System) that is built on top of
[Redux Toolkit](https://redux-toolkit.js.org/). The name for the DI-container is the same as SwaggerUI’s – **System**. 
Generally the API of Swagger Adjust  is backward compatible with SwaggerUI.
When I say “generally” I mean it in a way that the default configuration of Swagger Adjust 
is incompatible with SwaggerUI, but can be adapted by configuration to be fully compatible.
Swagger Adjust is a free and Open Source software and it comes with a 
dual license – Apache License Version 2.0 is used for original code fragments,
documentation fragments and original ideas that comes from SwaggerUI and BSD 3-Clause License
is used for everything else. 

To demonstrate Swagger Adjust power I’ve built a [simple TodoList app](https://char0n.github.io/swagger-adjust/) which is composed of two plugins: 
[todoList](https://github.com/char0n/swagger-adjust/tree/main/demo/src/todo-list) and [todoListEnhancer](https://github.com/char0n/swagger-adjust/tree/main/demo/src/todo-list-enhancer). todoList plugin has basic functionality of displaying a list of TodoItems 
and adding a new one. todoListEnhancer adds additional functionality like completion, deletion and also 
batch operations on the entire TodoList.

<figure class="figure d-block">
  <img src="{{ '/assets/img/blog/swagger-adjust-todo-list-plugin.png' | relative_url }}" width="533" height="256" class="figure-img rounded mx-auto d-block img-fluid" alt="todoList plugin" />
  <figcaption class="figure-caption text-center">todoList plugin</figcaption>
</figure>

todoList plugin looks like this when it renders. It contains AppBar, input for creating new todo
items and list of already added todo items.

<figure class="figure d-block clearfix">
  <img src="{{ '/assets/img/blog/swagger-adjust-todo-list-plugin-dir-structure.png' | relative_url }}" width="230" height="242" class="float-md-end" alt="todoList plugin" />
  <figcaption class="figure-caption">
    Directory structure of todoList plugin looks pretty standard. We have a components directory 
    associated with all the Redux sweetness: actions.js, selectors.js and reducers.js.
    There is nothing special about these 3 files. They’re unaware of the fact that they're 
    part of plugging-able DI-container and are built in the same way as they’d be built using Redux Toolkit.
    Please take a moment here and <a href="https://github.com/char0n/swagger-adjust/tree/main/demo/src/todo-list">browse these files</a> yourself to see the actual code. 
  </figcaption>
</figure>

The only non-standard file is [plugin.js](https://github.com/char0n/swagger-adjust/blob/main/demo/src/todo-list/plugin.js). This file imports everything that todoPlugin consists of and 
exports a function returning an [object with specialized shape](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#format) that our DI-container recognizes.

{% highlight javascript linenos %}
import TodoListLayout from './components/TodoListLayout';
import TodoListAppBar from './components/TodoListAppBar';
import TodoListInput from './components/TodoListInput';
import TodoList from './components/TodoList';
import TodoListItem from './components/TodoListItem';
import { addItem } from './actions';
import { selectTodoList, selectTodoListItems } from './selectors';
import reducers, { initialState } from './reducers';

const TodoListPlugin = () => {
  return {
    components: {
      TodoListLayout,
      TodoListAppBar,
      TodoListInput,
      TodoList,
      TodoListItem,
    },
    fn: {
      todoList: {},
    },
    statePlugins: {
      todoList: {
        initialState,
        actions: { addItem },
        selectors: { selectTodoList, selectTodoListItems },
        reducers,
      },
    },
  };
};

export default TodoListPlugin;
{% endhighlight %}

Now this is where real magic happens. Instead of using static imports in React components, 
we get everything we need from DI-container using specialized [hooks](https://github.com/char0n/swagger-adjust/blob/main/docs/usage/hooks-api.md). For the TodoList component
to properly render we need to select all todo items from the state and then use TodoListItem
component to render them. 

{% highlight jsx linenos %}
import React from 'react';
import List from '@material-ui/core/List';
import Divider from '@material-ui/core/Divider';
import { makeStyles } from '@material-ui/core/styles';
import { useSystemComponent, useSystemSelector } from 'swagger-adjust';

const useStyles = makeStyles(() => ({
  root: {
    paddingTop: 0,
    paddingBottom: 0,
  },
}));

const TodoList = () => {
  const classes = useStyles();
  const items = useSystemSelector('todoList', 'selectTodoListItems');
  const TodoListItem = useSystemComponent('TodoListItem');

  return (
    <List className={classes.root}>
      {items.map((item) => (
        <React.Fragment key={item.id}>
          <TodoListItem item={item} />
          <Divider />
        </React.Fragment>
      ))}
    </List>
  );
};

export default TodoList;
{% endhighlight %}

DI-container has its own [React Context](https://reactjs.org/docs/context.html) called [SystemContext](https://github.com/char0n/swagger-adjust/blob/main/docs/usage/api.md#systemcontext) which accepts it’s instance. 
First thing we do is – we compose a list of plugins and pass it to the System class constructor
(represents DI-container). DI-container automatically compiles the plugins under the hood and 
creates a store instance for us. Then we wrap our main App component into [react-redux](https://react-redux.js.org/api/provider) and
SystemContext providers. This allows hooks mentioned above to access anything from the DI-container 
within React components.

{% highlight jsx linenos %}
/**
 * This component is just responsible to render the main TodoList component.
 * We need to this to allow other plugins to override TodoListLayout.
 */

const App = () => {
  const TodoListLayout = useSystemComponent('TodoListLayout');

  return <TodoListLayout />;
};

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <SystemContext.Provider value={system.getSystem}>
        <App />
      </SystemContext.Provider>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
{% endhighlight %}

Let’s look into our second plugin – todoListEnhancer. As mentioned before this plugin enhances the 
original todoList plugin and adds additional features. Enhancement doesn’t happen via todoList 
plugin code modification but rather by it’s configuration. And this is where Swagger Adjust really shines.

<figure class="figure d-block">
  <img src="{{ '/assets/img/blog/swagger-adjust-todo-list-enhancer-plugin.png' | relative_url }}" width="533" height="256" class="figure-img rounded mx-auto d-block img-fluid" alt="todoListEnhancer plugin" />
  <figcaption class="figure-caption text-center">todoListEnhancer plugin</figcaption>
</figure>

Every todo item in todoListEnhancer is now associated with creation time, checkbox for completion, 
bin icon for deletion and at the bottom of TodoList there are two buttons for batch operations. 

Here we’re presented with todoListEnhancer directory structure and the source code of plugin.js.
Inside [plugin.js](https://github.com/char0n/swagger-adjust/blob/main/demo/src/todo-list-enhancer/plugin.js) we’re replacing the TodoListItem component with new one, we’re adding
new TodoListBatchOperations component and wrapping the TodoList component with a new one
that composes the original TodoList component with  TodoListBatchOperations.
Again, please take a moment here and [browse these files](https://github.com/char0n/swagger-adjust/tree/main/demo/src/todo-list-enhancer) yourself to see the actual code. 
Additionally, we’re exposing time formatting function to the plugin so that other plugins
could easily override how time is formatted.

<figure class="figure d-block">
  <img src="{{ '/assets/img/blog/swagger-adjust-todo-list-enhancer-plugin-dir-structure.png' | relative_url }}" width="252" height="202" class="figure-img rounded d-block img-fluid" alt="todoListEnhancer plugin directory structure" />
</figure>

{% highlight javascript linenos %}
import { assocPath } from 'ramda';

import { completeItem, uncompleteItem, deleteItem, completeAll, deleteAll } from './actions';
import { formatTimestamp } from './fn';
import reducers from './reducers';
import TodoList from './components/TodoList';
import TodoListItem from './components/TodoListItem';
import TodoListBatchOperations from './components/TodoListBatchOperations';

const TodoListEnhancerPlugin = (system) => {
  const todoListFn = assocPath(['formatTimestamp'], formatTimestamp, system.fn.todoList);

  return {
    components: {
      TodoListItem,
      TodoListBatchOperations,
    },
    wrapComponents: {
      TodoList,
    },
    fn: {
      todoList: todoListFn,
    },
    statePlugins: {
      todoList: {
        actions: {
          completeItem,
          uncompleteItem,
          deleteItem,
          completeAll,
          deleteAll,
        },
        wrapActions: {
          addItem: (oriAction) => (payload) => {
            oriAction({ ...payload, completed: false, createdAt: Date.now() });
          },
        },
        reducers,
      },
    },
  };
};
{% endhighlight %}

export default TodoListEnhancerPlugin;

The last thing we need to do, is to register the todoListEnhancer plugin with the DI-Container.

{% highlight javascript linenos %}
const plugins = [TodoListPlugin, TodoListEnhancerPlugin];
const system = new System({ plugins });
const store = system.getStore();
{% endhighlight %}

When building React+Redux features in this architecture, one always has to think hard how to expose
React components, Redux code and additional business or presentational logic to the plugin so that
other plugins can easily enhance it. This works also in reverse – when building features that need 
to be "protected" from enhancements, rather use classical static ES6 imports which prohibits anybody
from enhancing your plugins.

### Advanced usage patterns

During the course of writing Swagger Adjust and using it on different projects I’ve identified
multiple advanced usage patterns that need to be mentioned.  These advanced usage patterns include:

- [various messaging patterns (Routing and Transformation) when wrapping actions](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#routing-patterns)
- [providing initial state for a state plugin](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#initial-state)
- [override reducers from another state plugin](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#reducers)
- [composing selectors in single state plugin](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#composing-selectors-from-single-plugin)
- [composing selectors from different plugins](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#composing-selectors-from-different-plugins)
- [registering and using hooks](https://github.com/char0n/swagger-adjust/blob/main/docs/customization/plugin-api.md#hooks)

It’s important for Swagger Adjust consumers to understand these usage patterns properly as they set conventions on how Swagger Adjust was designed to be used.

### Word on conventions

Conventions are very important when producing any code. When utilized properly, it makes code 
that was produced by different people look like it was produced by a single person.
Redux Toolkit has done a great job setting up conventions and standards for React+Redux applications.
Swagger Adjust adopts all these conventions and adds a couple of its own:

- use [createAction](https://github.com/char0n/swagger-adjust/blob/main/docs/usage/api.md#createaction) helper to create action creators
- use following convention for action type: “namespace/action”
- use [createAsyncThunk](https://github.com/char0n/swagger-adjust/blob/main/docs/usage/api.md#createasyncthunk) to create async actions or handle side effects
- structure Files as Feature Folders as demonstrated in this article
- name your selectors in following naming scheme: select<Name>
- create memoized selectors using [createSelector](https://github.com/char0n/swagger-adjust/blob/main/docs/usage/api.md#createselector) utility

Consult [Redux Style Guide](https://redux.js.org/style-guide/style-guide) for additional conventions.

### Closing words

Swagger Adjust it’s an ideal framework for creating User Interfaces (UI) with external extensibility
requirement. It should work pretty well for creating renderers for different specifications like OpenApi,
AsyncApi or ApiBlueprint (as SwaggerUI demonstrated). It allows external extensibility via configuration 
and plugins, and it does bring much more features than demonstrated in this article. 
I welcome you to consult our documentation to learn more.

You can find all code demonstrated in this article in [Swagger Adjust GitHub repository](https://github.com/char0n/swagger-adjust).