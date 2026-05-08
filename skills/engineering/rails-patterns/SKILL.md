---
name: rails-patterns
description: Object-oriented design patterns for Ruby on Rails. Use when deciding whether to extract a service object, form object, decorator, or PORO; when a model is getting fat; when business logic doesn't belong in a controller; or when applying classic OOP patterns (decorator, strategy, observer, command) to Rails code.
---

# Rails Patterns

Rails gives you models, controllers, and views. Reality gives you business logic that doesn't fit neatly into any of them. This skill covers when and how to extract that logic into well-named Ruby objects.

## The Core Question

Before extracting anything, ask: **where does this logic want to live?**

- Logic about a single ActiveRecord object → **model method or concern**
- Logic that orchestrates multiple models or external services → **service object**
- Logic for validating/coercing user input across multiple models → **form object**
- Logic for presenting a model differently in different contexts → **decorator**
- Logic that varies by type or context → **strategy**
- Logic that reacts to events in other objects → **observer / callback**

Extraction has a cost — more files, more indirection, more to hold in your head. Only extract when the alternative is clearly worse.

## When to Keep Logic in the Model

Rails models can absorb a lot before they become a problem. Prefer a model method over extraction when:

- The logic only concerns that model's data
- The logic is called from multiple places (extraction doesn't simplify callers)
- The operation is simple enough to name with one method

Fat model is not inherently bad. **Tangled model** is — when a model knows about unrelated domains, calls external services, or triggers side-effects that surprise callers.

## Service Objects

Extract when an operation:
- Crosses multiple model boundaries
- Calls external services (APIs, mailers, payment processors)
- Has complex branching that would clutter a controller or model
- Needs to be tested in isolation from HTTP

**Structure:**

```ruby
# app/services/checkout_order.rb
class CheckoutOrder
  def initialize(order, payment_params)
    @order = order
    @payment_params = payment_params
  end

  def call
    return failure("Order already checked out") if @order.checked_out?

    ActiveRecord::Base.transaction do
      charge = PaymentGateway.charge(@payment_params)
      @order.update!(status: :paid, charge_id: charge.id)
      OrderMailer.confirmation(@order).deliver_later
    end

    success
  rescue PaymentGateway::Error => e
    failure(e.message)
  end

  private

  def success = Result.new(ok: true)
  def failure(msg) = Result.new(ok: false, error: msg)
end
```

**Call it from the controller:**

```ruby
def create
  result = CheckoutOrder.new(@order, payment_params).call
  if result.ok?
    redirect_to @order, notice: "Order confirmed"
  else
    flash.now[:alert] = result.error
    render :new
  end
end
```

**Naming:** verb + noun — `CheckoutOrder`, `SendWelcomeEmail`, `RecalculateInventory`. If you can't name it, you haven't decided what it does yet.

## Form Objects

Extract when a form:
- Maps to multiple models
- Applies validations that don't belong on any single model
- Transforms input before persistence

```ruby
# app/forms/registration_form.rb
class RegistrationForm
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :email, :string
  attribute :password, :string
  attribute :company_name, :string

  validates :email, presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :password, length: { minimum: 8 }
  validates :company_name, presence: true

  def save
    return false unless valid?

    ActiveRecord::Base.transaction do
      account = Account.create!(name: company_name)
      User.create!(email: email, password: password, account: account)
    end

    true
  end
end
```

`ActiveModel::Model` gives you `valid?`, `errors`, and compatibility with form helpers — use it.

## Decorators

Extract when you need to present a model differently without polluting the model with view-level logic.

Plain Ruby decorator — no gem required:

```ruby
# app/decorators/user_decorator.rb
class UserDecorator < SimpleDelegator
  def display_name
    full_name.presence || email.split("@").first.titleize
  end

  def avatar_url
    "https://gravatar.com/avatar/#{Digest::MD5.hexdigest(email)}"
  end
end
```

```ruby
# In controller or helper
@user = UserDecorator.new(current_user)
```

`SimpleDelegator` forwards every method to the wrapped object. The decorator only defines the methods it adds. Tests call the decorator directly.

Use a decorator when the same model is displayed differently in different contexts (admin vs. public, email vs. web). Don't use it as a dumping ground for every helper method.

## Strategy Pattern

Extract when the same operation has multiple implementations that vary by context, type, or configuration.

```ruby
# app/strategies/shipping_rate_strategy.rb
class FlatRateShipping
  def calculate(order) = Money.new(5_00)
end

class WeightBasedShipping
  def calculate(order)
    Money.new(order.total_weight_kg * 2_50)
  end
end

class FreeShipping
  def calculate(order) = Money.new(0)
end
```

```ruby
class Order < ApplicationRecord
  def shipping_cost
    shipping_strategy.calculate(self)
  end

  private

  def shipping_strategy
    case shipping_tier
    when "flat"   then FlatRateShipping.new
    when "weight" then WeightBasedShipping.new
    when "free"   then FreeShipping.new
    end
  end
end
```

The model picks the strategy; callers only see `order.shipping_cost`. Adding a new shipping type is one new class, no changes to `Order`.

## Observer / Event Pattern

Rails `ActiveSupport::Notifications` provides a lightweight publish/subscribe mechanism that keeps models from directly triggering unrelated side-effects.

```ruby
# Publish from the model
class Order < ApplicationRecord
  after_commit :publish_paid, on: :update, if: :saved_change_to_status?

  private

  def publish_paid
    return unless status == "paid"
    ActiveSupport::Notifications.instrument("order.paid", order: self)
  end
end

# Subscribe in an initializer
ActiveSupport::Notifications.subscribe("order.paid") do |*, payload|
  OrderMailer.confirmation(payload[:order]).deliver_later
end
```

Use this when a model's state change should trigger behavior in an unrelated domain (mailers, analytics, webhooks) and you want to avoid coupling the model to those concerns.

## Concerns

Rails concerns (`app/models/concerns/`, `app/controllers/concerns/`) are modules for extracting **reusable behavior shared across multiple classes**.

Use concerns for:
- Cross-cutting behavior (soft delete, taggable, auditable) shared by several models
- Controller helpers shared by several controllers

Do not use concerns to decompose a single fat model into multiple files — that just hides the problem. If the behavior is only used in one place, a private method or service object is clearer.

```ruby
# app/models/concerns/soft_deletable.rb
module SoftDeletable
  extend ActiveSupport::Concern

  included do
    scope :active, -> { where(deleted_at: nil) }
  end

  def soft_delete
    update!(deleted_at: Time.current)
  end

  def deleted? = deleted_at.present?
end
```

## Decision Tree

```
New logic to write — where does it go?

Is it only about one model's data?
  → model method (or concern if shared across models)

Does it orchestrate multiple models or call external services?
  → service object

Does it validate/coerce multi-model form input?
  → form object

Does it format a model for display without changing data?
  → decorator

Does the same operation vary by type/configuration?
  → strategy

Does a state change need to trigger unrelated side-effects?
  → ActiveSupport::Notifications + subscriber

Is it behavior shared identically across several models?
  → concern
```
