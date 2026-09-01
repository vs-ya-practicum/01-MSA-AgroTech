# ADR-001: Основа Центральной платформы для MVP

**Статус:** ⚠️ На рассмотрении  
**Участники:** Владелец продукта; архитектор решения  
**Дата:** 2026-09-01

---

## 1. Контекст

MVP-система свиноферм должна собирать данные нескольких ферм. Оператор должен контролировать фермы и анализировать События фермы.

Каждый Агент фермы должен работать при недоступности интернет-соединения. Он должен сохранять данные локально и отправлять их после восстановления связи.

Требуется сравнить два варианта реализации Центральной платформы. Основной вариант создаёт Центральную платформу с новым API синхронизации ферм и новым Центральным хранилищем. В альтернативном варианте Центральная платформа использует и адаптирует Существующий IoT-шлюз и TimescaleDB. Существующий IoT-шлюз предоставляет API синхронизации ферм, а TimescaleDB расширяется данными и схемой свиноводческих ферм. Обе системы остаются внешними для Центральной платформы.

## 2. Требования

### Функциональные

| Категория             | Требование                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| Мониторинг            | Оператор контролирует несколько ферм и анализирует События фермы.                                       |
| Локальные уведомления | Дежурный сотрудник фермы получает Локальные уведомления и реагирует на проблемы фермы.                  |
| Устройства            | Система получает телеметрию и видео от устройств и камер фермы. Система передаёт им команды управления. |
| Синхронизация         | Агент фермы сохраняет данные локально и синхронизирует их после восстановления связи.                   |
| Интеграция            | Центральная платформа публикует Опубликованные метрики в Существующий Kafka.                            |

### Нефункциональные

| Категория     | Требование                                                                                                                                                        | Критичность |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| Автономность  | Локальные уведомления и управление устройствами не зависят от интернет-соединения.                                                                                | Высокая     |
| Изоляция      | Изменения MVP не должны нарушать работу Существующего IoT-шлюза.                                                                                                  | Высокая     |
| Оповещение    | От обнаружения нештатной ситуации Видеоаналитикой до Локального уведомления проходит не более 5 секунд.                                                           | Высокая     |
| Синхронизация | Задержка синхронизации между Агентом фермы и Центральной платформой не превышает 10 минут при нормальном соединении. Ограничение не учитывает проблемы со связью. | Высокая     |

## 3. Основной вариант

### Описание

Основной вариант создаёт Центральную платформу. Её API синхронизации ферм принимает данные, синхронизированные Агентами фермы.

Агент фермы включает Локальный шлюз устройств и Интеграцию с камерами. Локальный шлюз устройств отправляет команды управления, а Агент фермы отправляет Локальные уведомления независимо от интернет-соединения.

Существующий Kafka остаётся асинхронным корпоративным каналом для Опубликованных метрик. Kafka не участвует в Локальных уведомлениях, управлении устройствами или работе Агента фермы при отсутствии интернет-соединения.

### Контекстная диаграмма

[Исходный код основной C1-диаграммы](01-01-primary.c4.context.puml)

<details>
<summary>MVP-система свиноферм — System Context</summary>

![MVP-система свиноферм — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtHRkF65NtdLn1fdnRTr98eacet0IHAUca31DsYVMYAW9QvZh2ov99qrcuHm6itRHTRPFkGfnHe6nJeitPZfRATbN-1_8L-IUyCX-AGCoI7bbSaDXm3jZMSCtFkdJk7bqFITXwFfuUcRHx8UzOZysZY_j8wThkJyVxqm9gwyWY-MROaEUjcJsppyDVZFVSJkRcpGzKVcCSVM8vZxbiFnx-tv4PxXqRztDgtfdl3sTGrPfEPBVzWY0lJecUE7k_RiwFfNj3EjSsf0ms01jjsPkzPDjkEQcXR8zUSxa-i1myc6DErBVFzz-K_I3AKtVLW8wOJV1mFm1xxZ3i4wVGX-CkQkIU75h8sy4Kp3japNNFNT0eWVdTa7HLe3j_TyMXyQC8C2eDCsvuz8St99Acuu1fgwXArV0QMxSoc1GOVkT1rLdDqn75FmC_c4ykP7LYe0_vKxw8fDUsJzKHpUbGv1mnk50HXzX0FlMSwdrXxIMUF7aFKtyLG6887EfoOusamVCQAG1xOIa54XBrDXOg2IwuTaPivE9xxsRLKDW-yGiic4QowibdWjPn9WXde3RaZai6oVU9O423JaRKxEouzI0AzecdaQLFEki-34_d84X6SPXpQOyVwU8EuORsmWPvKXXAHbihm2LMcCeI8u8ZnoLfXT-4SMdCguw6O_5Gda48buZBYgKhOkRY6PFKY8apEYUDQ1vKHfKhG8O9K2IwX65M2316YV2xR5A5AU2OcGDK8d51y63Q5XQTG3oAYCm8KNFzPTcLqfogCYFHKXKweFbNXGmH8YCusDQWg_ed9K6M2A5GY7g72ObIqariNxkZ1WaZPU2C3x-XAK-ibev6gPNnCDeiP7-zehPuQrFyYQ6w3uNApZQlwOtD_DZKdf7OdHTfAeSpHjD9vX69MEX6YOY8yg5xvjHivX9QmqhqIqwwIsAIkU-iO6C1Q-rY4cAgZB-QsGbGhJ4plegyD1Q1SxhoTdieGMc_rL8PHkDTJXD3TYGJFHR4DdyhuARxZKp8jANjt9PjFew7glNwI3rKhpATbQ4DZPqErA8LowkvBYJpYEoyb4edjkcnB2NWEONTTIlGAPyTbwspoqmKHOXrSaiIesfBIXLAuPwKBfHDYqeNI6INJXUsAX72wKBgV-Eb2IRJodZZPR7zTz3QwY4BSyRsRxJsdKWakulTkjlVKokRt9jx4mokLy531UpZRV9H5w0vCxXCjXNl94whDdqWeJQosH_qkdtxPecXLXb2Xfr_A4A_AUEeMJy5iKyI4dr_95R9RzKJkwW3CjUGVaWUuVwFenuh-uuRwMt3DJi5oDm2bqpEChjOK1AY38IczbW2WOGofekK26eDXhEXv06EOA6esG1iofP45QB3jXvqiWCe13BC0FHv0oG0CU00j0y1hd_3iXe0-3w1c0Bf5ZUQvTP0vquCy4pOD3SxKjVF2kvyrQzmLrylgdxj4s-j1kWr14MXftLwlepRUdCo80vfjJTSfRMdhkgeqKW4QiRRVKPLMIz52V_KKBa3XhXu2QR00RPXL-iqaG1rcbq4Z3cWDyqel3kZb0i8eaGG703DeG4S0C6nIWBu8GAK08kt9MfNID9WjtQA0NlK0b0qx0n60HW5KWVOa_4WiwVcHqOg7TqDfPvRM5bNZisGwrSpFf-fifyvswcodpdQ4iXsSYIHpaMGs8Z6P29kBnB8HII0JYMKZaaWcGcSZjpiTj8gUqdf2W60M9U41I3OYYMOYOJOYYMOYuMALH3EHS2r9ecVnq4yL0tHlQNHnjwudBYN1q9CJrn84z9dWom6qcV3E1lJO1PG9YETJMYuWlaI926G0aY8Go4mgW2Cpb90bIqB8g7pAdktEt2hiJjM0KW5jliOFElmp83_H2I0jxYbJv4gP1WXSCiO0IYR0O05jaGjbSS1WCu3QwIWXm52QaRkY3m3eT7GYf3hEijgH_rIwMotybIHxKd114GAwz1K0plgRDA2D0NqAq8lQ47Gh6u1ptdMCjDOodWuOTdNgxBF-a058TUlR1Q0dRgXhYMAm03eqXapgss538nPycfee1XjyMrWFjrlJsLif6WVG5j80MlaK8wqzhPM9gsrkgSz0RmKWcKtbR4ubSxCy0bt-rc4gmCXQAHm0x-fqe1JTu-Xc0FIYsovl6W1Uvm9Yao75kjTJydQz1jpQhLKCZMq0U6SGWxlYb40Y59r167fuZ0OOqM6c674raCHk2qJ6I2dYadfgbiG7Y7OWQa0Dk7D0tkPKXG7h5PS3C9WrdGFG6TN81XYCBcbfUqqgeqi5qwdwlVZwlVZwlVZlwtlnzOlnzOlnzOlnzOlnHTzqu_-nvVINIkfKeaubwbJYxKqbCXT_dKxKwKITJjJfH5ewuOxTYIL_yAiFx_jFlQN_r3_p5jvttbo6Zw-yfVVQM_b_z1R-gVUT_B_JB-M7-9LNsG02G5wIo6jVxxptdkpzmxl1BUVGoSe__MZwqTJxobjvh_qpg5rvrp30gNyERSvaw7W1GroXQl-lwASSZlCZsPl3uJbgx_sNndmDhL_wpw7xfVzYtTv_BdjB6NS5eteNc7M4yi-HLOWM6XCTyXQoTuCPlOGnmDntP4gzJKUUmGSJ_N-Hc7bVGjDleRTJCEYvTuayj48TmpWNkDihYX8s_bymvalK27uIe-VhDju23Bt1zf_xVmvR_1qghf3ju1Gqp0BumqGiO8WLOAu32y1wC1bwMK3JR-9kdIUC0_T4j9AMFZBjW_wHUp8uE9pCJYmB_vtPw3_K8Bk5OWER-nz4ZhWQyuT8U85il4OX0KrNsEuvdgw_0U-WEStltRG3eL4NE8w-XGw_GEPUnznrxdyXuz6nq_tp00I3eKzFuV0PDW9nXyeb6WJtZsBn31cvmevUXe52tFmKD_qSZ_ywYic0mGMEgs2gq7H2wslqwolqAnuE7L_1vsZkaF_y5wYVTz64hFzg1ap8uNX_27D2bXax6zw_IJIll4jYp9o4_np5511xdWp3hmB70-E_GBCb2Rib4CAVwCaDm4laps0Y_Jy5pdg5VBzUQT5GyP2U-vyZWzY_1nA6clNfU6GvNL0eym3Mc2Ep1duDF9yXSK36OIy_0mgNyi_CHuzDR4zQ2-2Q242yJa4oJd6O04UyQB6AF2DQS8ccy1hDz3mnqMZMBh6kGCKD_ewKRGc-88T817dVH9vADTyw7ZlkUBgFpO29NMlC_VlG83xjFv7ltKjLpcXoxrWeeMAn3Yh0tRidCVAGHlQVMzQna1iJJmyUf5DSu84kY7RWcSCzen6HJe1w1A6-nAvO_JIcpy5olyBqaTlHQW2sDpZQOglxd5sYW5a2vpWjp3GU5m5JwFObdjIhq1rmQeqgiPfGghCs0Ard6EN_)
</details>

### Аргументация

| Критерий           | Обоснование                                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Изоляция           | Центральная платформа отделяет модель, API и эксплуатацию MVP от Существующего IoT-шлюза.                                                                                      |
| Автономность фермы | Локальный буфер сохраняет телеметрию, События фермы и состояние синхронизации. Локальные уведомления и управление устройствами выполняются при отсутствии интернет-соединения. |
| Интеграция         | Kafka передаёт Опубликованные метрики корпоративным потребителям. Он не участвует в Локальных уведомлениях и управлении устройствами.                                          |

#### Последствия

**✅ Положительные:**

- MVP не изменяет Существующий IoT-шлюз.
- Команда определяет API синхронизации ферм, модель данных и правила хранения для фермерского домена.
- Отказ интернет-соединения не блокирует Локальные уведомления и управление устройствами.
- Центральная платформа изолирует домен MVP и поддерживает бесшовное развитие в SaaS-платформу.

**⚠️ Негативные:**

- MVP дублирует часть возможностей Существующего IoT-шлюза и TimescaleDB.

**↗️ Зависимости:**

- Существующий Kafka и контракты Опубликованных метрик.
- Локальная инфраструктура, устройства и камеры каждой фермы.

### Компромиссы

Основной вариант изолирует новый домен MVP от Существующего IoT-шлюза и позволяет самостоятельно определить API синхронизации ферм, модель данных, правила хранения, правила доступа и операционный мониторинг. Он позволяет комфортно надстраивать будущую SaaS-платформу, но для этого приходится реализовать, развернуть, эксплуатировать, контролировать и защитить API синхронизации ферм и Центральное хранилище.

## 4. Альтернатива

### Расширение Существующего IoT-шлюза и TimescaleDB

Альтернативный вариант использует и расширяет Существующий IoT-шлюз и TimescaleDB как внешние системы. Существующий IoT-шлюз предоставляет API синхронизации ферм, а TimescaleDB расширяется данными и схемой свиноводческих ферм.

Агент фермы и Существующий Kafka работают так же, как в основном варианте.

### Контекстная диаграмма

[Исходный код альтернативной C1-диаграммы](01-02-alternative.c4.context.puml)

<details>
<summary>Альтернативная MVP-система свиноферм — System Context</summary>

![Альтернативная MVP-система свиноферм — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjl85RxdKn3RNWMRhVL7KMrs0OcIW4MxmA89sejkiQ1bnYj4bboARkmM2zXnlTj55aalzg9eWVwWm5xBZhrHx4H-1UeL-YGzSpYKXfmXERGSaKWv0HABcc_cEsVE-UPeH3hZH_tXhccRE-gMzT3S6pWFh7sdEUXl3tUieQFsu2_BLfJnejjZszpzTN_B-K8jRsmmrtVC_K-iyTZSjkxt_s2ffTe6Y_g5jMqDjxgZeMEC1YDR_L4N6zUZPVOURTkZlU6Mry-npU4OEW0Dlk_8th9ilX_JqRPwZZdS7bZtxWqGqxGjywEFr3ygAhJDnMGzRX1y7oVWt_jIE0KTz3xuonev1xiMiTRpHJa8sJ8TSzCSfq3yRi_QIz4T_dRwlVwk2Iie3J9jU_IOTgUBf673AypIXQwA6LZsU3HCCNdFWQ7ZkfDtniu1-DbyR8r7EnOPGBpKcsH9JVjWiT224JN16uOm2hmmkuzJRvdZBwojiBDxZo3gRsCgXE14kmEpF_IcZya8ue6rD494sjjCg5In9TgnTBC09t8_dqjPyy0CpPg4btLPiy1SZYN1JVIMtB6KmQnzChOWG8OzQtEqdtcGU7h5qaZIfeIyJu9J-OWJ4ImpTkt-sFfyXRZXlR22dkI64fMMtF3nLIOtXAZWoF69ox3RS0whERdnK41-yXD8lX896ObKnHzSNaFYHf4Hfl71sB5sSYDAkQ131IaNN7mnoWKPAaJ9NDOfGhdmJ420yX4ullXmR58BJweHP4Id1oZu_6VPkT6Tl32YqfCNEhxwv8KF5I0fEklKeBpu9o13kGaYNub4X5ABKTf1Rjo4iti94N7mHYVUqAlbUYiSZSnLpiVqi9ZpyKQzqjAywx_qkjjWk5gkuQLstzmU3Sq1lReH8MsPK1PeMkOyV37Bd0XLCHaUpAZYgoivX9MmpBqIqAwCs4JclRKF323NFfSXfkda3_UrmlAgD36zIVwiA03PSXUTzEI6qEAe9pUC_BEUD8HkJoH4BWeS-EJ6Jy4JdunfAV6dA_5yIYLDRxN3V9YhqdmgHhrhRAoe5fbGZZnzoP17yEGbGoA1KvTrAO788VofIuPU4Pouh9rDShaWGwo1BKoCkPfHkP09zxXo8ND2NBcGEQDmkR1UaP0k5pBtaxXSo8XMqXqdgvslotwDBgCGj_pTpVhkKya55_dlRjPtryhgtqsyZPjNSk6XbD_XhFDM5gblO1BlQ4azIf9GhNv7GcPYjJxgjtdtonf5ApU4KjtzaeLuvSPJDxWBPfqY9dt_IgAGtMWa-gqEm1nB_J6zWVitLNs_gl-afFqMNBEHijqEeDfe6KQpNfO4Q9qkRQsA1A1aT9bMJmIKEjr0qvC0HZVKj7X0lSkrKXoWnFVlDk80cW3GZGEqH81g3A0Z0jHZ0AB-8S-k26YB05eCe9dMQ95REx4htKaoOTNG42nTBIcyst6h9iouLjpumXIjBIPfbYH5e59ljbeDhVJsP4OUKAxLTPtHbfgkQzLI9A1EhMqtj6gbKgtx_cfLXO2gC7iefCG3wjsurYw70LetldLAGK2bczHQGK0h4U17YI8v0PZ10XeIW6wP0RHb01e3aEbFSrM9qc2-DTC2Mla3C3PiT6G0TGQWIVIduKTZIK-EZ4eml4lLMcohh54rlWoNK-NaUggeTefgfwXsYcf7gjh1IYHSYuIh4OMhHFXQ952DA1ALIA0QKMGg4ROQkTbsK4cxfRMa0DugAj8Jq6f4aQr4_6f4aQr4_6HLP2iHFvSKsLtSzrFE0CqR6ftUhOk5YoWmzF32LIG1RIxu4W0rBhpZ0Iq-WM81mNggdWW8fYWLWHX0M0GycOa426I64R9m2maPKq_PeysHauVJgGAGAQ0crlXEGxm3YWiT3r8HRfaodvHPWCH7nW2W6WimU41DveEoFA2p6a1hDAe-mAYMb-vQlW10ew5J8TMnogej_QUnmzRz7scnftWVA7n0a_q4W5L_cGNK4D1c0Arb7ugkn0Emvbt4I6KXuz60RbDdTf_5YnX0f5lVBG2xS5sz7ce63s13eykLlWq-D0B1LzT2pU23RmtvSBESZZzAgGi0DIaDO9MlQaJrPxKoz6bRs8eTw9q0XAkfXCEfS6sMHA0fFZgC11XncI80Y3wTTgffppYQCG0zxR7BsmQ0ruM0m68eWU7rYBfTBm0tTciUGsCT0D4EOWWphkf9H5Nd48RklyS2ZENRN3E2Qg38VIsmD4Q949V8hLcHds1v0b402i2j0v8Eft8CM6HS0i3WSZe1e7Eg4GymE5sgHvqrQPmkfIodYkVYY-VYY-VY_r-VYoyUZ2yUZ2yUZ2yUZ4_xf9lubor7FrHIb195AL6K4kzkAH6R_4KvKPGJHJbHb1DHvOJJTmQMwlx5lPf_Et_YNimFtJVk17wQkcVuqtFrarzzUdT-v4xdHwI3-zgTgF3o37gySMVpFo7cjVhVm-_K-_Wyh6e0Ex1CKUWpOUzz-A7g_iEznfvaxDdyyBFXPqFtR-xCVJD_0bTdxXLCS3W_XZvFL1Zu0gQu99Vdtv9_LN-UzrLtGdYH_kvB5lDlw7qwVqge0sc___ofwavL70fcTS-HzH8rFoPM4LfaJlAMUw6wryZe5EO0SzzJ6Matn-e8NfZaFny9cFSTTFq1HWFtpP-whuY7PcHWcESSXxraAA7n_u8vJqad-9SQFLdqSI_0q6kq_tZ-jT_Zxt3XajWEJY7JN01_M8WBc686c2lF0h0UJ8PHBg3hYw1R9o7Zm3rBMc5B7vhsJdlFEUZix0v61vQ5luHs-T_NeDqh1mxwudl2dAu36eTuSC0L6HAwpj0f4nK3w6Z-PxJgELruSD4t961WbZFu-Hcyci8IV3nwS1TwNisVkI-9HnxqTwnnpnnOxPQAReGb8SHFq6bVUo57y2fCTqGuak7K-LSSkHcuTKPiEKLY4np7SoYX19Uc9EXW4KxOq28cdU6wlCOBi8Jp4nKD8GjuzW5nvmiwszD027kBTevvyWEuu1an_2eG3eGupeP11SGHD5NnrH6y_SJp4EL69g6h1fJ9EiqmaAOisoanadGzmVdVB7FEOt2EndcXIC8LUb-H7y_8ZpZTnFCZ4via1C1Y6ES34d2BNna20RVRt-v2p54P4KJRVsYspjnNr9W9JUy9klumu3PVO_3LCOJ2fJkbQJK5GlYAZKyFFFKY6YFjAyzPPyJt2oLPJXLCsSdyX1Z4_p8KFvMit_TxrhW9ycpkW8QCLTx06yGoMYd2aHGI16zKPjrsDB_F9Ilww3c9cYlop_U2l7h5KUCKIkXVQxy_Tlh3RV3JfmFJUJYoTr9uz-VcmqTcf1z93zwFgFAmuHnY4W8_b7pSWxwXEychXQcJK1fHKo_Hn6lFxGlfnBTeVhelbd6OQ3oQ0McpsA7kteKEmQJwgNhdJkI-ktJqBM-pSE5YaRA0ktD7uJQNABB_NFCkw7FZilNUlMYA5pZHELLcFe78fe92CaLNp7uMsDqzCRr4-iJjH6k0xJL6Mq0xZta11CmKE0Tf8LDHKY3Vlq8uNjFYwgNlbgbeS_LLGL8JKFjZD26UfnU_daU1rIa6twNF3mh8vKNSKvYzPydg4htnFm00)
</details>

| Вариант                                          | Плюсы                                                                                                                        | Минусы                                                                                                              | Причина отсутствия выбора на данном этапе                                                         |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Расширение Существующего IoT-шлюза и TimescaleDB | Использует Существующий IoT-шлюз для API синхронизации ферм и TimescaleDB, расширенную данными и схемой свиноводческих ферм. | Требует оценки пригодности и изменения Существующего IoT-шлюза и TimescaleDB, а также согласования изоляции данных. | Данные не подтверждают поддержку API синхронизации ферм, изоляции данных и нужных правил доступа. |

## 5. Риски

1. **Несовместимость Существующего IoT-шлюза или TimescaleDB с требованиями MVP**  
   _Меры:_ до выбора альтернативы проверить пригодность Существующего IoT-шлюза для предоставления API синхронизации ферм и TimescaleDB для расширения данными и схемой свиноводческих ферм. Проверить изоляцию данных, правила хранения, правила доступа и влияние изменений на текущих пользователей. Убедиться, что изменения не нарушают работу Существующего IoT-шлюза для текущих пользователей.

2. **Недоступность интернет-соединения на ферме**  
   _Меры:_ хранить данные в Агенте фермы и синхронизировать их после восстановления связи.

3. **Нарушение Локальных уведомлений или управления устройствами из-за корпоративных интеграций**  
   _Меры:_ не использовать Kafka для Локальных уведомлений, управления устройствами и передачи видео или аудио в реальном времени.
