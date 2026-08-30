# ADR-001: Выбор основного системного контекста MVP-платформы мониторинга поголовья

- [1. Контекст](#1-контекст)
- [2. Требования](#2-требования)
  - [Функциональные](#функциональные)
  - [Нефункциональные](#нефункциональные)
- [3. Решение](#3-решение)
  - [Описание](#описание)
  - [Основная контекстная диаграмма](#основная-контекстная-диаграмма)
  - [Аргументация](#аргументация)
    - [Последствия](#последствия)
- [4. Альтернативы](#4-альтернативы)
  - [Дополнительное жизнеспособное переиспользование центральных систем «АгроТех»](#дополнительное-жизнеспособное-переиспользование-центральных-систем-агротех)
  - [Альтернативная контекстная диаграмма](#альтернативная-контекстная-диаграмма)
- [5. Риски](#5-риски)

---

**Статус:** ✅ Принято  
**Участники:** Владелец продукта; архитектор решения  
**Дата:** 2026-08-28

## 1. Контекст

«АгроТех» создаёт отдельную MVP-платформу для мониторинга свиноводческих ферм. Бизнесу необходимо выбрать вариант системного контекста, который показывает, какие существующие системы полезно переиспользовать и какие возможности необходимо добавить в MVP.

Платформа должна работать на нескольких фермах по принципу «центральный сервер — агенты». Каждая ферма должна продолжать локальный мониторинг, управление оборудованием и Локальные уведомления Дежурному сотруднику фермы независимо от доступности интернет-связи. Локальное операционное приложение фермы накапливает Исходящие уведомления и после восстановления связи публикует их в Центральную систему управления фермами. Поэтому ни одна внешняя корпоративная система не может быть обязательной частью локального операционного пути.

Рассмотрены два варианта: минимальное жизнеспособное переиспользование ERP-системы и дополнительное переиспользование центральных Брокера сообщений и Озера данных.

Канонические названия участников, систем и оборудования определены в [Ubiquitous Language](../ubiquitous-language.md).

## 2. Требования

### Функциональные

| Возможность             | Требование к системному контексту                                                                                                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Мониторинг и оповещения | Платформа выявляет нештатные состояния свиней и оборудования, передаёт центральные уведомления Оператору центрального управления и Локальные уведомления Дежурному сотруднику фермы. |
| Управление фермой       | Платформа управляет кормушками и поилками, контролирует состояние систем фильтрации воды, учитывает поголовье и поддерживает видеокамеры.                                                        |
| Корм                    | Платформа получает складские остатки корма и данные о персонале, доступе, аутентификации и авторизации из ERP-системы 1С:Агро; учёт потребления и прогноз расхода остаются ответственностью MVP. |
| Локальная работа        | Локальные приложения ферм работают без интернета, сохраняют данные и Исходящие уведомления, передают Локальные уведомления Дежурному сотруднику фермы и публикуют накопленные данные в Центральную систему управления фермами после восстановления связи. |
| Роли                    | Оператор центрального управления анализирует события, организует пополнение корма и планирует операции через Центральную систему управления фермами. Дежурный сотрудник фермы получает Локальные уведомления, проверяет условия на ферме, оценивает необходимость вмешательства, координирует ремонт и управляет оборудованием. |
| Интеграции              | Платформа публикует базовые и пользовательские метрики для внешних систем и предоставляет API для будущего веб- или мобильного приложения.                                                       |

### Нефункциональные

| Категория      | Требование                                                                                                 | Критичность |
| -------------- | ---------------------------------------------------------------------------------------------------------- | ----------- |
| Доступность    | Достаточно высокая отказоустойчивость — 99,95%.                                                            | Высокая     |
| Оповещения     | От возникновения нештатной ситуации, выявленной видеоаналитикой, до оповещения проходит не более 5 секунд. | Высокая     |
| Видеоаналитика | Реагирует в реальном времени, на уровне миллисекунд.                                                       | Высокая     |
| Связность      | При нестабильном Wi-Fi ферма использует основной и резервный интернет-каналы.                              | Высокая     |
| Расширяемость  | Новый функционал можно добавлять без изменения существующего.                                              | Высокая     |

## 3. Решение

### Описание

Выбран основной вариант с минимальным жизнеспособным переиспользованием существующих систем:

- Платформа мониторинга поголовья MVP содержит Центральную систему управления фермами и Локальные операционные приложения ферм. Она владеет локальным операционным путём: обработкой данных устройств, управлением оборудованием, Локальными уведомлениями, накоплением данных и последующей синхронизацией.
- Оператор центрального управления анализирует события, организует пополнение корма и планирует операции через Центральную систему управления фермами. Дежурный сотрудник фермы получает Локальные уведомления, проверяет условия на ферме, оценивает необходимость вмешательства, координирует ремонт и управляет оборудованием через Локальное операционное приложение фермы.
- ERP-система 1С:Агро используется в режиме чтения как источник данных о персонале, аутентификации, авторизации и складских остатках корма. Платформа владеет учётом потребления и прогнозом расхода корма.
- UWB/RTLS-система, оборудование и устройства фермы, основной и резервный интернет-каналы, а также Внешняя система-потребитель метрик взаимодействуют с MVP согласно основному системному контексту.
- Использование существующих Брокера сообщений и Озера данных откладывается: они не требуются для локальной работы, оповещений в реальном времени или синхронизации MVP.

### Основная контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Платформа мониторинга поголовья MVP — System Context</summary>

![Платформа мониторинга поголовья MVP — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTSjl65Rx7Ks3INhGUCnB_GJV934cHLtOdOpdrXUFHG2GiOKmI303QLZkPiUKaTaSPgqrxqUb5FBxfjI9RaI9RyYiGhz0dwTc37ow01R0KPGAJOcN9v6A_tM_FdlFXO0b8ndrrD9PrUIZsbNloP63UKXwPpO6wDHeg8rFimGz55mJ3Q_PGbyUtrRwvBPPMLwdweVpeXc8OyfQohlvP4OlLLHfrNTbIHlskDZBRsa3JnTzqiOHQjEJU_IrTcupwTZjJbqS6D00QuRQQtbVqS3kgeQxqJ7cqDL0--MI0cAQko9zz9lv54A5iUeFrGftWSHmWVEnhvX1Ef-jWBqKpTyOAcQrjYv8VqfTDULCsva1yDL4cSpI7dwRQKySoh20tIDPrxQ7Jt5ca6XQiePOkKCjceEY6DffZy9u9NSSr9qSCSmViB3zK36sea0xOIxr9bbJMTxo5PlHOOnneCxt0ThDr7Bel6zjAFsZitdtm-iiO2Y4uq7WWgoDx-9Y881POIY54kArDkKe5I-8yPcPcu5Zc3yTIsZmmGjCcOKTTsYmmbcD9X0RwGEQOoM3QDZ4KS91HJzdK7gNk9BPUqJIIj2aXxfFWZdp4YGYc6MDTDPID1VmcR8K5z2Gpb1nfoGmVLsKoGyWH73u-SH5s6SQXDISp5lB9JtOSoLKY9YEMghYTysj8N2yymcJi6AOop8mePOAE8qYPuEAAKIR8E4AKp6MP8fG9o_W4A1kEuufFYCtSmZDN3poYqmCKNFyfUcPq9okC7Ed92XrNVRB2nn4WBZhBrA2isCSdGvbn8bU9M8Jc5gDvExbmOOvk1qJ4nfipyAfKATLRGM-aQaCsTZQBGpPUhPTRDRl-fjrSXucBfIfMhQdobZQI1qxjQeIqfKAPeMcfyt35B7KYZehnyA1wPTSkP11QmbAtYa-xKc8J6VVA8s00iRR1GwmcaQ_GnmgpMcvYKfDyBIW0wN9dxVHaXf2trPCPHkvUppo4BayaM2RoRVXanax-7P-KQGdnkolnV8h5cjHQ2_AXQhdvbDlrRdjrGRL8XNBaxakAFFmxBoaIyUswB4i9M0O9xxgaQ1N6ZilIsIId2pp46bY2nAZQbDA5LBZ7fGkf4mgb2wap2gOBonK9xdGXTJknquMKQ2NTSRBO_Zhln-Wy2d79dzqixpwLJ73X_-ncUNUjBFxPn8UuUIKJ5fhpCvnbtih2zGbCuXqjS_UI95IBtv6GcbWjZ_fbtlsoHD7A3A6vxdv9GxmoOwaBt0MpJ17ZldybKSWkr9FpgGx0J4NyhNC3zvsAzAWYhHHhTy4qgtEMom5K6grskralSG9gdQvJMcKEGB7TfKez4L3iT7r5IWAqku5IYmVKkw5IZ0CKm-sxZJX0ZG7enW5QB40b1j1X0Uen05RxW6KN1Aon0BKOG7FUIRFCsebTwKxI51Pr3SRILPFSUorkrPWHruhhdncYLM-GPf5J1ChrPglLg1K_dCpm0qhLkYHHsbALf5gb60ceExDTQzGgvNAbxjghLM42AipeSI35CA3UZIjhfI2WreqldQ8VKEucbPOVq4e4k5uYS0u0qw013Gv0jqG1rdW0DGh0qzw9LI5AWyEbEI-WbJq0DOVL3W-WJW5g7EqTztDyIKhsZBBVlOkLQcmfBr4rlWwcKwNaV2hFTl9i9yzsyco7Azl1J2IOYmIp4I6KYOHp4Lys8d1a8hviHE39HEXiv6AdW_AyfxGM5y1UHO5x02SR4NWp4JSR4NWp4JTO1TvCn8qbWVSixjefOu3c1ISzlrczXOj8CAJWmfKv0MiXvqi0L4Fk7GzeXGCe5k3Ff-g907-8EY8G0mYAW2qpaG26p5125Ym18QFoALtRrCmix4xbW5G1LRR6TnhiCo0xqR4XPUOfa-TAcGPmN3Bw09LOG3iCgF9SAFC3Eei1Qer6nGMqAwMPkMPl0D1eI8wGIfXbLMVsgz7TrjsNJh8ds1SKBg19Nm5Wrb-Y0LK4h560rgoDWox70p3dzNoaxCbuDA3RbAYpZ_Sc1X1frbyNW5wuhbGFb7OOGBj6DvJwDiAkuNE-UYrGsc7dMq8yN2ocuxTIwWn0bKi3QEMhjAFQqrePkDfcblmCzAi01BEfXCsfO6wMHA39tZgC1BJZ8eK1O5sTTYhpxd4qOm3IlDik7ne0LcS2V8iX-BgN8l9sAGTSsAnPT8rb05Xda3OpuYgsH5Ia4A8jkSTeG7jsE5JQVZKG-3uMc4r6Y12No5Epm3_0R0SY1-I0ImSaRKvb6E15N0AW7Ohf180KKeruG3kaI_MelQPQI9VcJAVov-Bpv-Bpv-B_Nv-Bpn-Cpn-Cpn-Cpn-Cd_T9jr0PAFTCqTH4NTtQDiM-IjH4rUpV1Yr4FsoIfnXvYf6d6B_U52Cs-FCq8qypyZGZJpEYqWnJDGUAE7qvVJizi7QjRwRdrkFfk-c123_EfsVJ4siNgqxWzMjI_HwgNyFtM_W-jBwtziKRV_fS_E_ZVuhh-9Iis0PkC2z1S9uK--ZJJyNfZu0xWcuExEv468YyVMVjVJdwSZJzDnaBXd-C7PzOZwsdS7nNXBSkxZkWS2B2FtXtD3q03bIpCyBi7BaTGirVeGMXll-HICcoR8ZtP7sueK6DJFwg297EsLzfGPuBsW4dynyoEaxcOFhM-fwGWwFM4-ZZ2VHoP7SBSo7GKxUHjJSzWbvn27BWPtSEdgt4wHjekX_1wAe8ndYFWnv21ouOclv4ci71DsYT0tA0aBWgagQa4wWvXqx9prz4x850TgqzDFSJCeBrBQuDWHwxYq6Cw_qY233kG7sW6ARMkxynr4Og65WTRMsC1x9vJzE7OEoHPYgGlev5xLvyMm7yuG3fa3ayWza_0Ji1QP6O3zU5K49wqzT8c3JyZjISemsEOB97jXL8vQ5mvywsQOwDQoihMwgvFTdykASDLnuOXHsvCDP1kjJUPBYoMYpSM6yMcX13jvJUzihcGDjS6SemGtrbigb-DG6Y4wCm07-Po5lAnyE--8LNBLvtgk_U5PHHNoIp3JjOv-RdXO0d_AuuVNLj-ZUSoFdlqODUMK-93sEZGtZz0jwnV0XZ0og9eGw8vv8rXVQxjWT2us_0KgVkWirM7Dtcs1TLnCJGm_IDRMphMx0bZaL2zfIuZaZ-iwE2B7tubu603oZws7alXkR-nUtMoirRrzS31i0P_uCu7FhdkHi6vyxIlhDTim0ltyFNWQCYPzYIhBNrtFPs0dGYv0JNdsq24hZ7n5tWlUrXZsU1OUsZaMnpeeb8duVmSXQFn62a4x2KzSpwEp5Hf3qc3pStT7DWH1e5b94PdxWcJucLaDuljYy0dO1o_k3CrLQIT-WGUrUzfO8kdfFbTEPDfENKMt1l4I16q4mWHBPkkNvnYDFSYvmUwYcSqDGn_b6myEp-XOj029_PIdqYtcx-iO1aqSZED7u4EcSuVrk5N9q_HkmXoYEfzNVdwuUoJ2JVX_9eewUDHkJ2-O5gxZ0O_u1TFuU1zaCHLo16H0iTmQgTO3NOJ4Ixxs9GdU9qNf8J0zZ-6L33QVy4BuxnJB5dhoqk1n7-OtlLtRuygQFxjBMM7N1kvL0nTRLdaEaPaw6YUnud-4zwXG80uwNwcdZb2lSfsN_UiVRTHX5d7Mhkb7aOf-uhLmHAJ7XPlqHgHFuFI6I6939E8AqNR86qNY1fMWrFA2q4ei4elp1RbcuX6upfzqwwyREhTrUT804pO-XYwh9t4G63IGHY-S9pEFGKwRqZwgW_jeDZ73b-RTkLh0rQzXny1sS5NEB7HMe1kcmkfvZO47Ey8G8nNuB4noCaASZaLIXfyY889WkwyioXUeoE5T0-idHC_VFaxWzKdkO6aI-Td9uxoQHedylFq2neQM86qW2jVi8toqIPOTj_KTa3Sly3)

</details>

[Исходный код диаграммы](./01-01-primary.c4.context.puml)

### Аргументация

| Критерий           | Обоснование                                                                                                                       |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| Автономность фермы | Критический локальный путь не зависит от доступности внешних корпоративных систем.                                                |
| Переиспользование  | ERP уже владеет нужными данными о персонале, доступе и складских остатках корма; дублировать их в MVP не требуется.               |
| Граница MVP        | Новые возможности мониторинга, видеоаналитики, локального управления и синхронизации остаются в границе MVP.                      |
| Срок и риск MVP    | В первой версии исключаются дополнительные контракты, доступы и эксплуатационные зависимости от Брокера сообщений и Озера данных. |

#### Последствия

**✅ Положительные:**

- Локальная работа и оповещения сохраняются при недоступности внешних систем.
- ERP переиспользуется без изменения её поведения.
- Граница ответственности MVP остаётся понятной для последующей декомпозиции на контейнеры и сервисы.

**⚠️ Негативные:**

- MVP должна реализовать и эксплуатировать собственный центральный путь публикации метрик.
- Хранение исторических сырых данных в корпоративном Озере данных не входит в первую версию.

**↗️ Зависимости:**

- Контракт ERP для персонала, аутентификации, авторизации и складских остатков корма.
- Поставщик UWB/RTLS-системы, её покрытие, точность и доступность меток.
- Каналы связи и локальная инфраструктура каждой фермы.

## 4. Альтернативы

### Дополнительное жизнеспособное переиспользование центральных систем «АгроТех»

Альтернативный вариант сохраняет независимый локальный операционный путь MVP и ERP-интеграцию, но после успешной синхронизации публикует события и метрики в существующий Брокер сообщений. Брокер передаёт сырые события и телеметрию в существующее Озеро данных и доставляет метрики внешним потребителям.

### Альтернативная контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Платформа мониторинга поголовья MVP — Alternative System Context</summary>

![Платформа мониторинга поголовья MVP — Alternative System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjj65hxdKn3TlTWDujhwfx9f04cMhf954QURYoGmQ9cn2KkYIbA9tKK19sb-2XVnjjkBnLuao5xipTuuJboxJkm0VGBn5VP9zimXAGt98Jco7978acvSQpJVp7VEdFFD8KNQneRQwykwt1LNbRloe6FUK3RDMaTTwtMLdYcsuPkY2u8nwdPVb_itrLLpNSmlB53jNNdpcc8Oyfgof7wZYBdI0eswggmflTMMrZCRMaVJnT-ry0ZqgCljZJLT6_HMxNwcBlSCw00qWdqrVLNHW_seZhhIDkNUMaUvVBc3c9gko5UkY7yLH3XMHfEr0uFW-pX1yBrlcLCuWow1lnJDtEehn5hR5talP5KsvHNPc03nzK0PJD0TlfjgM-tBi8BS85dNjVjETsUHodXW2xLqlbOs0qKtjDu4axTD63ggEtd7CBV0p_9znT2w2Xc0lTGhP4bbVMkqq8mHoumt6imeiCDi2QTUbOrrPTNlxFO6HFrvJ8KGdAZVaTMUFNr4HX0FpEGWHBYzJOLA4OzOEyPkPk1OxW_cKj8yC4EJ9c5dNT8iC9SZIGGc-aZk6CjWqZun50YGNbjPqJOJ3n9RhsWQSTeKa_TnS4S-eaG4ouo-hXhAyXHn4_J257gI6agEjAI6ZwiogI7a20uVdwWCEm_dq9gJ6Wzvv2SzWUGg4PCHIrNSmVar96eK7c4ojWnJwQP6b591nn6aL71nnIWLP1mXYkSoIn5AXMSy0fIEm7759y1cOk6PQ0GUqMa32ix_53qrkfCMHexqf8MEgpvfuUC841UTMMfGMlpZaQ7K196hH2n24ulHf8ES-M3ERYT4o8KtP-85gPYlrFtHIBK6VEnSB0xuUA5Ig9VjzkjsTnqC5_CbR5fKvJMj9tUSreKGQKk4CaFJ4kVXYbdYH1mLu-51ZSfkdSeXj8GbxXMFTYN49ZJlbKrW0Bcsp4EiB96lmCSAurPkObADV4qf0CboPrtfIGsXqQMUr31oh_LCGkZyH8Bb8iy5dzJuoNl59s5QGlJLbMW-kLnPgY_w-L2jt7mAZKghiJ2bMYH2EVJgIu8yl5TU4YJYkUeoAoLWEIHurIL1hp2kkCoSJNovm4EiYeUV6DMQKBcG2FUeSY5nGe5o8N56_d9XjYB1NIuaxYTskP0GhRWxJgQxlixxCJgFGfppPpUpkqyb5LpuFxkPtLqhqtysyJ5kNac5XoRy36UMjx9mVG8JUqVBnAF42TNqToGa9bQpetwUTx_CKBHIGsYYkr-I4A_KUEeCTy7CKiIuxr-95R8pZUHygWCmKn5_xzp0VQieRHQb-Lpv3hXcOSBZV035QhtHg5Nod81oi-KSzG878DTeKKSb5f1hjZo750Tej7n7EHfGQGMEN1GW5-pVgaO1oWn0AmfGPm6A4O0c2r290B3w-pmx9M2H1IX706gJ6iroQpDofPjn9amR6eobAyM5zsBKgZ4phXurFZD5IwD9QZbE4IXKQlLwjPpxU3B33yYNAf94QKj9aih5N2YWubYxM2qN2uLYnVLNlSW459dPuq1oGK2b5NKivls0SYlwQEQyW48hxgXx0VLOW1ib0kS4O0ODg780MdaAiCW3A5C0dlvEhWfX6XmyQfC2wka3K3OiD7a05GfGvkZlX9yJIr9yP1IyuPqhbYAFmWnLurj_EPMFhwUoQYUhTh9g9wjskAeTh4JyjOY_6X42bKYm5l5K8m979UAfHWIUIeIkHiws7HGctTBgN01x5GNk2PngHE2jHDngHE2jHDna5NWh4JUN1DvTtFLJoW2rCnezkLj72nTIO4Z-XIjm0XO3mHS3A0N2EnfG3oPG9C1RJrLY0TuKTKGW0k0N0LjcGW4CcQ64p7_uX8ogftHjHJFJS7Kg0oGAAB4rlbbbxu3iGiU659XR9i-PCWtWE6Nq08gHW4OGKE8vKUO7DAS3bAlLeWje5FDZTurV0A1QbHmXbR3AAetzLwQ7hRW_EiMUO9zGk80QVGQ0LN-U1fGGi4W1wkC-3heG3S2QTnGZXP6CXmDQDOdQVKOl8W2XRlsq0FJ2jQIAxsW40NHej0AbRpKO6fxWgvHzHoCOV3F8XxFbTFIbb0e3KEBI05hvYesm_hHM-iwscKUs0tqI07yr5NDnob-RnH6eiIyTXW8QKPd20B3EJfl5IQznr280qgINNJus0BpE17WMG_0CBuNKxL86EBDRqnWQim2mTf06C-EAjaGKfG2Y8RdlqO36-Et0qV2gWS3tiS3O6254N4AtPe5_Wl4LY0oG0SuT47TnAiM0KSR501g1d8u1I07LY0Oq0hfK2RlML0xeqeJbLFPSVFPSVFPS_F_hS_7PW_7PW_7PW_7PW_6JFkaME3hANLCqDL5NrzPDSLKbQg9gzc-35i8VDibAZAp4o4gCJxV4Y4p-hCp8oeoipCZAZB0omrJDZY8ENmxV3VUiXzPtmrDhU_X-k2V2jzFXoV38UeXDH_3p6zBy0PhVmBzty6_V-i7Q5Q_zvKlnlzi_YxKE6cEgznHn2P-O5Hl04smK1EUfiS---48Sle0n3c38FNje4IObBzzRExTxjtl3Vv9vWSesJd9aRLkFuFs78hnqSK-0pf48_y6hW-4UyA6wdH2MfyXp7rg-Xnx4ZDtFH4gYPKEyA-lTPGrQPF8NHeYCZlzY2_ASqYTepB_9x6ZCtl2TzGCX1-zQ3s2C1p3AWJqis4AWnsudQsTu0AFY5EIDNrmRHhuIXs-Xwsu8euiYEEC3JheF0xXWwFgQT8Cttw9tziWRXCH5aNGbWq3BAGnAllyguWW4yj3QGNS_83DOZt5j2FJGNGpYtD4lXG3dTY0835DhRontjPuA3bPxQylzZcpUrVGkEBkdcGgKidrHknlTLu1us4Cwn8QdOFq3y1EG5edxS5q89QGtV8E4ISSdfEKKVN00nXxONY2D-yAjM-kcsJSkpS-lgURwOENpjjQTlsVCRSbpVHraJ6qFklCBkRbhIxMv6kJJ3QMzFh_IqLRckp9OgCyFLjIl1q1qOCnr85u6yfhoUNTL_6hKB5vrck_S4PJUgaYiPGJOA-iH2K-CX7tu-JcyGikey03Al_rd-3SqxNJuB-Zz-BTtjtkN3VKRvL8kV-NM-97EEvVdIUiLiXRC1pqzVs025adPL0sJE5i6AqsmKdJany-qUVtBEL_C_Y4tV7N9vVL7Z7MMEQnenoo6HhAaUoJ7IFH1_uTsha3dxy0DnsveZMCJ0_pGa_ya6624uLixBAp7iEeu5n6NOnBa8lcVdR_ecC2lC26EqFlEQv5f__17zFun8IuYSyWaLrc6lqPR_uGj0SymMVViuLvREyHmI3bYrtiYP6ZiSIZbhlqdjPPNT6q3IYWsxHVWBP8ZfqGkI6901bYFG-aUaDXxxEOGYk_t68w0sYP8pp0XsXFAcDHronrvGn5Xu_5J_UfcVVxwZQjBlZ13gdyd0eHwTUhAugcJwkzjgPg37p_0rvwpgvnWJvBxu6PxaGv6Ydc4Ui0ED2BaXqG-uBLjs_POAArT34KxQ34GoPZxyEDOdydoaK4W7gsdreya44CTChY_igoR7OCAlS3wCVKQ8_yHyHBI-zLUH19Btftq9yTMUsTvZsctSt6K4334Cv8qZjramOz7QKKB4Be9GjSEOZVxzj7CdL3pS7-5OaVjun-C2rht3rm4GlZ4thcFn9krFyyXMNIoOyOBe7E2zjkxahllsx6zZzibQVKEvnc7yamet_joRxcjzNhaeiezrTnYCFu9Xtu64-q6T6sEE14zT02hTeJDu3DFDgCvBqcX0Bv_2jJGxDVmmo5M3Zlssk9o4Aauj5VT7MkqzT5ZsdilMzRSngvYwchR8EOPWwwYZo9Ey1P1StC0ZDwwBuaNBd2NQDuwnDfrEuLK8PJjb7iOfTo52m8bsRoiNo8rKWxiaSmCR4HHsv3rdBqDMS-HDBtd754x3X3rP_cPsR9q2zbWJdzmoizVNBsxw2G9k1bJ5qlPdRC861ZXo-Kpsx0_KgGFZgYZ_jW1ZddanBNjLx8swDbJY1sq2hX4pulKV7JPN8wnq2NkU4i4OhA2cOz7G5AGoQj04JtA830MTEMfG_KG0ykdVMJfcFetahi_KNMxcqIUqmjI3-39XMXNJ2Ve5lGqSGFfW5u_uhFILuhOteQ1NcEK7JlQm3Xte7TN-vo7BeQSOFH87H3opOOCngK5SwdT0eOv0Jlst5h1YRvjwjm3J_N6TT9OabZCOYKO2V_i2WWjazDk2y3eVm00)

</details>

[Исходный код диаграммы](./01-02-alternative.c4.context.puml)

| Вариант                                                           | Плюсы                                                                                                            | Минусы                                                                                                                            | Почему отклонён                                                                                                         |
| ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Дополнительное переиспользование Брокера сообщений и Озера данных | Использует существующие центральные возможности публикации событий, метрик и хранения исторических сырых данных. | Требует проверки контрактов, прав доступа, изоляции, хранения, резервного копирования, поддержки интеграций и общей эксплуатации. | Дополнительные зависимости не улучшают обязательную локальную работу фермы и увеличивают объём согласований и риск MVP. |

## 5. Риски

1. **Недоступность внешних систем или WAN-каналов**  
   _Меры:_ локально сохранять данные и Исходящие уведомления, передавать Локальные уведомления Дежурному сотруднику фермы, использовать основной и резервный интернет-каналы и синхронизировать накопленные данные с Центральной системой управления фермами после восстановления связи.

2. **Неполные, недоступные или несовместимые данные ERP**  
   _Меры:_ согласовать контракт только на чтение, проверить состав и актуальность данных о персонале, доступе и остатках корма до запуска MVP.

3. **Неточность UWB/RTLS и видеоаналитики**  
   _Меры:_ проверить покрытие, доступность меток, точность позиционирования, ночную съёмку, освещение, слепые зоны и качество модели на репрезентативных данных ферм.

4. **Потеря накопленных локальных данных при восстановлении связи**  
   _Меры:_ применять надёжное локальное хранение, подтверждение доставки, идемпотентную синхронизацию и удалять либо помечать данные как доставленные только после успешной передачи.
