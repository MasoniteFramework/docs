# Masonite Billing

# Requires

* Masonite 2.0.0+

# Installation

Installing Masonite Billing is simple. We just need a new configuration file, 2 new migrations and extend our User model.

Pip Install

```
$ pip install masonite-billing
```

# Add the Service Provider

Simply add the Masonite Billing Service Provider to your providers list:

```python
from billing.providers import BillingProvider
​
PROVIDERS = [
    ...
​
    # Third Party Providers
    BillingProvider,
    
    # Application Providers
    ...
]
```

This will add a new install:billing command to craft. Just run:

```
$ craft install:billing
```

This will create a new configuration file in config/billing.py

# Configuration File

All billing information will be located in the config/billing.py file and has 2 important constants:

```python
...
​
DRIVER = 'stripe'
​
...
​
DRIVERS = {
    'stripe': {
        'client': os.getenv('STRIPE_CLIENT'),
        'secret': os.getenv('STRIPE_SECRET'),
        'currency': 'usd',
    }
}
```

The DRIVER is the processor that Masonite billing will use and the DRIVERS constant is the configuration setting for the driver.

Although there is the DRIVER constant, Masonite Billing currently only supports Stripe. Other drivers like Braintree will be supported in later releases which should be backwards compatible.

# Migrations

We'll need to create 2 new migrations: one to add columns to the users table and one migration to create a new subscriptions table. Just create these migration files with craft and copy and paste the migration into those files and migrate them.

Let's first add 2 new columns to our users table.

```
$ craft migration add_subscription_info_to_users --table users
```

Now just add this column to the migration file:

```python
with self.schema.table('users') as table:
    table.string('customer_id').nullable()
    table.string('plan_id').nullable()
```

Now let's add a new subscriptions table.

```
$ craft migration create_subscriptions_table --create subscriptions
```

```python
with self.schema.create('subscriptions') as table:
    table.increments('id')
    table.integer('user_id').unsigned()
    table.string('plan')
    table.string('plan_id')
    table.string('plan_name')
    table.timestamp('trial_ends_at').nullable()
    table.timestamp('ends_at').nullable()
    table.timestamps()
```

Now just migrate the new migrations:

```
$ craft migrate
```

# Stripe Authentication Keys

Just add your Stripe public and secret keys to your .env file:

```
STRIPE_CLIENT=pk_Njsd993hdc...
STRIPE_SECRET=sk_sjs8yd78H8...
```

# Billable Model

Masonite Billing consists of a model that should be inherited by whatever model you want to add subscription billing information to. In this example here, we will focus on adding the billing integration to our User model.

```python
from config.database import Model
from billing.models.Billable import Billable
​
class User(Model, Billable):
    __fillable__ = ['name', 'email', 'password']
    __auth__ = 'email'
```

Once that is added we will now have a plethora of methods we can use to subscribe and charge our user.

Read more about how to handle subscription and payment information in the Usage documentation.

​# Getting Started

Below you will notice we are using a tok_amex token, you may use this token for testing purposes but this token in production should be the token returned when processing your stripe form.

It's also important to note that the subscription records in your database are never deleted but are updated instead. So canceling a subscription does not delete that subscription from your database but only sets when the subscription ended. This is good if you want to dump the data into an email campaign tool to try and get back any lost customers.

## Subscribing to Plans

To subscribe a user to plans, we can use the subscribe method like so:

```python
user.subscribe('masonite-test', 'tok_amex')
```

This method retuns a string of the subscription token such as sub_j8sy7dbdns8d7sd..

If you try to subscribe a user to a plan and the plan does not exist in Stripe then Masonite will throw a `billing.exceptions.PlanNotFound` exception.

## Checking Subscriptions

If you want to check if the user is subscribed you have a few options:

You may check if the user is subscribed to any plan:

```python
user.is_subscribed()
```

or you can check if the user is subscribed to a specific plan:

```python
user.is_subscribed('masonite-test')
```

You may also check if a user was subscribed but their plan has expired:

```python
user.was_subscribed()
```

or you can obviously check if the user was subscribed to a specific plan:

```python
user.was_subscribed('masonite-test')
```

## Getting The Plan Name

If you need to get the name of the plan the user is on you can do so using the plan() method:

```python
user.plan() # returns Masonite Test
```

This will return the name of the plan in Stripe, not the plan ID. For example, our plan ID might be masonite-test but the plan name could be "Awesome Plan."

## Trialing

If the plan you are subscribing a user to has a trial set inside Stripe then the user will be automatically enrolled in a trial for that time. We can check if the user is on a trial using the on_trial() method. In our examples here, the masonite-test plan has a 30 day free trial set inside Stripe.

For example:

```python
user.subscribe('masonite-test') # plan has a 30 day trial
user.on_trial() # returns True
```

We may also want to skip the trial and charge the user immediately for the plan:

```python
user.skip_trial().subscribe('masonite-test') # plan has a 30 day trial
user.on_trial() # returns False
```

We can also specify the amount of days the user should be on a trial for when subscribing them to a plan:

```python
user.trial(days=4).subscribe('masonite-test') # plan now has a 4 day trial
user.on_trial() # returns True
```

## Swapping Plans

We can swap subscription plans at anytime using the swap() method:

```python
user.subscribe('masonite-test') # plan has a 30 day trial
user.is_subscribed('masonite-test') # returns True
user.on_trial() # returns True
​
user.swap('new-plan') # returns True
​
user.is_subscribed('masonite-test') # returns False
user.is_subscribed('new-plan') # returns True
```

## Canceling

If you want to cancel a user's subscription we can use the cancel() method. 

This will cancel the users plan but continue the subscription until the period ends.

```python
user.subscribe('masonite-test')
user.is_subscribed() # returns True
user.cancel() # returns True
user.is_subscribed() # returns True
```

Notice here that the last is_subscribed() method still returned True even after we canceled. The user will continue the subscription until the period ends. For example if it is a 1 month subscription and the user canceled 1 week in, Masonite Billing will continue the subscription for the remaining 3 weeks.

If you wish to cancel the user immediately we can specify the now=Trueparameter:

```python
user.subscribe('masonite-test')
user.is_subscribed() # returns True
user.cancel(now=True) # returns True
user.is_subscribed() # now returns False
```

The period between canceling a subscriptions and the period running out can be caught using the user.is_canceled() method:

```python
user.subscribe('masonite-test')
user.cancel() # returns True
user.is_canceled() # returns True
```

Again this will only return true if the user has an active subscription but has chosen to cancel it. If the subscription is canceled immediately, this will return False.

```python
user.subscribe('masonite-test')
user.cancel(now=True) # returns True
user.is_canceled() # returns False
```

## Resuming Plans
If you don't cancel a plan immediately, there will be a period between when the user canceled and when the plan can be resumed. We can use the resume() method here:

```python
user.subscribe('masonite-test')
​
user.cancel() # returns True
user.is_canceled() # returns True
​
user.resume() # returns True
user.is_canceled() # returns False
```

## Updating Card Information

Sometimes a user will want to switch or update their card information. We can use the card() method and pass a token to it:

```python
user.card('tok_amex') # updates the users card
```

## Charging users

If you want to make one off transactions for customers you can do so easily using the charge() method:

You can charge a card by passing a token:

```python
user.charge(999, token='tok_amex') # charges the users card
```

You can also charge a customer directly by leaving out the token which will charge whatever card the customer has on file.

```python
user.charge(999) # charges the customer
```

You can also add a description and metadata (as a dictionary) for the charge:

```python
user.charge(999, description='Charges for Flowers', metadata={'flower_type': 'tulips, roses'})
```

# Webhooks

Webhooks allow your application to interact directly with stripe when certain actions happen such as when a user's subscription has expired. You can use these webhooks to catch any events emitted.

## Getting Started

Masonite Billing also allows you to tie into Stripe webhooks. If the subscription is cancelled in Stripe, Stripe will send your server a webhook and Masonite Billing will update the database accordingly.

Be sure to setup the correct url in the the Stripe dashboard inside your Webhook Settings. The url to setup should be `http://your-domain.com/stripe/webhook` or if you're testing with Ngrok it should be something like `http://684b1285.ngrok.io/stripe/webhook`.

If you go the Ngrok route be sure to read the Testing With Ngrok section

## Webhook Controller

We can simply put the webhook controller in our routes/web.py file just like any other controller but it can point to the packages controller:

```python
...
Post().route('/stripe/webhook', '/billing.controllers.WebhookController@handle')
...
```

This will send all Stripe traffic to the server and handle any hooks accordingly.

## Testing With Ngrok

Most developers use Ngrok for testing incoming webhooks. Ngrok is a freemium HTTP tunneling service that allows extrernal requests to tunnel into your localhost and port environment. It's exellent for testing things like this as well as other things like OAuth.

If you use Ngrok you will get a subdomain like: `http://684b1285.ngrok.io` . Because this is a subdomain, Masonite needs to know which subdomain to support so you're route will have to be: 

```python
...
Post().domain('684b1285').route('/stripe/webhook', '/billing.controllers.WebhookController@handle')
...
```

in order to catch the Ngrok subdomain. This setup will allow you to send test webhooks from Stripe to your server.

You can read more about subdomains in the Subdomain Routing section of the Routing documentation.

## Csrf Protection

External incoming API calls typically will not be able to be CSRF protected because they will not know the specific token connected to a request. We will need to put an exception to the /stripe/webhook route:

```python
class CsrfMiddleware: 
  ...
 
  exempt = [
    '/stripe/webhook'
  ]
     
```

## Creating Custom Hooks

Currently the webhook controller only handles when a subscription is canceled but can handle any hook you like. If you need to create a custom hook you can inherit from the WebhookController and add the needed controller methods:

```python
from billing.controllers import WebhookController
​
class CustomWebhookController(WebhookController):
​
    def handle_resource_event(self):
        return 'Webhook Handled'
```

## Routes

You'll also have to specify a new location of your controller which should now be located in your normal controllers directory:

```python
...
Post().route('/stripe/webhook', 'CustomWebhookController@handle'),
...
```

## Events

Events are specific types of webhooks that Stripe will send out when certain actions occur.

You can see a list of stripe events here: `https://stripe.com/docs/api#event_types​`

You'll notice that we have a handle_resource_event method. The WebhookController will take all incoming webhook events into the handle method and dispatch them to other methods.

How it does this is simply takes the event lets say the charge.dispute.created Stripe event (see the link above for a list of more events) and parses it into a string that looks like:

```
handle_charge_dispute_created
```
You'll notice we just replaced the . with _ and prefixed a handle to it. This looks like a method call. If we simply create a method now:

```python
from billing.controllers import WebhookController
​
class CustomWebhookController(WebhookController):
​
    def handle_charge_dispute_created(self):
        return 'Webhook Handled'
```

This method will be called whenever we receive a webhook for that event. If no method is found for the incoming webhook then the server will return a Webhook Not Supported string which you may see while development testing your Stripe webhooks.