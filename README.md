# Team Dashboard Plugins

Data Source Plugins for [Team Dashboard](https://github.com/fdietz/team_dashboard)

## Use Plugins in your Team Dashboard

Download the individual ruby file directly in the corresponding directory of your RAILS_ROOT/app/sources.

For example:

    number/hockey_app.rb -> RAILS_ROOT/app/sources/number/hockey_app.rb.

After a page refresh (no restart required) the new source plugin should be automatically available.

Note, that some plugins require either Gemfile or configuration changes. Check out the documentation provided in the ruby file directly.

## Contribute
Did you implement your own data source plugin and think they might be useful to others, too? Fork this project and send me a pull request, please!

## Implement your own Data Source Plugin
You will find example plugins for each type of data source. Use these to get started quickly.

Following an overview of the existing data source types.

### Datapoints
The datapoints source supports data for rendering graphs and aggregated values. Following a minimal implementation.

    class Example < Sources::Datapoints::Base
      def get(targets, from, to, options = {})
        result = []
        targets.each do |target|
          # retrieve the actual data here
          result << { 'target' => "demo.example1", 'datapoints' => [[1, 123456], [7, 123466]] }
        end
        result
      end

      def available_targets(options = {})
        ["demo.example1", "demo.example2"]
      end

      def supports_target_browsing?
        true
      end
    end

Note the datapoints array consists of pairs of number values (y-value and timestamp for the x-value of the graph). This is similar to how Graphite or Ganglia structure their json data for graph data.

For datapoints sources it makes sense to enable users to browse them easily. The widget editor dialog features an autosuggest textfield. it requires the <code>available_targets</code> method to return an array of strings. Additionally <code>supports_target_browsing?</code> should return true.

### CI (Continous Integration Server)
The CI data source delivers build status results.

    class Demo < Sources::Ci::Base
      def get(server_url, project, options = {})
        {
          :label             => "Demo name",
          :last_build_time   => Time.now.iso8601,
          :last_build_status => 0, # success
          :current_status    => 1  # building
        }
      end
    end

### Number
The number data source supports a single integer value and an optional label.

    class Example < Sources::Number::Base
      def get(options = {})
        # retrieve actual data here
        { :value => 115, :label => "example label" }
      end
    end

### Boolean
The boolean data source supports a single boolean value and an optional label.

    class Example < Sources::Boolean::Base
      def get(options = {})
        # retrieve actual data here
        { :value => true, :label => "example label" }
      end
    end

### Global configuration for your data source
It some cases it makes sense to have a global configuration in the Rails app instead of a separate option as part of the widget configuration.

Let's have a look at how its done for for the Graphite data source. There's an entry in application.rb.

    module TeamDashboard
      class Application < Rails::Application
        config.graphite_url = ENV['GRAPHITE_URL']
      end
    end

Note that it is set to the environment variable GRAPHITE_URL, which makes it easy to set without changing the rails app directly. Additionally, this configuration can be set in the environment specific config files.

The graphite data source now can easily access the configuration. In order to determine if the data source is configured correctly you have to implement the <code>available?</code> method:

    class Graphite < Sources::Datapoints::Base
      def available?
        Rails.configuration.graphite_url.present?
      end
    end

Only if <code>available?</code> returns true will the data source be available in the widget editor.

## Contributors

* Marno Krahmer