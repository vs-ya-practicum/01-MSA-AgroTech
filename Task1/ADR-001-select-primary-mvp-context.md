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
| Локальная работа        | Локальные приложения ферм работают без интернета, сохраняют данные и Исходящие уведомления в Локальном операционном хранилище данных, передают Локальные уведомления Дежурному сотруднику фермы и публикуют накопленные данные в Центральную систему управления фермами после восстановления связи. |
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
- Платформа включает локальную UWB/RTLS-возможность для идентификации, позиционирования, анализа движения и подсчёта поголовья. Оборудование и устройства фермы, основной и резервный интернет-каналы, а также Внешняя система-потребитель метрик взаимодействуют с MVP согласно основному системному контексту.
- Использование существующих Брокера сообщений и Озера данных откладывается: они не требуются для локальной работы, оповещений в реальном времени или синхронизации MVP.

### Основная контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Платформа мониторинга поголовья MVP — System Context</summary>

![Платформа мониторинга поголовья MVP — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjj65hxdKn3TlGhYslgdicq1IPQk6g1ekhiNQM3G4cCJaKYLefAuYmA9qwIxwo3P5BtQnIR8pLuxZjKeTkA-0laA-oHxPZYaICwG75gEI6GvWHrxED_CDsVE-UPeH5gJMwesbWrv90wKc_9qQ7wZt3LRGtL76ocQAVRXcs88miHhTiUGntzM1-QkMDxOyDMFvBjNbSb4tb6kgTyhOgcsuKTzgUmesg2dQsP77-g6-BiUBbIBjjo_jMFeKstWj3CDMPj00w11jzMDWMBGxNmD3QLlojhEKFdiio76j0n5_k8ByI-226NR6wnFTOAlum7eQpymXo2TNWDxARgvDrRGR1rRb8EGWMpAs_8a1UAxgJ9DqHo-csfV7SkmWjmWsJ3qEwGvMQGwBhZ6jtIXMZO3nPZeMeh1-oPq7TSSNPcOUs1d-OuoqKSAwe2zrDjeIMLZprjeHezrneKEqmiSDxk6Xnx8arrb43Ps_nPu_KKCXI5ueF5GLZLd-9Y8G1POIK54kAtjSvKgBedpM9YPWMEPduwbh7dW2CsQX1DrMRF0iHnBWXhe0vbZ8ODPssIYW8DeVMLRlvkvapXwvQUHf4q9SPy49_8H9o8upHWRwaJPMi9lQ2iieIUveKIa9JTyN9N93I4YE7nyuYBi8epZrvpSM2WWF_bn95U9c8nOgk9spgyXSRtm2DDaRs8gezo8KYxe447A1HTNZ791XWXHCfTLYb0kB1CGe7muZYi-59lKmfEg1nxHwGCANlyfHcvq9o-CYFJaXOwhFddXGmI8YyugDIWlzWd8K6wSo5KY5g7KOfIsatCNvkXEG4GSl961DwHgkT4EUwElbh8nEIocRBpHgBJhJltNJdC39YwMQxXgKvLtT4qUajgD26dBX393qpBduOfPvaI8Yl7myFNAhbtA87u9ozmg0UtAY4rat2jtWG74sXOFiRg4_b5lAongkOb9BVHlIG78bZlhf2StXBoZdjmmSizwqX2wE95WcIXmu9CREmLFV3Ac9SIVhiJpAPNgKdippCTNoysdqcdqEXjBgaKcb2DFNpBa4JnvoP18uDHbLKh0CWXzwfAXLHWdBYjdavmky11huX8cvglDA5t8X7jSkf0v8IfTo9nHE5rOhKXmfmkPsucTBcH4A-cEa-NErtdVHkTHY0j-xsPrzwdaWWl_UpUhksjb-VScFiJDAxcmKChtS5PvAmlNEp29TxIaxYL9g9Q_8o4piLeTzOky-sM5efKRGgdkVib2l79ZgNFS1RDA4UE-VoLHoCxL4tbN1s2c8lwUtC1zlIhThKhhfVeDCCr6od8nW6gptUcq6cLEGBtR8wLTuG2KEZrVQIG2IjrUe4X9W4ulLEhnW4QFAgKuG8bktsl60UeCG2yEq6O1gZ60BWlGY06mseSikoHWamMenm1QQIVDCcitTgMxILDOrZKOIrTBSk_Dk5LZHbmxhdzcYDQyGLebJX6eD5hjThDU-d0ommyerngIvDEMcYJLgwL8G8FCThDPhrOgrOPhhtQL2QWoeuT0IZIWqOihc-KmeDwBBzrI453f9PLs4D1E1BXU8d0E0DFm0veSW5xP1zZa0THz09xs95Q5A0scIoije9q_W6yE6rqUGCC7g7Eq9-v7V4bAzenAqBrBrLfigQnGDNu8frFbv7ogo7QARAV8TefiXolRmPb8E1S9PoC2bOdGkKWW6n4uCf50DYBmP2B-REHysq4bxPRMvW8ugoXm3q2o4O4t4t6p4O4t4t63LU3DHDnO4dXtSTTEEGEqpZdfz6Rr5YuYmP322rVX16nIpfS0g57k7Gzeqm4K2mZcKur4G314YGZ40C8YuCXC980XCpuX2vUGaFdoAKFVriqyd4uLW4m1DRR6Tvli7P2Tw3YG2dFBv7cbx0Tml6GC0AgnW0uDgF6yKEO7T9S3r9lDgWleLCiBSor-0K2pAH4XbN2MLLlOh-xljk7-I98zmNb1uG9Q_bS0EEil-m4r3DZq0TgBDWHTYGVWdDVpaOeduz60NaloxJxUBp60IBD-N03_mlMaHgXqQ83VDNfKwjkaNIFWV8rwg7Hevrj1F9mlfkEFKXeCG8rB0_pAL-r4jVThPUZLDhCKEz170GXdKmc7K-7SB8b0Ytrq60den4KA0y1wTTgjfZtZQCK0fBJ7BXyQ05Pd0WAB8GIwboBoTga0dDkiUNIDLG1OEqY76N5LHoAg4eNeIEurFw2pk4oLJb0D1Bwt1HQJ4IB49N9h5lW7M9n05802SE60fCEf70EyY4i0TAYOJW18b6h40pgKBZMYpfhgb2wbJAUAv-ABv-ABv-B_Nv-BBnwCBnwCBnwCBnwCJ_ka6rM6oarJD7NHK7TsJN6W8ZLHTUUlGGlH3viKAKQHOXGfniUROiG6Vv5c56b6aMOKQKPKcc6gvb0HhHVMgNLexzi_McVsFUkTTIZ2jpFhlJMtzt7L77v-ZQf_W-hNy7KANqVsO_kfUFLFNud_lVUBU0q_9IjsW1lCIn38asAVVFwvQ3q7t0owENIw4-r7rinw3mD2bNLgFuRUSRUY_S3w3LSUmGWpJE3fjzgtclKF-1cQ0f4tK7SFsitiVT6-3xXNzW5q2mr5Quu8mbGmq6q4JTpH7m5-Bje_uLzdrXlH-a-OYFt0VeBwdTlteTUPzSv-m6GbWh5cZh7WQ_w9wDi1v0d-W1m3LdOaQ_AEWZupPqk7wpBwE9E0QNw13d-5GU_1q0VMMpmbn0V6F4OZMIVUIFO1DiM_OI8FOLB7Q7M09SmRRBFWCxliBDCfzFeJiCCsGEjruitnm9gXIHt1sCTmuTrYMfT5R9b3M0sV1US14zXFNRk-Ggk9cO8tE6P79WiRuwPiZBO6KtDlIncDX_gUek0_TBluY1niYIMT4DlcENZ3t_6ieB7BywBM6FlNloX3XlmBCIONViEa4SjVgIv5u7YAreuvnyjiFj4cXpM7fgWJ74llyFUt8kxX5VQeWylOJwnZ-ndGyGBk8MxoVh7-qEiPs0LPvX0t2n0unfKmF_iXTelxsD2e4pmBY4uIPSW7lByJ0ZumL6yh4rFltzeQwPeAOQ1gErlZeMpUr8qHk8EccmgykXcB-itujeBrJq8CKKAXV1yMP8uyUf-i5g9qW5VhDPdTt7w4QiwmJP5xp9n9eSeZuVhrNTCSJwwih--evkvq-zE-Fbg_FLdRazV61kniQdywMjyehLszrbfhWSH-e_HtrxU7-lRwI8OP6klJRVMxAH2TJjQ6uD5J2FbFHmFnZrwr-2Mfld53KBI1Y6PBXq3tww_M_6u9lbwoNbu1yKFpECCU_XASzszeqj3a23ao_7Q8_2g8SrmIwwCZd8V8Gw3zlXCCqFX792xkSYty1pkXjunaaK-mgXuxfaPBV-Ou27oT81SHqNzEfA25f_zI37Y0Oep9xo8T_C_PZkr83neBUzXRPze-jIEQ-3CqHelgnEOxF7Vay8GLTF5Nn9me8uh94szk2nKu4B5H8BGTjN1DS8JKrZw8d1dMDzZOrJ7-S3HwTZ-Z00CA84dEKXs1Dhsn7zFJY6X7n28wtE0IOrS56N9rwnWR5rJTgJboHo8LGVTnTpJcRE_8sbPVrpHq_71RDVTuvklhUmqj0IOvWpMVu-fJD4MqQllO-qwmCLw4zx1NyCCRBFa7ZcUGRUwnKukz4VVbAOA_pvWPUPKZnJJK_WHDRp8TAORdhqBSVhws1jt4wziLyT8blXpcGoGdbou9yVJm3EAcYFc_8CnW3x2V8UOiVujABFWuC385vj3S-Oo8jxOts3np73wmtoBvsaSnXLKC6H3j9zYcA5YmFgBna1E3o05rdtRnhZP5EQJ8SNfiFs7hCQm-OkWNtxbFUe5eI8hEJVR84rGShTZQ89zEXE509e7bKC9PwtrCzwcZt2dMUq6NpUK4BodQ4OwHHATBZFXuK9g6cRoarDOBMvnt-sDr8NbFd1Yb9VuFldICcPSZpyGvemWNdpckotHIlaaboXhRtClAB93x7m00)

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

![Платформа мониторинга поголовья MVP — Alternative System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjl85RxdKn3RNWNnslgNqcm0IRQkDi1Yisqlai2W9SOc8ebQYahiBG8uoIQxMmT9iyXLYoP8BthJ6yUnDehjU85z0l8L-YGzSpYKXkIG75cEIAHYOCSUpJVpdJFdV7D8aLR_jjRjAORIaLlgBMNGDhzHjyrgMzliTjIkAJVXcsf8Kd_SxQwXzFwijSmjERkomhHtbEshQh-lRAhNjEzKELDOOL5VgfjgjzNGksPTR-k6_Bi67e4UDQLvUzFG1zsMqyyqb6uVEW2DO5_TQAb6i1_JqL2RfjBTRAkNBxSHKpLKvSeL-I-I3CV6UB9cO11y7IS8ldQFEmKTz1huIzNDdPvAh7LyaVL2MegfR2ZzAH3V3jJ15DtXkwarjPu2AoWCKWn3lqkxqqKgue4jpDBvMlaCLAElTwUOl6d2q57ToIjzSmVyhDnL-tf79GFmbtg3BAbYx8mNcZDYaVD2dHi5JfXTmwbRIdzBRVcTtRmDKNyUKo449-gr5QthJ1-H4SG3Spa84M5lCw6InoFMZecREJYU-uEvb3GFpD2aIJXPbpGBpEL84e69Ff8x9ZAOj4_wAWH8jwbkwDk91ucZLooDE6sAoViuE9MFA171CgDdQ7rrVOQu2NfX1Zr93IKgBQdXuwfCQWXHmH7Z4vLXv-4SLdDIuo6F_AGda5mbuZBYgOexkBY6H8qY8apzdRwfTb8ZIgcWGmKf5LnSCKe56If4yLpcAKAfy8n7WD8HEAxu1DXCBJnJZI0YEar0mVcVQgH6TzB2Y4fFMkYuwfCMFbI0XEZCKuFIuX-F3AKcY5mbuX6QMeoc7UJC1tTs9o52BhpHYLVA-Mofvez6fZNWOtgnEE3ZbLAkLdJQltQw6s2ud2rWqwgcREfTfKrRLqAaBH7A74rBd8ShPeaJeIecmeCPbTywaqDO2KlSAnxjIeXDQDwhsy00ScrTX5YnJFu5tbQOj0eJArV9lna585dkl2izgI4qljIJ6aRkjPvf29sVIF1SvBdWane_UQ_u94nBYhww4iqdaocMQwj-FaohC9zSlTIehyoe5ebG3htwaY0FxvMN18buhhhCImbu3WbUTKdGAvmhBdDd4rykY12huE4dnhGcL2uamZsgN4YSKA1SI9oHlroOhqW8bmk9-ubVBYH4A-wEazckhuk-ZIwY4EVytityxbD91HVnzsxcTzVAxEzDV8oRLrBXeIdVmvddhIn2xy34tj4ozIXnGZNx7Gc9YTNyg9_dtIzp5AtK49hgxfSan2irdZh3NJ3p537X-rTYXUnC8z5tTG1cglBlwGtSr_FbxNnvELEy2QvPcV8u7q2-KglNgwMi8A2urg17BIS0oDGRp56A1MJM6fwZ70UeDtn7CHfGQWIEJ1GW4-pVg4G1YXn08mfGum7o4O0r7g0K0U3rztbsHi0g3r2C05IdDPhdrhN8bLwBCs7MqE0iNI4kl5UZLesRSRMeyRafMXXFKiq8Ya2kLAtLAiNCnvCPSK2sK2gN6MqfbCl5V2OKKABMhbQA-LmkNtBzLSjp0Nbkzb189WWeDQAErQmVK6n47siPBo3NY3jgNa0j5k16YIGu0PZ10Ye2W4QM0QoA08eCGAG_pLKfJ8E3HtLQG2rz0CQ6bJKHG8a150Nwq_2ZiLIEZuoSDxmp-KBaaPkZQjppbrFP-7fgKUqigfr5jREeTeIg7Qn4_BM8lngH0fL8i1RnL2EIG2NYgKOaaKg4hKREjXtafjtIQa80PnKbuGbeDIA9LY9kDIA9LY9kiagYbOYRIvBeBkxwAMM0wXcDdjwjuuKBAJ3A_eNB2G9M0y4N0oW4mZiQK0icK2J0Mq-LOW7U5AKY407mYu0ZCw40ZimmGkO_V4B6r5E6lg6RQRWwjG0a2YZmDNwjmjy1-OME0ybnjqoHCsKM873Aw07a8m7r8A0WSg8i3bYR3L2iLF8ke9xFJjmr-GK0bKgP2caPgwp2n7z5TjYI-oCjzYJdXC85LDap0ApwiooWW81L1b2RzA7eN3G0QzvnZEJ6CXuEQ5JBpEup_YK24EhMJml0BboZNF8TzI203Or6eFIj14F34tobek-e1uDl3lbmjfoElfHIuW0AGXh0Ab--7jQVrKhVsJRtMEn0dmJ0Nqt5N9porsPn1AhyIuUXW7fKfd00lBFJjVoqrpYg4O3oj9TTFZO0lCu5U1P3yWnV3gdRoml0cTsQnj2O1u2tWzIv6PTt929V3Y3gPVSr5b2Vl1muwburaCJU5fWO8uM8I-ZMB8bFCBa2iG0i0ESEYBiubMB0EEDY0FL0JiS0oW7LY0RK0xfK2hlML0peqfJbrEAv-CLpyOldulzVdujVF1Y_U31-yM3yui7uQPzq2nnjzPOfcxfiQ9jRfjpIY9feklFNeANmXqqM9SQYn5YK69zkYH6P_8ioOr5cBCgCHPaHLcQOcjbMPUkLTMpjsm_ixwrJUzSwiVPb-7PgVR16zWDi6i7FRqdphz3y5hwEuUl0Vc8_aw_-wIlvlxilv6eRZJ6rEwfy3P-ObUl04soK9FhKs6TVV25RBs6C8Govxmmjsu-jeVK19eScwzX-0ZFX5BBzqFeL6mzWjY6IUNQZUwDh_Gr-Xgv0wXsqxKA_eVr0jky3xesz1yD2HzaQ4R9W5WBTJj35dVqnu4Uo_GF-EhJUoTQ__KJiX_PJCkx8lW-Z3gqJ-o6NbGoE6pgEWw_HPpAp6oXz_B0S0rQvetILJPLyViww3gyhvADDFAvv0GF-0fFkmjHxrdiqYV21EG_9JDRHU2Px3rtnJp3a4HXrI5OAM8BTu9i9d-55PycEOTGVW1twWApTqTZ6FMj8Z3g0kG_XXPE9MHTbzCm-h0RZmP771VOprwzlo6eYKuWCn-t4PNvdt5ACpdfhOEwigvrUMzzHLVoZjvEFoq5Fp1YGw9kN40r_HQkWiylplDOOu-iV0KVwueimfY_yYgG9ory2Gyh0yPYi7TV6Ytm-uIw7DOUkP13CfHFy_bx64TvWHErTn3Yn3ktdG6USS8-mouV9-iEef-0NufbzxEOXS8YDO9_z2CFYFZgQ389MG7RIB2CnCFwR8H03RT28lgatRwztzAu6QQ1rDzTxRSMyfHiT28UkRgfmfjEJzLlHVINhxvGOeKHI-Juio8X4z0EwMeJI7gxMMshTo7vCMaxHfoGyXevHfF50kdvzopHx_KlBovkQkJNO-BofTvRlz9TsbAMU0RkSrXnqbbSoIrUlLPUg8BVVgCsjvOsslh7SKS12OtcmeNqx0AA3_b8R8de0AVzvfoN_STmiVqcRRzwKr6vB9jPoKk0rHEPFn28mvO14AVo6bh6nkYV_zX_GmxVeytz1xqU_7T_eNkvhtwcNCjahroTF_Dwyl4nQhv2ru3u7xFdy4LWaTLlhcyJP2bXfWfKotPtyJDU-_cg99GLPcR5UNt9fOIxoZDad6U35ESc3Ikv8_3w9PUZ_m4bRwFmzaK4ti2QHZkao3ZWQZaUe_uTEK90WFNL26RwEI33Bv3ydfz4jWRzlGznWzEZlCjTwwpdw_eWGT_OWiWlzv6I9zJxQ-k3MXyq_OkhlUxOqI61YrmcHCJJsA9Hon_cyjVKDGxyDzJMVzalm5icGKqyUXz8TaiXpzO7es8Wq0aD0xJeAu943hfA5CYP5_tfRkGrxmJs9GvKhTCvMH5ZY2hnt-3Ax7zqZVuRE9CaThJx122C2IDSUXlYHn0VTAObjH-C8OZCDGmzMoLalDz0EoEvhxuMQXliT57rQ3pyuBsZTpqHmWG9iKKv27C1UzSv-4ZGZf1_TFCBb5rxYX1fiI-u-TeZEXLtUQJbmPw8DtYHrAnokjNULxdfJxtR9fQaxchaZOYypzX9P0YGvX3KVOVCnCP49UtJ6AtzDymP-U8Sbm9uJ6RJiUUAqOixZMECTWXqpmg9nOqSr3QtP9-Rr1ntL6Ak255NVBIt1CD7xtINvmWMnclPZ5AiNBaZHzD22A1EH_olA34J0VawOy-8jhD0Kuy2f7BbJ2zUtCfOwxz0z8qoV4oA7fsICGsUl80uazGNwb2GBxa9aFXB4i9K0TMQ745tj06NVlXUrwzbF-RiUh3vXo6vn8sQ30w8-AJepsOEne6AsehT1FfqCmqIcYULGmhEO-qZtcQFSKwptX2wVon4kATaH3eb4JrSeY_48Q1eoUHrGst7QudaOcwiJoNlgv6XGul_0bEVSypGIcNWE8SEBpvtGvGQfc56-6i5nBWpq1ZNeQA8_72l7JgJ95JpnzaWh4FBDXWmcUpvVG6XbmPs07shq96f4WssNEINWd-24C8kLV2JymaqrzZHurcq86FqF)

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
   _Меры:_ применять Локальное операционное хранилище данных, резервное копирование и восстановление Центрального операционного хранилища данных, подтверждение доставки, идемпотентную синхронизацию и удалять либо помечать данные как доставленные только после успешной передачи.
