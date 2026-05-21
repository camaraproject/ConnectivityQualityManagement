# CAMARA CQM APIs — Understanding Connectivity Quality Management

## 1. Executive Summary

Most applications consume mobile connectivity as best-effort connectivity: the network tries to deliver the best possible performance at any given moment, but the application does not explicitly request a specific connectivity performance from the operator. In practice, that performance can vary significantly depending on factors such as network deployment, coverage conditions, terrain, indoor or outdoor location, mobility, and current network usage.
Most applications consume mobile connectivity as best-effort: the network tries to deliver the best possible performance at any moment, but the application does not request a specific connectivity quality. For some, an API consumer (aka Application Service Provider (ASP)) needs more predictable connectivity quality for a specific application, device, user category or enterprise context — for example a live video uplink during an event, payment terminals in a pop-up store, several cameras and intercoms in the same venue, or temporary enterprise connectivity at a defined site.
The CAMARA Connectivity Quality Management (CQM) portfolio addresses these needs through a set of API tools. They are not interchangeable. They differ by **when** connectivity quality is needed, **where** it applies, **how many devices** are involved, and whether the need is covered by a single QoS profile or by a richer reserved connectivity environment.


At a high level, CQM is based on the idea that some API consumers or connectivity users need well-defined connectivity performance, such as predictable uplink behaviour, latency characteristics, prioritisation, or minimum throughput. These needs may be linked to an application, a device, a service, a user category, an enterprise usage context, or a specific event scenario.
In CAMARA, these connectivity characteristics are typically described through **QoS profiles**, which can then be discovered, requested, assigned, reserved, or used as part of a richer connectivity environment.

The CQM portfolio can be summarised as follows:

- `qos-profiles` helps API consumers discover or reference the QoS profiles made available by the operator. The usage of this API for resolving the connectivity characteristics is optional.
- `quality-on-demand` enables an application to request an immediate, time-bounded QoS session.
- `qos-provisioning` supports longer-lived QoS assignment to a device, service or subscription.
- `qos-booking` introduces planned QoS: a reservation for a defined future time window and service area.
- `qos-booking-and-assignment` extends the booking idea to scenarios where several devices may need to be assigned or re-assigned to a reservation.
- `dedicated-network` and `dedicated-network-accesses` represent a more advanced connectivity environment, specifically when involving multiple QoS profiles and deeper operator-side preparation.

From an API consumer perspective, the CQM portfolio should be understood as a set of API tools for different connectivity-quality needs. The relevant tool depends on what the application and its usage needs to achieve: discovering available QoS profiles, requesting well-defined connectivity performance immediately, applying a longer-lived QoS assignment, reserving connectivity performance for a future time and service area, assigning several devices to a reservation, or using a richer reserved connectivity environment with multiple QoS profiles.
From an API consumer perspective, the CQM portfolio is best understood as a set of API tools for different connectivity-quality needs: discovering available capabilities, requesting well-defined connectivity performance immediately, applying a longer-lived QoS assignment, reserving connectivity performance for a future time and service area, managing device assignment, or using a reserved connectivity environment that may expose multiple QoS profiles.
This document explains the purpose and differences of those tools. It is not a normative specification, not a CSP implementation roadmap, and not a strict API selection guide. Actual API availability and supported capabilities vary by CSP and market, and API consumers should always check what their provider supports.

## 2. Introduction — Why Connectivity Quality Management Matters

Mobile connectivity is today mostly consumed as **best-effort connectivity**. This means that the network tries to provide the best possible performance at any given moment, but the actual experience may vary depending on many factors, such as network deployment, coverage conditions, terrain, indoor or outdoor location, mobility, and current network usage.

For many digital services, best-effort connectivity is sufficient. Users of many standard applications like messaging, browsing, background synchronisation can normally tolerate variable throughput, latency or varying radio conditions. Other services need more predictable connectivity performance under specific conditions: a live video application may need stable uplink performance during a contribution window, a point-of-sale terminal may need reliable connectivity during the opening hours of a temporary store, a media production team may need several devices to operate in the same venue, or an enterprise may need a specific connectivity behaviour for a limited period or within a defined location.

Connectivity Quality Management (CQM) addresses these needs by exposing API-based tools that allow application providers to discover, request, configure, reserve or manage connectivity quality in a standardised way.

The key concept is, that API consumers and application developers do not need to have detailed understanding of mobile network realizations. API consumers work with external-facing abstractions of network capabilities exposed by the operator, such as QoS profiles, QoS sessions, bookings, service areas, device assignments and dedicated connectivity environments.

In simple terms, CQM helps API consumers understand and request **well-defined connectivity performance** when ordinary best-effort behaviour is not enough for a specific service, moment, location or group of devices.

## 3. Core CQM Concepts in Plain Language

Before looking at each API, it is useful to understand a small set of shared CQM concepts.

| Concept | Plain-language meaning | Why it matters in CQM |
| --- | --- | --- |
 | Connectivity performance described by a QoS profile | A QoS profile describes offered connectivity performance characteristics, such as throughput, latency, priority or other network treatment parameters. | It provides the common reference used by CQM APIs when an API consumer wants to request, assign, reserve or discover well-defined connectivity performance. |
 | On-Demand QoS session | The application of a QoS profile to all applications of a device or, where supported, to a specific application flow, usually for a defined duration. | It explains the `quality-on-demand` model: requesting a specific QoS treatment when it is needed. |
 | Provisioning QoS Session | A longer-lived way of establishing or maintaining QoS behaviour for a device, service or subscription. | It explains `qos-provisioning`: QoS behaviour is not only requested temporarily on demand, but may also be configured more persistently. |
 | Booking | A planned reservation of connectivity performance for a future time window and, where applicable, a service area. | It explains `qos-booking` and related reservation-based APIs. |
 | Service area | The geographic area where the requested or reserved connectivity performance is expected to apply. | It is central to booking and dedicated connectivity scenarios, while keeping operator-internal topology hidden. |
 | Device assignment | The act of linking one or more devices to a reservation or reserved connectivity environment. | It explains why `qos-booking-and-assignment` and `dedicated-network-accesses` are relevant when devices must be assigned, replaced or managed over time. |
 | Network profile | A Dedicated Networks construct that describes a richer or multi-dimensional connectivity offering, potentially including multiple QoS profiles, aggregated capacity, device limits and related conditions. | |
Before looking at each API, it is useful to understand a small set of CQM concepts. These concepts are API-facing: they describe what an API consumer sees and works with, not how the operator implements them internally.

| Concept | Plain-language meaning | Why it matters in CQM |
| --- | --- | --- |
| QoS profile | A QoS profile describes offered connectivity performance characteristics, such as throughput, latency, priority or other network treatment parameters. | It is the common reference used across CQM APIs whenever an API consumer wants to request, assign or reserve well-defined connectivity performance. |
| Discovery of capabilities | Discovery allows an API consumer to retrieve available QoS profiles, network profiles or supported connectivity options before requesting, assigning or reserving them. | It lets the API consumer work with abstracted, understandable information rather than guessing what the CSP offers. |
| On-demand QoS session | A time-bounded application of a QoS profile to a device or, where supported, to a specific application flow, requested when the connectivity treatment is needed. | It explains the `quality-on-demand` model: ask for a defined QoS treatment at the moment it is needed. |
| Provisioned QoS session | A longer-lived QoS treatment associated with a device, service or subscription, established through provisioning rather than requested dynamically for a short duration. | It explains `qos-provisioning`: QoS behaviour can be configured persistently, not only requested on demand. |
| Connectivity booking | A reservation of connectivity quality for a future time window, service area and one or more devices. | It explains the reservation-based tools, where the API consumer needs advance commitment for a known future need. |
| Service area | The geographic area where the requested or reserved connectivity performance is expected to apply. | It is central to booking and reserved connectivity environments, while keeping operator-internal topology hidden. |
| Device assignment | The act of linking one or more devices to a provisioned QoS session, a connectivity booking or a reserved connectivity environment. | It explains why several APIs separate "what is reserved or provisioned" from "which devices use it", which is useful when device lists change over time. |
| Network profile | A Dedicated Networks construct representing a richer or multi-dimensional connectivity offering, potentially including multiple QoS profiles, aggregated capacity, device limits and related conditions. | It is the discovery-side concept for Dedicated Networks: what kind of reserved connectivity environment the CSP exposes, before any booking or device assignment. |

## 4. Common Journey: Live Event Connectivity

To make the portfolio easier to understand, this document uses one common illustrative journey: **connectivity quality around a live event**.
Imagine an application provider supporting a live media event. During the event, contributors use mobile devices or professional equipment to send live uplink video and audio. The event takes place in a known venue or outdoor area, and the relevant period may be known in advance.
The provider may face different connectivity-quality needs:

- discover which QoS profiles are available;
- request well-defined connectivity performance immediately for a device that is going live now;
- configure a longer-lived QoS behaviour for a service or device;
- reserve QoS for a future event window and venue area;
- manage several devices under a common reservation;
- use a more advanced dedicated connectivity environment when multilple QoS profiles may be available or managed for a complex production.
Around this event, the provider may face several different connectivity performance needs. These needs do not all apply to every event, and they are not steps of a single sequence. They are independent illustrations of what different CQM tools are intended for.

| Possible need in a live-event context | API tool / category | Main API or API family |
 | --- | --- | --- |
| Understand which QoS characteristics are available before requesting connectivity treatment | Discovery / support | `qos-profiles` |
| Request a well-defined QoS profile immediately for a device or flow that is going live now | On-demand connectivity management | `quality-on-demand` |
| Associate a QoS behaviour with a device, service or subscription more persistently | Longer-lived QoS assignment | `qos-provisioning` |
| Reserve connectivity quality for a known future time window and service area | Reservation-based connectivity management | `qos-booking` |
| Reserve connectivity first and assign or re-assign several devices later | Reservation and device assignment | `qos-booking-and-assignment` |
| Use a reserved connectivity environment where multiple QoS profiles may be available or managed | Dedicated connectivity environment | `dedicated-network`, with `dedicated-network-accesses` and `dedicated-network-profiles` where applicable |

The same logic can also be understood through adjacent examples such as a festival, a pop-up store, event point-of-sale terminals or temporary enterprise connectivity. The value of the journey is not that every scenario uses every API. Its value is that it makes the main CQM dimensions visible.

| ASP need dimension | API tool / category | Why it matters to the API consumer |
| --- | --- | --- |
| Understand which connectivity performance options are available | Discovery / support APIs, such as `qos-profiles` and `dedicated-network-profiles` | The ASP needs to know which QoS profiles or network profiles can be referenced before requesting, assigning or reserving connectivity treatment. |
| Request well-defined connectivity performance for immediate use | On-demand connectivity management, mainly `quality-on-demand` | The ASP needs a QoS session to be accepted for a device or flow that requires the defined performance now. |
| Apply connectivity treatment more persistently | Longer-lived QoS assignment, mainly `qos-provisioning` | The ASP or service provider may need a QoS behaviour to remain associated with a device, service or subscription until changed. |
| Reserve connectivity performance for a future time and service area | Reservation-based connectivity management, mainly `qos-booking` | The ASP needs to know in advance whether the requested QoS profile can be reserved for a given time window and location. |
| Manage several devices under a reservation | Reservation plus device assignment, mainly `qos-booking-and-assignment` | The ASP may need to reserve connectivity first and assign or re-assign devices later, especially when the final device list is not known at booking time. |
| Possible need around the live event | What the API consumer is trying to do | CQM tool category |
| --- | --- | --- |
| Find out which QoS profiles or network profiles the CSP exposes | Discovery of capabilities before any request | Discovery / support APIs |
| Make a single device or flow go live now with defined uplink performance | Get a time-bounded QoS treatment immediately | On-demand connectivity management |
| Keep a defined QoS behaviour associated with a device, service or subscription beyond a short session | Apply a longer-lived QoS assignment | Longer-lived QoS assignment |
| Secure defined connectivity performance in advance for a known future window and venue area | Reserve connectivity quality ahead of time | Reservation-based connectivity management |
| Reserve connectivity for the event and later assign or change the devices that will use it | Combine reservation with device assignment | Reservation-based connectivity management |
| Operate several devices and traffic types in the same venue under one reserved environment that may expose multiple QoS profiles | Use a reserved connectivity environment | Reservation-based connectivity management (Dedicated Networks family) |

The same logic can be transposed to adjacent contexts such as a festival, a pop-up store, temporary point-of-sale terminals or short-term enterprise connectivity at a site. The journey is illustrative: it shows the main dimensions (timing, geography, devices, profile model) that distinguish the CQM tools. It does **not** imply that every scenario uses every API or that the APIs must be invoked together or in sequence.

## 5. Scope and Portfolio Grouping

This document explains the CQM API portfolio at product and concept level. It is written for non-telco experts, API consumers, developers and technical product people who need to understand what each API is for without reading the YAML specifications or understanding internal mobile network topology.
The document covers the following CQM APIs, using the **Fall25 CQM portfolio** as the reference view:

- `quality-on-demand`
- `qos-provisioning`
- `qos-booking`
- `qos-booking-and-assignment`
- `dedicated-network` and `dedicated-network-accesses`
Discovery APIs:
- `qos-profiles`
- `dedicated-network-profiles`

This document is **not**:

- a CAMARA CQM Working Group harmonisation plan;
- an operator implementation roadmap;
- a mandatory deployment sequence;
- a strict API selection guide;
- a normative CAMARA specification;
- a comparison of YAML endpoints, schemas or payloads.
This section explains how the in-scope APIs are grouped, so the rest of the document can be navigated by purpose rather than by API name alone.

The CQM APIs in scope are grouped as follows:

- **Discovery / support APIs** — expose information about what the CSP makes available, without changing connectivity behaviour:
  - `qos-profiles`
  - `dedicated-network-profiles`
- **On-demand connectivity management** — request a defined QoS treatment for immediate, time-bounded use:
  - `quality-on-demand`
- **Longer-lived QoS assignment** — associate a QoS treatment with a device, service or subscription persistently:
  - `qos-provisioning`
- **Reservation-based connectivity management** — commit connectivity quality in advance for a future time, service area and one or more devices, with or without a richer reserved environment:
  - `qos-booking`
  - `qos-booking-and-assignment`
  - `dedicated-network`
  - `dedicated-network-accesses`

This document is intended for non-telco experts, API consumers, ASPs, developers and technical product people who need to understand what each API is for without reading YAML specifications or studying mobile network topology.

It is **not**:

- a normative CAMARA specification;
- a CSP implementation roadmap;
- a strict API selection decision tree;
- a comparison of YAML endpoints, schemas or payloads;
- an internal Working Group harmonisation document.

CAMARA specification availability and CSP availability are not the same thing. API consumers should always check which CQM APIs and which capabilities their provider supports.

## 6. API Deep Dives

Each API is explained with the same compact structure: the need it addresses, what it controls or exposes, the API consumer takeaway, and what it should not be confused with. Endpoint-level and schema-level detail is intentionally out of scope.

### 6.1 `quality-on-demand` — Immediate QoS sessions

| Aspect | Explanation |
| --- | --- |
| Problem addressed | The application needs improved connectivity now, for example when a user starts a live video stream. |
| Controls | A temporary QoS session for a device or flow, associated with a requested QoS profile and a defined duration. |
| Developer takeaway | This is the immediate, time-bounded QoS pattern. It is the simplest “do something now” model. |
| Not meant for | Future reservations, persistent configuration, multi-device orchestration or dedicated connectivity environments. |

`quality-on-demand` is about immediate QoS activation. It is not a booking mechanism and should not be explained as one.

### 6.2 `qos-provisioning` — Long-lived QoS assignment

| Aspect | Explanation |
| --- | --- |
| Problem addressed | A device, service or subscription needs to be associated with a QoS profile more persistently than a temporary session. |
| Controls | Longer-lived assignment or provisioning of QoS behaviour, depending on the operator’s implementation and service model. |
| Developer takeaway | The key distinction from `quality-on-demand` is persistence. This is closer to configured QoS behaviour. |
| Not meant for | Future time-and-area reservations, multi-device booking or dedicated network environments. |

`qos-provisioning` is useful when the service model requires a more stable QoS configuration rather than a session requested at the moment of use.

### 6.3 `qos-booking` — Planned QoS for one device, time window and service area

| Aspect | Explanation |
| --- | --- |
| Problem addressed | The application knows in advance that connectivity quality will be needed at a specific time and place. |
| Controls | A planned QoS reservation for one device, one QoS profile, one future time window and one service area. |
| Developer takeaway | The essential difference from `quality-on-demand` is planning. The essential difference from `qos-provisioning` is that the reservation is bounded by time and area. |
| Not meant for | Full capacity inquiry, prediction, confidence scoring or multi-device orchestration. |

A booking request may fail if the operator cannot support the requested QoS profile in the requested area and time window. In simple early patterns, the practical model is: request, receive a response, adjust if needed, and retry.

### 6.4 `qos-booking-and-assignment` — Multi-device reservation and assignment

| Aspect | Explanation |
| --- | --- |
| Problem addressed | Several devices need to consume a reserved connectivity capability, and the final device list may not be known at booking time or may change during the event. |
| Controls | A reservation concept plus the assignment and re-assignment of devices to that reservation. |
| Developer takeaway | The value is not simply “more devices”. The value is separating the reservation from the device assignment. |
| Not meant for | Being the default starting point for all CQM use cases or replacing Dedicated Networks when multiple QoS profiles or deeper preparation are required. |

This pattern is more complex than simple booking. If the scenario can be handled through independent bookings per device, the simpler approach may be sufficient.

### 6.5 `dedicated-network` — Advanced dedicated connectivity environment

| Aspect | Explanation |
| --- | --- |
| Problem addressed | A complex scenario needs a richer prepared environment, potentially with multiple QoS profiles, multiple devices and stronger operator-side preparation. |
| Controls | A dedicated connectivity environment that may involve a network profile, one or more QoS profiles, a defined area, a time window and device access management. |
| Developer takeaway | This is an advanced CQM pattern. It may be powerful, but it is not the simplest way to understand or adopt CQM. |
| Not meant for | Lightweight QoS boosts, simple one-device reservations or the mandatory baseline for all CQM use cases. |

Dedicated Networks become relevant when one QoS profile is not enough, or when the scenario requires a prepared connectivity environment with richer operational control.

### 6.6 `qos-profiles` — Discovering available QoS characteristics

| Aspect | Explanation |
| --- | --- |
| Problem addressed | An application cannot request, assign or reserve a QoS profile if it does not know which profiles are available or how they are identified. |
| Controls | It does not control connectivity directly. It exposes information about QoS profiles that may be referenced by other CQM APIs. |
| Developer takeaway | Treat `qos-profiles` as a supporting catalogue. It is similar to learning the available product options before requesting one of them. |
| Not meant for | Activating QoS, reserving QoS, provisioning a device or creating a dedicated network. |

### 6.1 Discovery / support APIs

#### `qos-profiles` — Discovering available QoS profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to know which QoS profiles the CSP makes available before requesting, assigning or reserving connectivity performance. |
| What it exposes | A catalogue of QoS profiles that can be referenced by the other CQM APIs. |
| API consumer takeaway | Treat `qos-profiles` as the discovery layer for QoS profile references. It does not change connectivity behaviour. |
| Not to be confused with | Activating, reserving, provisioning or assigning a QoS treatment. |

#### `dedicated-network-profiles` — Discovering available network profiles

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer needs to know which network profiles the CSP exposes for Dedicated Networks scenarios before booking or assigning devices to a reserved connectivity environment. |
| What it exposes | A catalogue of network profiles, which may describe richer or multi-dimensional connectivity offerings (potentially including multiple QoS profiles, aggregated capacity, device limits and related conditions). |
| API consumer takeaway | This is the discovery API of the Dedicated Networks family. Its role is similar to `qos-profiles`, but for network profiles rather than individual QoS profiles. |
| Not to be confused with | Booking a reserved environment or managing device access. |

### 6.2 On-demand connectivity management

#### `quality-on-demand` — Immediate, time-bounded QoS sessions

| Aspect | Explanation |
| --- | --- |
| Need addressed | The application needs a defined QoS treatment now — for example when a device starts a live uplink. |
| What it controls | An on-demand QoS session for a device or, where supported, a specific application flow, associated with a QoS profile and a defined duration. |
| API consumer takeaway | This is the immediate "apply a QoS profile now, for a bounded duration" tool. |
| Not to be confused with | Future reservations, persistent QoS configuration, multi-device assignment or reserved connectivity environments. |

### 6.3 Longer-lived QoS assignment

#### `qos-provisioning` — Provisioned QoS for a device, service or subscription

| Aspect | Explanation |
| --- | --- |
| Need addressed | A device, service or subscription needs to be associated with a QoS profile persistently, beyond an on-demand session. |
| What it controls | A provisioned QoS session: a longer-lived assignment or configuration of a QoS profile against a device, service or subscription, until changed. |
| API consumer takeaway | The key distinction from `quality-on-demand` is persistence: the QoS treatment is configured, not requested at the moment of use. |
| Not to be confused with | Future time-and-area reservations or reserved connectivity environments. |

### 6.4 Reservation-based connectivity management

#### `qos-booking` — Connectivity booking for a future window and service area

| Aspect | Explanation |
| --- | --- |
| Need addressed | The API consumer knows in advance that defined connectivity performance will be needed at a specific time and place. |
| What it controls | A connectivity booking for a QoS profile, a future time window and a service area, applicable to a device. |
| API consumer takeaway | The essential difference from `quality-on-demand` is planning. The essential difference from `qos-provisioning` is that the commitment is bounded by time and service area. |
| Not to be confused with | Multi-device assignment, network profiles or reserved connectivity environments. |

A booking request may be accepted or rejected based on what the CSP can support in the requested time window and service area. API consumers need clear expectations on acceptance, rejection, reliability and expected system behaviour. Where the CSP exposes them, mechanisms such as capacity pre-checks or confidence indications can support these expectations, but they should only be assumed when the API or the CSP offering explicitly supports them.

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

## 7. Comparative Matrix and Purpose-Oriented Navigation

The CQM APIs are related because they all deal with connectivity quality. However, they differ substantially in scope and complexity.

The CQM APIs share the topic of connectivity quality but differ in scope. The most useful comparison is by capability axes: timing, geography, device model and profile model.

| API | Timing | Area | Device model | QoS/profile model | Complexity |
| --- | --- | --- | --- | --- | --- |
| `qos-profiles` | Not applicable | Not applicable | Not applicable | Catalogue of QoS profiles | Low |
| `quality-on-demand` | Immediate / session duration | Not the primary pattern | Single device or flow | One QoS profile per session | Low / Medium |
| `qos-provisioning` | Longer-lived / persistent | Not the primary pattern | Device, service or subscription | One QoS profile assignment | Low / Medium |
| `qos-booking` | Future time window | Defined service area | One device | One QoS profile per booking | Medium |
| `qos-booking-and-assignment` | Future time window | Defined service area | Multiple devices / assignment | Typically one QoS profile per reservation pattern | High |
| `dedicated-network` and `dedicated-network-accesses` | Reserved environment lifecycle | Defined or prepared service area | Multiple accesses or devices | Multiple QoS profiles / network profile | High |

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
| `dedicated-network-accesses` | Reservation-based (Dedicated Networks) | Aligned with the reserved environment | Inherited from the reserved environment | Device-level access management | Inherited from the reserved environment |

This matrix is intended to make portfolio roles visible. It is **not** a strict API selection guide and does not guarantee CSP support for any specific combination.

---

## 8. Key Takeaways, Guardrails and Sources

### 8.1 Key takeaways

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
The CQM portfolio is best read as a set of API tools for different connectivity-quality needs. The main dimensions that distinguish them are:

- whether the need is immediate, persistent or planned for a future window;
- whether a service area is part of the request;
- whether one device or several devices are involved, and whether device assignment must be managed over time;
- whether one QoS profile is enough, or the scenario benefits from a reserved environment that may expose multiple QoS profiles and related conditions.

Discovery / support APIs help the API consumer see what is available. On-demand and longer-lived QoS tools apply a QoS profile to a device, service or flow. Reservation-based tools commit connectivity quality in advance, with the Dedicated Networks family extending this to reserved connectivity environments with device access management.

### 8.2 Guardrails for external readers

When explaining or using the CQM portfolio, the following guardrails matter:

1. **Do not expose operator topology.**  
API consumers should have only a high level of understanding of a cellular system and its intrinsic behaviour. Concepts like cells, cell-edges and radio carriers are understood on high level. API consumers should not need to have detailed understanding of the network topology, including cells, grids, carriers, radio layers or internal coverage planning.

2. **Do not imply universal support.**  
CAMARA specification and operator availability are not the same thing. API consumers should check which APIs and what capabilities their provider supports. API consumers should not assume that all possible capabilities and features are supported by a CSP. For example, QoS Profile description within the QoS Profile API allows usage of features like DSCP or L4S, which may not be leveraged by the offering CSP.

3. **Do not turn the portfolio into a strict decision tree.**  
The same business scenario may be addressed differently depending on operator capabilities, market maturity and commercial context.

4. **Choose the API tool according to the actual connectivity need.**  
API consumers should not assume that every CQM scenario requires multi-device assignment or a dedicated connectivity environment. Some needs may be addressed through immediate QoS, longer-lived QoS assignment, or a simple planned reservation. Other scenarios may require device assignment or a reserved environment where multiple QoS profiles are available. The relevant API tool depends on the need, the CSP offering and the capabilities available in the target market.

5. **Be clear about expected system behaviour.**  
API consumers need to understand what they can expect from each CQM tool: whether a request can be accepted, why it may be rejected, what happens when the requested connectivity performance cannot be supported, and how changes are communicated. Confidence levels, capacity pre-checks or prediction mechanisms may be useful for some scenarios, but they should only be described as available when they are explicitly supported by the API or the CSP offering.

6. **Keep internal CQM mechanics out of external explanations.**  
1. **Do not assume operator topology.** API consumers work with external abstractions such as QoS profiles, QoS sessions, connectivity bookings, service areas, device assignments and network profiles. They do not need to reason about internal network topology.
2. **Do not assume universal CSP support.** CAMARA specification availability and CSP availability are not the same thing. API consumers should check which APIs and which capabilities their CSP supports. For example, features that may be expressed in a QoS profile description are not necessarily activated in every CSP offering.
3. **Do not turn the portfolio into a strict decision tree.** The same scenario may be addressed differently depending on CSP capabilities, market maturity and commercial context.
4. **Match the API tool to the actual need.** Some needs are addressed by an on-demand QoS session, others by a longer-lived assignment, a simple connectivity booking, a booking with device assignment, or a reserved connectivity environment. The relevant API tool depends on the API consumer need, the CSP offering and market availability.
5. **Be clear on expected system behaviour.** API consumers need clear expectations on acceptance, rejection, reliability and expected system behaviour. Where the CSP exposes mechanisms such as capacity pre-checks or confidence indications, they should be described only when explicitly supported by the API or the CSP offering.

### 8.3 Sources and version notes

This document should be anchored in:

- the Fall25 CAMARA specifications for the in-scope CQM APIs;
- public or shareable CAMARA material related to CQM;
- use-case driven CQM comparison material, especially event, media production, pop-up store and data-boost scenarios;
- the SIM Swap / Device Swap / Tenure / Number Recycling explanatory whitepaper as inspiration for communication style and structure.

- the published CAMARA specifications for the in-scope CQM APIs;
- public CAMARA material related to CQM;
- use-case-driven CQM explanatory material, especially event, media production, pop-up store and data-boost scenarios.

## Appendix (e.g. change log)
