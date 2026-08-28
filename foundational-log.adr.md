# Architecture Decision Records Log

- [Шаблон ADR для исследования системного контекста](#шаблон-adr-для-исследования-системного-контекста)
- [ADR-000: Establish Ubiquitous Language as the Authoritative Domain Vocabulary](#adr-000-establish-ubiquitous-language-as-the-authoritative-domain-vocabulary)
  - [Context](#context)
  - [Decision](#decision)
  - [Alternatives](#alternatives)
  - [Consequences](#consequences)
- [ADR-001: Name the Farm-local Software Farm Edge Application](#adr-001-name-the-farm-local-software-farm-edge-application)
  - [Context](#context-1)
  - [Decision](#decision-1)
  - [Rationale](#rationale)
  - [Alternatives](#alternatives-1)
  - [Consequences](#consequences-1)
- [ADR-002: Choose the MVP Integration and Reuse Strategy for the Livestock Monitoring Platform](#adr-002-choose-the-mvp-integration-and-reuse-strategy-for-the-livestock-monitoring-platform)
  - [Context](#context-2)
  - [Decision](#decision-2)
  - [Alternatives](#alternatives-2)
  - [Consequences](#consequences-2)
- [ADR-003: Reuse ERP Data for Feed Stock and Farm Personnel](#adr-003-reuse-erp-data-for-feed-stock-and-farm-personnel)
  - [Context](#context-3)
  - [Decision](#decision-3)
  - [Rationale](#rationale-1)
  - [Alternatives](#alternatives-3)
  - [Consequences](#consequences-3)
- [ADR-005: Choose the MVP Data Lake Strategy](#adr-005-choose-the-mvp-data-lake-strategy)
  - [Context](#context-4)
  - [Decision](#decision-4)
  - [Alternatives](#alternatives-4)
  - [Consequences](#consequences-4)
- [ADR-004: Keep Farm-local Operations Independent of the Existing IoT Platform](#adr-004-keep-farm-local-operations-independent-of-the-existing-iot-platform)
  - [Context](#context-5)
  - [Decision](#decision-5)
  - [Proof of Concept](#proof-of-concept)
  - [Alternatives](#alternatives-5)
  - [Consequences](#consequences-5)
- [ADR-006: Choose the MVP Data Warehouse Strategy](#adr-006-choose-the-mvp-data-warehouse-strategy)
  - [Context](#context-6)
  - [Decision](#decision-6)
  - [Alternatives](#alternatives-6)
  - [Consequences](#consequences-6)
- [ADR-007: Reject Microphone-Based Behavior Analysis for the MVP](#adr-007-reject-microphone-based-behavior-analysis-for-the-mvp)
  - [Context](#context-7)
  - [Decision](#decision-7)
  - [Rationale](#rationale-2)
  - [Consequences](#consequences-7)
- [ADR-008: Use UWB for Livestock Identity, Position, Movement, and Counting](#adr-008-use-uwb-for-livestock-identity-position-movement-and-counting)
  - [Context](#context-8)
  - [Decision](#decision-8)
  - [Alternatives](#alternatives-7)
  - [Consequences](#consequences-8)
- [ADR-009: Use Video Analysis for Visual Livestock Indicators](#adr-009-use-video-analysis-for-visual-livestock-indicators)
  - [Context](#context-9)
  - [Decision](#decision-9)
  - [Alternatives](#alternatives-8)
  - [Consequences](#consequences-9)

---

## Шаблон ADR для исследования системного контекста

На этапе исследования системного контекста используется облегчённый расширенный шаблон ADR на основе подхода Майкла Найгарда:

- статус, участники и дата;
- контекст: известные факты, ограничения и неопределённости;
- решение: граница системы, вариант переиспользования или архитектурное допущение;
- альтернативы: только реалистичные варианты уровня системного контекста;
- последствия: влияние на C4 Level 1, допущения и последующие решения.

При необходимости добавляются разделы «Обоснование» и «Проверка гипотезы / PoC».

Полный шаблон ADR из учебных материалов не используется на этом этапе, потому что он предназначен для детально проработанных архитектурных решений. Он включает функциональные и нефункциональные требования, Use Cases или FURPS+, описание реализации, диаграммы, детальное сравнение альтернатив, риски и меры по их снижению.

Цель исследования системного контекста — определить границы целевой системы, внешних пользователей и систем, возможные интеграции и варианты переиспользования существующих возможностей. Детали контейнеров, технологий, реализации и полного покрытия нефункциональных требований на этом этапе ещё не определены. Поэтому облегчённый ADR сохраняет историю значимых решений и их альтернатив, но не дублирует требования, Use Cases и диаграммы. Полный шаблон следует применять для последующих решений уровня контейнеров, технологий и конкретных интеграций.

## ADR-000: Establish Ubiquitous Language as the Authoritative Domain Vocabulary

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-26

### Context

The course evidence uses terms that are ambiguous or undefined for the target architecture. Architecture and modelling artefacts require one canonical vocabulary.

### Decision

Use [Ubiquitous Language](ubiquitous-language.md) as the source of truth for canonical domain terms, not as an explanatory glossary. Use its Russian/English canonical pairs in all new artefacts. Keep original evidence terms only as parenthesized references.

### Alternatives

- Use the course evidence terms directly in every artefact.
- Maintain a non-authoritative glossary alongside independent terminology in each artefact.

### Consequences

- Change a term in Ubiquitous Language before aligning dependent artefacts.
- Do not introduce competing synonyms in a separate glossary.
- Record architecture-significant terminology choices in separate ADRs.

---

## ADR-001: Name the Farm-local Software Farm Edge Application

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-26

### Context

The evidence calls farm-local software `агент фермы` but does not define its architecture or responsibilities. The term `agent` is ambiguous because it is also widely used for AI agents. The evidence does require farm-local operation during disconnection, local alerts, equipment control, and synchronisation with the central platform.

### Decision

Use **Периферийное приложение фермы / Farm Edge Application** as the canonical name for the farm-local software. Preserve `(агент фермы)` only as the original evidence reference.

### Rationale

The selected term describes deployable software running near farm equipment without asserting an unsupported gateway role. Its conventional meaning fits the required local processing and disconnected operation; [AWS](https://docs.aws.amazon.com/solutions/tactical-edge-application-deployment-on-aws/) and [Azure](https://learn.microsoft.com/en-us/azure/iot/iot-introduction) use `edge application` for locally deployed software that processes data near devices and can operate with limited connectivity.

### Alternatives

- **Agent:** rejected because it is ambiguous with AI-agent terminology and the evidence does not define its software role.
- **Farm Edge Gateway:** deferred as TBC because it conventionally implies an integration and connectivity role that the evidence does not establish.

### Consequences

- Use **Периферийное приложение фермы / Farm Edge Application** in new diagrams and ADRs.
- Keep **Периферийный шлюз фермы / Farm Edge Gateway** as a separate TBC concept.
- Update dependent artefacts after changing the canonical term in Ubiquitous Language.

---

## ADR-002: Choose the MVP Integration and Reuse Strategy for the Livestock Monitoring Platform

**Status:** ⚠️ Under review  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-26

### Context

Task 1 requires a primary and an alternative C1 solution that show which existing AgroTech systems can be useful and which systems must be added. The Livestock Monitoring Platform remains the system scope for both variants.

### Decision

Pending comparison of two variants:

- **Reuse as-is:** integrate existing AgroTech systems through their current interfaces and capabilities.
- **Extension-led reuse:** use only additive extensions that preserve existing behavior; implement uncovered livestock functionality in the MVP.

### Alternatives

- **Reuse as-is:** lower change risk in existing systems, but may leave more livestock functionality to the MVP.
- **Extension-led reuse:** may reuse more existing capabilities, but requires proof that additive extensions preserve existing behavior.

### Consequences

- Prepare one C1 diagram for each variant and record the comparison in the Task 1 ADR.
- Treat ERP, Kafka, the IoT Platform, the existing web portal and mobile application, Data Lake, Data Warehouse, Analytics Module, and BI System as TBC reuse candidates.
- ADR-003 records the accepted ERP reuse decision.

---

## ADR-003: Reuse ERP Data for Feed Stock and Farm Personnel

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-26

### Context

The existing ERP System (1С:Агро) is responsible for finance, warehouse, and HR. The existing web portal synchronises with it through REST. The MVP must track feed stock and forecast consumption; it also requires roles and authentication.

### Decision

Reuse the existing ERP System as a read-only source of farm-personnel data, authentication, authorisation, and feed-stock data through its current REST integration capability. Do not modify ERP behavior.

The Central Monitoring Platform owns livestock feed-consumption tracking and forecasting. ERP remains the source of personnel, authentication, authorisation, and warehouse data.

### Rationale

This reuse avoids duplicating the infrastructure already used by the company's existing pig farms while preserving the ERP System's current behavior. It does not assume that ERP implements livestock-specific forecasting.

### Alternatives

- Duplicate feed-stock and personnel reference data in the Livestock Monitoring Platform.
- Modify the ERP System to add livestock-specific functionality.
- Do not use ERP personnel data; maintain application users independently.

### Consequences

- Implement a read-only ERP integration for personnel, authentication, authorisation, and feed-stock data.
- Keep livestock feed-consumption tracking and forecasting in the Central Monitoring Platform.

---

## ADR-005: Choose the MVP Data Lake Strategy

**Status:** ⚠️ Under review  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-27

### Context

The [Ubiquitous Language](ubiquitous-language.md) defines **Озеро данных / Data Lake** as the existing MinIO S3-compatible raw-data store. The evidence identifies it as an existing corporate capability, but does not require it for the livestock MVP.

The MVP requires local operation, offline work, real-time alerts, and synchronization. Historical data storage must not become a dependency of the local operational path.

### Decision

Prefer reusing the existing Data Lake for synchronized raw events, telemetry, and historical livestock-monitoring data, subject to validation of access, isolation, retention, backup, and integration requirements.

### Alternatives

- **Create a separate MVP Data Lake:** use when the existing store cannot provide sufficient tenant isolation, retention, backup, ownership, or integration control. This provides greater independence but adds deployment and operational work.
- **Defer the Data Lake:** use when the MVP needs only local operation, real-time alerts, and synchronization, and historical analysis is not yet required. This reduces initial scope but postpones historical storage and analytics integration.

### Consequences

Keep the Data Lake outside the local real-time and offline-critical paths. Store data locally first when disconnected, then synchronize it to the preferred or selected Data Lake after connectivity is restored.

---

## ADR-004: Keep Farm-local Operations Independent of the Existing IoT Platform

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-28

### Context

The evidence requires every farm to continue operating, controlling equipment, buffering data, and notifying local staff when connectivity to external systems is unavailable. Synchronization with the Central Management System occurs after connectivity is restored.

The existing AgroTech IoT Platform is external to the MVP. The evidence does not establish that it can satisfy the required independent farm-local operational path. Making local operation depend on it would create a shared external dependency as farms are added.

### Decision

Do not reuse the existing external IoT Platform on the farm-local operational path.

Implement farm-local device integration, telemetry collection, equipment control, local buffering, local notifications, and synchronization through the Local Edge/IoT Gateway and Farm Local Operations Application. Each farm operates independently when connectivity to external systems is unavailable.

The existing IoT Platform may be considered later only as a consumer of synchronized, non-critical data through a separately recorded central integration decision.

### Alternatives

- **Reuse the existing IoT Platform:** rejected because it is external to the MVP and cannot be part of the required independent farm-local operational path.
- **Build a complete new IoT platform:** rejected because it adds substantially more deployment, operational, and long-term maintenance work than the required livestock functionality justifies.

### Consequences

Each farm has an independent Farm Local System that continues operating when connectivity to external systems is unavailable. Add a farm by deploying and configuring its local systems, without adding a dependency on the existing external IoT Platform.

Keep external systems outside the local device-control, local-notification, UWB/RTLS, and real-time video-alert paths. The Local Edge/IoT Gateway owns local device integration, telemetry normalization, equipment control, buffering, and synchronization.

---

## ADR-006: Choose the MVP Data Warehouse Strategy

**Status:** ⚠️ Under review  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-27

### Context

The [Ubiquitous Language](ubiquitous-language.md) defines **Хранилище данных / Data Warehouse** as the existing ClickHouse structured-data store. The evidence identifies it as a possible reuse candidate, but does not require it for the livestock MVP.

The MVP requires current operational views, synchronized livestock data, and basic reporting. The evidence does not specify data volumes, retention, analytical complexity, reporting load, or mandatory BI integration that would require a separate Data Warehouse.

### Decision

Prefer use the Central Management System’s operational database with simple reporting read models. Keep the Data Warehouse as an optional later integration for historical analysis and reporting.

### Alternatives

- **Reuse the existing Data Warehouse:** use when the MVP requires historical or complex reporting and the existing ClickHouse store provides acceptable access, isolation, retention, backup, and integration. This avoids new infrastructure but creates an additional integration dependency.
- **Create a separate MVP Data Warehouse:** use when stronger ownership, isolation, scale, or workload separation is required. This provides greater independence but adds deployment and operational work.

### Consequences

Keep reporting queries isolated from operational transactions through simple read models. Do not make the Data Warehouse a dependency of local operation, offline work, synchronization, or real-time alerts. Reconsider the decision when reporting volume, retention, analytical complexity, or BI requirements are defined.

---

## ADR-007: Reject Microphone-Based Behavior Analysis for the MVP

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-27

### Context

The forum evidence proposes microphones as a possible separate input for detecting audio indicators of livestock distress. The formal requirements do not require audio analysis. The existing partner ML model is defined for video-stream processing and does not provide audio analysis.

### Decision

Reject microphone-based behavior analysis for the MVP. Keep the Video Analytics System and the partner ML model focused on video processing and use the other agreed livestock inputs.

### Rationale

Adding microphones would require a separate audio-processing capability, including audio feature extraction, model support, validation, and event correlation with video and livestock telemetry. This is outside the confirmed MVP capability of the existing ML model and would add unjustified complexity.

### Consequences

- Keep microphones outside the active MVP context diagram.
- Record the microphone concept as `[REJECTED]` in the UL.
- Reconsider audio analysis only if a suitable audio-capable model and explicit requirements are introduced.

---

## ADR-008: Use UWB for Livestock Identity, Position, Movement, and Counting

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-27

### Context

The MVP needs to work with individual livestock identity and movement data. In this context, active UWB ear tags and fixed UWB anchors/receivers provide:

- individual livestock identification through tag IDs;
- position/location data within the anchor coverage area;
- movement data derived from changing positions and tag transmissions;
- counting of animals with functioning tags detected within the coverage area.

UWB can provide counting if every livestock animal carries an active UWB tag. The system counts distinct tag IDs currently detected in the covered area. This is an identity-based count, not a count of untagged animals. Missing tags, failed transmissions, or animals outside anchor coverage can make the count wrong. UWB livestock research supports identity, position, and movement tracking, but does not establish unconditional counting reliability. [Microsoft livestock UWB study](https://www.microsoft.com/en-us/research/wp-content/uploads/2015/10/baszun.pdf)

The canonical definition of the UWB/RTLS System, including its composition, provider installation and configuration, and local integration boundary, is maintained in the [Ubiquitous Language](ubiquitous-language.md).

> [!WARNING]
> The evidence says that some complete livestock products cannot be bought or adapted because of legal constraints or implementation cost. That does not automatically rule out an RTLS component, but provider and product availability, legal suitability, and total cost remain explicit selection criteria.

### Decision

Use an off-the-shelf, locally deployed UWB/RTLS System in each Farm Local System, with active UWB ear tags and fixed UWB anchors/receivers, as the primary source for livestock identity, position, movement trajectories, activity indicators, zone transitions, group distribution, and counting of functioning tagged animals within the coverage area.

Use UWB-derived movement data for movement-pattern detection, including speed, distance, prolonged immobility, repeated rapid movement, and unusual transitions between zones. Treat all UWB-derived results as valid only within the limits of tag operation, anchor coverage, positioning accuracy, and data quality.

UWB does not replace visual analysis. It does not determine visual appearance, posture, fighting, wounds, or other visual livestock-condition indicators.

### Alternatives

- **Reuse the existing IoT Platform for UWB/RTLS:** rejected by ADR-004 because the required local positioning and monitoring path must remain independent of external systems.
- **Use video analysis for counting:** rejected as the primary counting method because occlusion, overlap, lighting, camera placement, and dirt can reduce reliability. Video may remain a secondary estimate or validation signal.
- **Use RFID for identity and counting:** not selected because the current design already uses UWB for identity, position, and movement, and the evidence does not require an additional RFID capability.
- **Use manual counting:** possible as an operational fallback, but does not provide continuous automated monitoring.

### Consequences

- Make UWB the primary MVP source for identity, location, trajectories, movement patterns, and tagged-animal counting.
- Faster MVP delivery: the provider supplies, installs, and configures an existing UWB/RTLS solution instead of the MVP team building that specialised capability.
- Additional cost and supplier dependency: procurement, installation, calibration, support, licensing, maintenance, and replacement terms must be agreed with the provider.
- The UWB/RTLS System continues to provide local identity, position, movement, and tagged-animal-counting results when connectivity to external systems is unavailable.
- Do not assign movement tracking to video when valid UWB data is available.
- Keep UWB-derived movement patterns separate from visual behavior and appearance indicators.
- Do not treat raw UWB coordinates as behavior classification without a suitable interpretation model.
- Record coverage, tag availability, positioning accuracy, and data-quality limitations with UWB-derived results.

---

## ADR-009: Use Video Analysis for Visual Livestock Indicators

**Status:** ✅ Accepted  
**Participants:** Project owner; architecture modeller  
**Date:** 2026-08-27

### Context

The formal requirements require assessment of livestock condition from appearance and behavior, including illness, death, unrest, and fighting. The existing partner ML model processes video.

Real livestock IoT practice supports video analysis for visible group behavior and appearance indicators, but its reliability depends on the farm, camera placement, occlusion, lighting, and model validation. Published pig studies demonstrate useful counting and behavior classification, but also report degraded performance in crowded, occluded, or poorly lit conditions.

### Decision

Use the Video Analytics System and partner ML model for visual livestock indicators that UWB cannot provide: visible abnormal appearance, posture, fighting, wounds, and other visually observable signs that can generate an operator alert.

Do not use video+ML as the primary source for movement tracking, movement trajectories, zone transitions, movement-pattern analysis, or tagged-animal counting. Its reliability is too low and too environment-dependent for the MVP’s expected continuous tracking and analysis. When valid UWB data is available, use UWB for these responsibilities. Video may provide supplementary visual context for movement-related alerts.

Treat ML results as detected visual indicators and alerts, not veterinary diagnoses or guaranteed classifications.

### Alternatives

- **Use video+ML for movement tracking and movement-pattern analysis:** reject because video tracking is affected by occlusion, overlap, lighting, camera placement, dirt, and animal similarity, resulting in insufficient reliability for the MVP’s expected continuous tracking and analysis. Retain video for visual appearance and behavior indicators that UWB cannot observe.
- **Reject video analysis entirely:** not selected because the evidence explicitly requires visual appearance and behavior assessment, for which video is the available MVP input.

### Consequences

- Keep video cameras and the partner ML model in the MVP context.
- Treat ML results as detected visual indicators and alerts, not veterinary diagnoses or guaranteed classifications.
- Validate precision, recall, false-alert rate, coverage, lighting, occlusion, and latency on representative farm data before relying on alerts operationally.
