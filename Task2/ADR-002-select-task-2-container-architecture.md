# ADR-002: Выбор контейнерной архитектуры MVP-платформы мониторинга поголовья

**Статус:** ✅ Принято  
**Участники:** Владелец продукта; архитектор решения  
**Дата:** 2026-08-30

---

## 1. Контекст

Для MVP необходимы два варианта контейнерной архитектуры. Каждая ферма должна продолжать локальный мониторинг, Локальные уведомления и управление оборудованием при недоступности внешней связи. Центральная система управления фермами получает синхронизированные данные, использует ERP-систему 1С:Агро в режиме чтения и передаёт метрики внешним потребителям.

---

## 2. Требования

### Функциональные

| Категория | Требование |
| --- | --- |
| Мониторинг | Обрабатывать визуальные признаки, UWB/RTLS-данные, телеметрию оборудования и формировать Локальные уведомления. |
| Управление | Передавать локальные команды кормушкам, поилкам и системе фильтрации воды. |
| Синхронизация | Накапливать локальные операционные данные и синхронизировать их с Центральной системой управления фермами после восстановления связи. |
| ERP | Получать сведения о персонале, аутентификации, авторизации и остатках корма из ERP-системы 1С:Агро. |
| Метрики | Передавать базовые и пользовательские метрики внешним системам. |

### Нефункциональные

| Категория | Требование | Критичность |
| --- | --- | --- |
| Автономность | Локальные оповещения и управление оборудованием не зависят от WAN и внешних корпоративных систем. | Высокая |
| Оповещения | Путь от видеоаналитики до Локального уведомления занимает не более 5 секунд. | Высокая |
| Синхронизация | Повторная передача накопленных записей не создаёт дубликаты. | Высокая |
| Расширяемость | Новые центральные потребители событий и метрик можно подключать без изменения локального пути фермы. | Высокая |

---

## 3. Решение

### Описание

Ограниченные контексты, которые направляют дальнейшую декомпозицию на микросервисы, разделены между Локальной системой фермы и Центральной системой управления фермами. Контексты Локальной системы фермы реализуются только локально и продолжают работать при недоступности WAN. Контексты Центральной системы управления фермами реализуются только централизованно. Обмен между этими границами выполняется только через синхронизационный контракт; центральные контексты не участвуют в локальном пути оповещения и управления оборудованием.

#### Локальная система фермы

1. **Farm Operations**
   - Работает локально при недоступности WAN.
   - Хранит операционные данные и синхронизирует их после восстановления связи.
   - Предоставляет локальный интерфейс Дежурному сотруднику фермы.

2. **Video Analytics**
   - Анализирует видеопотоки для определения состояния и поведения животных.
   - Выявляет драки, дистресс, давку, болезни и смертность.

3. **Livestock Identity, Position, Movement, and Counting**
   - Идентифицирует отдельных животных по идентификаторам активных UWB-меток на ушах.
   - Определяет положение животных, траектории движения, переходы между зонами и показатели активности через UWB/RTLS-систему.
   - Выявляет паттерны движения, включая скорость, расстояние, длительную неподвижность, повторяющееся быстрое движение и необычные переходы между зонами.
   - Подсчитывает животных с работающими метками, обнаруженных в зоне покрытия UWB.

4. **Equipment Control**
   - Контролирует кормушки, поилки и системы фильтрации воды.
   - Передаёт команды управления оборудованием.
   - Получает состояние оборудования и результаты выполнения команд.

5. **Operational Messaging**
   - Создаёт Оповещения на основе локальной аналитики и Сигналов оборудования.
   - Доставляет Локальные уведомления Дежурному сотруднику фермы через локальный интерфейс независимо от доступности интернет-связи.
   - Хранит Исходящие уведомления локально и публикует их в Центральную систему управления фермами при доступной связи.

#### Центральная система

6. **Central Farm Management System**
   - Представляет Оператору центрального управления Текущие состояния, Инциденты, историю Инцидентов и состояние оборудования.
   - Доставляет Оператору центрального управления Центральные уведомления.
   - Принимает синхронизированные операционные данные от Локальных операционных приложений ферм.
   - Передаёт Локальным операционным приложениям ферм операционные команды и подтверждения синхронизации.

7. **Access Management**
   - Получает из ERP System 1C:Agro данные о персонале, аутентификации, авторизации и доступе.
   - Применяет роли и права доступа ERP System 1C:Agro для пользователей Central Farm Management System.
   - Контролирует правила доступа к функциям и API Оператора центрального управления.

8. **Feed Inventory and Forecasting**
   - Получает данные об остатках корма.
   - Прогнозирует расход корма.
   - Поддерживает планирование пополнения запасов.

9. **Platform Metrics**
   - Публикует базовые метрики.
   - Поддерживает добавление пользовательских метрик.
   - Передаёт базовые и пользовательские метрики Внешней системе-потребителю метрик.

#### Переиспользуемая система

**ERP System 1C:Agro** остаётся внешней переиспользуемой системой, а не ограниченным контекстом MVP. Она ведёт идентификаторы персонала, аутентифицирует и авторизует пользователей, а также предоставляет Central Farm Management System данные о персонале, аутентификации, авторизации и доступе.

Оба варианта используют одинаковую Локальную систему фермы: Локальный MQTT-брокер передаёт визуальные события, нормализованные сигналы, состояние и запросы управления. Накопленные операционные данные передаются через Центральный MQTT-брокер по MQTT over TLS с QoS 1; команды и подтверждения передаются в обратном направлении через отдельные темы.

Выбран Вариант 1: автономная ферма с прямой центральной интеграцией. Вариант обеспечивает минимальное число центральных зависимостей MVP и прямой экспорт метрик. Вариант 2 сохраняется как альтернатива для увеличения числа центральных потребителей событий и метрик через Брокер сообщений.

<details>
<summary>Платформа мониторинга поголовья MVP — Primary Container</summary>

![Платформа мониторинга поголовья MVP — Primary Container](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTSjn45hwVfr31Ne3NnXxFhy6uQcRigSqMi86OvG8ebpAZE5CPpnYD7CSiL2KE-M53nS1IjHGii5celTdQAkFOn3X_LJo1vXLuaZrzkgLfgLjIo-ECH7QKYfD8VRg_FdtErwTFjvJUXLPdNJVrDQrfdDCtsjQoSScgj5khdJMZOsaD-66OcKpFBRPfwkjljPhMUMrsPeQxlwPVUjNezVHLuqphVKFB5cPugLUCLQFJh7SxLgtRxfhQitMyX19LlN5XrUnkT9gqd6NgdHuK01XYsQxPD4on75VGD1gMtbbj6_FpRPIfc8Q-iA3z9QF1TTPjh25KWi-n0V7PXz8cMALdG5z6rzfQDqXlgIvclI9DtTBFwhqO4kzj61inYiDFgzLehUimWif2kcbsDrbnDaX5lF0EDtI-kt84XjdhTc8qth2Wwh3Yv4dFsW8zwvj6hxjca0haGtsM3AbkRha3BQcnA7bGare1DRCps7HJxvqtcdvbDow0rPz4KoY23QstzLQ7DX_Y4KG38xa846LjSwQInoko7mCrI-HawXTzAMaSwA59Ww1UbpGAzELG4493JqWT0nfCMYSz0moaqp3ETYybRYIKhtWOKTmKuVTHuem-maW4munrizKpLeQm6r4BG_19QY0nQaaD7eTbKWE84OuQdZ0FEmdby9oJ6Wrvw2SzXkGmaHIHZ5MSojKv9AmM5MBgRVKiOoqrf9GAE8oGKe758QDKW65458rbb2IK2iru22WTXkEGZu0cDl74ga65T1eW1Ui_mqmDxwG54QEUjC1ns2SjU1W1AS4P9GUbHJyU6agD4JbC90CKcupYLdBiIzewTOY0n1jhUAQSdorL_TR8tHLqp9B5WevdIhbgaTv_WnOteUFQR05lBRRqrMv7RxExCm7KbWXa2QSbZiCXiyI1C1PJmS7LAhyxb49u2ajSAnxkIWXDeDyRbm01-DgA2h1YcVmIjXK6TvM1bIlarv04a2nsMQOdDO3SL4zg43cvdZY0JeuaP2ho97nIeoTlnYTXM9dmx4eudcosMAuk-l5mTvNnv6gbUcrcIBP8179WzYL17DxCIu90F5cNKJ61J25YrYL1hKWoBYD74nqkg02RmyiFZBkRKBYG2FQmS25nG4Aua3WYVxWmMf9G3XSIrvCyN4W8LjI9a-7owwhRw2eCSS9xDwCxfv8ABEfxDwCxjJBytiIJEBoI2Wt5tCCPvL4MfHsOo1CjiMk98ghXJoGaHbQZWtwIfrz6I5gf0HJhz4iIv9KQJHtZ5CmeIKpv_4eaaHsh9hQh0sAMeVsE7U1-Er--b2zFPullWcfcObud8v2VgzPgbTAiea1ngSwkQav185khSrSfKY2xLFTSvIY1Mjrt5SC5IdNXoeO9PCNozRamWQ94e1ucK9K9v4C4bcG2fH01MNcVPeSKM9G954C4Ad4xBLFhKkX8BqLrOLZJa0nT8Sgy5yD6JUfnrR3wfIvQS1kfP1L985UgLAjpnUoJenbrWTb2gLpckALGBXVpsK214kljubmndylbIuw-gdcfG5xgFKmaAmgKwc7NugnVe5WFlvQoNe5SFUggUWMgaGAEbMGK6u1ky09p2WBrMKvWKKMWo0ceb6U-cWdYOF6gn1Mefa-0wyFCaef0YHCeAfHdviTigHnj6JclUMVpXT0hDqBM-D0VJir6nrFZQ6SSxOoZdN6qenJjO2JYZqNyqKX6Y4J4MCGJZMGK8X5FD99HYKJuQEHuqq4kxfHMLHAWevXHRe158ndLICI9HZAgaOZZh1dLICJnfOpgBExeAMK2bMDsEhvQtO4B233A_e7BAGei2iON8L0Gp3jSe2ewKAY0DvugHGfuNPIHG8Y0dmGepGGAI6Y68pB_vICoBfuoksUxLXgoKsE1H0KASevVcfFFWF90Xuhaf5EcoagP5r1OCde4yg42DL6WeB9GLXTO6awWE3UNTmHg-Tc1kWR_086vkJ8ZqZ96MOM1_efyjINdhopOoz05XIDGuLS06FNFyW8555Za1Ag3Camw5ow0CQzh8pcNneC5wfKoD_ku_mWH25JhqoN03rozNF9TDL60Dusw4FhEYQRXCRvIqNVLHECRWJySpwV3KoabYK11YGDuvilNWihpNEbRRKklyGpqL0Zuewc8v9G_Dei2K96d3WC5Qc6U8X6GhKwNyd5p79KGWNBSjCkJ5W2jImKyWv7nL5yEYDlBOu5ZgpMDfZ4A0TaCKfDwN9vIHBuiIDJApZDUe3PuB5mrBnjarBO51ft9191BuDISKMzWa84O2um5JbmWAZcLOW7Nuo84Qe9FHmYK1TO85wW9l5GAoZKL1LwA6KwDtuiVlnS_VY_-__Mz-F6ByUCNuySlnezVZ8_xfblSJ-ou7_W9HvlFbHSN2y9nzi7TqRusdnJgi8-79GX5-71OWbZy7mrBoolqQT2D_8DX8mSK7LueOglco2y_DkwkChPAaVmwgGWZAVXXGKOQC0bnHXf0-KEDaJE7J57IQ2CDsf877CaXYtffTUYFHoRspkh8a9_WHqf7-CPgMl34-KXf4k-hfaLFyJzMEigtLLM_CnhvikfnAfBy1pNEJtaRLZlYuqVrsRcSmBFSNO5d5-UMAlMIP6rNhvONoyEkbaQELyPVYO9mMIjH50vNHO7WgfJV7KeL7dPAL1zUJae0IfIdTwrUH1QdMfejpmXPdC5TvOXghbR9LORTPHazNecd9ml2yVHaKJ1FZmHnSgiOcIfuJqzM7nvFJm9AbAVtDdzRtmtTJC4tGpVJyBtGpHHyAtGpDTy9tKpDDq97ZoJAnPl6nT_KP3vwl18dJnQ4u-N9ec1k7WbYT9DvilhmE7eIK2B3zdOpObEbMYWKXQK4TrV0j5HOAYrLdrJOFdAyih0zKH1ks9ue2YTiZmB1LIc_EvGgF65xelhmXkq9G66R2y7x2-EZaEEZaEEZa4_lKSXGvnyVXnmVXnmVXnmVXmmw3ccrhBQXsT_Pz-tT_dR_8_jH_xBzmDxLuCSZ-w5zsD_6MuVmzxlazcEuVHT-tuVVU_rF-ZlQgty-hVrw-GljjDbQqyqjhURiBqBrxAkspxpyicP_2x87KDKkhLBhNxSFx8VGADoox_S_WHQmQgr_rNwCD_UWbGC4iVDEvvsE_Idy7Oe2c7jmxpAKE-XlQ_qh8FTZ_nPK2mKdDSGBM8cWVOyLGjoF4VT3MgTzeDb7W-uUaYBtgPHRDSfGpDT1ul0PZQDvlQSrZ8vbml2jwHrzrLY3VwrquRaEhg1v-lu5jFbJ_ohKzB1_o_uPCPEUGplx2EP80zsJbXxqRs5VloDOGU86G48eHE57ZgQWaWEYfpseTnyUF1Xe2XH0jFU8F8C6TgYEhaABE6IaY4Rqw3Pe7qngyES1r7sNx_kXPo1OHKJLH6a8d5H7rNnWF_3hvfnkhgqqDwoj5MDjlTtTCWpyJtZTkOagPaZ3UwSptskzRxoOdLructvD_jtvQN9dWS0HlYy_FyzjvWF6TkkYqREwZGihQzrE2qQjrLbTMM_hrhckkQOnPnGm_rkmsrr00me-x5y1r0VO3PaH3rI-2xyFSPo_G0tYWE8ma0uUJcgLqwSciKfuT0rlepLJGyH1Fi9X9yBx_LjelrvZ9H8K6HP2lo4kGeo8A0St4xrdB7cBbkY3QSEPe8EKyOXubWmBUO2TeJP3y7nCokuxAkXVGvNS4Nfzo0pQLPMq0VfWVr3g9MhNzx3WdWFj26NvAf4BG85xO75NS0ntiR0xTFP1jCQeuxT8NBD2LBG2XcKQcg898Iky-lhoyXHGsMLqTG3zJcTwUc3WvB7MlGWHq_8hPoQrrxjdjAnZxjFJ0QfZ1kEQdjHoY6_jGxUHCkoVG8VCOj6BDiFiv3tBMZbhTYy0Aj6xVt3CNNik-y8B4yyFwQw4KgXN-jhvckT6gJTov3eqX9Lsj-7-Ppq3bd2kK6oXpe6CSOWCzvCxIVdcXI-9tI2pmttxv_wLIOtDR6JmFkPcWVk2Kd2U92P7A7oRF9oKy1GIaUEc1mFt26QZOEVWr1N2ACxbPPOOYbDXbQ-5-LE9GqZFkniMX6kQ_ILOr3LqbHreux1_JJe5sp-_v5S-WWzX5zS9zzYiDp3Yk1gdoWudem1BlUDn6H8g814y4kOFEdDzpi8hZqgf1Tz3WICq6a4N6exLFUZhPnWFNILz9kptoXJE_QHDADg_JLIo2uQ-3OL8qTsNIE22mqEm_IHeRn_Wtu2Ysmo7vkA-GQ8txSstge64Th7LDBeh4CUrjs3Pq7EyEuPl8uKRpLLZ-bHtUQf_4-xUjk-9jXVWYlw0o-qTr1OrtAJLbLRNMbdLBMDJtueDMs7caKyu4kQzYJ7RhYSkXoR8S70pJ-10CBRmpYl1dO4ndNu3viCfB-CZkB-XRecdeGEAFad20y3hsj4-YNEfBP6Wms45Jwn02XnoCHLnIfz9Kl_4Cnn2zpOspwwOLbiqi0_5QUrRUQoDxUzZs7o3-kC9ps-mH6sjazLCpCaD3PimraT8gbVO2kS8_YHBdrt_N7Q71MRmSr9xgpLLRy6qTHTro2qK9yaguQjfPtb6nWignC356zwX9ICS71hINznwa_q7UqvNMvrpfjwpp8s6jH59a9-xZ7Y44JoPXMbFlW6WDrpMxl-r_nayPe5s60RAEWrz3UYw5sdeKzZcDeuuwVqENS3sTwOfts6h3pcA_X8TWa8ZQqo6_o4kbJ-YQm4suu1zkIvA_hrFlTXPEdjMNb6z8ZPkwQizeDCc_6wSxveHdRE_eki_i9ERJlUmNJAXExtvthiQnSdY87156ZdyXj64vIfYR49O0apJYpL1eWB9HBKzI13WB4_2QWJy5M4FV8ohEhgq7XBs9Z2ykNAkrRRe12_b4V9JmYUVesFT14mxGeWyHTIDM0y0s27UXivfrEHmLZ_2JlrZy6-Q4A4iBWNhtbmpB9FCga26lGqmTWvr5COz6Ejx0CJ77T_28nf5FSJe8OeD-hUYDTiX-3g0ftMnPMt9qXLtm40XkaIrxS7VRvCua4mad2w1cdxvZ_sfVHV7_LzG-jelzuUaPUDIgsUHq5bVDRkMqJWVb9nOUkEqRoRKilQT5nquBBL4YEiAJcsWcKaocJn2jeRmOzBDYk4SHZOqWmRNN7U7ySDC9YmNDZK58c6fXHUrYOdaqtiJ4nb9rWh1Hona4F-JJDbXlw-phkqv3UoXTzv44gT1Cngc_MZIJ-ISFYyp3dfuZ2GVQ2Xi63tZ7B06GLNTWUhPcirfKyY_q2Xuq5l9QkYOUI5dd8F74piIkfGwTZSaLg6y4FMlG0CjokcXBxnHDkL_uWhj0MOK47POXG5Bf_WL1Inc0YloHa-HM93tK5UxGDpR_0htCLif714IyAvztbW-SrgQJG5x93FX9uCLKKOQ-20kVzWl1nGOdh4ydH2SzM_vuKAOw0F4DplrvkaeHNyPZ65IOnpBvQSc9UbbOJaahj3zi2LhqihfKmul4b_7mCeDJ7QuJWQCEMTI_6eIEodBNNWd6ktNZtyeu1ZRHtwmqkLm9a2fOJ2-SIOeDkjvkGm9xwuJlapIJICQqEzmQJcCUg0l68hAZI8CC0zAcH_kU5H5auC-R6eOr7bOKDxVGIze7j3kBVb-otEDhcaQRQHlxTJYyqFf9Snfeh4CRqFI1Ol08vZzycRu8I2Xp-z99vyZpohbFbtL1iG0OZ_ixr0GebwIySBUij2RweEoAjIivilIJaQgtq-aj6ACSzXYfNyxZlzuDFY4Unmsv46BlQTjr8Fw6MlWF_kD3RmAzScssTZCiSisCWT0bAFVEDkA6Ha4jVNSjyS44u8Xjfw-Kj_hZ0dbEDFca6X2Had5c8G41nqbRo9bMf95UQlosalGlrQRj0znT_HuSBa1p_ouYxi3xbBeiP29sCNj87_yUPtwCPbHtMCXpbxORNhsWQwtdEIek_8NelcbCylJVrXUFWqwH8Sbb7X4DgLkq7O9EyInlCWK5_R_l_OHTFCnKkisdu7u4PSr9AzwYzl88g1OD3XOsPEUaJ8F-0ISlt_OjUa_dldJQrmdCkSXGjPwxg6oJDjODJfDpMojdhSoyyzoNtgOvF-nyC5yWvmOU_5P-itTXOc9LwZanCJyD3vQq3x8V33FJfejkCNnSEg2Dl_cxsSB5RZl57OBu8dN1NQEKMFuPnOOPc_bF2Ad0le1ZP0Md09i59nqMaHJz7LujnNw5dn8A_JL-WNDJVLU9GVRPA5eUBi-_T73_I7jqdU3e5rtYulQZoJZn6rnnOL1tniCwpw-Z0WmwCnu5tlzW6n2qBPlq5qWCEidrFtlwIOJ8IwM-gAvlh1Dg70ez7qBTIZqtOq5vTHmIElWMDGnC_y3)
</details>

[Исходный код основной C2-диаграммы](02-01-primary.c4.container.puml)

<details>
<summary>Платформа мониторинга поголовья MVP — Alternative Container</summary>

![Платформа мониторинга поголовья MVP — Alternative Container](https://puml.livingmodel.dev/a9f2k7xq/svg/xLrRRzp65Nxdho2QFYIg56crLzcAWDsL5WrW5xQbr0zn850hMbvujLIub6MdCU1BVAi3gqu3D70Jfsw2llH5aQrObYqPo2zO_eNyafuvCzmTaaDog9MNZBiKB4lSEJFVENFEDsSEBsfVRBJMD5DRLPVryzfwqrhGBrlbPcEbjQgtBBKEttHJKThTPXkcjdQsiMnTK2Sd9hZpgzhbatgxhQteyutFT3MJd-2bJkWhUckvPhIigj4qJFMT6Xw-5XMjVd75DDPRoxITPMgjDZG06FwsXhcic_vsN4DJhrjQQwMfJqytKQPiwjhCZFeNHOLZgJjOtTS9Veu3-3-xAXo2TJeFzj8DwygQJhIbjfXqYonhbhQajMD8VBgkhyTe3j-jHhsnfi4CIWjffcbiiEPiaWfuu1bkwZndnGXqissqOWnUjw3hiERaaxPr1UoiRUXjOrKd7OYdUebCgMPUwKwqeCU2u8Egq0kecytZqCjQ-uA-x3Ls_I9u_L4CXI8uq5fJQxJey24HGImma8C0aROsvoevF2Brw9bP82Solp-MaiQ14Peq21frIQF0M0u5uHle3PcZHuD9swIjWuEqwlgISJbn9w5ynSE8ugQ8k8yIP_GHHYAOPgoPZRQ-s8VV-AtG1v-a1XAZbjJWSLWcDO0OuSZX2Okmep0EppcfiP2BVjBZI0uJ2H69MCNfN9v3mdgH8QRsbRQbhwQ6b581Xn5IAh0uP9GAC8o8eh4CaeHIOHaN0QN3SHpoyQ69JJonUf0XdJf8mVvFDrF3Esb1nAWdBN0SzaaB7aP0Kd06oK5fiO-BXbBZH0uJYG359gEudHpw48vE0oAWyCO6dYZb9eiLhpToPtqsPiLYduqdYjbAWPu_GvkReBWwcSTJimrjnMXfJNPs8e3Q4e4iuBJ4SJXabZWGnc8oEBXUnMVxCWXFOObRnSLT2Q49Z7lzCY20M5kK0LOeaI_VPONUMMbWfJBvwfC0aiKkglIa1b2tr9CQH4wj9mwWeoC9aObS1P_KsCbTyKaObX9UNGd7ayaKIfLPBnxkh3IURBLOgqxqoHQ9G0wilYI8mrrvIH28g-eoA2OG6SHVTKdGAeAAoy3HHASBCi2cyF02uyucb2uaWZqiNKWSa2zTI1oHDrqOB4b8fmk9sqcSBYG4A-gEa_xgwxANqMKOueYltGpkFfLKO96_TZEukrRwlpRn9cvUIOM5Obx36UIjB59NO2BlQ8dTInHHzNz7Gc9aDJZeHtdtom19ApM0OjtzaWHvfSPIXxWBPf0a9dt_IYIH7QedTbK7n2nT_JsxWVlZNEbohZIUANm2ffc8UHoDG6wgKgsMYvEI0eMv6ZigMGc1JBN67SL8WSnSpNMKeWIgDSzH21Serdn79amWuszVcmeJA0W4Qc425P50BaHWJYHG314GjVTOja-1MP508KIW75TfaLddGcTwBag5Vbr3C7NvAFUU3PirOSHLmleNXcY-EqWv8qa2sMAvKfagPDuSpSWBJEQBfHB7BVbIgP3B10eKcRQpKuLSDfihElQgv8G2EM7qC96CNw1O2pjc9xq2XLhuCPTn2sHhKKV5BL29572yH94S0DJW1QOa16gJdC2iZ4214v1fps9L2U9W_r6EAr19dm2dmyISZ42H4oX8j6Vkntof5EqPMRTxPtBvq2CxGDQuwasd9gFpgM6sCynsXjdECDkHodOm4_7c8jviHF5b8lvSn9MDA1APY2iRKMGo4JuRETnoa8sxf5Ma1EWiAj83i6n4aSr4d6n4aSr4d61LP3CH9vOKsLNSiLFA1Cg7L3g-MRiJ5v1WbBmJbvKKcFKvNuH0tkVUuG8LVm252hZpgMAaW3j46Gc427X9WD9Ce820PZWYynuU8kFoATDOCgmqLAU60eaAvCKSFpSbNW75YGuLoGgNJ9cTCYyWiMLq2UH21Qf-WRpCHbbUOAu_WSBKLCuHgEOcU-Rg_G82KrCbHgGbpBBoFViL-6wBpeyisLFeXi8HAFCx0Cpw9tc1F0hCSWALNXicdGqNm9otwoFPBeq72zJA9MxrwVuI8X1erhTBW9-uMgdeEQf-0TurQhxKTyhl6YxdAnOyHzNlV0E8XyF5T7WffIWGo4jn0CzykMfGUvuhFRjjuJ5SWTuA0Mys5L6SykPcKG3Auj9Xe40rB5840gBTwLmkResZ729GYbjsUTC2O7MXW6io55VtfO2ylJGKEBHPq-WQWn0GhI1LOSJbA4NaIZw9QidvZ1UezZxs7LKt6oXobmLwoYW1v1AuD2lo0_Gg44E1eS2H2qGLfr8iq8su286gBwOZ14e-rWWNgFfugHXKQohuU2bcEZLyBdxuNFpmkVZ_r-VYXm_63n-C7puOFtmmFkwJRd5ViTDzmKyusbosD3kRzztEtZixsCVcaq8TzlAm1A7uNXoM81RlIyFIyWXz6cmZVc7Om055fnUIs2fPykN5nfsLnbOkaA-ZoZ2IWX-MPAG1ao_FI0CeRwenSEOG6KgORQJ1MkA48pbaKG-jzltooCIUMHqOyYDyIUa0drXD2vuuBobDuddLjDWf_ijA1_cqgknxHYCVLZrCHu8_KEDy5uzXDIDUVbIRdChwU9Oxw-FPsQcvSgqesDlLAgNPKh-xfO7Z5V5NeY2wh9Ke2eUhea1mNOhFzcKA5pibQWyt9oK09IhI3QiTKSMf52TB4xugJk-iT4OrLIrdo_rUPHayNa6a9ml2YVHaKR18ZmHnTBiOaIduI4_M7gv8Jm9ALAItDtvRxmtTIC4xGpVIyBxGZHIyAtGZDUy9tKZDEq87ZoGgn9VrIx-fnNpmU0L1dYm89ygJHS72F1B4u1RpPEtX2lGae4IcxStbY8igbNo-uDjAS6TzcERoSyMvofjAsmUELvIs9mgYcxOdYi99swD0S5sAp_Pb2bVQdgWzt6bx0b3OnONWwml3Mo67jq8ERuLyUs-530t-uUsGmziXXxT33c-73BeTqcfOJLtjVDzvsTcoRzXVT0xiQvrNdIqLlXrqzZkxzWqyjGi_FoMdNyFff_3l9VpRjh-qDzMJVpwb_dhjQxNSH6MinYLThJhN6648zcRRtttmWTfv3F8xqDqMxLQrRtTsElimC9pglBI_X56mUzM-sNcD9xTXf1q4idckTQxL-Hly34q1q7CuTmtQxTWtLFiwoFratuTkeU6eYfW1Bn7iF6UD4FjhnBvF--pig9szdigxfCbBAjNj6cKevjiWiVixbQDwhQtMzPPbmXIkQYrjHL-5tnOD-5o3S51TkdyDO_vituIUzktxdHU8cMWEupn3C7igs9-CzCg-Zxf-Jx22n1s0X528qVSSIq4dEyHEsz3lC_ZWLSzIH1NUWlQceyfFH5iS4QQFsMUdyyghr7dDN5rSNhUkBEghQqtZYgxZNz3jBYnKepwzvbnhkjtuJ3-McPpvcBuA_fFfSN9cXi3nlHn-Ufgx4WyOc-1eRSkeNrnSDLeDC7UZjRAurjIiyuQvgh98yc7-Zy_XjW0DpCMkVHrGxw0Q8k_hJWwSVjNPnGbwx7QDKRLywaEq_NMYkdqBFZgWBaYRs9j4CxoCvxwxaVSZWJ9C5MTf3DBaWFYpsrt9yFG3H4Cd7AB7laVQFiCJu4dsBTJdYG_sBlwyxRGB681-yApNAbfD6YCDaWOi4hqMOIfDNPMSD0oWawSN5iOWwgzXL02aSwtnyPvBaOzLun8a60idvaVLqyQycd4SR7myG5ysJTt97aM2m4dVxpopxpEIE82VRv38WpO_W-BCHz1lDy8czrFBMbmoZOk0IX1GFpeUfhwRUV_zaVVwZ10InJGGFEFya-SHOG1mHDGtXCMcKU_-SPjUO3f7ABPGbyOWtKLI-RbBw1uE_OOu3Zb3pdTUsDT7LROAaDcxvypSg23coGY7YsLEwv1mTWvtMCVGNoQa7qj4Ar7JMBSW9GbMYMeJ7hK1C_h4vLfWtrsCc0C4W6ivMRP1QKggVkSyUM8C5nOY1MUFqpx8iiulXSmRdcEdUtIX5WRZfSQoRYp2Sjsy0XbYst7C66w9zAClh-ZZ7neBO_PTEFkWyvptjyvJn-Oak6uI_T61Ng0bjhqBjEDmKVOdOous36jnHRFq3UrARD0Id2ccIW5btCNrVSkLVC4GX24vpWoS1XOVF2E6AVFHsShu6M3oCJTLzH8bdWL6YNyJ9_cP3sL6tDVjE_P3-mO1XjVPVSZMDvOMJQlfzuohVYfzB4wfSBXdc1tTeRvzn9mAsuZ66iasOnBg5cQ7u0rVe8DUPnde7lnFKjCjBty-mTzmfHjLppR6QWsmxLEqAfV8Zv9ayD6uapw3Qq2-ILTBQBG3GKbxeKw8S33qYRvuUKJqHz0SLHkjywRMjiprkhKUIJPVTTbb3nCrmlnKa-y0w9tkScV_rNu87qEZA0oKBkhQAb1VEz97nt3C6pZZHFjDki6mDySfF-6e-npTVOE-JA6HFG33luzRcI-m8JCoyQzkT97Vdz40TBOshWGxIYlYuvQsqWOUN8P_zGk66Q5Svn5Dyy5FxZhguRXazN6q-S6zwK3QtWdSU4HElwx3hfLYN8Or54YY7MknGGE2uzFSaMRfPFKWD42zuX5P4navltIsVNt2tW1oDXVFDveMNIo5F4A-NnNaW4_0lh2IqtK2Vdu0ls5Umz4UHDOl_uMjy5E4_MzeVUkNbpohzFuW3SyimZzJusKN_NAZRP5qHLin3KklNmZQIisTEUNXOZNJUNBCWSM8cSJVTSmx8Do19hP0fHi-_xxDjktWEDsjOoyP9tDsWCDiEP7LtRKZYvFBBW5p0GfxpE-nzyFUMaXNChAMiRomyu9eSKPRMcfO9qyV5vKhGbR2A6YhUhkjmNhkNlrywmlbpWDVQZUV7Oj800Esur6uCB5hQXU1QWKBtbV8_7TXX4tVlc6Cy06QPmSYPnTFWvbKoec8O0_zujlUxtoY9fv1v-IgRfaaNuFLkhqERCpDQnJ6RI2Zvm34ixWzXe_sCRB9znUHovLzFphqMmHV2_0qBZKiID2Ki0BHs1Y8XUtyZwaZ8ydNl4P654552C57R7KG_w7hMACqgQTxjMkiYaMoQRhETVM-pLJRTWROHe-xYto1BaZfj7CmwbqzENjUOyI_Jtg8MK032uLDeskU0pPHQAedq3qBQcTCkc210aY0w0hRw0oEje8TwY7BDiuy1yydAXAw5poq6WAla0ADNPCiwXOkwvlEUYdqA4T3JqufxSh_Wd6VetFTex33EWpOQCRlA63p5zYHEsMDn8AhDDfg2zAD6_mM7VaSvmL8mBrNFBCmVshKCsjEBOcm-vUz7QGYp3OHbtUl9GOKc1EpYfml8xRlU-52Rku1n0yxzj6fA4D_4unXL6KSwzw2UkiRMvvTAKjjNBjIBsp17XwBp9GNIQnZDjjTR3Sv9GFcd7CfVlUDIdubAAou5nhrrti_vd6EEtjUiCBoWH9Wr30OtpaB55lrt5o6XFVKMP97wPKKkelQv0ftk9A3BfZ_YvqY330FIfeVdhXCHSk77cno6EHvs6UyVu1Tq3rWtFlYQndlrWtJr9j8t-g7i-_rPPUme8d6qhyF2NU9FXvvI7Z4NSk7X8H-_bou-EovjeOlQIaX80Vmwz7v7WKXrIJrGTIMfPFK7fHLeMUvM1Og6Mb-Bv7IZZ7FOGcu_I1E_BWi-8Or3floe1tsspRhGNh6clY7lx69bw4-qGKrjd9iuLYlEI3IsM-S2uYA28BSVkxn8S64O8XjfqVoLpLZGZdCijadcf1Pad4ctwQTpfAxE30kg0oSBV_fEEZNQ9BnPRQ0N3NAjJnkOGcfknDw7QDizY2WRFOWj6pcI-RdvXV6_xYmS0fCYF5A676Ft8XpXuvBo26EutcAM-BaYAyB-JSOvPEdJt53Udgazeog3U6ut_OvIgyWXjrzpUyzXHKsqHZ7rVaiITsviPIMOUdgac5Pngfgd8-854Mm5G3dkKcsMYgfxgXaxh4gGefof5e1ty15Hrr5JgTCwIfpucGufSOjLn7nVKLlBQi4gQ9SW0s5skxUdAOqzHNoeTbOkM0fq-zmRumOvN-P-NowJkuyE_OEVNVlpCZ82IevCZ8zZX_Dg9yhdq-pEzPckitn9jSPTVgZFqpcot3UQTnjW7VEph3x8LLcyOaPPcDtvpmYfmFw8ewG4Pm6lTgvwjJBt-Hmp7qbXw9FyNT7eG_lXun2tmWmetQlVjmanM_HHY5yNCzqqMSIGiQbZpJ1k7zdMS4s3I91FV0QrkyE1OC-xXYcFZv32J1eteANwY6GNpbZtw4NHY4ctf3wFtZl1s2LsB3hik5Gw6CYyb3e8oSpqfURGaQ7EAOyeFmF)
</details>

[Исходный код альтернативной C2-диаграммы](02-02-alternative.c4.container.puml)

### Аргументация

| Критерий | Обоснование |
| --- | --- |
| Автономность фермы | В обоих вариантах Локальные уведомления, обработка видео, UWB/RTLS и управление оборудованием выполняются локально. |
| Синхронизация | Локальное операционное хранилище хранит outbox до подтверждения доставки; MQTT over TLS с QoS 1 обеспечивает асинхронную синхронизацию, а идемпотентная обработка защищает от повторной доставки. |
| ERP | Центральная система использует ERP-систему 1С:Агро только для чтения через REST/HTTPS. |
| Выбор MVP | Прямая передача метрик по HTTPS/JSON уменьшает число центральных контрактов и эксплуатационных зависимостей. |

#### Интеграционные решения

Следующие локальные интеграции применяются в обоих вариантах; центральные интеграции Варианта 1 указаны отдельно.

| Граница | Выбранный подход и технология | Обоснование | Компромисс и риск |
| --- | --- | --- | --- |
| Конечные устройства фермы — Локальный edge/IoT-шлюз | Адаптеры протоколов производителей и промышленных протоколов. | Кормушки, поилки и системы фильтрации воды неоднородны; шлюз изолирует Локальную систему фермы от особенностей конкретных устройств. | Для каждого нового типа оборудования требуется адаптер и проверка контракта производителя. |
| UWB/RTLS-система — Локальный edge/IoT-шлюз | Локальный API производителя. | UWB/RTLS-система является готовым локально развёрнутым решением поставщика; её API — фактическая граница интеграции. | Изменение API, калибровка и поддержка поставщика создают зависимость. |
| Система видеоаналитики — Локальный MQTT-брокер — Локальное операционное приложение фермы | MQTT, QoS 1; публикация и подписка на визуальные события. | Передаются только выявленные признаки, а не видеопотоки. Асинхронная доставка отделяет видеоаналитику от потребителя и поддерживает Локальное уведомление в пределах 5 секунд. | MQTT-брокер — дополнительный локальный контейнер; необходимы управление темами, учётными данными и версиями сообщений. |
| Локальный edge/IoT-шлюз — Локальный MQTT-брокер — Локальное операционное приложение фермы | MQTT, QoS 1; публикация нормализованных сигналов, состояния и результатов команд. | Асинхронная доставка подходит для сигналов и состояния. Буфер необработанных сигналов edge-шлюза сохраняет данные, пока локальное приложение или брокер недоступны. | Доставка имеет семантику как минимум один раз; приложение должно идемпотентно обрабатывать повторные события. |
| Локальное операционное приложение фермы — Локальный MQTT-брокер — Локальный edge/IoT-шлюз | MQTT, QoS 1; запросы управления с correlation ID, последующая публикация результата и состояния. | Управление оборудованием может быть длительным; асинхронный запрос не удерживает синхронное соединение до результата выполнения. | Нужны таймауты, обработка отсутствующего подтверждения и идемпотентность запросов управления. |
| Локальное операционное приложение фермы — Центральный MQTT-брокер — Центральная система управления фермами | MQTT over TLS, QoS 1; store-and-forward, outbox, подтверждение доставки и идемпотентность. | MQTT подходит для нестабильных каналов связи и небольших операционных сообщений. Локальный outbox сохраняет записи до подтверждённой обработки Центральной системой управления фермами. | Доставка имеет семантику как минимум один раз; нужны идемпотентность потребителя, управление темами, учётными данными и мониторинг брокера. |
| Центральная система управления фермами — Центральный MQTT-брокер — Локальное операционное приложение фермы | MQTT over TLS, QoS 1; отдельные темы команд и подтверждений. | Команды и подтверждения передаются асинхронно и не являются частью локального критического пути при недоступности WAN. | Нужны correlation ID, таймауты, повторная доставка и проверка актуальности команд после восстановления связи. |
| Центральная система управления фермами — ERP System 1C:Agro | Read-only REST/HTTPS. | ERP остаётся источником персонала, аутентификации, авторизации, доступа и остатков корма без изменения поведения ERP. | Неполный или изменившийся контракт ERP может ограничить центральные функции. |
| Центральная система управления фермами — Внешняя система-потребитель метрик | Версионированный HTTPS/JSON API метрик. | В основном варианте подтверждён один потребитель; прямая интеграция уменьшает число центральных зависимостей MVP. | Каждый новый потребитель потребует отдельного контракта или последующего перехода к брокерной интеграции. |

#### Альтернативные центральные интеграции

| Граница | Выбранный подход и технология | Обоснование | Компромисс и риск |
| --- | --- | --- | --- |
| Центральная система управления фермами — Брокер сообщений | RabbitMQ, AMQP; публикация событий и метрик после приёма данных через Центральный MQTT-брокер. | Брокер отделяет центральных потребителей от Центральной системы управления фермами и позволяет добавлять новых подписчиков. | Необходимы эксплуатация RabbitMQ, версии контрактов, подтверждения доставки и обработка повторных сообщений. |
| Брокер сообщений — Внешняя система-потребитель метрик | AMQP; доставка метрик. | Потребитель получает метрики через центральную шину, а не через отдельную прямую интеграцию с Центральной системой управления фермами. | Потребитель зависит от доступности брокера и совместимости AMQP-контракта. |
| Брокер сообщений — Озеро данных | AMQP; S3 API; передача данных мониторинга для исторического хранения. | Озеро данных получает данные мониторинга после центральной синхронизации и остаётся вне локального критического пути. | Требуются контроль доступа, изоляция, хранение, резервное копирование и контроль доставки данных. |

#### Последствия

**✅ Положительные:**

- Локальный критический путь не зависит от WAN, Брокера сообщений и Озера данных.
- ERP переиспользуется без изменения её поведения.
- Повторная синхронизация не создаёт дубликаты при соблюдении идемпотентности.

**⚠️ Негативные:**

- Центральная система самостоятельно реализует экспорт метрик и интеграционные контракты.
- Подключение новых центральных потребителей требует новых прямых интеграций.

**↗️ Зависимости:**

- Контракт ERP для персонала, аутентификации, авторизации и остатков корма.
- Поставщик UWB/RTLS-системы и локальные протоколы оборудования.
- Основной и резервный интернет-каналы каждой фермы.

---

### 4. Альтернативы

| Вариант | Плюсы | Минусы | Почему отклонён |
| --- | --- | --- | --- |
| Автономная ферма и Брокер сообщений | Упрощает подключение центральных потребителей; Брокер сообщений доставляет события и метрики, а Озеро данных получает исторические сырые данные. | Требует эксплуатации RabbitMQ, контрактов AMQP, контроля доставки, прав доступа, изоляции и хранения данных. | Не улучшает обязательный локальный путь MVP и увеличивает центральные зависимости. |

---

### 5. Риски

1. **Недоступность WAN или внешних систем**  
   *Меры:* сохранять локальные данные и outbox, передавать Локальные уведомления локально и синхронизировать накопленные записи после восстановления связи.

2. **Неполные или несовместимые данные ERP**  
   *Меры:* согласовать read-only контракт, проверить состав и актуальность данных до запуска MVP.

3. **Неточность UWB/RTLS и видеоаналитики**  
   *Меры:* проверить покрытие, доступность меток, качество видеопотока и характеристики модели на репрезентативных данных ферм.

4. **Потеря или дублирование записей при восстановлении связи**  
   *Меры:* сохранять записи в outbox до подтверждения доставки, применять идемпотентные операции и контролировать результаты повторной синхронизации.

5. **Недоступность Локального MQTT-брокера или Локального операционного приложения фермы при передаче сигналов edge-шлюза**  
   *Меры:* сохранять сигналы в Буфере необработанных сигналов edge-шлюза, использовать MQTT QoS 1, повторять публикацию после восстановления доступности и идемпотентно обрабатывать события по идентификатору события.

6. **Изменение контракта Внешней системы-потребителя метрик**  
   *Меры:* версионировать HTTPS/JSON API метрик и сохранять совместимость контрактов в пределах согласованного периода.

7. **Недоступность Центрального MQTT-брокера или повторная доставка после восстановления WAN**  
   *Меры:* сохранять записи в локальном outbox до подтверждённой обработки, использовать MQTT QoS 1, идемпотентно обрабатывать сообщения, контролировать задержку доставки и доступность брокера.
