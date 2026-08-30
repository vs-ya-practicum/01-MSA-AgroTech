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
- UWB/RTLS-система, оборудование и устройства фермы, основной и резервный интернет-каналы, а также Внешняя система-потребитель метрик взаимодействуют с MVP согласно основному системному контексту.
- Использование существующих Брокера сообщений и Озера данных откладывается: они не требуются для локальной работы, оповещений в реальном времени или синхронизации MVP.

### Основная контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Платформа мониторинга поголовья MVP — System Context</summary>

![Платформа мониторинга поголовья MVP — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjj65hxdKn3TlGhYslgNicq1IRQk4g28q-r56XYqnDX2956bg2JUHO74QPBkEaWstLvjiGroizUEOoMgxTYlGBx2FicUEJEK-3CaHvOZ4baoi2EFvflvpfbplZcaI7jmfzthAxhIbLlgRMNOCMwezurgfxtLwwezGsx2DrMNfC6asprTwN_RRXdRSdPbnT7ULUvVKmS3PKjTR_z5bJE55IVgghgbzbeDhMVKjOwcotzeuE7hKLEQTxPqRTXhqNw6hlG6q05e-FjgUalL_VqS7NMrQIYzhOxwnHSTn5HrLVdgA_claWp7vcIofcyGV1ydyB_t0tSADkWw-4lLZ9s-IgobliYw8It5K3QLmGo8xuVgS8RkyDreDzjz1LPG6AJekdQFTMUBLCG3MnnBvsdbCr3rWTQROVAc0KE7TIVl38mTyBDoJnreNPKCm5_gJRAaYhupMMZEY4NE6tLk5D0mMyUfMyfWMsrvdTsy0r5_4LCX12VgTvHsZquVaX740miv215XRpj29Oz7f1rJDtDmFFVxSobk7fYXSPEWMHSt2ypbK1A-YJwIEwOo6BTF1Ye4IA-fRchtOmyIgbTE6b7Q597tKN0c7s4YWcL6NsyFr8qvuiRlXJdq936KcBGaXe-jCeaXn0H7Z4zOXbs4SvoQanWFkUGdEO5aAn6N4KzLxC75DIHi515X6kmC3BMR655A11qcI8dWOejH8iWm8OhciaWHIeHdN0AKZC2nnSV7PcRXcMa44T5f0WhE_rGzCRgJ54PCUf92nrQVfF1X0YH4Pv4Qb1J_k6GeCK5aAn6FqCnYDEiWvpwuizE42BZmnYPUAUUpfPet6XsjFX-pYyK-7w-KShKYRRzEk-jWk9mjODDgMzdIUag7jQu4I5iib3cQ5ZiFMynY9y9KJ8I7OrH-wrmESKfOx5vnQLTCR0BpNhqF323NDaI85Slad-zZXMchCB5obVoRKm3YvSwxqfCOGfDBFObXP5_hcONGnOa4pqMk2pw9yPFxYa_CjAJmgolXV3APOhcswkNZQ1NcawkN6lMLETKY5ig1LrzYvE6-yX8Z4TTLbqKf0SyX_gikCNg5SyLbuMoYomKHOXKylCGShJ6L2x5m3oiNOYVaAnTYP-Gj5nOh4iBbGkn-ufSBCT6AkkDalklheX-ZYoZ41NzsixZxL1B1HVopcyNTjJB_PnCVu-QLH7Xenizm5dahYz0dC95tjCmyIfHGpNz7GcnYjJZg5tdtom95Ap64Phhx9GxnIeodpd4Np295JFZ-bKWXEzT8x5CTW1cg_4Ts0_VDVFb-lhoSATu2rwpCU5mC85-frUlLKbOGK5nhiAEM4m1awWt7KOe4PDOQhgCS1QWtF4Sn751g-8vC623Zxz-eX06A740Z353Z0V8XW3KUe1G2uFNtU7PEm2eFK0m1L6SrckVMjT2LNeioOTxGu2nT8IgyLyDMZPjnjR3nkIbQc4nIpGYAGAvKhTKgnSp7anbnGBPGAfSTsb8ebulvJ22WnAnThHJpkLo-PFkhbkS2yjpiOP2C7r1gX1shMI-Ws0W_rZ9kGAuHTTJSW5eam8uIIN02CCC9g0W06ba7O5K4K7G0HFgpN9M2DDX_L6S5r987SDYmiYO2A3a0HO7-BFnOB9MZ8oFd3kzClX1wv1Qe6ZzuowbiT3sLLZjfjPDMEscr8rJjO2NYhKMyrOZagqJyjOYh6f44AX5NDIA9L2BEQkHysq5krYsj9WIWgoW9Jy2g4Kcq4h6h4Kcq4h6JLHAjHEnSaaHtSTjF2GDKpsdqx6wTB5n0WL7sBbnE4B3g2xu8GC4Ntk60cZ-1GW7kUgeK2N2dA1E1481N1AZC108uCkCGCk_X4J97FQLhcvgHXAjJAI1MG86lyMiL_Wx8BtGe9CVTCaNEb9q0WLD65o0V2gZx0GMH4sLnmDfyW6AbahS1zNnswgxf3m2eLCfCICjOPHMc_YiwXotPBrcn9z4J2XjGTPu1ODMVTG8A25Xr06hJFWoT2mTWpJk9aTn4neC1ZMhPiVjCVWW11Bhrqm8u5wvHBdcEkX_W38s6h_Ij-4F35NobekUe-uDl0VbmlfmElvHIuW0AGXhWLBvyFQY_KoizPzlS8zs1FWc0jvgAk3ZbhSsY25Jvbmu30VMmJE40U6UdQ_bPht5KGm3bMI-xV6m0U9qBS2s6v1g-752tbrF0kTsQnD1O182tWzIv6PUd4f4l-n3rilsU4r2VlkqxwcuraCG-5fWQ8mM8I-3MB8bFCBq2aG9Im8K3eYvE9HWmoRW8GDsNqn60iayrmW5rdowLWgur5NswD6CvbJuNdpuNdpuN___wN7pwO7pwO7pwO7pwOFoiJxhvZevwsv0DJTRRMzk6t6eJDMbhzBT1Iy4FcwGbHbfYf2N6frjYX2P_McQaPKPQPgHbHb2PORIDZYgRhynZSz_QjNuqJwq7vecvByEtK_E3ERPsiMaChz-IvZDeVWjVn_1rO3spNiZN_lor_Dy7lyZh-9IiN0TkO9SaiIV5FlloIzdy3N0Z66QV3YTRJyoH-G4cX2Rps7e6e-EmilN8FCF60vXXX0HUVDVxhcV-0rv3Lo3o7jeUGB-HjIjR3m7tnjg3OQ6ZR8u9GJ05WNOdw6BF_WJmOzbwYZ-EpFUo-Hyl4UkHzPoCExOUmgWZyyHwn6KbWxD6r5dmDVvCTkm0oW3_G8uEAzjLUigMIlvcpeO6xolapvcuNFCB3FWE9dq0K--PlwD9X0_CUKXcCeycCrbxw8f_Wo6FmQX3iZh04km6tqpvZ2xJPJg6KPy2E_G1MQ-ZYOrxveWOTG1p7y8R9rEpBilecNrO3OS7nouNM2zilxuXgubC8HgesudBlCwuhUZTZTRGsDbGk_sEjgEg-8jkftyY1pqnPn2YRtw3QFWxMWMTRPuNjSOONx_w7Ec9ByAOlN66f0dBTxuXPU1uJDQEQ-DbFfzWbyEQGrSo2ERI2Nx_NSOHtc14xLt6E34FhPT0Pv9mZx7BX-dwmwYdu1VYcNtivY9mY8rWd_KOm-8XEfeCWbP0Th8i8p4m-JqX40ETzbrrO6ZDEnjThTU6D6ZtjZRw7SMuhUbT28UUPgXmTjENjTlXVINpNumOeKHI-I4io9X4z2vRBK9f3rVhBRDkR3qXBQVeKn8U8seKQJoGRjxSDep-uChoybRRs1vkVjxKkijt1qixob9VXvsjtHnsbrSoIzVMgqjLaDWRQdDxUREZRIvt5R1GNnvkjhyV0j7XOAa34Js4bF-ysvA_cJJBLrdphLkIscl9n5f_2gnT_th96POGwndpzHKGFsB7AKRuQmZUln6ZeSi12MJuQP_4bHj7GnBraGhdFeaGwBzBam4w_qZ4nLwkQVnW44wMaIto4Qhg8NKrMVfJ6Y3mTKH2H2R_qKmX2-__JJ4G0QhUPp_BFik_-RQsVFt6rNMF-MZnFqduOXoUsabroXRsX0Rw4hmyWt_xR8lvW3t9Ibi_qT15KNHf86usd2YdM6BHh-xkL9kdv4OENIRwStcIqMECBxwJYNIH7SnMYZ5Qmn8Pqs_ViTFjap4FuELKF-WUTqhMmdfglIJylQaslBUveHkTGQ2tYKpoP8GAFR4GMVrEWyonjx0Tw6Tc8lNF2KOQaHSM0p34JoHEcBb40OycaJJLt3qPlGFkfTfi1zm1MhSN64Uucq0PrUxZdwBpM_SpaJEW01i0JOm3s0dUMy_yPWJqOmiPB6xm5gU60T6tTub3T2xieRJbm9w9DRYZtAuVkDRUKteRJQtN8nTxxhQD7H5x7MClaIL0aYDOyp4s7nCJoQhjOguVeJDUUIk6D_3YFMwmUpGoM57nZBPYek5O4_rrZXbYqQIngnfwkpaWvWs6NLMVnAiKLZqjBS4muRl95VdI9R6AyMEKWfSkIU7qq88m4v7_AyOCTboIvlnu2ohZn3XmwZBkrCBLeuo5n7jqpnZJ1weRibdiafp2FOCuaCWf-fGa2mebcOy4CQWPK7Vi5AAhxQFisV_2TZ_hEN_tWzKd39rRtTYnqG5HhzAUcorloq0sgAjdhDPzDztCQD22Cr4-iMxUEy_U0KPuzFJSDXnCTeKpjcFZ7a1bWcwPL1seN_54vo5Q-O9QDKFGJkdokHnXN19t799zPhPQMeo7JvYHoMlVbZ7H7ZnLSmhELBUVKw7nxrD_SbJmt5CvabvucY73cozf_d4pJSoeI6tcktlUlGB8_Gy0)

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

![Платформа мониторинга поголовья MVP — Alternative System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTSjD85hxNKwXczueYapZ-3SjGPJln5LHD0RDp0LHASKHmOLiUMOPajgZYPu2P3GNBB5SxjL3inTxiJGWnc2I4gda2wHNsIVRqwPRTabfIEmwMYhMeXAJTN_TtJf_pzP4inTsRZNP7CxMMkgvVrtfDwxA-QPMRZOrsIszRQXs-wQQYT8VTRfjQv_l6kdL3NLnOuDfRskO5lTlLDlHBZHzrDPDVu57dzGszlLupsbRLQ1gc-eSQ7e4U5Qr-SyCqUkrrsiyojNONEW2DO5_3NDVDO3-kewdNBQszqTJFdcqYfcpgshbpwbyK5OwruMJrm23uEauGVEsES0esw2NmbsvOMnsTM4jziUY5h6kMjgPrnq3yqDDxOtI7xrQZtkXei8BI8CqqZTkiErka0XxOmYsThrNCG3UxHdkCoUiM31tLdRpIjRR0pzfjlMkqT3A0UAdNo99gvjPmeGKZ5WGlL8LHGCFi4awzhdLlwEj-PzTlGjIVn5G8mOawJQtHfjD7P0Jnm5GEGaJQsroevF28jMFaPW5EvFvWBYND0pCqQH8qwv9cWRaSII8mqMTonqW6a_P9LuS0QTVrDMCpyI2XUiNJYDEccBoFWpFvY18HB3CwPgEhhquGDq4lJA0dgQ74f2KrV5oLIGqX9ZXoVA8ox2ISmsjEQZpaaP_q19AhH4968bLn1vVNaAXHP8IfkzMrz5PgH2aLT9WWfOABAqQf8CE4A9xBD4Ke5Pxn250w0iSLdm2RiOLdh15aHASEA3Z_qytKw4vQ637fIGiTLttImeS9a1ITQMfGMlpZaQ7K196hH292OulHk8CS-n3EJXCYvC8RctYXb5iiLlpHoBK6VCmk5WTyl53CLWgq_IBjReBXwc8UcvORseRHrfgiTI54sXAXBD2qn7cuOfOu4QPYCZouKSMj4pc4bx34lUBHheJOXEQzlWaC8DTMPOWLIkHVu6s5KQiqiLAP_9jG09BbBhhIanf2mqizgM7aNkiPXz39YOJ8HPuBFgdna_UAJyAqbEYhAz5yCfb2gRBivyErIlF9LekrwiA4Qf48vT2hBmdoy5vvIP28vwhBj9H0v93WLPS4lIAuuZ9rDl7bWWon9JpynBZMXCg5HBX7bGk94mgK2uapyfSBqnK9wN8XSJ-9ouM4QCNTSJBPzNNPjz5b5EA4txkPtdqggU0Y_zxDzEvQcVozYSznyqegF3JcUpZJl9L5wXsOs3jQnXubJgWcloCXCR6Q7lMJlFjbYgAL6a9ZtVsIX7YbnbF7k0jccY8cVVzAh90TQoJshWx0B5tz8xk1-qgkj9ahpMSAry0r2sCU9mF8BLMgrN9nKH9GMAcnevAL06IgDUuenW8oApNFKOe3L6k-en0DADO2HoOAa0dshor50Ge2G2qAK141SX605H6W604GzVTvTaB0iWXGY02KnpLQvDQLo9LUYJDXqj0GB5q-BhoNerPDc76LgF65APeVJbBEI8f0jbYkL9OAcSydC_A0nNonLEAq9Lyg5NAPK42HMRkyLCXbixcYwwzAJWZ82RE7GJ91GB4MTIml-W65MlInal42ihMue-85L681RfGeaXE06JnWIG9GM-G0opA00WUGwSzoLGdJuE1H7XTGIH-0is5XHGPGv005YVui_5WibU8Z8-iDxqmk7tbafwWQT_pbr69yFJMhTcRLpgpQcLKxKjKELYB-MiHVZIY1IYHOYtYg4KMY4l5K8ufC9S9N8yVR3hBZRcaLAG1THKLw0bQDABALY5kDABALY9kiYcmbukQI8hkBktvA6Q1yJAF7T-jmuK8AZ99_uRAIWEL0yCK0ye7mZWPKWWaK2V3MKyLOW3T5cGX40FmYG6Kc522G6Kx8_8TFoBXwoZJM32iDLwTcW4G1UR76hoo9Tq1neKCXMU6MANEcp0CaJXazW5mae1e4v6LEbEK1AvC12ajBEHTGpIsEt3Nw1G1BIoKcf2Miil8Z_nNuOOlkZwpOK-W9XGieysS0MFKlye0y0fOvG6NKXw6pqG2iUOSnaXtAU3YWLYvnky_mbmX0g5k_B02_SBLIqNTKWm0-D6g1qdSf61gUu2iMV4Sr67nJo8VZvNJqfPIY092NqW1U-NBLiFwyLlhEjeN7R0Vw8W3-QYhcufI_DeiZK1PVEWm5LACoHG0GdPskvCQznb6E09J6lUpokG7WTI70inYAP_XIIDrUcW6ExTOqXiOq0A8Tf2hCk1oLY5mfWAYMtDTuG7NqSk2eUjL0aNjRO6IC4Y8keLkp8Z_1w0h43303d3WWxk9KYW73Z8i1L0Cv7GCe1LGZ6b0Dw58nx5fJ8Q1BOvPJi-VYPy_5pvwB_tzzBdxsOFpimVZPW_6p1-F7VT8jS3JrwvPg6QhPsBXXgUiDeYODW_urQ2NyOPDPYJ4hCMObnfTROaGc_wpCc9KPip9ZLcQ4bHbMmshggltAFh1tdFlEJ_QHSzS-j7TK-7PaVxG7pdri6i3FRqdp9sX-2ry7yBNhF76UgHV-_Atwtxilr78JZR4QjtJr4ZungrQ19zYeAEofiQ---KQrNy8OVHXoXmwjEe_illqH9eSc-y1v0ZFX5AhpmFw4ZRimMn_9FBlQljgs_mO_GrSWzHxQxaA_ldDVTUu1xesp3SD2HzKU4B9W5WBTJj35dVqHu0UgynX_xTllLVlVVYBE0-Sf6NVWt8DH-_QXyq38IWN7zQdZu6lmbShj1beNFop7X5LkQMrjGoUVdxDgmEiQ-MWJZsjUm83lODAxCFMs_G5D8dnWpZqoaxq_dCdPHbVy4mnv24RjaPK2bc0t-6R4ft-QBja1ZFeOsA4FoDhj3stSjll4g5sOUmzUE1oPTLf5p-p0Qd0U77XSu3np_Vg6h2OoXSYWRYSkyplZkcQsLjTxrjQgtkeqZIrTnpzwE_gu7FJ4X069lda9qV1Nj0ewkpnFQeqnlduBED8NNuGnU-4JaYOitmM6L87Z0LaxeOsdnNp2NGvh3bt989XBX_ZzWueZlC68sZwDSMBlES-1pZ3X7cANZwFrXr6Fm2_4CplOpKDW3nl1FkSXXiKzT3GP1As0x6HPHc9W-3T380QQe15TowZVN6qPxGQaGQEziTffQjPrmsn1EBGDIuSpdOvgN8_kgzZ_OCG89PBAzs111YIYxxFL8fIsSRNUCki6pYFISeG-9U7HfqQHnbtbofKRbjNfdfcVtsXODtfhNzUDrlojxjoMDjSnOPThr7kj-ONCt8LBvRaooErblNvZVgrfhCstDB3GdE-jDNxe0T5UTwu94Ts3bF-wjQv-DsnMpxFcQzSKlRsk4ci5AV0Q8lCNOX6OiakY57v3o_XOtLP__m_euLlq-R-WzyFV3wwsptORF-fdCeldheoU-BrsTfwqdYDh8NmEsFFv8x18-cQZQn5dQs2b1LQgR7VoCrsv-EqSJmgoCcE_FkFImbmK6RD3Ci2Beyc3IaubVeV4ClI_JzCMElz4PD0Dh56aOxeC0uw5upxg_nuD2XAaHpIKuMkV1BDA_gCvZMu9_7qRuWQpmtvNXTRRpz7t-uGutOF8BlGBpHBxGxJrmQqFct_6rD_nR6cGmCIkGo9ZQEn-AEKM_RorrJNJk0drjPZsI_0MoP0ZJnw7qksJo7FrWUZOW3G2Gq3TfGhWaGEXaeKo9aN_QbEx2Nl1dK0cVFTzPVxYvVENV466LFzEv0nrwSWLsIEMw8TK-ERWnq_mRuUL71-n9ybyS3DT9B99UlP4B3u4gaUn96v-ytQdU_M8N9_Rfyb-T7gey0FK6t4eawsCL3JkpZ50QvYmenplS9NF3Xbp5tuSxIUqvZaY4UyyTfuJ_dxfxTrUMpMjPfUBwK1W2BSLkl4J2v7L1neTNAt1Af9VcOdKFuUOpsIxOPa6G_nCif6PIo9fVvYllAwXTy2zD3lSjDv5wxP30mch2oYn6ntycCAWTR-IxGqeG450vMSNAeFtpfEW6I7zs4A6RtRmaY2neGXmguOzT2xKLBHbrvs9DNYZtAqdXTRUrjghTQFT9XS2RpMiBHbxkR7do18WoJwi-G2R3uY9dCYWCrxvAyWtyCDxBBYsQMIm8lC9RSL4mx66-p4_PeH5myQMRfcDUfUOr-srT7COhqfKDJqt1yD4LnTdr5EdvCuWFiUfmQbJIZGzj23AHEJ_YZ43oILfBewtiB9UZeEWJXTEBNqseM9X-HxTCy3qEIIRpn4POu_kpCI1H4xHfoHPK2Z9V2I8GSs0EhSVowvsWBAloeYgCPoduXe3Lfymv0kA0LTE056_qXwRhMZBG3Qegv_OkS-xqMP6WnQSYVA9vr7Rnzaxm0YVdXxRXjtXhl29xTYu1r2Pe5icLGVgLppnlOTMFgDMZH6q8xfYBliObyGTUsIV6U_SIev7G9YHoUl0bZ7K7ZnrvmLdfDjFgT04zwa_SMTqmbDxabvusgd2Yyzf_WapJSueNrb9dGi3lS6WsX-9gC3AeHEPGWIlrNZs1o3acmqPZCf4iGgoObGuWJZKtAgC1TjTxYnIV4uSmYnstqB2BrozuA-SJBgN0gF_0G00)

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
