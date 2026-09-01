# ADR-002: Высокоуровневое видение контейнерных решений MVP

**Статус:** ⚠️ На рассмотрении  
**Участники:** Владелец продукта; архитектор решения  
**Дата:** 2026-09-01

**Универсальный язык:** [ubiquitous-language.ru.md](../ubiquitous-language.ru.md)

---

## 1. Контекст

MVP-система свиноферм должна работать с несколькими фермами. Оператор должен контролировать фермы и анализировать События фермы. Агент фермы должен продолжать локальную работу при отсутствии интернет-соединения, сохранять данные и синхронизировать их после восстановления связи.

Task 2 требует сравнить два варианта контейнерной архитектуры: выделенную Центральную платформу и расширение Существующего IoT-шлюза и TimescaleDB.

Архитектура использует две границы развёртывания. Граница фермы содержит Устройства и камеры фермы, Агент фермы, Локальное хранилище и Локальное S3 (MinIO). Граница Центральной платформы содержит Центральную платформу и Центральное S3 (MinIO). В альтернативном варианте Существующий IoT-шлюз и TimescaleDB остаются внешними системами.

---

## 2. Требования

### Функциональные

| Категория        | Требование                                                                                                                  |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Мониторинг       | Оператор контролирует несколько ферм и анализирует События фермы.                                                           |
| Локальная работа | Агент фермы получает телеметрию и видео, управляет устройствами, обнаруживает инциденты и отправляет Локальные уведомления. |
| Синхронизация    | Агент фермы сохраняет данные локально и синхронизирует их после восстановления связи.                                       |
| Интеграция       | Центральная платформа публикует Опубликованные метрики в Существующий Kafka.                                                |

### Нефункциональные

| Категория     | Требование                                                                                              | Критичность |
| ------------- | ------------------------------------------------------------------------------------------------------- | ----------- |
| Автономность  | Локальные уведомления и управление устройствами не зависят от интернет-соединения или Kafka.            | Высокая     |
| Изоляция      | Изменения MVP не должны нарушать работу Существующего IoT-шлюза.                                        | Высокая     |
| Оповещение    | От обнаружения нештатной ситуации Видеоаналитикой до Локального уведомления проходит не более 5 секунд. | Высокая     |
| Синхронизация | Задержка синхронизации не превышает 10 минут при нормальном соединении.                                 | Высокая     |

---

## 3. Решение

### Описание

Основным вариантом принимается выделенная Центральная платформа. Её Центральный сервис синхронизации принимает, обрабатывает и сохраняет данные от Агентов фермы в собственном Центральном хранилище. Центральное S3 (MinIO) хранит выбранные видео- и аудиофрагменты инцидентов и аудита-свидетельства.

Агент фермы работает автономно. Локальный буфер подготавливает данные к синхронизации, Локальное хранилище сохраняет структурированные данные, а Локальное S3 (MinIO) сохраняет бинарные свидетельства. После восстановления связи Локальный сервис синхронизации публикует данные через выбранный надёжный транспорт в Центральный сервис синхронизации.

Существующий Kafka используется как асинхронный корпоративный канал для Опубликованных метрик. Для Farm Synchronization используется отдельный облачный Выделенный Kafka как надёжная асинхронная очередь. Он хранит горячую историю около 30 дней; Сервис архивирования сохраняет более старые сообщения в виде архивных объектов в Существующем S3-озере данных не реже одного раза в месяц. Выделенный Kafka не используется для Локальных уведомлений, управления устройствами, автономной работы или передачи видео и аудио в реальном времени.

### Контейнерная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>. Откройте диаграмму в новой вкладке и переключите страницу в формат `Raw` чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

MVP-система свиноферм — Основной вариант — Container

<details>
<summary>Диаграмма: MVP-система свиноферм — Основной вариант — Container</summary>

![MVP-система свиноферм — Основной вариант — Container](https://puml.livingmodel.dev/a9f2k7xq/png/rLXRJnjN47xthpZX2KiG4ZbFYIeb1Qj19K1YqYXFriOin8elQBrEWwfAld19QXH2cuUgLKcZgkthOhom67lv2pD_eR-acZcxzjcB7MZLXxuOiNlEp3Spttndpjwjc9fXLei5yU2hvMciWuDrR80DFR04rg4D3lJ1nIsmiGOzyLVjlO13hDD3QFFlcO0sM5W31opeOuFNp9PBffOlwKRYscBwINRsLZQpydGX_TdDHE9QlfGhL5TryTmqDogtPsOCxUlhwtdpUVLPjQ8RkNB9r4lczLovEBDHq4e4RhfYhXRopsPcRqqlqwF73nPcYbh5r8sPsLlPWQ_h6zLY8M7cpOB-duMJMBZtTEdnIlR9_Chzx4BwY_JYt6GoaRc_z2Ixd7wKMLhCBZrUMPXVJ4yc4ubbtQYKIvFb3TtGpB8n9IRW42xOjuKDSB4sGSz-0fSyOGrSE0S7QzW46ni2-c1Z7Rhq6FVeh_2HJmb21nQjXrDrpqTmuGXRs007zmVhiIN04MpAnHfqE0_AhWxOSCQvQjE-YQIFVKqpYdDLSpDTt2YKDtMTyBu76qwmYJNeOmlEAB4kmSScTCWiT1M_7E473g69ks19Vp-32rs2Ztjin1ROX3ROq04NUd0EDkFR9zHO0niiE0vcnX9mmHaxujKKeU8qcSXiLaozc4s_CYVrL_cAcI-jVwcjlT08paTiuXlABSMBJNoBRy21Cy4B6F0V181BCiHu9k28FNR93UNid01QH3FeupQ_w_iawdAYB_YNYkqC-KMhYPJ7_6e7773Y0MTISvgfBMWlz56eRR159ZKDBfnox2x18cRqiONR7CeXRb4Ri7DUJEM2DlGyKmxq-0q7O77zY8HlmAROQhWjePCzmTpw3cpe4dsXFOJzURbQMjMCpSdAPYatNrepj8ffL7DcrT0bxQbC5kvIvRYiqCSjZh_kKQUB3UOKDwgQK2Bu84tYcuGGOjYG3wjwLPzSrLVpEStKLmSb_XvRJ6h995c-GNc7TVu5BEZW0PpukABLvCSkxd9-RUZWtXY0TmGSKybndxZEWNAIYGLCPrUsCxWYTKDmkzXmDf5CV9k8P3Adbqn3AomND7EjR1JzvmJx1xzKk4ED466hvDa5fxo1M-oUsoASlh2JMExBG4BMm9ATpKhPfNSSpIUNzfbmbYyx7bU9R2uzUGV7qZtHI-deATdkB12SKGlQr7zi1tTYsfKi3iIMYpCbI8BmWFnqFMZaXPHHQianfuxrb7j4jeDDQYlOPnTEoF05DfdG3dHzHJfK7musUq2ernfUfgZNWpNGZDpp_4lToEZ6otnEzZhZ9YskgrqOm8NxOwGAwqCBlEoTQiG3ZTkIYrGv2VGKBBXWkN36jc4mWhbdFeyoPbasV1mbupI6iVqULHv5nUA87DX5aKInghdoebFfffoaGPguyUk995n69hfYHquL7NvbKwnIeCVaA3KwGPcKc7oGByql9ISysHp8xUKoI5HiOujFAWMy1n7SQGx4ugFQeKc8rnrJQ0od9QLvnrllDwAoYJ8mFIfgiEArZCQ8qGAc7joUGYcd8OyWEdL6XRR-9ayCaZZVfpVRM5oWMd0Iy0Oi_g-9hwa1IKl0vGpoV-gSA3Sx755ZA1sKGHSwWp5ASKM1d0wOoi28VhoUioKFeRD8srDSzzRraidjwUjU67jWKc8A-5h5ddlGeyVJtBSNFBXQGxBKmCRNSfRoYF8hM714KkjBi0Ljhd6V1wy-JH1zfIHWtvRPKiUtGL03NSEtGYR9enb9BUCCo7GpMZheSJreZEkpq3NXX5NAWnSY8ny4X3vi-seJPQnQ36TXfN5eWAlJRcO0rhspHzuNXaQf7t2VxCJqVQ6SaygehUln9HtHrS65VBGUCTkiw9WNKCEO-aoDlXfCNKtPl0owtc2pBzarvhYGZ7MuS5Z79l_oJKDQyZ9tXuSbZsnxS2hhWNKAErpT7itYJIy45teZivwAJtX8PsEQpFt_IUybyomL-f5UK2wcCMFWZz5QKBptLrQMCnD9tX-z70OqxH_V-bJpll27B7l3E5ZuzbA2n8tSOFz4D4NHs02FR3qk6ucyAbu3KFuCwziX2wrFIJx54KbkU5ZwKvRMKpnb2iFMK0a3idJfYNzAJ6GUBdmAI4hEdAvFSLzK7O6x2hOHS_QOOO3SfyIzvNcbf5uLetUs3z6f-ZpOc59tV9078oQrgsksVoUymzzSI6LTNcYBcpT87AalcuJ7Qx2-Vx4P9B_7q9YI3b_g-jwWJ567GutfDfNH1vSgTm-RWIbV_ZjgOX_SUkCHnLn1meZ-yMKXv3hYQiJNZYWR0ryZGXhjKNq4ryQUmFwd7Uyu2N9iH2E4FrgClsnSwJ0V0VwIBHhicAksPuIOaKzd7_xLpLTUH4DVM9HlCyb4ugvUMgqM2ty3 'MVP-система свиноферм — Основной вариант — Container')

</details>

[Исходный код основной C2-диаграммы](02-01-primary.c4.container.puml)

### Аргументация

| Критерий        | Обоснование                                                                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Изоляция        | Новый домен, API, модель данных, правила хранения и эксплуатация не изменяют Существующий IoT-шлюз.                                                                                                                     |
| Автономность    | Локальные данные и свидетельства сохраняются в Границе фермы; локальные реакции не зависят от сети.                                                                                                                     |
| Развитие в SaaS | Собственная Центральная платформа не связывает будущих клиентов с текущей IoT-платформой компании.                                                                                                                      |
| Интеграция      | Существующий Kafka повторно используется для публикации Опубликованных метрик корпоративным потребителям.                                                                                                               |
| Технологии      | Выбранный надёжный транспорт используется для синхронизации, локальные протоколы производителей — для устройств и камер, Kafka — для асинхронных метрик, SQL — для структурированных данных, S3 API — для свидетельств. |

#### Последствия

**✅ Положительные:**

- команда самостоятельно определяет Центральный сервис синхронизации, его интерфейс и транспорт, а также модель данных;
- изменения MVP изолированы от текущей нагрузки Существующего IoT-шлюза;
- проще контролировать доступ, хранение и развитие в SaaS-платформу;
- отказ интернет-соединения не блокирует Локальные уведомления и управление устройствами.

**⚠️ Негативные:**

- MVP дублирует часть возможностей Существующего IoT-шлюза и TimescaleDB;
- требуется развернуть, эксплуатировать, контролировать и защищать дополнительные контейнеры.

**↗️ Зависимости:**

- Существующий Kafka и контракты Опубликованных метрик;
- локальная инфраструктура, устройства и камеры каждой фермы;
- Локальный буфер, Локальное хранилище и Локальное S3 для автономной работы.

---

### 4. Альтернативы

### Контейнерная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>. Откройте диаграмму в новой вкладке и переключите страницу в формат `Raw` чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.ы

<details>
<summary>Диаграмма: MVP-система свиноферм — Альтернативный вариант — Container</summary>

MVP-система свиноферм — Альтернативный вариант — Container

![MVP-система свиноферм — Альтернативный вариант — Container](https://puml.livingmodel.dev/a9f2k7xq/svg/rLXRJnjN47xthpZX2IGJ4f6dH9MI04fG2P2ODCeJMin2hFY2rkiagAha2xVK24AQXwfLaqPLszV5iB0OU_aBCt-XlwIQEMVNUxF9HQhKbqIipvwP-UQRxtndRvTDpJ0hXRnuyDtyADR0mHhMmOOEM09hq087kk3Y1jXOXOxuf_fEm1iunrrQXLNeWeLrSA05NMp2cO0MM5W51opeOftNJvIAffOhwaRgokpKayM96ukPXQSpKzzSJrt95RFvoh8kdfdcMldcs9YXlRowcZEVLPOgPTt8begcNZIlPak5iRMyLgGyHylcSZwtD3PnOtIU7Zr-C3DMqCgcRenDt5ZqGrrTgnJoAJDdvlN_fBAPEq_d7YyiFfbUkByuCtLlQdPoU2ILkJ_tP75-wb5cRdPnxl72pFJirF18AZMl6-LISRYqfXkQMJBIOWZUmmN7f8WkLeVesI_WKY2iWWldu60L6s1ZNK0NRAn1cnxZBlqll2BIWf83YzR3QV2TZ-328JQf8DptrsDJW2DuAnUhSCoG1Duw1XlE6BOMlJSqukM-eXc5oOgvFbLOovVMTPtoVGSsd623a6Eyi0OkfOyDEAPjeHs8onL-u18Qk0sMZFShkD2cz76NDx45Dg05DXo32nqu1vlpswUiiGesM70KHiOISC687V9ggZ0GT2ILMI-RUc5nwfKvhB_AbSrSSVLRRUMvHkbyn0Q-9cofNcpW7hu61yu4B-24_w84k2Mz7Cy477B4DeKXpCufGOiO1rtSvD-wH3LQscQWB_XVdrxo9rfD_3pYdxR00ISvuSpuf6PgCzfplL_MDjWYCpuABfnoxIwbHSpeOXCtkPJtk44JmS5vCRKBMj1HMpdGuL-u08lxHoHy3JRLLiLDRp9mKp2tVWGRsaHVQ2MdFLqoxscc_b9R7mJsT6bX57VW7FVWb1DzotpVuGpE16ubji1x8oqOl1hk42sOBgf7IQth44S9dnP3uSe7mQxH52NsmimLz79Mo-kJIsanjE3_TJUSC-uJ5mx1edr2JKZA52nQxI-23Wsd8x26cwnRhfmgfLpG0XUESLkfWOERFjLxkTujLOhBch4-N5ulPgUBAuPMDerArgmOkjGTcXCBjxtWLFo6oapERXlhFDJMG6p4zoaXH4yG7rRqYZwyh2_difgfB_ipzXQRh2hdWTxuyzKRjD_0WcCyW1Clh_WuyMENooVuZd5tG8At11pHpE4-YGqNoioZCMGzSQMUWYl6hmdMAnlEXaPIFwHYI6Rrecbe-VcyPgwKZ8BtdDB-oPiLtA9snpWd1_s28Q_Z1eTdNOh25msIA3woaCXkO4bfvPEgJRznDPSiPOLbH2neaO2ndYOEcqrWUQSLWviMfE5uG0hSLbkxqEAZHOhn4PVE1nBpLkg9JSULu9XjEA6DBx31X7IWxKdw--13_sML28bLNLLA0nh6K3Eopt8lT2EZ6ozoMIcMlN4FobWeBzmVeFL1mU1bRuARgAHnKtA9a9U9de855omopc292LKmkUJn83EUp02ftMBuGQuuFJSof4RQr-9k7mddXN_wPAqBQ66JZx40k2GpDAhkA7VIudFS8SV4wuu8VKwdAMLZIwtloaCW-1BrU5Gfrp51H0g0kyfPMCbpHXwZ_t05JtKrFG5vPd-YKzVBzEeVKire_BoOolXOpA5gs2OezyL_DN27o9McwN4PGVuhw2VbouwqZhsn44gWc_5GLJuGrE4fHuHemEiPBIcGPp5AKrrtLlMYoTI38mbtDFUGZM8AU7F8aJlGeSUZpCaBDhLMZonLFhbiEQ6ytUzWmI5vES-cMT3Y7dVPbNN9NdG3BG7xfaGhwEtyeloPnJqXGL8q8ndXF4CIr8YtZix5D9iE5ZQQkXEUG9LUX8pOfCLOZ-tzUNOpFhKXenmO779tDNgR6O0rfQlooEvjIlE0-xHFmjpdIrajdp5BXhQg9xUqprI75rhzx4TC0PBwa-v_RqWhyUNZfLTxlnbN2BhgqDsNx1eWXPAn3ZSEQzZWV_aQ8dTIoDtYWroHRHTEPJ-mHcL7kziXezvG9RXiaf9H7qy6FAAp2KFc_l-aznDnbWhzICy7Rgq95kNdU2-etliB2_EPeH5-Ftvp36dQ5ry9WzjxmX_PMHb5i73lamI91xdEyObe0KNZ3TYETBXj9F91yVAJyloLjm_jqBnCyhqwLEfn4nt596nBBfcgcCCw3nDPpgdYBHxqIFcL6kVv-rly0ODKoUM5jhX-ZSI4KcvGFegGNS-a-XF6jpENM-YmGUygqqI0HL5AldK6IN0G4-uaVzN1HWXT-MS_0-TVOfAJId2aqOo-s3j6GiT2zVbw4EYrEXL3j_k8h4c3aHaNT-QdLQ0aeCEtuuGWoQTfrDw7JXRlQaln7yweu34DwjVsY1dvgl1qlMUbzAxjz66Y1qWifmWQVqjHJZ0EyjDHqgrBVcG9thhEBhjUIVKP05NeQf700qVWTY3LFfV10Qu-ofI0fWsqDzv7DNLMXmMfJtBHhnM3s_bPJgbFyfy8MrXUFnUollpspu8FNtLbbZVOoAULm4UPaLJgjbvShXJo_m80 'MVP-система свиноферм — Альтернативный вариант — Container')

</details>

[Исходный код альтернативной C2-диаграммы](02-02-alternative.c4.container.puml)

| Вариант                                          | Плюсы                                                                                                       | Минусы                                                                                                                | Почему отклонен                                                                                                                                          |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Выделенная Центральная платформа                 | Изолирует новый домен, API и данные от текущей IoT-платформы; поддерживает развитие в SaaS.                 | Дублирует часть возможностей Существующего IoT-шлюза и TimescaleDB; требует новых операционных затрат.                | Не отклонён; выбран как основной вариант.                                                                                                                |
| Расширение Существующего IoT-шлюза и TimescaleDB | Повторно использует существующие центральные возможности и может сократить объём первоначальной разработки. | Требует проверки пригодности, изоляции данных, правил доступа, хранения и влияния изменений на текущих пользователей. | Данные не подтверждают, что системы поддерживают Центральный сервис синхронизации, нужную изоляцию и правила доступа без риска для текущей эксплуатации. |

Повторное использование офисного Существующего S3-озера данных отклонено отдельно. Оно недоступно Агенту фермы при отсутствии интернет-соединения, не обеспечивает ферма-локальное хранение свидетельств, а его пригодность для изоляции клиентов, разграничения доступа, хранения и будущей SaaS-модели не подтверждена. Такое повторное использование связало бы MVP с владением и эксплуатационными ограничениями существующей платформы. Поэтому оба варианта используют новое Локальное S3 (MinIO), а Центральное S3 (MinIO) сохраняется и в альтернативном варианте.

---

### 5. Риски

1. **Существующий IoT-шлюз или TimescaleDB несовместимы с альтернативой.**  
   _Меры:_ проверить Центральный сервис синхронизации, его интерфейс и транспорт, схему данных, изоляцию, правила доступа, хранение и влияние изменений на текущих пользователей.

2. **Потеря интернет-соединения приводит к потере данных или задержке синхронизации.**  
   _Меры:_ использовать Локальный буфер, Локальное хранилище и Локальное S3; синхронизировать данные после восстановления связи.

3. **Корпоративная интеграция нарушает локальные реакции.**  
   _Меры:_ не использовать Kafka или Интернет для Локальных уведомлений, управления устройствами и автономной работы.

4. **Недостаточная изоляция данных в повторно используемых системах.**  
   _Меры:_ до выбора альтернативы подтвердить разделение данных ферм, доступ по ролям и правила хранения; при отсутствии подтверждения использовать основной вариант.
