# The RPM Contrib Gem

## Getting Started

To use the rpm_contrib gem, install the `rpm_contrib` gem from rubygems.org.
It will also install the required version of the `newrelic_rpm` gem if it's not
already installed.

For Rails 3.0 and when using Bundler, add these dependencies to your Gemfile:

    gem 'rpm_contrib'
    gem 'newrelic_rpm'

### Troubleshooting Startup

If you've set up your gems to load as described above and you are still not seeing 
data in RPM, there may be a bug in detecting your framework.  Try setting the 
environment variable `NEWRELIC_DISPATCHER` to the name of your app server (Camping, 
Resque, Rake, etc), and please report to us if this fixes the problem so we can
fix the auto-detection logic.

If this does not help then set the `log_level` to `debug` in the `newrelic.yml` file
and examine the `newrelic_agent.log` file for errors after restarting your app.


## Supported Frameworks

A number of frameworks are supported in the contrib gem.  They are all turned on
by default but you can add settings to your newrelic.yml to disable any of them.
### Sinatra view instrumentation

This adds instrumentation to the `render` methods in Sinatra::Base

You can disable it with `disable_sinatra_template` in your newrelic.yml file.

### AWS/S3

Get metrics on how S3 is performing for you in production. To disable AWS/S3, set
'disable_aws-s3' to true in your newrelic.yml file.  For more information on this
instrumentation, check out [our blog](http://blog.newrelic.com/2010/07/06/monitoring-aws-s3/).

# How to Add Custom Instrumentation

We encourage contributions to this project and will provide whatever assistance
we can to those wishing to develop instrumentation for other open source Ruby
libraries.

When adding instrumentation to this gem, be sure and get familiar with the [RPM
Agent API](http://newrelic.github.com/rpm/classes/NewRelic/Agent.html) and
contact support@newrelic.com with any questions.

There are several extension points in the agent you can take advantage of with
this gem.

* Custom tracers which measure methods and give visibility to
  otherwise unmeasured libraries.
* Samplers which sample some value about once a minute.
* Dispatcher support for web request handlers which would otherwise be undetected.
  In order for the agent to turn on in 'auto' mode it needs to discover a 
  web dispatcher, or be [started manually](http://support.newrelic.com/faqs/general/manual-start).
* Framework support, for alternatives to Rails like Camping or Ramaze

## Custom Tracers

Custom tracers for frameworks should be added to the `lib/rpm_contrib/instrumentation`
directory.  These files are loaded at the time the Agent starts.  **They will not
be loaded if the Agent does not start up.** 

It is important that you wrap any instrumentation with the checks necessary to
determine if the code being instrumented is loaded.  You can't add code to the
contrib gem that will break when run in any other context besides yours.


For details on how to define custom tracers, refer to the [support documentation
on adding custom
tracers](http://support.newrelic.com/faqs/docs/custom-metric-collection).  You
can also get detailed information on the API from the [Agent method tracing
rdocs](http://newrelic.github.com/rpm/classes/NewRelic/Agent/MethodTracer.html),
especially the
[add_method_tracer](http://newrelic.github.com/rpm/classes/NewRelic/Agent/MethodTracer/ClassMethods.html)
docs.

A good example can be found in `lib/rpm_contrib/instrumentation/paperclip.rb`.

## Samplers

You can add samplers which will record metrics approximately once a minute.
Samplers are useful for capturing generic instrumentation for display in [custom
views](http://support.newrelic.com/faqs/docs/custom-dashboard-specification).

Samplers should extend the
[`NewRelic::Agent::Sampler`](http://newrelic.github.com/rpm/classes/NewRelic/Agent/Sampler.html)
class.  They should be placed in the `samplers` directory.

Refer to examples in the RPM agent to see how to get started.

## Supporting New Dispatchers

If you want to add support for a new dispatcher which is not being recognized by
default by the RPM agent, add code to the `rpm_contrib/detection` directory.
This code needs to define a module in the `NewRelic::LocalEnvironment` class.
This module will be accessed at the time environment detection takes place, when
the agent is initialized.

This module should define the method `discover_dispatcher` and return the name
of the dispatcher if detected, or defer to super. 

