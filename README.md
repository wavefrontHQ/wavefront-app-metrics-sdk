# Wavefront App Metrics Reporter [![travis build status](https://travis-ci.com/wavefrontHQ/wavefront-appmetrics-csharp-sdk.svg?branch=master)](https://travis-ci.com/wavefrontHQ/wavefront-appmetrics-csharp-sdk)

This package provides support for reporting metrics recorded by App Metrics to Wavefront via proxy or direct ingestion.

## Dependencies
  * .NET Standard (>= 2.0)
  * App.Metrics (>= 2.1.0)
  * Wavefront.CSharp.SDK (>= 0.2.0) (https://github.com/wavefrontHQ/wavefront-csharp-sdk)

## Usage

### Instantiate a WavefrontSender (either WavefrontProxyClient or WavefrontDirectIngestionClient)
Refer to this page (https://github.com/wavefrontHQ/wavefront-csharp-sdk/blob/master/README.md)
for instructions on how to instantiate WavefrontProxyClient or WavefrontDirectIngestionClient.

### Option 1: Enable the Wavefront reporter using Report.ToWavefront(wavefrontSender)
```
  /*
   * Using the wavefrontSender instantiated by following the above instructions,
   * enable reporting of metrics to Wavefront.
   */
  var metrics = new MetricsBuilder()
    .Report.ToWavefront(wavefrontSender)
    .Build();
```

### Option 2: Configure and enable the Wavefront reporter using Report.ToWavefront(options)
```
  /*
   * Using the wavefrontSender instantiated by following the above instructions,
   * enable reporting of metrics to Wavefront and set the source for your metrics. 
   */
  var metrics = new MetricsBuilder()
    .Report.ToWavefront(
      options => {
        options.WavefrontSender = wavefrontDirectIngestionClient;
        options.Source = "appServer1";
      })
    .Build();
```

### Configuration
The Wavefront reporter has the following configuration options:
  * WavefrontSender - the client that handles sending of metrics to Wavefront via proxy or direct ingestion.
  * Source - the source for your metrics.
  * Filter - the filter used to filter metrics just for this reporter.
  * FlushInterval - the delay between reporting metrics.

### Running the reporter
If you have an ASP.NET Core application, refer to this page
(https://www.app-metrics.io/web-monitoring/aspnet-core/reporting/)
for instructions on how to schedule reporting.

Otherwise, you can run all configured reports using the `ReportRunner` on `IMetricsRoot`:

``` 
  await metrics.ReportRunner.RunAllAsync();
```

Or you can use the `AppMetricsTaskScheduler` to schedule the reporting of metrics:

```
  var scheduler = new AppMetricsTaskScheduler(
    TimeSpan.FromSeconds(10),
    async () =>
    {
      await Task.WhenAll(metrics.ReportRunner.RunAllAsync());
    });
  scheduler.Start();
```

### Wavefront-specific entities that you can report
```
  /* 
   * Wavefront Delta Counter
   * 
   * Configure and instantiate using DeltaCounterOptions.Builder.
   * Do not update option fields after instantiation.
   */
  var myDeltaCounter = DeltaCounterOptions
    .Builder("myDeltaCounter")
    .MeasurementUnit(Unit.Calls)
    .Tags(new MetricTags("cluster", "us-west"))
    .Build();

  ...
  metrics.Measure.Counter.Increment(myDeltaCounter);

```