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

Выбран Вариант 1: автономная ферма с прямой центральной интеграцией. Вариант обеспечивает минимальное число центральных зависимостей MVP. Вариант 2 сохраняется как альтернатива для увеличения числа центральных потребителей событий и метрик.

<details>
<summary>Платформа мониторинга поголовья MVP — Primary Container</summary>

![Платформа мониторинга поголовья MVP — Primary Container](https://puml.livingmodel.dev/a9f2k7xq/svg/xLrRRnl75NxdhrWaVMXLAXB5gsn504c9g0kd5IodUOW3OKskPS8KgIvNbfN6W2t7bbC7Lf86g96qITq4VIaAgB8KoRePo2yW_q9-IS-ScLtExipUH8cxSRc2PMavP-OxPyxvvinbL-rRzUQoggjBIasxeQuqZAlQ7QFOg2ysbxIceLJXcwOd4cshsAgkBhzNhnatbOdnSUx-adhd7QtTLXUr-Vg7cfBAZlDIbxL5hLchj9f6kTLewSgR5Ro44YMrUcjHRwqqQxISeQlDDXG066BPbbxJTB4SLr3NgeRQN6neKrCDb2dgcZezhVmfeS1rtMgiAbI2dsC3ucTtfKsmIkV1NbhBM5lMYBRK5XDsaPfggDVLTWY9FwveAo6AmtUZNgqlgz23WOLKNM-jikAiat9uuHskwnntvGWqlTrgXcYyQa3LNiN99srZ3Uoihchjrf96Af1tzNNIfQg-PdMqfCQSv8Eor0kecyrZqpMrVLEhEOrTlGLUVnPDeGWsjDnGwqtQl4T440iCv2900bkRSvKCNhvwzCmiaPEPNuobg75WX4ODWaPTr2WmbZr120sTapbwD1YrJTeQE4YpgbrltOdSIIXVyJ3ykCadxlt46NruaGYc6SjwlQqjzE4teXNwu9FOG6BK4Xiy9il41X0Zd61ul2BiB8p3SqviB6IZd_Wuaib4KaGoLZ4h3yuXNhK48QRsMjlGbc93Ih60mmWf5bXCCee564P4_bW6IKAni8oDWEBXE2Rv26X24q-e6eAGJXMaOFwdwR7XdRWWOjGJ5pWc-yG53oEWG706oK5niO-DXcBZH2OJoG259gEmbPpwahPE0y9buOqrF5x8JEHBJc_axWesPel5Wet7y-bIZjw_GelheBWoaSLRCtLriTLK6-pkk0krHG9PmccHup39B78WZCM2uE1gbT_joo0yXKLk5HjtHOJ6DUwrEu00OcqX2B1SWNm9smgzku615Ohagqy2Y1QxRAKdDe2ifPxO83BNUi80EZkIa9d8jk0J6plPLtmYXfNmNbtnnfDAvGgb6ISUxcvWFEbolb8UxvCj8e7ikleI8Gxxoak4G6ohBeDY0fb1n5MN2AqYMN4PE1h_T248i4cydC2ukn6b2v5WzqeN8WSaf0kH8tAc2uCbYS3fGkHsagSB4S7oEt7It_fwq6tq80nnnditWpkd4WiimVTk1dTgfV-zYVCul18B2uNSmndaKPP0Ep2-9rf2r-97LFsVI8YCh0O7_InFlmoGj683ADJfboZ8ApQMEiKfc46IME3pAxv4TggQsAuEY1cQyWjsWFlzJE5EfZ2Modq0fXaFUPsDG6QoL2uNynC11NAp5NQLqW44KkKAT-Lz1LAp5TjLy1CeLnnNpbiWNn6kb9T0IYnVcVGIo4a4Abu29Pb0naDWLYQGzn2GbNTOja-16Pb0paEW65PfcLbdFNjwra-5Vbr3qdLPF_UUyUerQSILlEgNXcZMQgIO2aW2wNonL9hCfSwFPe8BJ6JpXGB7BTb28PT9kGhacROpaxbCEft9c_OgPQG26MdqC96KA92lU5qp4qw1NCNxcatP1T8Llwja5oZv2fXUaWZO0AZ12qm64AXCS08pGGHod42GyYnM4suSB5x5i0Ab-0bmEepF1X78SmAv0EMP-p5VAlXxHjhktgbCrlDA3v0rxZhJgGd_V6gOxGopdM6sCyns0cKxc8auSn5dDf8GCX4n5x5b8ua0cOWj6qa4oKJuREHqmq4wx91M2YH0Up4Hk06MZII2PY9cDf88cecOmPe8cecOiPG8EegRTegPGF6KIeStgzLnBWb6mTbnwO02CuBp-GXa1VVs5YY90UGfOC-dyhu2zX1b9E0Xu2G1IZEk0XAQuOZCUJc8ZCkdzDRrbX67rQcXGAG2MJd7pqxAHq1veaD5qj8XCyXCcHS8C6Mq2MGy1SgYG3R8H3cum6n_0hd9oOmfKCvCzCpL-mK49YSBZ4WBc6LbU_RByTNcpHzPifUW4mfJeCZF033hd-05iYWmmmcKUcMOTDfR07DUoqVI5esx2rIA1MxqiNxn470rw-ibm7TSfP1tN6LHW7UDYf3wJegkONE-VCvnbKNd6q0yd2wclPTIyXA1R20Eu9alKtOhptEbOxOjlOOZq6iXuCocV1QddBcP7u2YVEdGLQ3i5IaI0TdiT3OJTesZw25G2BliSju2O7MfWAqp4hRg2ovvUs4eS6gpnj4r1Y4W6q7AqeZBK8h850I9Si7yZ1SezpuMhhAT3HB1jWLwoYHSoCLrQ4u4Rw2t0Z4K60gSkO3VubIC1Qo8yn4e2p7j8r0GMCDRe2pmKjvjhIady5B8T6huNFpmkVZXS_7_hy_53n-C7puOFtmmVlXWVDWdtSA-OiTwmOytsaowC3EJ5Owpz-uEzh7vg51xlJmiGYZ2YyCYnE9yQLXS7g6FWstaBmmRE23_z28WjbAQV3cnSNS3OolcoDTPPHXHmVTACkA0ISWpuW3AcMeCd3baXf9c6t6mbZpXY0wPtqEhVRyyCh9dLWU6_0nVKZh09rRZWYVCIqgZU5urBdOA_xBIGJwj6lGzexuFgvwc8iaVg36_ozim6ZulFwfCJAO5dkNk2ZmxCpbRhEGbSxjAgJ1Jw7UsD72yClwA58J5Mf6YCBdA3mHNfVnkNwQmiLEazh1pKWHG_2AzPRHzLd5A-Od2kB2AqxiREAEQB1VJnNvtMGQFLnBfqO8m8prQ52pIVK6StIn6PWe-qgErXotIeu3Y5-djrP_NUqDNO_3EqDKul2zqDGRl2bsDpNj2LsFpZj318_4ByPfs-sSrc0yUhoJ8emLXHdcqA5YO-u8Ot60UhJriWHu55D-qlL7psLGfPRCvOIh1tHKmpMPdyxEbyqhR1uvNbhP72i9AsoD5OQRjVY2uAkLt-pA5BMsFr1xsj3q2A8b4GxjXA4PBqUkBDmsbLYTXNc_HlsQTSDzt61wH71wH71wHV7sFI7e6__2OvF2OvF2OvF2OfDinIQDkD3IbystdiBFLNUz-t3dftkiSTROK-7RIEUxiTzVnrZxy_8BSVWMtNy2_G_YttVsqkwcyyuSvvQTxNofpUdr9rTUKihdl2DMpjzs-yVRRIkSRaDs5ghPebKhtKMUtSmoDmgtEOVTJQ06hLheFEg_mvZQqieiWDgyrhpKxdy7FK1J0xC6zUr1kjxkkTE-3t7-xJw1QA9XK42zW9OATFLO8SRz2tCUqpiwkqZdegRjFYXnIAQjgbA6O7u74_XiAH_DgMwbgJKE7xbjIc-gYjWI_BRJWSnL2GR7f_YMq-KFt0THqt7tIUOcOYURGpWw2EL30zgIbe-uJrFLRWXKaDW0IGY46Fp0j1PNi4ZjjGxqxyC5Hpr9W06Az4_8PDB19RVG0MiGk8qKKOaUhmSvkKe7_Tw7k5xpk-xQEO1KHKnEZ8N1I7JNpRkV8QPiRghwqK5inrXQqfUL6QqtJy8_pMcCJDKoVZdUjETMkVwXTI4rClq_VCl_1r1YvCqtW2E-TdvhYDla1OwD-MsiRhUgjXQLMimwzLcykBYmtLED6IrzIM30AcFyb-EqMe043xtVl0-fTL4Fcn3sJRy6_VUpdBz62sA7O3KJ1_QHIdBkKn2hXeuTu6xsPEY9syW5sEn7UwJv1_xKxAv6WoB0GnWq94U94n3YuoMW_UybxjCGUp1hc0ErcZ1CIMJ8iv0DKXleCmVC9ARjZcg3x44toND1wdpcqPIff0_I3dLwfYzIlzx3WjWdj06NvAf4Bm83Ru77tiG-tiB3LTPrTVulHmA-Jl6Q1c6W174lN5D7blpTzGFbbwgstHdxLfm-JEAEkwcZdQvumf2xACKxV41OQhQhrbsrweMDUdjWyFGR3Q1_3_WUBkHra-Oou8D8TtE-yxDvFAepk2Gb-mb7ZeM0K73o8Jn9UMoSV9YN1Yz5f-kvkpsVSGzJTOpXpUOIPUTd3BOJXWeJQryAW4YYmYEQj5GDo6ANp33pg8PJRwMv26_lTXz9ngVFoejFu2DuZvDYOJDYU3WKz9mvhSMfizmZryDJdjf0XuoSejWjDEqcLqlaNBEUmcPHwy1uA7A3J2BPGiA_sGDVFCKbuG7I6HE6TowCu898seMZtAJ795ZZwEXGYHRSkaj4SkuTW-q6mtWx0tu2Ywmo7Ok7U82cDykvx9GpoLOdTRjThMci1afl66kJIRJEwGyGsPZjQRL4RkzIwEjfz37UVTlP4ttC9HMSMOca7jVbrDsbreTuo5XPLGrjLrxnXWnd6hbozF3zg9qb4-XU4GvmJVLPqOpBCW6wMwNT8laJDJmOlCv1iOmufiC-DpSIF7Rr8NHhF20WwhgnUNz2DXjWdTyMHu1jvpeRjxs3wjK5T-8o719Zgr9T9LXnoF41V847-CVBGVPOf7y3_98NUSjB_SpR0m_Uaybvzj587fd-1DkGc74cIRNurPgRvfB-WGaoTx6a2-Yp2mQuX-kBMZkmFk9cgrfitTBLjw2jLOyMNKxwmIEG0Cq4oS55D_Wv0DooYw_wv-pbyp18sBmmqKAlg4Z1SszVHHx7DTUnnelqcdGXrDySeHM2hnnohFSE0eD38N8NXFyOfrySqfsGa3Vw5VaMoBFBx3cRmM-OKp9R1-sb5VDnG5zl0G3NuLxtPqdsKwtn5vn7W9uzDzR1TCWQQsdnddzKWl-wwpcnykr_JO6PDCTPW9B-b-O9yxap3zf7RrtiJJHmOIMAFWByYx84Vu-o0Jj7wXBqAfAWltAWt33ecIdc4V9VmoHSOM8y1qwQGLOuISoFMNG2sZxT1EOMw70w41wZKttg_quar7O6aOAsRIvgXau48oB2z0ixEeVR3k0rzlGT07Dpn3Nn44uzZ773zsA3xnD-oJOAlINPdRjUDDTcqzpauA2HaQBPj-FafIPt8GCBP4gZfntztFkkym7x_9vH--EDXdxIitQct3P9jgejwoz2gDzqckRDNvXmZePBgFBzWmc5B58IuxkFG1fP9ai7a1DaQHkoajRg2OnZPC7FhN7tPwiQxYOJNciheA8XuJL4lA2CZqIyJZOma9AiV2DudzqVyvxdqWte_OgfjcmriOtG-HXAdUIOwPkSawIHopfxt60STdsAn22parOssVdP9sz5Kpw5wDiqnsnJcyJHnx6ahcK2SSdt1x6Vt_aH5F6SVfrP3uXM1vsxEIPkhHvaQEjARmAxy3vpK7E4a765xLUWosmXVaKl-xrgHFNlonOAyXxRQ0k9UvoU5hzXCuO2GmBNcs5YFhQzSdP-JBa20FvAHy5EJoCV64jB41t4vqtuv8DVqZAqK2ibPzuaJBgI93a1yiwFlplaP-faxXgJ2E9PRqabAbYc5wP0ugNN2bim-YtENJ5uaiOw9bPMOR79AklHv2DxkTSFdfbyu5mFimv0rLNIkwx1z44b8AZzzydqMdGOcH0_8yg_J1h-vUdLkVkotyx__NL9aYPSAwD2uhbPlSPgxzEFfDJysHqms33l8LJAW4L9-OQPOR6NjX4qxDgcBUEcftIRJp4ipITC_rkaW2BwpOxgOfQ-j6iSg4sULuDmkZm31AlObUzaEGTYkR2r1cbGZBXwqDsMpbK1eDzZdUzAHvy0sHJcaKrgN141Ki_Cj2aBAIrQ8q1WixyQuPPGADGSBPAcIeRgBi2Yj69CSDbFfFarAtB8tPG0rSHv76TLcmNFMsAlBtMPwhrklk-aPEav_JXqVXADy9qpI3JVvtCOrfmGFCNS4eDDTMPWEUISigzsM9KespgEPp15Bh6W-0OCBPm7VUSIf1Z1nTzLT8RST2I_LGdQXiAQ0gByJPro84j0V0eLjc0DVvyWMo2V1bZpspKqrN4bP7-NNG2ugp2jshROmcOP-tRJs6TWQFgaNacp_Zk8M0b2zlu8iAWYHmirxNWdB3bkuNU9CDX_Hk2NsiSuIc7j5J-c6ENL3SoNKckOBgVli_DKnxBkaWW5AAF20P6WRj5rYv30kwRkUXVh_1nAy--Zvzr0nAoJ_YqObASiJRgE7W6APLcyQJpGZPOxurRP0Rfcu2UcmqhPE8YKQsg86kGQUFavClSczDY39_pBzqLIL73Ewy2PzWUlqoCXbAZaoCZM67qqh7oK-ccB7awQjudYYSLgPUlVN4zaYt3SBMmNmcEGqE_ocCFpZqmoplN8UaLa1VHk3Hmjc0TOBvjgPJrFqsMgh5Vf8jKShz3djQSLQrtr0JaF9pejuj-km7pqfxj4kVT04shNsiropSMu_AomC-X2Qb_huP1l0e2FX2zJwYEmuqBOtw9OFkFKvgVyTtL4YtCVMkUZ2djUEapSKUdW_E1Jw8C1qu7LWZzOXi6XW9lu7)
</details>

[Исходный код основной C2-диаграммы](02-01-primary.c4.container.puml)

<details>
<summary>Платформа мониторинга поголовья MVP — Alternative Container</summary>

![Платформа мониторинга поголовья MVP — Alternative Container](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTSjl65Rx7Ks2QNhIg58d_fADeXgJ8QJfsQbjAVH5dE109onoJX4f2aPN6Cx4Snt7jYUlKCyqqqwHkCxtfZMnBja9BzaoU07o5F4dF_W1O00jW8Ke4aX8UUsHWp-wtPy_vzjl5ZmRNshqDfQzqvPPwHTdiwAlgTRtSQQ_tkcfFbvlmZzgNf85LRAklR5ngj_IhSdfXWJdVLQwVKmS3PLrTQN-aogdS0cjrLbrNUwswrjEhMaVhos_KyU4fKL6QrzRxscQlHShfVQKtW080mrjMwxVKlhSSKx2lDdMbjzvH5nSxsARSLvMb9VbFaWp7cjLOqrC9leuRy5wxmMs2Lhe2_b8rVNj3HRqblaWxJLgAhgmfWmWMVznKDoCKXt_rThEzeS082Xif_RwsHOlJGShZ0vzXXivrbez0xG-qNeJ6cpfK7LGSNHdes-1dPKiTQ5qLLS0VwZKqf4f_snfeJervpeKgDmf8cAtWfblAuAhQSZkxUGsY_YIQmYQueOsEqkwHvWCo0dbW8WS28knj9bIo-0ZjX-rcZXtF_TvSYXi7pj2uGP2iYni5pkL049w6JiaTDWt6xPE12W7IQwfhslNOWuJm5GiZZ9j2yZxCdD9748bWcR7HRm_KnXXnu_N267oI64YKMXA3nsIPn02YX2E69oZ3Ji8vBESanaCE-abE89bCn4N4OnMpSd4E2Qf5X9W6smDTxIQ6b181Xn9I8h2OP9G8C9I8mh5CaeGIuHa70IKZS4poyQ29J3oHQX0XdIPOmVfFxIU6TvA2Y59FKk2Ox9CKF9I0XE1CaeEIuXy73IKcY4mcuW6AJ4PHApdsmMsT98JFnXjjUA6OJHSgxcXapdfyJ3UBFJvUA6GgUNB-8YdUXuxBwHm-jTnMrhMUqg5d5toeBHR874wB7OT9Ph43eImcWeEfbNzsB8Um51QxLnpS5HCQtxnNhmC2oBM62B1y4Vtnt5Qmpme3AvRHdp4985xil9sUn02ojdeIWyZSwua2wEH8WkSYnuPFOlpatF696POKlBiIZ2ULoXShosuyp5bXF9bgeLvT69CjOe7ik_iI8mxdpakCG1oxBfDY0fv3lBikCNg5i-CoSJJXSa44M0aVRc3CsPZaGYpOW-H2x80ySY5sH6wvC5cI49OBiVk9BnTYWXNsnCbu--kYjz556EA4xzrCxZcLH60HltSpkQTMnhytSHeFhoJ2Gn7luKpoKHQXEp2XJxH4hYMCgCP_8Y4sifeSz9Dy-cM2f9KOG96UVec3l1BZgMCy1JD94XD-VYMKo8vL4xsh0sQwAl-IFi3zVhPuFLkSJ-K_0DSiH3nEnY1RgbIhvK9Qq21VgzEZaX4mI5Nhp548DKZLwewZ66PGhRkEVB11eUuvKa46AM_vUYd88Cyng0SPL7W6sG236i-W4630A-_ox9W6opo3V811EMgdUMwj1OvqBQmBuuO6P-XoOU6z73Hgt8oh1DNFJT6SrKWv9KW2cKAvKYdbKwT7C-86wLoXM6Iu9LSivhCfNuC2xUroAP_DPB85qr-LBDSWoyqUQfBo6XJgGSToscsGhmSVjPJJ85CFEofEWqgeWHabac03q0tMe2HWK4yp1iiY1dd6GAGypLN9ZuEzHpcgGILv1amV5ceY1WN682zGdeOVZQLYU6HadE6TokO2Zym4MUE6Mqwbm_NKLErCrSvKxKpLZf3QmKh4hKNSQaJoA16l5d6e4Kb0YJZKY2IYH5WrShpf81DrIgi86P1Hb8GRe6f44bKYfXgHH9M8cQoIg18nSqaIdSLDFoNCe7pCJaTtgpLmFWAZw1wuZA31iYVuGWnodl0EDgXu4oZGmAcd2g46pXIb911Wu2O1GZE-1XoQOOZCVRY8ZD5JVMrDqvEmEpKrYDKWn-VuMea_0_A53Z79SATCaPKoQo2mP7GOP0CDgbw3dCX2MToWDfv1lbJAcWRLRDfsb_qVC2YLYfH8YrXbvMp_vTbg2-QFLEn9P45X6fJP5G1M_MdM88SDbXc3Ybs6McU23R3cjM8aOz6ulq6zN6Hc7-i_0GQ-Rlrv6R03LoyMN4VLQy26HjqZVKlUq704No7lEghUu9j0FXmlfuEtKWeSWvmG1x3CbwtwbMUvqhNQvXxJ6UXdOU1MKo6RKsvj5WQWpDywz3Me1cKAnu2tEgrbe-vnb0CCYb6tNKxR0BpEDN0CXkIelkYZsujJWsExDOcXCGa3tWnIvMPSbb14jkYng1RDQwn1rRxiEQfEDf34RWlOdP5yoCLtQfR46x1t8AO6KuCJDmZRd4gmWPLn8GPLJqw761Gzh15iKFNmKi5lhodluQM8SchwNlpqlVZfU_7_h-_5Jr-CdxuOFtqnVlfYVDGttQ9-OiVwm4ymscockBoSyppERf-Tx6lpSQ4E-dXOZ50y7mwB4Olxes59UOK-2RxXVp1iue32vOKWjae6_N5ZOyuAOoldqP-JKXXnmGyI6Kd0vD4PIG3bbXeJPmwUexXg8mdUuWkE-925lRGwziSZOtjdTMB8J_0ZfHDyOpKfUA9yf3IEzrMJugVe7okTvDkgejyP3NrPzJWLSNv1ZVallmsz4_BnetgwbF7mB7FMmxFBfLgvNk2ixUgLud9ntDNInF7o-2jM41PhnOh2vAem44oL_BDZkSB1Jh7wmyb9CK09otHD7uJiubGAwUA2PnV7FYkig4hLSgOyxbsMoUFbP7gy8CnCZnS5pVHG42UtYk6vWitqUFtXoFGue8Hb-c3hf_NTqAq4V3DqAmdV2zrAmBT2jnBpdT2jn7mZTF98mbAyfNxuavhC9u-Na-JnWZ2pF5uKDCr3GKnkCe_N7uv4ZmDAg6plj49kgbHokRndAS6SzM2gvMg5MkMqPFl4yV9aUwmWBDaUAmfJjeU1OAhadntB5GxP7gi_dB8z1YYIr56lwBAkoVts-bLTRhLHchSryjki9V_x3jD79AUFI4uVaVpvFY8PcFpJnoIdZqbE7vESFYRfzvYatjOxgcnyQxmqTaSxeq-DLwDFZ4DZLuP_NXb7ni5e1vywW9-VeTEluVGp-FiI_ZuTtHyzaC_zuRpyuoUFv787TqPlVwZALVFU8pH1ltZxYxVVberlm7uVgjibrSgZpunzumWQXbF6oz5zQ0LNBuzk6Q_noQVGqZu6yk1oxtBF-0ly34K1q0iuzmcKsn_joAEROFTaT0-gXOApCiOCU96XyO8MmjXVO-n7f4vZNpQ6TfSFK96NnCgg6jiGp9-1nS4lP8RgbO7SL7jw7uQmg_IKTRKB_sjeS5s1T90TVNy4RJuVtOAQZaRtZEynPjHpQ6SFWndAu7_Kqk7e7kxhjmWhMDm1I1WASlhGz1HKie_yz1JgtOCBXxQdK5TO3uuUc5rvWdgBMuJXe_xPDmxTdRgYzBkDrgQ-tL2x6nrjMrNnRzQr9XRIep6Zvd9lST3-I3sJIY-zJpuH_y7YF3gpXE1uFXg_kCZSeGUC7GYqWQurhpMwMgyDxcxtrXiR7KM_elMxCiqa3-R_U09k5z30M1oCRWBgVTmDNlHPWmEd3us3oxtvUHl-9NLDBfz_nyG_Fu_7xn5sC1weMHbCKVyFPb71MTmYNBgDJ-Ce9G66etU8XnCPxutkuRXq1Y6o8C1n8PmFAFHHS23VuHk8pkSgsKZbk8iwovny_Npr2cKC3mkwW3j3GWhXkOlAxfak6DrsUipu9rGEWUdkpqFZaSkBZpsEEi0_FxMzoyL4BkpPfS8zGnAtZRH90xcY0V7LvuR-TsOioBzAlVdcpA_7Z6MKRoHaNUrypMOyDrGPoXWRGgEZDQrVJsEda7a4OWiC3PnE1pZzdrlKoq9tX0QdOewzAqo-Gg6AAaJL6j-FRiww40RmvImdut7E-YQfVomnVWn8DFDm9bm4ZqPBDcyOSHBj6u4SGcIaRUgWNcJZAmZ3sr1k3xBuBc2upVMkyRs_huYR_B8kCFWUEx80cZs0PlUXOJSX7eIwujpPEJnlxM56kezcgrq8enteyKFa3fURlgIIWWqh6fylSDj3HDW7Bas2St2BuuCFsorLQu1uw6o3NXsOoHSXzR2QK5lhwlmxskhSw7Cu-uNnmjdj6boMrnLTtLAs-ObZF3E77f7CBUHS7FRVOpS_XMH0XLvYX-_ZQnoXOQPEs90Zj8ssfZSefE3kldUfCdznzUpAd9FxN0FxLoIdyBo04-Ub9vS0v3vunuy-ziXi1oryIcPIQpf-JPNBSviPe3GAEPlCY7X4oN1pT0VYPWUv0D_UzxXWSskjqTSxtg6-8VaE-9TMV0-n_a0yHqJYFw3RTwovOVJdqKEuJ7K0lYF1-xrZLjmrbIuatq2M0dm_HqRSYki7H3EF7ioJiCSj7fd9Xdo9nuZ0GhAMOdSwboX462xiEgHuqF_ti6RS9UuBWe-Sg2lh0yYS5lnjNjNw0Rq0RkCgNxVY_ZjHiX3-dwFUqB5756idjMlsFK0HHaGIMI38E091k8Lr0SyKi1niRw1L0m5_W-pekCaE0073aqo4dxz0V8lGCsX0AFxmNrWUFSDU_HUKllt3IrOJsB-yXDK5t5zfujO6wlNsG4VJc0BhIrrjNlMJrxMBvrr9AgUCnsTCM7HLWuBt9fvW83t1nxlGfHtVx0DYiPOJrlnBH35kPjSCJMibXrCFRT7xE1uwxCezbxyVOXyE4L12uuZ8xvhp6Uh5HMLjhQsVk_0MRmcBK_5p4icEQG1u6W6r2DGOXa7jgeC1B2iRQttjcjhtWViD-0L5yIkN1BZSMydCuVQVuZPlOWo7Vle1hjz3j08JELjF64GyPNUKQp1Ztv0un7M4MVS8H89RlCwXxCAUs8VqF20J39g06GVn5y9Fu8SNU2rvZ-5UvFqN9BtDkg318U5dfawoY2UpCJz0p9DTLU-ZIHaOVB09J6c5ZHIqx9-RWsg2Lbjdv9aPePqCmJs9cHddNFZRrTNpA_O49t6MynXZY5R5C3s-__gqDY7SlU2e72p77IgVwMMGnnr3uYCuSNSV0pQQ6XGmPn70MO0o6jdTZvKBPzrZuUzp33HG5_l0vAlqO2nE3O2Rzf6ZD1JsSQv2Ip05xR2g-pKLNqDlC5vSNJa_wzPZT2MBKl8-0uWxgsDSRj741-GhDEWsYaTcZEwvuOBWyNZ8uuu8sMCZV6YwpBrwSEVy7aeMxsgGxeloKkt7kz_bi3-D8PGuClQ2Sod8_QoJcM3qA7P-zuubbGKIltp-7RB8UeKJ42iiQyPum7J11-tn1-p8L1MKgksy1nt6MnHzuUxEkGkhgpAPPyNx8KuJWYFZt5FaxN2A3KCKNzlYLWIJFOtRius8VurliJz10QErB-uDdIsmeoalGRqdv_dGoPsxQMdEtawcDbvOEofxICAHrDX1UGxcpmcZcSH09IzS7FfC-_E_M_dzkxCc5HvIPYQa2B4SPRuyLhxmnvJR8N_yKL0oQXuZ-XiqZcCRI_2dfcnvxLbsK6-YL0XSJdWcjDhAwZp6YZT5YEuS8dA-Gppjt5u8xu5Bv1E_0gmdc7A6jZ8Cro4S-UWGlkw-bCzTECz0TI4X_WqJmnZl8oOqBQatRjFsqe8RQ-Q2XTjl4UP04DwILp8kNfDwiCYLrpHTrxgoTYKa3oIE8gOt6b74t_NJm_PywAk89RHuaj6g5qsXqDq7M42ZVvWbghd8T2nIP-rbuQvZEMVEH9VAxxufgRsMZ3ehILT1wCW3wxwpr57NqKNyh9cq-0RpakWi-v-bZnURwARocJV8v_gMPcRE4ikPcSLvV6b9_bZwU97UZ5woYkFdLvRanVT-aywLuRnPs2g07ufPeeywo7JG5fReC3ahPn6P5P1tRmDQC0lOMo2okUrncitHh15iYhouPxL2tjSBQ8My1BWaewqil2rn2-rXyEvd1hVh6Yxo6690k-GfPcZNseObYS2vPSrimqQ5GTvm2Ee-Vcq2O11DWFUqWG4EpRP1Lv1jaDFg_h_T6-VqmG94tP_YMUaclUa_98C3ruEXa2T3nQ6GfqoNX7SD0rg7F2Oy8FqF)
</details>

[Исходный код альтернативной C2-диаграммы](02-02-alternative.c4.container.puml)

### Аргументация

| Критерий | Обоснование |
| --- | --- |
| Автономность фермы | В обоих вариантах Локальные уведомления, обработка видео, UWB/RTLS и управление оборудованием выполняются локально. |
| Синхронизация | Локальное операционное хранилище хранит outbox до подтверждения доставки; HTTPS-синхронизация идемпотентна. |
| ERP | Центральная система использует ERP-систему 1С:Агро только для чтения через REST/HTTPS. |
| Выбор MVP | Прямая передача метрик по HTTPS/JSON уменьшает число центральных контрактов и эксплуатационных зависимостей. |

#### Интеграционные решения

| Граница | Выбранный подход и технология | Обоснование | Компромисс и риск |
| --- | --- | --- | --- |
| Конечные устройства фермы — Локальный edge/IoT-шлюз | Адаптеры протоколов производителей и промышленных протоколов. | Кормушки, поилки и системы фильтрации воды неоднородны; шлюз изолирует Локальную систему фермы от особенностей конкретных устройств. | Для каждого нового типа оборудования требуется адаптер и проверка контракта производителя. |
| UWB/RTLS-система — Локальный edge/IoT-шлюз | Локальный API производителя. | UWB/RTLS-система является готовым локально развёрнутым решением поставщика; её API — фактическая граница интеграции. | Изменение API, калибровка и поддержка поставщика создают зависимость. |
| Система видеоаналитики — Локальное операционное приложение фермы | Локальный HTTPS/JSON callback с визуальным событием. | Передаются только выявленные признаки, а не видеопотоки; один локальный потребитель получает событие для Локального уведомления в пределах 5 секунд. | Приложение и видеоаналитика связаны прямым контрактом; версию события необходимо контролировать. |
| Локальный edge/IoT-шлюз — Локальное операционное приложение фермы | Локальный HTTPS/JSON API приёма событий с идентификатором события, подтверждением приёма и повторной передачей. | Подтверждён только один потребитель нормализованных сигналов. Буфер необработанных сигналов edge-шлюза сохраняет данные, пока локальное приложение недоступно. Это исключает отдельный локальный брокер из MVP. | Доставка имеет семантику как минимум один раз; приложение должно идемпотентно обрабатывать повторные события. |
| Локальное операционное приложение фермы — Локальный edge/IoT-шлюз | Локальный HTTPS/JSON command API. | Управление оборудованием требует запроса, подтверждения принятия команды и результата выполнения. | Долгие операции оборудования нельзя удерживать в HTTP-запросе; их результат передаётся как последующее состояние или событие. |
| Локальное операционное приложение фермы — Центральная система управления фермами | HTTPS/JSON, store-and-forward, outbox, подтверждение доставки и идемпотентность. | Ферма продолжает работу без WAN и передаёт накопленные данные после восстановления связи. | Синхронизированное состояние может отставать от локального до восстановления связи. |
| Центральная система управления фермами — ERP System 1C:Agro | Read-only REST/HTTPS. | ERP остаётся источником персонала, аутентификации, авторизации, доступа и остатков корма без изменения поведения ERP. | Неполный или изменившийся контракт ERP может ограничить центральные функции. |
| Центральная система управления фермами — Внешняя система-потребитель метрик | Версионированный HTTPS/JSON API метрик. | В основном варианте подтверждён один потребитель; прямая интеграция уменьшает число центральных зависимостей MVP. | Каждый новый потребитель потребует отдельного контракта или последующего перехода к брокерной интеграции. |

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

5. **Недоступность Локального операционного приложения фермы при передаче сигналов edge-шлюза**  
   *Меры:* сохранять сигналы в Буфере необработанных сигналов edge-шлюза, повторять доставку после подтверждения приёма и идемпотентно обрабатывать события по идентификатору события.

6. **Изменение контракта Внешней системы-потребителя метрик**  
   *Меры:* версионировать HTTPS/JSON API метрик и сохранять совместимость контрактов в пределах согласованного периода.
