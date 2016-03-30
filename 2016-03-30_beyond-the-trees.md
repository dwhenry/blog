# Seeing beyond the trees

It can be easy to get caught up in a solution. Once given a problem and time to consider alternatives, it can be easy to 
go down the path of implementation blindly form that point on.

Take the following problem:

> Given a set of cucumber features we want to progressively enhance the implementation, and we want to be able to test 
both the javascript and non-javascript versions of the code.

## Solution 1 - Duplicate the features

This is pretty easy, I just copy the feature and add the `@javascript` tag. 

This however feels a little dirty, it would require you to remember to update both versions of the feature each time you
make a change and your test files are now twice the size. The benefit being that if your javascript and non-javascript 
implementation deviate then having duplicate features may be desired.

# Solution 2 - Add a @with_and_without_javascript tag

This tag would then tell cucumber the test need to be run twice, once with javascript and once without.

Thus removing the need for duplicate tests, admittedly it then required your step definition to work with and without 
javascript enabled. They pointed me to an implementation they had used before, unfortunately it was for a different 
version of cucumber. 

Rewriting the implementation took about a day and the below is the final product:

```
require 'cucumber/core/version'
if Cucumber::Core::Version.to_s != '1.4.0'
  raise 'Potentially incompatible version of cucumber - This hack is only tested against Cucumber::Core 1.4.0'
end

module Cucumber
  module Core
    module Gherkin
      class AstBuilder
        class FeatureBuilder
          def initialize(*)
            super
            @language = Ast::LanguageDelegator.new(attributes[:language], ::Gherkin::Dialect.for(attributes[:language]))
            @background_builder = BackgroundBuilder.new(file, attributes[:background]) if attributes[:background]
            @scenario_definition_builders = map_sd_to_builders(file, attributes[:scenario_definitions])
          end

          def map_sd_to_builders(file, scenario_definitions)
            scenario_definitions.flat_map do |sd|
              if sd[:tags].detect { |tag| tag[:name] == '@with_and_without_javascript' }
                map_sd_to_js_and_non_js_builders(file, sd)
              else
                [init_scenario_builder(file, sd)]
              end
            end
          end

          def map_sd_to_js_and_non_js_builders(file, sd)
            scenario_definitions = []

            unless ENV['RUN_JAVASCRIPT_TESTS_ONLY']
              scenario_definitions << (no_js_sd = sd.deep_dup)
              add_tag(no_js_sd, '@with_and_without_javascript', '@no-javascript')
            end

            scenario_definitions << (js_sd = sd.deep_dup)
            add_tag(js_sd, '@with_and_without_javascript', '@javascript')

            scenario_definitions.map { |scenario_definition| init_scenario_builder(file, scenario_definition) }
          end

          def init_scenario_builder(file, sd)
            sd[:type] == :Scenario ? ScenarioBuilder.new(file, sd) : ScenarioOutlineBuilder.new(file, sd)
          end

          def add_tag(attributes, base_tag, new_tag)
            tag = attributes[:tags].detect { |t| t[:name] == base_tag }
            attributes[:tags] << tag.deep_dup.tap { |t| t[:name] = new_tag }
          end
        end
      end
    end
  end
end
```

This essentially hooks into the builder logic and create two node instead of one for any scenario tagged as
`@with_and_with_javascript`. I quite like this implementation, however having it locked to a single version
of cucumber - which given I am hacking on the internal of the application - is likely to change with the
next release, makes this approach very brittle.

## Solution 3 - Run the tests twice.

Isn't that what I am doing above? Yes, but is there a way to do this without hacking on cucumber code?

The answer is yes, and it is actually pretty easy to do. 

Start by clearly marking the tests you want to run with javascript enabled using teh default `@javascript` tag. This
way when you run cucumber, your test the standard path. Next add an additional tag - I choose `@noscipt` to any tests
you would like run a second time with javascript disabled.

These tests will be run separately, and this is done by adding a new rake` task:

```
namespace :cucumber do
  desc 'Run cucumber features tagged with @javascript rack_test driver'
  task :javascript_disabled do
    puts '*' * 50
    puts 'Running cucumber features with JS disabled...'
    puts '*' * 50
    system('WITHOUT_JS=true cucumber --tags @noscript')
  end
end

task default: 'cucumber:javascript_disabled'
```

> You'll notice I have made this new task a default task so that it is run whenever I call `rake`.

So the task itself is just runs cuucmber, filtering on my new tag `@noscipt`, the only other thing is the `WITHOUT_JS` 
environment variable that has been passed in. This is becuase I need to over-write or more accurately, disable the 
`@javascript` tag which exists on the scenarios.

The following change allows me to do this:

```
if ENV['WITHOUT_JS']
  Capybara.javascript_driver = :rack_test
else
  require 'capybara/poltergeist'

  Capybara.javascript_driver = :poltergeist
  Capybara.default_max_wait_time = 20
end
```

Ideally I would like to get this to run without having to pass the environment variable, but don't yet have a 
solution to do this. However this has all the benefits of solution 2, without the drawback of being locked to a 
specific gem version. In addition it required far less hacking and was significantly quicker to implement.
