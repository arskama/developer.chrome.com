---
layout: 'layouts/doc-post.njk'
title: Compute Pressure API
description: >
  Compute Pressure offers high-level states that represents the pressure on the system. It allows 
  the implementation to use the right underlying hardware metrics to ensure that users can take 
  advantage of all the processing power available to them as long as the system is not under 
  unmanageable stress.
subhead: >
  Get informed about your system compute pressure.
date: 2023-03-15
authors:
  - kenchris
  - arskama
tags:
  - api
  - performance
  - monitoring
---

The Compute Pressure API offers high-level states that represents the pressure on the system.
It allows the implementation to use the right underlying hardware metrics to ensure that users can
take advantage of all the processing power available to them as long as the system is not under 
unmanageable stress.

## Current status

<div class="table-wrapper scrollbar">

| Step                                     | Status                              |
| ---------------------------------------- | ----------------------------------  |
| 1. Create explainer                      | [Complete][explainer]               |
| 2. Create initial draft of specification | [Complete][specs]                   |
| 3. Gather feedback & iterate on design   | [In progress](#feedback)            |
| 4. Origin trial                          | Not Started                         |
| 5. Launch                                | Not started                         |

</div>

### Try out the Compute Pressure API

To experiment with the Compute Pressure API locally, read this [page][how-to]

## Interfaces

Pressure Observer API can be run in the following contexts:
- Window
- Dedicated Worker
- Shared Worker

The Compute Pressure API defines two new interfaces.

`PressureObserver`: The primary interface for the Pressure Observer API. Provides methods for 
creating and managing an observer which can watch any number of sources at a pre-defined sample 
rate.
Each observer can asynchronously observe pressure changes trend in a system.

`PressureRecord`: Describes the pressure trend at a specific moment of transition.
Objects of this type can only be obtained in two ways: as an input to your PressureObserver
 callback, or by calling PressureObserver.takeRecords().


### PressureObserver

When an `PressureObserver` object is created, it's configured to watch at a given sample rate the 
pressure of platform collector supported sources. The source can be selected anytime during the 
lifetime of the PressureObserver object. The sample rate cannot be changed after the creation of 
the object.

#### Constructor

`PressureObserver(callback, options)`: Creates a new `PressureObserver` object which will execute
 a specified callback function when it detects that a change in the values of the source being observed has happened.

The constructor takes as parameter a [callback](#callback) function and [options](#options)(optional)

##### Callback {: #callback }

'pressureUpdateCallback()': callback holding the observer handle as well as a sequence of unread `PressureRecord`s

##### Options {: #options }

`PressureObserverOptions`: At the moment only contains the sample rate,`sampleRate`, at which user requests updates.

#### Methods

`PressureObserver.observe(source)`: Tells the 'PressureObserver' a source to observe.

`PressureObserver.unobserve(source)`: Tells the 'PressureObserver' to stop observer a selected source.

`PressureObserver.disconnect()`: Tells the 'PressureObserver' to stop to observe all sources.

`PressureObserver.takeRecords()`: Returns a sequence of [records](#records), since the last callback invoke.

`PressureObserver.supportedSources()`(read only): Returns supported source types by the platform collector.

##### Parameters

`source`: Define the source to be observed, for example `cpu`

### PressureRecord

The `PressureRecord` interface of the Pressure Observer API describes the pressure trend, of a 
source at a specific moment of transition.

#### Instance Properties

`PressureRecord.source` (Read only): The origin source from which the record is coming.

`PressureRecord.state` (Read only): Pressure state recorded.

`PressureRecord.factors` (Read only): Contributing pressure factors, can be more than one.

`PressureRecord.time`(Read only): High resolution timestamp.

##Feedback {: #feedback }

The Compute Pressure team wants to hear about your experiences with the Compute Pressure API.
Please provide your feedback and file issues on [GitHub][issues].

## Examples

### Is the Compute Pressure API supported?

```js
if ('PressureObserver' in globalThis) {
  // The Launch Handler API is supported.
}
```
### Creating a pressure observer

Create the pressure observer by calling its constructor and passing it a callback function to be 
run whenever there is a pressure update:

```js
let observer = new PressureObserver(callback, {sampleRate : 0.5 })
```

A sample rate, `sampleRate`, of 0.5, means that there will be updates at most every two seconds.

The sample rate requested cannot always be served by the system. It is restricted by the 
implementation.

### Using a pressure observer

There is only one way to start a pressure observer.

```js
observer.observe('cpu');
```
In this example the `cpu` is the pressure source we are interested in. For now, it is the only one 
available. In the future, we can consider other sources such as `gpu`, `power`.

Everytime we want to observe a new source, we need to call `ComputePressure.observe(source).

On the opposite, if we want to stop to observe a source. The following example should be followed:

```js
observer.unobserve('cpu');
```

In order to unobserve all sources at once, the following call is necessary:

```js
observer.disconnect();
```

### Retrieving pressure records

```js
function callback(records) {
  const lastRecord = records[recordss.length - 1];
  console.log(`Current pressure ${lastRecords.state}`);
}

const observer = new PressureObserver(callback, { sampleRate: 1 });
await observer.observe("cpu");

// forced record reading
const records = observer.takeRecords();
```

### Tell us about the API design
Is there anything about the API that does not work as you expected? Do you see any missing method
or property for your usage of the API? File a spec issue or comment on an existing one in the
corresping [GitHub repo][issues].

### Report a problem with the implementation
Did you find a bug with Chromium's implementation? Or is the implementation different from the spec?
File a bug at [new.crbug.com](https://new.crbug.com). Be sure to include as much detail as you can,
simple instructions for reproducing, and enter [Blink>PerformanceAPIs>ComputePressure][blink-component] in the **Components** box.



## Helpful links {: #helpful }
- [Specifications] [specs]
- [Public explainer][explainer]
- [Compute Pressure API Demo][demo] | [Compute Pressure API Demo source][demo-source]
- [Chromium tracking bug][cr-bug]
- [ChromeStatus.com entry][cr-status]
- Blink Component: [`Blink>PerformanceAPIs>ComputePressure`][blink-component]
- [TAG Review](https://github.com/w3ctag/design-reviews/issues/795)
- [Ready For Trial][dev-trial]
- [HOWTO page][how-to]

[specs]: https://w3c.github.io/compute-pressure/
[issues]: https://github.com/w3c/compute-pressure/issues
[demo]:  https://w3c.github.io/compute-pressure/demo/
[demo-source]: https://github.com/w3c/compute-pressure/tree/main/demo
[explainer]: https://github.com/w3c/compute-pressure#readme
[cr-bug]: https://bugs.chromium.org/p/chromium/issues/detail?id=1231886
[cr-status]: https://chromestatus.com/feature/5597608644968448
[blink-component]: https://bugs.chromium.org/p/chromium/issues/list?q=component:Blink%3EPerformanceAPIs%3EComputePressure
[dev-trial]: https://groups.google.com/a/chromium.org/g/blink-dev/c/-1ciwdn23J4/m/CuCT52x3DgAJ
[how-to]: https://github.com/w3c/compute-pressure/blob/main/HOWTO.md
