# CAMARA CQM APIs — Understanding Connectivity Quality Management

## 1. Executive Summary

Most applications consume mobile connectivity as best-effort connectivity: the network tries to deliver the best possible performance at any given moment, but the application does not explicitly request a specific quality level from the operator. In practice, that performance can vary significantly depending on factors such as network deployment, coverage conditions, terrain, indoor or outdoor location, mobility, and current network usage.
Most applications consume mobile connectivity as best-effort: the network tries to deliver the best possible performance at any moment, but the application does not request a specific connectivity quality. For some, an API consumer (aka Application Service Provider (ASP)) needs more predictable connectivity quality for a specific application, device, user category or enterprise context — for example a live video uplink during an event, payment terminals in a pop-up store, several cameras and intercoms in the same venue, or temporary enterprise connectivity at a defined site.
The CAMARA Connectivity Quality Management (CQM) portfolio addresses their needs through several API tools and categories. These tools are not identical and should not be treated as interchangeable. They differ by **when** the connectivity quality is needed, **where** it applies, **how many devices** are involved, and whether the service requires a simple/single QoS profile or a richer dedicated connectivity environment.

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
This document explains the purpose and differences of these tools, without presenting them as a mandatory implementation sequence or as a universal decision tree. Actual API availability and supported capabilities may vary by CSP and market.

## 2. Introduction — Why Connectivity Quality Management Matters

Mobile connectivity is today mostly consumed as **best-effort connectivity**. This means that the network tries to provide the best possible performance at any given moment, but the actual experience may vary depending on many factors, such as network deployment, coverage conditions, terrain, indoor or outdoor location, mobility, and current network usage.

For many digital services, best-effort connectivity is sufficient. Users of many standard applications like messaging, browsing, background synchronisation can normally tolerate variable throughput, latency or varying radio conditions.

Other services, however, may need a more predictable connectivity performance under specific conditions. A live video application may need stable uplink performance during a contribution window. A point-of-sale terminal may need reliable connectivity during the opening hours of a temporary store. A media production team may need several devices to operate in the same venue. An enterprise may need a specific connectivity behaviour for a limited period or within a defined location.

Connectivity Quality Management (CQM) addresses these needs by exposing API-based tools that allow application providers to discover, request, configure, reserve or manage connectivity quality in a standardised way.

The key concept is, that API consumers and application developers do not need to have detailed understanding of mobile network realizations. API consumers work with external-facing abstractions of network capabilities exposed by the operator, such as QoS profiles, QoS sessions, bookings, service areas, device assignments and dedicated connectivity environments.

In simple terms, CQM helps API consumers understand and request **well-defined connectivity performance** when ordinary best-effort behaviour is not enough for a specific service, moment, location or group of devices.

## 3. CQM Concepts in Plain Language

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
 | Discovery of capabilities | The concept of discovery allows API consumer to retrieve information about offered capabilities and usable combinations | The API consumer only needs to know abstracted information in an understandable format |

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
The table below uses the live-event context only as an illustrative frame. It does **not** imply that the APIs must be used in sequence, that all APIs are used together, or that every operator supports every pattern. Each API tool addresses a different type of connectivity-quality need.

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
| Use several QoS profiles within a reserved connectivity environment | Dedicated connectivity environment, mainly `dedicated-network` with `dedicated-network-accesses` and `dedicated-network-profiles` where applicable | The ASP may need multiple QoS profiles for different devices, traffic types or production roles within the same broader connectivity environment. |

## 5. Portfolio Grouping

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
It should be read as an explanation of the portfolio structure and the main conceptual differences between the APIs. Specification availability and operator availability are not the same thing; API consumers should always check what their provider supports.

## 6. API Deep Dives

This section explains each API at product and concept level. It intentionally avoids endpoint-by-endpoint or schema-by-schema detail.

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

`qos-profiles` is the vocabulary layer of the portfolio. It supports the other APIs but is not the final service action.

## 7. Comparative Matrix and Portfolio Relationships

The CQM APIs are related because they all deal with connectivity quality. However, they differ substantially in scope and complexity.

The most useful comparison is not by endpoints or schemas, but by capability axes: time, geography, device model, profile model and operational complexity.

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

This matrix is intended to explain portfolio roles. It is **not** a strict API selection guide and should not be read as a guarantee of operator support.

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
- `dedicated-network` is the advanced dedicated-connectivity pattern.

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
External readers do not need WG harmonisation plans, Q1/Q3 options, governance topics or potential future API consolidation discussions to understand the API portfolio.

### 8.3 Sources and version notes

This document should be anchored in:

- the Fall25 CAMARA specifications for the in-scope CQM APIs;
- public or shareable CAMARA material related to CQM;
- use-case driven CQM comparison material, especially event, media production, pop-up store and data-boost scenarios;
- the SIM Swap / Device Swap / Tenure / Number Recycling explanatory whitepaper as inspiration for communication style and structure.

Internal CQM consolidation notes may be used as preparation material, but should not be exposed as visible external content unless the document is explicitly marked as internal.

## Appendix (e.g. change log)
