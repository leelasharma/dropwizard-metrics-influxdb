# dropwizard-metrics-influxdb

Thuis is a java plugin to write [dropwizard formatted metrics](https://dropwizard.github.io/metrics/3.1.0)
to [influxDb](https://influxdb.com).

## Conventions

### Reporter

The InfluxDbReporter extends the [ScheduledReporter](https://dropwizard.github.io/metrics/3.1.0/getting-started/#other-reporting) provided by dropwizard
and is responsible for converting dropwizard metrics into influxDb points.
These points can be sent using one of the senders. The reporter can be
configured to send metrics at regular times.

### Sender

* **HttpSender:**
  The HttpSender writes metrics to influxDb over http/https.
* **UdpSender:**
  The UdpSender writes metrics to influxDb using udp protocol.
  >**Note:** If using the UdpSender, packets greater than the 65KB can't be
sent. This also includes the MTU which varies, so the actual data size that can
be sent is even lower.

## Using library

* Create a metric registry

      MetricRegistry registry = new MetricRegistry();

* Build the InfluxDb config

      InfluxDbConfig config = new InfluxDbConfig.InfluxDbConfigBuilder()
                       .withBaseUrl("http://influxdb:8086")
                       .withDatabase("test")
                       .withAuthString("root:root")
                       .build();
  Refer the Getting credentials section below to get the details needed to
build the config.

* Create the reporter

      InfluxDbReporter reporter = InfluxDbReporter
                       .forRegistry(registry)
                       .convertRatesTo(TimeUnit.SECONDS)
                       .convertDurationsTo(TimeUnit.MILLISECONDS)
                       .filter(MetricFilter.ALL)
                       .withTags(metricsTags)
                       .build(sender);
  The metric tags above is actually a simple map and these tags are by default
applicable to all metrics sent using this reporter. Its advisable to have as
bare minimum the 'service' and 'host' tags included.

* Create the sender

      InfluxDbSender sender = new InfluxDbHttpSender(config)
  Please, use InfluxDbHttpSender unless you are very sure of the packet size use
InfluxDbUdpSender.

* Start the reporter and configure how often it needs to send metrics.

      reporter.start(10, TimeUnit.SECONDS);

* Register metrics in the registry, these will be reported to influxDb
periodically. Dropwizard already provides instrumentation for a lot of services
like Apache, EhCache, jersey etc.  These can be added to the metrics registry.
Refer [dropwizard instrumenting](https://dropwizard.github.io/metrics/3.1.0/manual/httpclient) for a full list of instrumented services.

      // Register memory usage, garbage collection metric set provided by
      // dropwizard's jvm instrumentation.
      registry.registerAll(new MemoryUsageGaugeSet());
      registry.registerAll(new GarbageCollectorMetricSet());
      // Register a custom metric like this timer
      registry.timer("testTimer");
  It is recommended to add MemoryUsageGaugeSet and GarbageCollectorMetricSet for
all java applications.

