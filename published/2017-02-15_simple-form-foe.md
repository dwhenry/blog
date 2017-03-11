# Simple ~~Form~~ Foe

Let me start by saying that I love the idea of `simple_form`. An easy way to generate forms with a standard - and automated - way of displaying errors when they occur and just generally give your whole site a consistent look and feel with very little overhead.

On my latest project, I have been looking to use `simple_form` and for the vanilla forms/field this certainly proved to be easy, however once I moved onto my more bespoke fields things soon became far more complicated.

To do this I needed to use 'custom render', and typically of an opensource application the documentation only covered vanialla use cases, so creating a custom render was an undocumented process. Normally this would not have been a problem as I am happy to dig into the code and search for help on google. The difficulty here was that `simple_form` didn't live up to its name as the internals of the gem are anything but simple. I'm sure the architecture fits together easily once you understand it, but for me, it all felt like black magic.

As such, I won't be using `simple_form` on my current project. Not because I think it is bad software or unfit for purpose, I have chosen not to use it, as even if I got it all working - and I feel confident that given the time I could - I would be fearful of future changes, as each time I needed to change it I would need to spend time and effort remembering how it worked. I feel this extra effort outweighs any benefit provided by the convenience provided.

This may be due to the complex nature of my application, or it may be that `simple_form` really is only suitable for vanilla forms.

## Final solution

For referrence I have provided the code I used in my final solution below:

### Helper method 'form_helper.rb':

```
module FormHelper
  def field(f, field_name:, hint: nil, &block)
    errors = Array(field_name).flat_map { |fn| f.object.errors[fn] }.uniq.sort
    render 'form/field', hint: hint, errors: errors, block: block
  end

  def field_tag(value:, error_proc: ->(_) { nil }, hint: nil, &block)
    errors = [error_proc.call(value)].compact
    render 'form/field', hint: hint, errors: errors, block: block
  end
end
```

### View partial 'form/_field.html.slim'

```
.form-group class=('has-danger' if errors.any?)
  = capture(&block)
  - if errors.any?
    .errors
      - errors.each do |error|
        .error= error
  - if hint
    small.text-muted=hint
```

### Example usage:

__Simple field__
```
= field(f, field_name: :date) do
  = f.label :date
  = f.date_field :date
```

__Multiple fields with shared error messaging__
```
= field(f, field_name: [:price, :currency])
  = f.label :price
    = f.text_field :price, type: :number, step: '0.01', min: '0'
    = f.text_field :currency
```

__Using '*_tag' helpers__

```
= field_tag(value: params[:ticker], error_proc: ->(value) { 'please enter ticker' if value.blank? }) do
  = label_tag :ticker
  = text_field_tag :ticker, params[:ticker]
```
