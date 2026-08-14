---
layout: post
title: "Handling async side effects in Redux"
description: "A survey of approaches for handling async side effects (AJAX requests) in Redux applications: redux-thunk and the 'Dirty Containers' anti-pattern it invites, a speculative middleware approach built on plain action objects, and why redux-observable is worth a look for new projects."
date: 2017-06-06 10:00:00 +0200
canonical_url: https://www.linkedin.com/pulse/handling-async-side-effects-redux-vladim%C3%ADr-gorej/
image:
  path: assets/img/blog/handling-async-side-effects-in-redux.webp
  width: 450
  height: 450
  caption: Handling async side effects in Redux
  object-fit: cover
---

<p class="lead">
  Properly handling async side effects in frontend applications is quite a complicated endeavor. I've tried many approaches, many libraries,
  visited quite a few conferences and meetups, and this article sums up what I've learned so far and what worked for me.
</p>

Let's reduce the problem to the most common async side effect denominator: AJAX requests. I will demonstrate various concepts on a frontend application using the
<a href="https://github.com/reactjs/redux" target="_blank" rel="noopener noreferrer">Redux</a> library (a functional implementation of the Flux pattern), so I will
assume the reader's intermediate understanding of Redux concepts. This article deals more with concepts and pseudo-code, rather than concrete runnable implementations.

## Redux Thunk

If you're building a small to mid-size project, I think you're good with <a href="https://github.com/gaearon/redux-thunk" target="_blank" rel="noopener noreferrer">redux-thunk</a>.
Redux Thunk <a href="https://github.com/reactjs/redux/blob/master/docs/advanced/Middleware.md" target="_blank" rel="noopener noreferrer">middleware</a> allows you to
write action creators that return a function instead of an action. The thunk can be used to delay the dispatch of an action, or to dispatch only if a certain condition is met.

This is how you usually define your async action creators in the redux-thunk ecosystem.

{% highlight javascript linenos %}
import { createAction } from 'redux-actions';

const fetchUserRequestStarted = createAction('FETCH_USER_REQUEST_STARTED');
const fetchUserSuccess = createAction('FETCH_USER_SUCCESS');
const fetchUserFailure = createAction('FETCH_USER_FAILURE');
const fetchUser = userId => dispatch => {
  dispatch(fetchUserRequestStarted(userId));
  return axios.get(`/users/${userId}`)
    .then(response => dispatch(fetchUserSuccess(response.data))
    .catch(error => Promise.reject(dispatch(fetchUserFailure(error)));
}

const updateUserRequestStarted = createAction('UPDATE_USER_REQUEST_STARTED');
const updateUserSuccess = createAction('UPDATE_USER_SUCCESS');
const updateUserFailure = createAction('UPDATE_USER_FAILURE');
const updateUser = user => dispatch => {
  dispatch(updateUserRequestStarted(user));
  return axios.put(`/users/${user.id}`)
    .then(response => dispatch(updateUserSuccess(response.data))
    .catch(error => Promise.reject(dispatch(updateUserFailure(error)));
}
{% endhighlight %}

We define three sync action creators: `fetchUserRequestStarted`, `fetchUserSuccess`, and `fetchUserFailure`. Then we define the thunk that dispatches these sync
action creators at various stages of an asynchronous AJAX request. Notice that we return a promise from our thunk. It allows us to build control flows with promises.

{% highlight javascript linenos %}
const fetchUserAndProfile = userId => dispatch => dispatch(fetchUser(userId))
  .then(user => dispatch(fetchUserProfile(user.profileId));
{% endhighlight %}

`fetchUserProfile` is just an ordinary thunk that fetches our user profile, just like `fetchUser` fetches our user. Returning a promise from a thunk is a very common
pattern in redux-thunk, but IMHO it is not a very fortunate one. It allows you to handle some of your side effects directly in your container.

{% highlight javascript linenos %}
import UserCard from './components/UserCard';
import { connect } from 'react-redux';

const mapStateToProps = null;

const mapDispatchToProps = dispatch => ({
  onComponentMount: userId => dispatch(fetchUser(userId)),
  onUpdate: user => dispatch(updateUser(user))
    .then(() => dispatch(toastr.show('User updated on user card'))
    .catch(error => sentry.log(error));
)};

export default connect(null, mapDispatchToProps)(UserCard);
{% endhighlight %}

This is what I call <strong>Dirty Containers</strong>. If you're using redux-thunk, I'll bet my shoes that you did something like that in your React application ;]
Please refrain from doing this. This is not a proper way to handle our side effects. There are better alternatives. Keep the side effects from your React
components and containers as far away as you can. They'll thank you later. With this approach, you lose the possibility to cancel pending AJAX requests if
you're using native promises.

So what are the alternatives?

A couple of months ago I had a very interesting information exchange with
<a href="https://github.com/ericelliott/speculation/issues/9#issuecomment-275062382" target="_blank" rel="noopener noreferrer">Eric Elliott</a>. He implemented a
library called <a href="https://github.com/ericelliott/speculation" target="_blank" rel="noopener noreferrer">Speculation</a> that solves the problem of cancellation
of native Promises. It builds on a theory that all action creators should return dispatched actions only, and not promises, and that only code that creates
promises should know how to cancel them.

Eric Elliott had this to say about it:

<blockquote class="blockquote">
  <p>
    All my action creators return action objects, not promises. Those action objects get dispatched to the store, where it's possible that some async
    middleware might be listening.
  </p>
  <p>
    That middleware handler is responsible for both the creation and potential cancellation of promises. Speculations would work well in that context, and
    you'll maintain the pattern of using action objects to signal user intentions and application state changes, including the maintenance of async event state.
  </p>
  <footer class="blockquote-footer"><cite title="Eric Elliott">Eric Elliott</cite></footer>
</blockquote>

I will not go into detail about how I would implement such a middleware. It is out of the scope of this article. But let's just assume we already have one.
Let's call it Speculation Middleware. How would our action creators and container look then? Let's see...

`actions.js`

{% highlight javascript linenos %}
import { createAction } from 'redux-actions';

const fetchUserRequestStarted = createAction('FETCH_USER_REQUEST_STARTED');
const fetchUserSuccess = createAction('FETCH_USER_SUCCESS');
const fetchUserFailure = createAction('FETCH_USER_FAILURE');
const fetchUserCancel = createAction('FETCH_USER_CANCEL');
const fetchUser = createAction('FETCH_USER');

const updateUserRequestStarted = createAction('UPDATE_USER_REQUEST_STARTED');
const updateUserSuccess = createAction('UPDATE_USER_SUCCESS');
const updateUserFailure = createAction('UPDATE_USER_FAILURE');
const updateUserCancel = createAction('UPDATE_USER_CANCEL');
const updateUser = createAction('UPDATE_USER');
{% endhighlight %}

`UserCard.js`

{% highlight javascript linenos %}
import UserCard from './components/UserCard';
import { connect } from 'react-redux';

const mapStateToProps = null;

const mapDispatchToProps = dispatch => ({
  onComponentMount: userId => dispatch(fetchUser(userId)),
  onUpdate: user => dispatch(updateUser(user))
)};

export default connect(null, mapDispatchToProps)(UserCard);
{% endhighlight %}

OK, now we have a side-effect-free container, business-logic-free action creators, and all async processing hidden in the Speculation middleware. But how do we
show the toastr saying `User updated on user card`? How do we bind context to a generic action creator like `updateUser`? Wrap your generic action creator into
a more specific action creator and use <a href="https://github.com/acdlite/flux-standard-action#meta" target="_blank" rel="noopener noreferrer">meta</a> on the
Flux Standard Action to decorate the action with context. Our Speculation middleware will delegate this `meta` property further when dispatching the action
creators `updateUserRequestStarted`, `updateUserSuccess`, `updateUserFailure`, and `updateUserCancel` respectively. Then you can have a middleware that listens
for the generic action type and specific meta and reacts accordingly.

`specificActions.js`

{% highlight javascript linenos %}
import { identity, always } from 'ramda';

import { updateUser } from './actions.js';

const updateUserOnUserCard = createAction(
  updateUser.toString(),
  identity,
  always('userCard')
);
{% endhighlight %}

`UserCard.js`

{% highlight javascript linenos %}
import UserCard from './components/UserCard';
import { connect } from 'react-redux';

const mapStateToProps = null;

const mapDispatchToProps = dispatch => ({
  onComponentMount: userId => dispatch(fetchUser(userId)),
  onUpdate: user => dispatch(updateUserOnUserCard(user))
)};

export default connect(null, mapDispatchToProps)(UserCard);
{% endhighlight %}

So that's about it. If you ended up using Dirty Containers, this article may help you correct your errors. You can even get rid of Dirty Containers without
implementing the Speculation middleware (if you don't necessarily need promise cancellation) by defining more specific actions and moving the side effects
from Dirty Containers to middleware.

If you're building a new application, I'd strongly recommend you look at <a href="https://redux-observable.js.org/" target="_blank" rel="noopener noreferrer">redux-observable</a>.

---

<div class="alert alert-info" role="alert">
  All code snippets in this article are licensed under <a href="https://www.apache.org/licenses/LICENSE-2.0" target="_blank" rel="noopener noreferrer">Apache 2.0 license</a> using the following copyright notice: <strong>Copyright 2017 Vladimír Gorej</strong>.
</div>
