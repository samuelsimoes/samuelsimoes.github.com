---
layout: post
title:  "[Part 1] Rails with PayPal subscriptions guide"
date:   2014-11-01 16:00
categories: rails
---

Some people constantly email me with questions about how build a structure for handle PayPal subscriptions on Rails apps, in this post series I'll try to help you create that. I expect that you have a basic Rails knowledge because I'll focus more on the app code design instead Rails basics.

You already can see the full app in this **[Github repository](https://github.com/samuelsimoes/rails-paypal-subscriptions-sample)**. Enjoy!

## Persistence structure

We'll use the latest Rails version until now, the 4.1.X, we'll also use the greatest @fnando's rubygem **[paypal-recurring](https://github.com/samuelsimoes/paypal-recurring)**, this gem deals with all the PayPal API requests, which makes our life more easy. The gem's link points to my fork because the original version doesn't send the `reference` field to the PayPal endpoints, which is necessary for our implementation.

In the persistence layer we'll use this very simple database structure.

<div class="image-container">
  <img src="https://s3-us-west-2.amazonaws.com/samuel-blog/paypal-schema.png" alt="Our app MER schema" class="image-with-shadow half-image">
</div>

Notice the PayPal fields in subscription table (`paypal_payer_id` and `paypal_profile_id`), I'll get to that later, still in this table we have the `paid_until` and `canceled` columns that I believe needs no explanation.

The plans table have the required `price` column, which will precify the subscription, and a PayPal description, which is a column for a small and especific description for PayPal checkout page, this must be less or equal to 100 characters and can't contains any special character, so, in our `Plan` model will can override the paypal_description setter to always treat these description rules, like this:

{% highlight ruby %}
def paypal_description=(value)
  write_attribute(:paypal_description, I18n.transliterate(value))
end
{% endhighlight %}

The rest of schema basically deals with relations, subscription belongs to some plan.

## A quick overview on PayPal subscription flow

The process in our app starts in the plans page, the user choose a plan and click in the subscribe button, clicking in this button we create the subscription record for this user and redirect him to the PayPal checkout page, returning from successuful subscription profile creation authorization we start the subscription charging, after that, in a especific endpoint of our app, the PayPal will sends to us notifications about the subscription situation and with these infos we'll update the `paid_until` column, which will indicates in our app whether the subscription is active or not.

In other part of our app the user will have a page with their subscriptions infos, including status (canceled/running/payment pending), and a button for cancels it, clicking in the cancel button our app will submit a subscription profile canceling request to the PayPal API, after some time, PayPal will send to our app a notification about the cancellation situation and we'll toggle the `false` value in the `canceled` subscription column.

## Routine objects

To maintain the organization of our flow we'll split the whole subscription routine in four small and specialized objects. Below I'll show these objects details involved in the process. In the next post I'll show how to wire up everything.

###PaypalSubscription::DefaultOptions
For every PayPal request we need send options to identify for what subscription we are pretending make the action, for centralize the default options we'll create a very simple object, with one class method only, this object will return the options based on the subscription record on the first function argument.

Note that our subscriptable entity needs respond to `id`, `paypal_payer_id`, `paypal_profile_id`, `paypal_description`, `price`. The first four properties are columns of subscriptions table, the latter two are **delegates to the associated plan entity**, remember that.

{% highlight ruby %}
class PaypalSubscription::DefaultOptions
  def self.for(subscriptable)
    {
      period: :monthly,
      outstanding: :no_auto,
      frequency: 1,
      start_at: Time.current,
      trial_length: 0,
      payer_id: subscriptable.paypal_payer_id,
      profile_id: subscriptable.paypal_profile_id,
      reference: subscriptable.id,
      description: subscriptable.paypal_description,
      amount: subscriptable.price,
      currency: 'BRL',
      locale: 'pt_BR'
    }
  end
end
{% endhighlight %}

**Important!**
Notice that we defined no trial period (`trial_length`), and the charging period starting right now (`start_at`), this way the subscription profile will makes the first charge.

Some tutorials and even the gem documentation recommends you to make one first express checkout and then create the recurring profile, but **I highly don't recomend this**, because your express checkout payment can works fine, but your subscription profile creation can fails, in this situation you have a failed subscription profile but one charge in user account, which is very bad! Leaving the whole charging process to the subscription profile, if the subscription creation fails, the user won't be charged.

### PaypalSubscription::ResourceFacade
This object will hide the specifics PayPal gem API (the gem interface, not the PayPal REST API) interactions on our app. With this object we'll no longer need instanciate the gem object and each public class method express an action and returns the interested value of action, for exemple, the `checkout_url` action method will returns the checkout url, with this we don't need access the checkout_url property of PayPal gem object in other parts of our app and if the property name change only in this facade we will need change.

{% highlight ruby %}
class PaypalSubscription::ResourceFacade
  def self.checkout_url(options)
    process_action(action_name: :checkout, options: options).
      checkout_url
  end

  def self.make_recurring(options)
    process_action(action_name: :create_recurring_profile, options: options).
      profile_id
  end

  def self.cancel(options)
    process_action(action_name: :cancel, options: options)
  end

  protected

  def self.process_action(action_name:, options: {})
    ppr = PayPal::Recurring.new(options)

    response = ppr.send(action_name)

    raise response.errors if response.errors.present?

    response
  end
end
{% endhighlight %}

### PaypalSubscription::NotificationHandler

This object will take an action (cancelation or updates how long it's paid) depends on notification received from PayPal on our PayPal notifications endpoint, note that this object will receive a subscription record and a notification object, this notification is an instance of `PayPal::Recurring::Notification`.

{% highlight ruby %}
class PaypalSubscription::NotificationHandler
  def self.resolve!(*args)
    new(*args).resolve!
  end

  def initialize(subscription:, notification:)
    @subscription, @notification = subscription, notification
  end

  def resolve!
    if canceling_profile?
      @subscription.cancel!
    elsif update_profile?
      @subscription.update(paid_until: @notification.next_payment_date)
    end
  end

  private

  def update_profile?
    successful_recurring_payment? ||
      successful_active_recurring_payment_profile?
  end

  def canceling_profile?
    @notification.type == 'recurring_payment_profile_cancel'
  end

  def successful_active_recurring_payment_profile?
    @notification.recurring_payment_profile? && @notification.profile_status == 'Active'
  end

  def successful_recurring_payment?
    @notification.recurring_payment? && @notification.completed?
  end
end
{% endhighlight %}

###PaypalSubscription::RecurrenceCreator
This object will be reponsible to get the PayPal recurring payment profile id (which will delegate to our `PaypalSubscription::ResourceFacade` the profile creation on PayPal) and the payer id to save these informations on our subscription record.

{% highlight ruby %}
class PaypalSubscription::RecurrenceCreator
  def self.create!(*args)
    new(*args).create!
  end

  def initialize(subscription:, paypal_options:)
    raise MissingSubscription if subscription.blank?

    @subscription, @paypal_options = subscription, paypal_options
  end

  def create!
    @subscription.update(
      paypal_payer_id: @paypal_options[:payer_id],
      paypal_profile_id: profile_id
    )
  end

  private

  def profile_id
    PaypalSubscription::ResourceFacade.make_recurring(@paypal_options)
  end
end
{% endhighlight %}

##In the next episodes

In this first part we got how structure the app backbone, in the next post I'll show how clue everything in the controller layer. If you have any question ping me on [Twitter](http://twitter.com/samuelsimoes) or leave your comment below.

See you soon.

--- Edit ---

**[Part 2 - Rails with PayPal Subscription Guide](/rails/2015/03/19/part-2-rails-with-paypal-subscription-guide.html)**
