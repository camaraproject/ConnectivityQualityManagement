# CAMARA CQM APIs — Understanding Connectivity Quality Management

## 1. Executive Summary

Most applications consume mobile connectivity as best-effort connectivity: the network tries to deliver the best possible performance at any given moment, and the application does not explicitly request a specific connectivity performance from the operator.
In practice, that performance may vary depending on factors such as network deployment, coverage conditions, location, mobility and current network usage.

For many usage contexts, this is sufficient.
In other scenarios, an API consumer, such as an Application Service Provider (ASP), may need more predictable connectivity quality for a specific application, device, user category or enterprise context.

The CAMARA Connectivity Quality Management (CQM) portfolio addresses these needs through a set of tools or capabilities, exposed by different API for specific connectivity quality needs.

The CQM tools are not interchangeable.
The CAMARA Connectivity Quality Management (CQM) portfolio addresses these needs through a set of tools or capabilities, exposed by different API for specific connectivity quality needs.
The CAMARA Connectivity Quality Management (CQM) portfolio addresses these needs through a set of tools or capabilities, exposed by different API for specific connectivity quality needs. These tools are not interchangeable.
They differ by **when** connectivity quality is needed, **where** it applies, **how many devices** are involved, and **what** the application and its usage needs to achieve: discovering available QoS profiles, applying well-defined connectivity performance immediately or as a long-lived QoS assignment, reserving connectivity performance for a future time and service area, assigning several devices to a reservation, or using a richer reserved connectivity environment with multiple QoS profiles.

This document explains the purpose and differences of those tools.
It is not a normative specification, not a CSP implementation roadmap, and not a strict API selection guide.
Actual API availability and supported tools vary by CSP and markets, and API consumers should always check what their provider supports.
This document explains the purpose and conceptual differences of those tools.
It is not a normative specification, not a CSP implementation roadmap and not a strict API selection guide.
Actual API availability and supported capabilities vary by CSP and market.
The core APIs within scope are:

- `qos-profiles` API helps API consumers discover or reference the QoS profiles made available by the operator. The usage of this API for resolving the connectivity characteristics is optional.
- `quality-on-demand` API enables an application to request an immediate, time-bounded QoS session.
- `qos-provisioning` API supports longer-lived QoS assignment to a device, service or subscription.
- `qos-booking` API introduces planned QoS: a reservation for a defined future time window and service area.
- `qos-booking-and-assignment` API extends the booking idea to scenarios where several devices may need to be assigned or re-assigned to a reservation.
- `qos-profiles`,
- `quality-on-demand`,
- `qos-provisioning`,
- `qos-booking`,
- `qos-booking-and-assignment`,
- the Dedicated Networks API family: `dedicated-network`, `dedicated-network-accesses`, and`dedicated-network-profiles`.
Selected preview capabilities are explicitly identified where relevant.

## 2. Introduction — Why Connectivity Quality Management Matters

Mobile connectivity is today mostly consumed as **best-effort connectivity**.
This means that the network tries to provide the best possible performance at any given moment, but the actual experience may vary depending on many factors, such as network deployment, coverage conditions, terrain, indoor or outdoor location, mobility, and current network usage.

For many mobile services, best-effort connectivity is sufficient.
Users of many standard applications like messaging, browsing, background synchronisation can normally tolerate variable throughput, latency or varying radio conditions.
Other services need more predictable connectivity performance under specific conditions: a live video application may need stable uplink performance during a media production event, a point-of-sale terminal may need reliable connectivity during the opening hours of a temporary store, a media production team may need several devices to operate in the same venue, or an enterprise may need a specific connectivity behaviour for a limited period or within a defined location.

These services are often business critical and better connectivity quality is needed.
There is likely a business impact, when the request for connectivity quality is not accepted or when the promised connectivity quality is not provided at the required time and place.

Connectivity Quality Management (CQM) addresses these needs by exposing API-based tools that allow API consumers to discover, request, configure, reserve or manage connectivity quality in a standardised way.

The key concept is that API consumers and application developers do not need a detailed understanding of mobile network implementations.
API consumers work with external-facing abstractions of network capabilities exposed by the operator, such as QoS profiles, QoS sessions, bookings, service areas, device assignments and dedicated connectivity environments.

The mobile network can therefore be treated as a black box. 
The APIs expose the information that API consumers need: available capabilities, operations to request or manage them, and status information describing the outcome (success or non-success) of those operations.

In simple terms, CQM helps API consumers understand and request **well-defined connectivity performance** when ordinary best-effort behaviour is not enough for a specific service, moment, location or group of devices.
Further, CQM further helps API consumer to understand the responses to their requests and the potential changes. 

This document explains the CQM API portfolio at product and concept level.
It is written for non-telco experts, API consumers, developers and technical product people who need to understand the purpose of each CQM API and the conceptual differences between these APIs, without digesting the YAML specifications in detail or understanding internal mobile network topology.

CQM APIs also need to make non-success outcomes understandable.
Mobile connectivity relies on finite network resources, and a request cannot always be fulfilled under the requested conditions.
A non-success outcome or a later status change should therefore not be interpreted as an API failure: it may be an expected business outcome indicating that the requested connectivity quality is not available for the relevant context.

**Editor's Note: We should be clear on API responses, i.e. success cases and non-success cases. Spectrum and cellular capacity is always a limited & scarce resource and the network needs to provide sometimes a Non-Success response. The Non-Success cases are not failure cases from CSP perspective.**

## 3. Core CQM Concepts in Plain Language

Before looking at each API, it is useful to understand a small set of CQM concepts.
These concepts are API-facing: they describe what an API consumer sees and works with, not how the operator implements them internally.

**Editor's Note: Comment by Hubert on "Connectivity Performance Definition: I wonder if Connectivity Quality is not a term better understood by ASPs. Looking around I found this summry of difference between the terms quality and performance: "The core difference is that quality defines how well an object, service, or system meets requirements or standards, while performance measures how fast, efficiently, or powerfully it executes its functions." I'd suggest to change to 'quality' when we adress an ASP perspective and keep 'performance' when writing from CSP perspective.**

| Concept | Plain-language meaning | Why it matters in CQM |
| --- | --- | --- |
| Connectivity quality | Stability, reliability and consistency of data transmissions in terms of throughput, latency and jitter characteristics. | Describes the need from ASP perspective, i.e. "what is the perception?" |
| Connectivity performance | Performance of the connectivity, characterized by parameters such as throughput, latency, priority or other network treatment parameters. | Describes the connectivity from a CSP perspective, i.e. "what is provided?" |
| Discovery of capabilities | The ability to retrieve (likely?) available QoS profiles, network profiles or (supported?) eligible service areas before requesting or reserving them. | Allows the API consumer to understand the CSP exposes capabilities and its characteristics. |
| Connectivity booking | A reservation of defined connectivity quality for a future time window, service area and one or more devices. | Allows reserving of connectivity quality for a future time, either immenent or distant time horizon. |
| Service area | The geographic area where requested or reserved connectivity quality is expected to apply. | Gives the API consumer an understandable geographic abstraction of the coverage area of the service. |
| Device assignment | The act of linking one or more devices to a booking or reserved connectivity environment. | Explains the distinction between reserving connectivity and determining which devices may use it. |
||||
| **Double term**<br>Connectivity quality | The level of connectivity experience needed for a given usage context. It may be described through performance characteristics such as throughput, latency, priority or jitter. | It explains why ordinary best-effort connectivity may not be sufficient for some services or usage conditions. |
| QoS profile | A reusable description of offered connectivity-performance characteristics. | It is the common reference used when requesting, assigning or reserving defined connectivity quality. |
| On-demand QoS session | A time-bounded application of a QoS profile to a device or, where supported, to a specific application flow. | Explains the on-demand concept of spontaneously requesting an different/additional connectivity quality when it is needed. |
| Provisioned QoS assignment | A long-lived association of a QoS profile with a device, applied when the device connects to the access network. | More persistent QoS Profile assignment, similar to subscription level QoS profiles. |
| Network profile<br>**(tbd)** | A richer connectivity offering that may include multiple QoS profiles, aggregated capacity, device limits and related conditions. | It explains the Dedicated Networks family. |

## 4. Common Journey: Live Event Connectivity

**Editor's Note - Start:**
We can try to connect to the [GSMA Use-Case library](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma-open-gateway-resources/?category=use-cases). 

Enterprise / B2B
- [Remote Journalist](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-journalist/) 
- [Remote Live Broadcast](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-live-broadcast/)
- [Enhanced connectivity for retail stores](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/enhanced-connectivity-for-retail-stores/) 
- [QoD for PoS transactions at events​](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/qod-for-pos-transactions-at-events/)
- [Connection of remote venues](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/connection-of-remote-venues/)
- [Large Event Planning and Delivery](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/large-event-planning-and-delivery/)
- [Smart Video Surveillance](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/smart-video-surveillance/)

Consumer / B2C
- [Reliable network for social media and gaming](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/reliable-network-for-social-media-and-gaming/)
- [Immersive large events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/immersive-large-events/) Consumer Use Case

The business-critical operations of most of the above listed use-cases can be made more robust, when pre-reserving connectivity performance.
For business-critical operations, many ASPs prefer having _planed stability_, i.e. confidence that connectivity quality is provided, when needed. ASPs may be concerned about
* Impact of **not getting** the connectivity service (e.g. QoS Session) at time of requested.
* Impact of **not getting** the promised connectivity quality during the QoS service execution.

**Editor's Note - End**

## Usecases from GSMA Usecase library

| GSMA Usecase library | QoD | QoS<br>Provisioning | QOS<br>Booking | QOS Booking<br>& Assignment | Dedicated<br>Networks |
| --- | --- | --- | --- | --- | --- |
| [Remote Journalist](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-journalist/) | Yes[^1] | Yes[^2] | Yes | Yes | Yes |
| [Remote Live Broadcast](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/remote-live-broadcast/) | Yes[^1] | Yes[^2] | Yes | Yes | Yes |
| [Enhanced connectivity for retail stores](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/enhanced-connectivity-for-retail-stores/) | Yes[^1] | Yes[^2] | Yes | Yes | Yes [^3] |
| [QoD for PoS transactions at events​](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/qod-for-pos-transactions-at-events/)  | Yes[^1] | Yes[^2] | Yes | Yes | Yes |
| [Connection of remote venues](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/connection-of-remote-venues/) | Yes[^1] | Yes[^2] | Yes | Yes | Yes[^3] |
| [Large Event Planning and Delivery](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/large-event-planning-and-delivery/) [^4]  | Yes[^1] | Yes[^2] | Yes | Yes | Yes[^3] |
| [Smart Video Surveillance](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/smart-video-surveillance/) [^5] | Yes | Yes | Yes | Yes | Yes |
| [Reliable network for social media and gaming](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/reliable-network-for-social-media-and-gaming/) [^6] | Yes | ? | Yes | Yes | Yes |
| [Immersive large events](https://www.gsma.com/solutions-and-impact/gsma-open-gateway/gsma_resources/immersive-large-events/) [^7] | | | | | |

[^1]: Connectivity Quality only provided when request is accepted.

[^2]: Connectivity Quality only provided when there is capacity.

[^3]: This use-case seem to need multiple QoS profiles for different devices within the reservation.

[^4]: Unclear set of QOS Profiles / usage purposes.

[^5]: Unclear use-case descriptions: Surveilance cameras are often permanently mounted, temporary permanent (e.g. construction sites) or even mobile (e.g. tram, drones).

[^6]: Unclear use-case descriptions: "Social media" here assumed as "Social media consumption" (not production as by influencers). Gaming Sessions can be spontaneous or planned.

[^7]: Unclear use-case descriptions and unclear purpose of connectivity usage. For a successfult event outcome (in densely populated and congested areas), is seems generally better to reserve connectivity than relying on On-Demand availability.  

## Illustrative journey: connectivity quality around a live event

To make the portfolio easier to understand, this document uses one illustrative journey: **connectivity quality around a live event**.

A live-event context can create different connectivity needs depending on the production setup.
These needs do not all apply to every event, and they are not steps in a mandatory sequence.
The examples below illustrate how different CQM tools may become relevant under different usage conditions.

| Illustrative live-event situation | Concrete API consumer need | Relevant CQM tool(s) | What matters to the API consumer |
| --- | --- | --- | --- |
| Before preparing the event | Discover which QoS profiles, network profiles or eligible service areas the CSP exposes | `qos-profiles`, `dedicated-network-profiles` and, where exposed, service `areas` discovery | Understand which connectivity options can be referenced before requesting or reserving them |
| A reporter or contributor starts an unplanned / spontaneous live uplink<br> | Request defined connectivity performance immediately for a device or application flow. | `quality-on-demand` | Obtain a time-bounded QoS treatment when it is needed; the request may be accepted or rejected depending on current network conditions and the current UE location. <br>**Editor's Note: It is unclear, whether a QoS Profile is usable at any location in thenetwork coverage<br>Editor's Note: "Relevant" is the GSMA identification, because maturity. No configuration yet from ASPs. Other (currently sandbox) APIs support on-demand reservation with immediate activation.** |
| A broadcaster regularly uses the same field equipment across productions | Keep a QoS profile associated with a device whenever it connects to the access network | `qos-provisioning` | Apply a persistent QoS assignment without creating a new on-demand session for every use.<br>**Editor's Note: How much can the broadcaster rely on the QoS Profile performance, assuming that other broadcasters with similar equipment configuration may show up at the same event?<br>Missing Aspect: App-Flow vs Device Level.** |
| A single-camera contribution is scheduled in advance at a known venue | Reserve one QoS profile for one device during a future time window and service area | `qos-booking` | Secure a planned connectivity commitment for a known time and place |
| A small outside broadcast uses several cameras and devices that may be replaced during the event | Reserve connectivity first and assign or re-assign devices later | `qos-booking-and-assignment` | Decouple the reservation from the final device list<br>**Editor's Note: the DN API with a simple Network Profile can be used here as well.** |
| A complex production uses live video, intercom, preview feeds and operational traffic in the same venue | Use a reserved environment where several QoS profiles and device-access rules may be available | `Dedicated Networks` API family | Manage a richer reserved connectivity environment with multiple QoS profiles, different device types and related conditions<br>**Editor's Note: Should we clarify, that the QOD API may be used within the reserved connectivity?** |

The same logic can be transposed to adjacent contexts such as a festival, a pop-up store, temporary point-of-sale terminals or short-term enterprise connectivity at a site.

This journey is illustrative. 
It does **not** imply that every scenario uses every API, that the APIs must be invoked together, or that there is a universal decision tree for selecting an API.
The examples above describe the intended use of each tool.
API consumers should also consider non-success outcomes. 
Depending on the tool and the CSP offering, a request may be rejected, accepted with a status that evolves over time, or later become unavailable if the requested connectivity quality cannot be fulfilled anymore.


## 5. API Deep Dives

Each API is explained with the same compact structure: the need it addresses, what it controls or exposes, the API consumer takeaway, and what it should not be confused with. 
Endpoint-level and schema-level detail is intentionally out of scope.


API specific concepts:

| Concept | Plain-language meaning | Why it matters |
|---|---|---|
| QoS profile | A QoS profile describes offered connectivity performance characteristics, such as throughput, latency, priority or other network treatment parameters. | It is the common reference used across CQM APIs whenever an API consumer wants to request, assign or reserve well-defined connectivity performance. |
| Connection | Devices connection to the network with certain connectivity quality, e.g. best-effort or better. Each connection has an IP address on the device. | It explains the underlying foundations of Connectivity Quality concepts. | 
| Default connection | When the device connects to the network, it gets a default connection. The quality of the default connection is defined by the device subscription. | Provides connectivity with default (aka non-prioritized) connectivity quality. | 
| Prioritized Connection | A device connection where all App Flows (as default) are prioritized according to the assigned connectivity quality. The device gets the priorized connection, when the device is connected and the prioritization is active. | Provides connectivity with prioritized connectivity quality. |
| On-demand QoS session | A time-bounded instantiation of a QoS profile to a specific or all application flow of a device, requested when the connectivity treatment is needed. | It explains the `quality-on-demand` model: ask for a defined QoS treatment at the moment it is needed. |
| Provisioned QoS session | A longer-lived QoS treatment associated with a device, service or subscription, established through provisioning rather than requested dynamically for a short duration. | It explains `qos-provisioning`: QoS behaviour can be configured persistently, not only requested on demand. |
| Network profile | Is a representation of advanced characteristics of the Connectivity Quality, potentially including multiple QoS profiles, aggregated capacity, device limits and related conditions.| Concept presented by the API to the Application Developer. |

The CQM APIs in scope are grouped as follows:

- **On-demand connectivity management** — request a defined QoS treatment for immediate, time-bounded use:
  - `quality-on-demand`
- **Longer-lived QoS assignment** — associate a QoS treatment with a device, service or subscription persistently:
  - `qos-provisioning`
- **Reservation-based connectivity management** — commit connectivity quality in advance for a future time, service area and one or more devices, with or without a richer reserved environment:
  - `qos-booking`
  - `qos-booking-and-assignment`
  - `dedicated-network` and `dedicated-network-accesses`
- **Discovery / support APIs** — expose information about what the CSP makes available & where, without changing connectivity behaviour:
  - `qos-profiles`
  - `dedicated-network-profiles`
  - `dedicated-network-areas`


### 5.1 On-demand connectivity management

#### `quality-on-demand` — Immediate, time-bounded QoS sessions

| Aspect | Explanation |
| --- | --- |
| Need addressed | The application needs a defined QoS treatment now — for example when a device starts a live uplink. |
| What it controls | An on-demand QoS session for one or more App-Flows (even all App-Flows of a Device), associated with a QoS profile and a defined duration. |
| API consumer takeaway | This is the immediate "apply a QoS profile now, for a bounded duration" tool. |
| Not to be confused with | Future reservations, persistent QoS configuration, multi-device assignment or reserved connectivity environments. |

`quality-on-demand` is about immediate QoS activation. It is not a booking mechanism and should not be explained as one.

An on-demand QoS request may be rejected or may later become unavailable if the network cannot fulfil the requested QoS profile.
This may be because of high load or because the QoS profile is not supported at the current location.

### 5.2 Longer-lived QoS assignment

#### `qos-provisioning` — Provisioned QoS for a device, service or subscription

| Aspect | Explanation |
| --- | --- |
| Need addressed | A device, service or subscription needs to be associated with a QoS profile persistentlythan a temporary session. |
| What it controls | Longer-lived assignment or provisioning of QoS behaviour, depending on the operator’s implementation and service model. |
#### `qos-provisioning` — Longer-lived QoS assignment for a device

| Aspect | Explanation |
| --- | --- |
| Need addressed | A device needs to remain associated with a QoS profile beyond a short on-demand session. |
| What it controls | A longer-lived QoS assignment that remains in place until it is changed or revoked. The assigned QoS profile is applied when the device connects to the access network. |
| API consumer takeaway | The key distinction from `quality-on-demand` is persistence: the API consumer does not need to request a new short-lived QoS session every time the device reconnects. |
| Not to be confused with | A universal guarantee that identical measured performance will be available at every time and location, or with a future time-and-area reservation. |

`qos-provisioning` persists the association between a device and a QoS profile. It does not define an explicit service area or a future booking window. API consumers should therefore not interpret provisioning as a guarantee that identical measured performance will be available everywhere and at all times. Actual fulfilment remains subject to the QoS profile semantics, the CSP offering and network availability.

### 5.3 Reservation-based connectivity management

#### `qos-booking` — Connectivity booking for a future window and service area

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer knows in advance that defined connectivity performance will be needed at a specific time and place. |
| What it controls | A connectivity booking for a QoS profile, a future time window and a service area, applicable to a device. |
| API consumer takeaway | The essential difference from `quality-on-demand` is planning. The essential difference from `qos-provisioning` is that the commitment is bounded by time and service area. |
| Not to be confused with | Multi-device assignment, network profiles or reserved connectivity environments. |

A booking request may be accepted or rejected based on what the CSP can support in the requested time window and service area.
API consumers need clear expectations on acceptance, rejection, reliability and expected system behaviour.
Where the CSP exposes them, mechanisms such as capacity pre-checks or confidence indications can support these expectations, but they should only be assumed when the API or the CSP offering explicitly supports them.

#### `qos-booking-and-assignment` — Connectivity booking with device assignment

| Aspect | Explanation |
| --- | --- |
| Need addressed | Connectivity needs to be reserved for a future window and service area, while the devices that will use it are not all known at booking time or are expected to change. |
| What it controls | A connectivity booking together with the ability to assign and re-assign devices to that booking over its lifecycle. |
| API consumer takeaway | The value is the separation of the reservation from the device list, so that devices can be added, removed or replaced without re-creating the reservation. |
| Not to be confused with | A simple per-device booking, or a reserved connectivity environment that exposes multiple QoS profiles. |

#### `dedicated-network` — Reserved connectivity environment

| Aspect | Explanation |
| --- | --- |
| Need addressed | The use case calls for a reserved connectivity environment that may expose multiple QoS profiles, a defined service area, a time window and conditions such as aggregated capacity or device limits. |
| What it controls | A reserved environment based on a network profile, within which one or more QoS profiles can be made available to devices. |
| API consumer takeaway | Dedicated Networks are relevant when the CSP offering exposes a richer reserved connectivity environment, especially where multiple QoS profiles, device access management, aggregated capacity, device limits or additional conditions are relevant. Whether this API family is relevant depends on the API consumer need, the CSP offering and market availability. |
| Not to be confused with | A single-profile booking (`qos-booking`) or an on-demand QoS session (`quality-on-demand`). |

#### `dedicated-network-accesses` — Device access management for a reserved environment

| Aspect | Explanation |
| --- | --- |
| Need addressed | Devices need to be granted, changed or revoked access to a reserved connectivity environment over its lifecycle. |
| What it controls | The set of device accesses associated with a `dedicated-network`, including assignment and re-assignment of devices. |
| API consumer takeaway | This is the device-side companion to `dedicated-network`. It is the API consumer's tool to manage which devices may use the reserved environment. |
| Not to be confused with | Discovering network profiles (`dedicated-network-profiles`) or creating the reserved environment itself (`dedicated-network`). |


### 5.4 Discovery / support APIs
#### `qos-profiles` — Discovering available QoS characteristics

| Aspect | Explanation |
| --- | --- |
| Need addressed | An application cannot request, assign or reserve a QoS profile if it does not know which profiles are available or how they are identified. |
| Controls | It does not control connectivity directly. It exposes information about QoS profiles that may be referenced by other CQM APIs. |
| Developer takeaway | Treat `qos-profiles` as a supporting catalogue. It is similar to learning the available product options before requesting one of them. |
| Not meant for | Activating QoS, reserving QoS, provisioning a device or creating a dedicated network. |


#### `dedicated-network-profiles` — Discovering available network profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to know which network profiles the CSP exposes for Dedicated Networks scenarios before booking or assigning devices to a reserved connectivity environment. |
| What it exposes | A catalogue of network profiles, which may describe richer or multi-dimensional connectivity offerings (potentially including multiple QoS profiles, aggregated capacity, device limits and related conditions). |
| API consumer takeaway | This is the discovery API of the Dedicated Networks family. Its role is similar to `qos-profiles`, but for network profiles rather than individual QoS profiles. |
| Not to be confused with | Booking a reserved environment or managing device access. |

#### `dedicated-network-areas` — Discovering of service areas, supporting QoS and/or network profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to know which QoS profiles or Network Profiles are consistently supported within a certain service area |
| What it exposes | A set of service areas, can provide a specific connectivity performance. |
| API consumer takeaway |  |
| Not to be confused with |  |

#### Service-area discovery (`dedicated-network-areas`, current draft name)

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to identify eligible service areas and understand which QoS profiles and/or network profiles are supported in each area before creating a reservation. |
| What it exposes | A catalogue of service areas, including their geographic definition and the QoS profiles and/or network profiles supported within each area. |
| API consumer takeaway | This optional discovery capability helps the API consumer select an eligible service area without needing to understand the operator's internal network topology. |
| Not to be confused with | A generic coverage map, a guarantee that any arbitrary area can support the requested profile, or the creation of a booking or dedicated network. |

## 6. Comparative Matrix and Purpose-Oriented Navigation

The CQM APIs are related because they all deal with connectivity quality. However, they differ substantially in scope and complexity.

The CQM APIs share the topic of connectivity quality but differ in scope. The most useful comparison is by capability axes: timing, geography, device model and profile model.

| API | Timing | Area | Device model | QoS/profile model | Complexity |
| --- | --- | --- | --- | --- | --- |
| `qos-profiles` | Not applicable | Not applicable | Not applicable | Catalogue of QoS profiles | Low |
| `quality-on-demand` | Immediate / session duration | Not the primary pattern | Single device or flow | One QoS profile per session | Low / Medium |
| `qos-provisioning` | Longer-lived / persistent | Not the primary pattern | Device, service or subscription | One QoS profile assignment | Low / Medium |
| `qos-booking` | Future time window | Defined service area | One device | One QoS profile per booking | Medium |
| `qos-booking-and-assignment` | Future time window | Defined service area | Multiple devices / assignment | Typically one QoS profile per reservation pattern | High |

The main relationships are:

- `qos-profiles` supports the portfolio by exposing available QoS profile references.
- `quality-on-demand` and `qos-provisioning` both improve QoS, but differ in duration and persistence.
- `qos-booking` adds planning: a future time window and service area.
- `qos-booking-and-assignment` adds multi-device assignment behaviour to the reservation model.
- `dedicated-network` and `dedicated-network-accesses` allows for a richer reserved environment pattern, especially when multiple QoS profiles or deeper preparation are needed.

| API | Category | Timing | Service area | Device model | QoS / profile model |
| --- | --- | --- | --- | --- | --- |
| `qos-profiles` | Discovery / support | n/a | n/a | n/a | Catalogue of QoS profiles |
| `dedicated-network-profiles` | Discovery / support | n/a | n/a | n/a | Catalogue of network profiles |
| `quality-on-demand` | On-demand | Immediate, session duration | Not the primary dimension | Single device or flow | One QoS profile per session |
| `qos-provisioning` | Longer-lived assignment | Persistent until changed | Not the primary dimension | Device, service or subscription | One QoS profile assignment |
| `qos-booking` | Reservation-based | Future time window | Defined service area | One device per booking | One QoS profile per booking |
| `qos-booking-and-assignment` | Reservation-based | Future time window | Defined service area | Multiple devices via assignment | One QoS profile per reservation |
| `dedicated-network` | Reservation-based (Dedicated Networks) | Reserved environment lifecycle | Defined service area | Multiple devices via accesses | Potentially multiple QoS profiles via a network profile |
| API | Category | Timing | Service area | Device model | QoS / profile model |
| --- | --- | --- | --- | --- | --- |
| `qos-profiles` | Discovery / support | n/a | n/a | n/a | Catalogue of QoS profiles |
| `dedicated-network-profiles` | Discovery / support | n/a | n/a | n/a | Catalogue of network profiles |
| Service-area discovery (`dedicated-network-areas`, current draft name) | Discovery / support | n/a | Catalogue of eligible service areas | n/a | Associates areas with supported QoS profiles and/or network profiles |
| `quality-on-demand` | On-demand | Immediate, session duration | Not the primary dimension | Device or application flow | One QoS profile per session |
| `qos-provisioning` | Longer-lived assignment | Persists until changed or revoked | No explicit service area in the API contract | Device | One QoS profile assignment |
| `qos-booking` | Reservation-based | Future time window | Defined service area | One device per booking | One QoS profile per booking |
| `qos-booking-and-assignment` | Reservation-based | Future time window | Defined service area | Multiple devices via assignment | One QoS profile per reservation |
| `dedicated-network` | Reservation-based | Reserved-environment lifecycle | Defined service area | Multiple devices via accesses | Potentially multiple QoS profiles via a network profile |
| `dedicated-network-accesses` | Access management | Follows the reserved-environment lifecycle | Inherited from the reserved environment | Device-access management | Inherited from the reserved environment |

This matrix is intended to make portfolio roles visible.
It is **not** a strict API selection guide and does not guarantee CSP support for any specific combination.

---

## 7. Key Takeaways, Guardrails and Sources

### 7.1 Key takeaways

The CQM portfolio is best understood as a set of connectivity-quality control patterns.

The main differences between the APIs are:

- whether the need is immediate, persistent or planned;
- whether a service area and time window are part of the request;
- whether the pattern applies to one device or several devices;
- whether one QoS profile is enough or a richer multi-profile environment is needed;
- how much operational complexity the pattern introduces.

In short:

- `qos-profiles` is the supporting catalogue.
- `quality-on-demand` is the immediate session pattern.
- `qos-provisioning` is the longer-lived assignment pattern.
- `qos-booking` is the simple planned reservation pattern.
- `qos-booking-and-assignment` is the multi-device assignment pattern.
The CQM portfolio is best read as a set of API tools for different connectivity-quality needs. 
The main dimensions that distinguish them are:

- whether the need is immediate, persistent or planned for a future window;
- whether a service area is part of the request;
- whether one device or several devices are involved, and whether device assignment must be managed over time;
- whether one QoS profile is enough, or the scenario benefits from a reserved environment that may expose multiple QoS profiles and related conditions.

Discovery / support APIs help the API consumer see what is available.
On-demand and longer-lived QoS tools apply a QoS profile to a device, service or flow.
Reservation-based tools commit connectivity quality in advance, with the Dedicated Networks family extending this to reserved connectivity environments with device access management.

### 7.2 Guardrails for external readers

When explaining or using the CQM portfolio, the following guardrails matter:

1. **Do not expose operator topology.**  
API consumers should have only a high level of understanding of a cellular system and its intrinsic behaviour.
Concepts like cells, cell-edges and radio carriers are understood on high level.
API consumers should not need to have detailed understanding of the network topology, including cells, grids, carriers, radio layers or internal coverage planning.

2. **Do not imply universal support.**  
CAMARA specification and operator availability are not the same thing.
API consumers should check which APIs and what capabilities their provider supports.
API consumers should not assume that all possible capabilities and features are supported by a CSP.
For example, QoS Profile description within the QoS Profile API allows usage of features like DSCP or L4S, which may not be leveraged by the offering CSP.

3. **Do not turn the portfolio into a strict decision tree.**  
The same business scenario may be addressed differently depending on operator capabilities, market maturity and commercial context.

4. **Choose the API tool according to the actual connectivity need.**  
API consumers should not assume that every CQM scenario requires multi-device assignment or a dedicated connectivity environment.
Some needs may be addressed through immediate QoS, longer-lived QoS assignment, or a simple planned reservation.
Other scenarios may require device assignment or a reserved environment where multiple QoS profiles are available.
The relevant API tool depends on the need, the CSP offering and the capabilities available in the target market.

5. **Be clear about expected system behaviour.**  
API consumers need to understand what they can expect from each CQM tool: whether a request can be accepted, why it may be rejected, what happens when the requested connectivity performance cannot be supported, and how changes are communicated.
Confidence levels, capacity pre-checks or prediction mechanisms may be useful for some scenarios, but they should only be described as available when they are explicitly supported by the API or the CSP offering.

6. **Keep internal CQM mechanics out of external explanations.**  
1. **Do not assume operator topology.** API consumers work with external abstractions such as QoS profiles, QoS sessions, connectivity bookings, service areas, device assignments and network profiles.
They do not need to reason about internal network topology.
2. **Do not assume universal CSP support.** CAMARA specification availability and CSP availability are not the same thing.
API consumers should check which APIs and which capabilities their CSP supports.
For example, features that may be expressed in a QoS profile description are not necessarily activated in every CSP offering.
3. **Do not turn the portfolio into a strict decision tree.** The same scenario may be addressed differently depending on CSP capabilities, market maturity and commercial context.
4. **Match the API tool to the actual need.** Some needs are addressed by an on-demand QoS session, others by a longer-lived assignment, a simple connectivity booking, a booking with device assignment, or a reserved connectivity environment.
The relevant API tool depends on the API consumer need, the CSP offering and market availability.
5. **Be clear on expected system behaviour.** API consumers need clear expectations on acceptance, rejection, reliability and expected system behaviour.

### 7.3 Sources and version notes

This document should be anchored in:

- the Fall25 CAMARA specifications for the in-scope CQM APIs;
- public or shareable CAMARA material related to CQM;
- use-case driven CQM comparison material, especially event, media production, pop-up store and data-boost scenarios;
- the SIM Swap / Device Swap / Tenure / Number Recycling explanatory whitepaper as inspiration for communication style and structure.

- the published CAMARA specifications for the in-scope CQM APIs;
- public CAMARA material related to CQM;
- use-case-driven CQM explanatory material, especially event, media production, pop-up store and data-boost scenarios.

## Appendix (e.g. change log)
