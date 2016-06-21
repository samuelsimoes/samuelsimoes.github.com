---
layout: post
title:  "Facebook Flux and MVC"
date:   2016-02-27 12:00
categories: javascript
---

You probably are seeing the exponential growth of [Facebook Flux](https://facebook.github.io/flux/) pattern and its derivations. The main idea of Flux pattern is the one way data flow on the app and the clearly separated concerns layers.
h
Flux pattern defines three important layers on your app. The presentation layer that presents the app state, the store layer that holds the state itself and makes the states mutations and the action creator layer that dispatches commands that are intercepted by stores through the fourth Flux's piece, the dispatcher. The diagram above show the flow:

<div class="image-container">
  <a href="https://s3-us-west-2.amazonaws.com/samuel-blog/flux-explain.png" target="_blank">
    <img src="https://s3-us-west-2.amazonaws.com/samuel-blog/flux-explain.png" class="image-with-shadow half-image" />
  </a>
</div>

The data flow must happen only on one way, each cycle can roughly be described like:

1. The view layer invokes some action.
2. The action dispatches a message through the dispatcher.
3. Interested stores catch the message and make the necessary states mutation.
4. The affected stores notify the view layer to present their new state.

## The good parts of Flux

The one way data flow is one of the best insights of Flux, if you think it's what happens on your server request lifecycle and this works pretty well, the user on printed HTML in the browser make some action (click link, submit form and so on) that triggers a request, the server processes it and returns a new HTML with the "new state" and the cycle goes on.

Until recently nobody had brought it to the client side. The one way flow makes easy track how things work and happens on your app.

We can't forget about the concern separation that is also another good part.

## Simplifying with... MVC...

The pointless purism on keep actions communicating with stores only through the dispatcher, besides the fact to introduces more concepts to explain to newcomers, brings other problems, mainly when the stores need interact with each other (search about `waitFor` to see this).

Now, open your mind...

I like Flux, but I think it can be a little more simple and still reliable, to this we'll break some Flux rules and it will look like the MVC that you are familiar on your backend framework instead of "front-end MVC" out there.

If you are already yelling "you got everything wrong, Flux isn't MVC", "it's huge backstep", "MVC is totally broken" or something like this you can stop the reading here.

If you still here I already have put this "pattern" on production apps that I collaborate and other open source projects (like [Chrome Basecamp Notifier](https://github.com/samuelsimoes/chrome-basecamp-notifier)) and it performs really well and of course we have a [ToDo MVC using this](https://github.com/samuelsimoes/todomvc-fluxo).

### Actions

The first difference going to be the removal of the dispatcher. Our action creator (I will call only "actions" from now on) will communicate with stores directly.

To this task I like to create a class for a group of actions based on the app entities, like `PostActions`, `CommentsActions` and so on. Theses classes receive on the constructor the stores that it will manipulate, it's good because you see what "pieces of state" some "group of actions" have access.

Some example of this would be like this:

{% highlight javascript %}
class CommentActions {
  constructor (comments) {
    this.comments = comments;
  }

  update (commentID, data) {
    this.comments.update(commentID, data);
    // maybe some sync logic here if want
  }

  remove (commentID) {
    this.comments.remove(commentID);
    // maybe some sync logic here if want
  }
}
{% endhighlight %}

Or a complete example like the [todos actions on the ToDo MVC example](https://github.com/samuelsimoes/todomvc-fluxo/blob/master/src/actions/todos_actions.js).

But it goes very against the Flux recommendation:

>Nothing outside the store has any insight into how it manages the data for its domain, helping to keep a clear separation of concerns. Stores have no direct setter methods like setAsRead(), but instead have only a single way of getting new data into their self-contained world — the callback they register with the dispatcher.

But thus far I don't have any issues with this approach, since we are sending messages to our stores that not exposes how the state mutation occurs, despite Facebook saying the opposite.

This dispatchless and more straight approach brings a good benefit to understand on each app's actions how the stores are manipulated instead of chasing what stores catch the messages and figuring out crazy `waitFor` chains, we even don't need [constants with the actions names](https://github.com/facebook/flux/blob/1beef2f8a216d85cc1c25ac39182ab01aee022d3/examples/flux-chat/js/constants/ChatConstants.js#L17-L22) anymore.

It resembles the controller that you have on your backend MVC framework, on this layer you deal with messages to state mutations, state persistence (with the server or other persistence layer) and store hydration.

A very important care is **to keep your actions without complex state manipulation**, your actions must send "messages" and here I'm talking only about invoking methods on your stores, **all state mutations and side computations must happen on the store layer**.

On the example below we are calculating the cart's total on the hypothetical remove product action, if other action elsewhere also removes a product this computation won't happen leading to wrong state.

{% highlight javascript %}
class CartActions {
  constructor (cart) {
    this.cart = cart;
  }

  removeProduct (productID) {
    this.cart.removeProduct(productID);

    // wrong do it here, your cart store must compute this
    let total = this.cart.products.reduce(memo, product => {
      return ((product.price * product.quantity) + memo);
    }, 0);

    this.cart.set({ total: total });
  }
}
{% endhighlight %}

### Stores

The stores hold the app state and the logic to change it. The store exposes a public API with methods that make state mutations, **these methods are invoked only on action layer** because it makes predictable where we should look the state mutations commands. Yes, it resembles the model on MVC, whatever.

For this part of your app you can use literal objects and some pub/sub to notifies the view about the changes. I like to use some more robust state manager with some events capabilities. For this we have created the [Fluxo](https://github.com/fluxo-js/fluxo), a tiny lib that mimics the model and collection of Backbone.js with other useful additions to our one way workflow, I will write more about using Fluxo with this, but not on this yet.

The stores, like on Facebook's Flux, notifies the view layer about the state mutations and the view layer rerenders the new state.

You can check a complete example about the stores on the [Fluxo's ToDo MVC example store](https://github.com/samuelsimoes/todomvc-fluxo/blob/master/src/stores/todos_store.js).

### View layer

The view layer we keep like Facebook's Flux suggests, it presents the current store's state and if the user interacts with it the view invokes an action on the "actions layer".

To this task you can use anything that you want. We have using React.js that we found very straightforward and performative for this task and to connect [Fluxo](https://github.com/fluxo-js/fluxo) with React we use the [Fluxo stores connector](https://github.com/fluxo-js/fluxo-react-connect-stores).

## Finishing

Our "simplified flux" will look like this:

<div class="image-container">
  <a href="https://s3-us-west-2.amazonaws.com/samuel-blog/simplified-flux.png" target="_blank">
    <img src="https://s3-us-west-2.amazonaws.com/samuel-blog/simplified-flux.png" class="image-with-shadow half-image" />
  </a>
</div>

With this post I ain't trying to say that MVC is the best one and the Flux is worse, but it's working pretty well here with a good balance between complexity and reliability, I can check all actions on my app on a very clear layer with the visualization how the state orchestration is being done, not to mention the store's state mutation dependencies are super easy to setup and understand. The view layer is completely decoupled from this other layers and if React doesn't work anymore for our purposes we can change the view layer for any other view lib without big hassles.

If you liked, have some question or still thinking that I'm crazy to break the Flux rules write a comment below. I'm writing a practical and simple tutorial of this with React and Fluxo and I'll publish soon (but you already can read the [Fluxo's  Getting Started tutorial](https://github.com/fluxo-js/fluxo/wiki/Getting-Started)).

Thanks for your attention.
