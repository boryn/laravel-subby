# Plan Subscription Model

## Create a Subscription<a name="create-subscription"></a>

You can subscribe a user (or any model correctly traited) to a plan by using the `newSubscription()` function available
in the `HasSubscriptions` trait. First, retrieve an instance of your subscriber model, which typically will be your user
model and an instance of the plan your user is subscribing to. Once you have retrieved the model instance, you may use
the `newSubscription` method to create the model's subscription.

```php
$user = User::find(1);
$plan = Plan::find(1);

$user->newSubscription('main', $plan, 'Main subscription', 'Customer main subscription');
```

The first argument passed to `newSubscription` method should be the identifier tag of the subscription. If your
application offer a single subscription, you might call this `main` or `primary`. The second argument is the plan
instance your user is subscribing to, and the third argument is a human-readable name for your subscription. Fourth
argument is a description. Fifth argument is a start date.

## Change its Plan<a name="change-plan"></a>

You can change subscription plan easily as follows:

```php
$plan = Plan::find(2);
$subscription = PlanSubscription::find(1);

// Change subscription plan
$subscription->changePlan($plan);

// Change subscription plan and keep usage
$subscription->changePlan($plan, false);

// Change subscription plan, keep usage and invoicing data
$subscription->changePlan($plan, false, false);
```

Subscription usage data will be cleared by default, unless `false` is given as second parameter.

If you want the same billing frequency (`invoice_period` and `invoice_interval`) set third parameter to `true`
and subscription will inherit plan's billing frequency. If you want to keep current subscription invoice intervals, set
to `false`.

- Plan change will adjust existing features to the ones in the new plan.
- Change will also remove features attached to old plan.
- Existent features that where previously attached without plan but exist in the new plan now will use plan values.
- Features not attached to a plan and inexistent in new plan will remain the same.

## Subscriber's subscriptions

Retrieve subscriptions of subscriber.

```php
// Get user subscriptions
$user->subscriptions;

// Get user active subscriptions
$user->activeSubscriptions;
```

## Subscription Feature Usage<a name="subscription-feature-usage"></a>

There are multiple ways to determine the usage and ability of a particular feature in the subscriber's subscription, the
most common one is `canUseFeature`:

The `canUseFeature` method returns `true` or `false` depending on multiple factors:

- Subscription has not ended.
- Feature _is enabled_.
- Feature value isn't `0`/`false`/`NULL`.
- Or feature has remaining uses available.

```php
$user->subscription('main')->canUseFeature('social_profiles');
```

Other feature methods on the user subscription instance are:

- `getFeatureUsage`: returns how many times the user has used a particular feature.
- `getFeatureRemainings`: returns available uses for a particular feature.
- `getFeatureValue`: returns the feature value.

> All methods share the same signature: e.g. `$user->subscription('main')->getFeatureUsage('social_profiles');`.

### Record Feature Usage<a name="record-feature-usage"></a>

In order to effectively use the ability methods you will need to keep track of every usage of each feature (or at least
those that require it). You may use the `recordFeatureUsage` method available through the user `subscription()` method:

```php
$user->subscription('main')->recordFeatureUsage('social_profiles');
```

The `recordFeatureUsage` method accepts 3 parameters: the first one is the feature's tag, the second one is the quantity
of uses to add (default is `1`), and the third one indicates if the addition should be incremental (default behavior),
when disabled the usage will be override by the quantity provided. E.g.:

```php
// Increment by 1
$user->subscription('main')->recordFeatureUsage('social_profiles', 1);

// Override with 3
$user->subscription('main')->recordFeatureUsage('social_profiles', 3, false);
```

### Reduce Feature Usage<a name="reduce-feature-usage"></a>

Reducing the feature usage is _almost_ the same as incrementing it. Here we only _substract_ a given quantity (default
is `1`) to the actual usage:

```php
$user->subscription('main')->reduceFeatureUsage('social_profiles', 2);
```

### Clear the Subscription Usage data<a name="clear-subscription-usage-data"></a>

```php
$user->subscription('main')->usage()->delete();
```

## Check Subscription status<a name="check-subscription-status"></a>

For a subscription to be considered active one of the following must be `true`:

- Subscription has an active trial.
- Subscription `ends_at` is in the future.

Alternatively you can use the following methods available in the subscription model:

```php
$user->subscription('main')->isActive();
$user->subscription('main')->isCanceled();
$user->subscription('main')->hasEnded();
$user->subscription('main')->isOnTrial();
```

### Other<a name="other"></a>

```php 
// Check if subscription is free
$user->subscription('main')->isFree();

// Check subscriber to plan
$user->isSubscribedTo($planId);
```

> Canceled subscriptions with an active trial or `ends_at` in the future are considered active.

## Renew a Subscription<a name="renew-subscription"></a>

To renew a subscription you may use the `renew` method available in the subscription model. This will set a
new `ends_at` date based on the selected plan and _will clear the usage data_ of the subscription.

```php
$user->subscription('main')->renew();
```

_Canceled subscriptions with an ended period can't be renewed._

## Cancel a Subscription<a name="cancel-subscription"></a>

To cancel a subscription, simply use the `cancel` method on the user's subscription:

```php
$user->subscription('main')->cancel();
```

By default the subscription will remain active until the end of the period, you may pass `true` to end the
subscription _immediately_:

```php
$user->subscription('main')->cancel(true);
```

## Scopes<a name="scopes"></a>

```php
// Get subscriptions by plan
$subscriptions = PlanSubscription::byPlanId($planId)->get();

// Get bookings of the given user
$user = \App\Models\User::find(1);
$bookingsOfUser = PlanSubscription::ofSubscriber($user)->get(); 

// Get subscriptions with trial ending in 3 days
$subscriptions = PlanSubscription::findEndingTrial(3)->get();

// Get subscriptions with ended trial
$subscriptions = PlanSubscription::findEndedTrial()->get();

// Get subscriptions with period ending in 3 days
$subscriptions = PlanSubscription::findEndingPeriod(3)->get();

// Get subscriptions with ended period
$subscriptions = PlanSubscription::findEndedPeriod()->get();
```