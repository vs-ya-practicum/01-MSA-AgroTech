# ADR-001: Основа Центральной платформы для MVP

**Статус:** ⚠️ На рассмотрении  
**Участники:** Владелец продукта; архитектор решения  
**Дата:** 2026-09-01

**Универсальный язык:** [ubiquitous-language.ru.md](../ubiquitous-language.ru.md)

---

## 1. Контекст

MVP-система свиноферм должна собирать данные нескольких ферм. Оператор должен контролировать фермы и анализировать События фермы.

Каждый Агент фермы должен работать при недоступности интернет-соединения. Он должен сохранять данные локально и отправлять их после восстановления связи.

Требуется сравнить два варианта реализации Центральной платформы. Основной вариант создаёт Центральную платформу с новым Центральным сервисом синхронизации и новым Центральным хранилищем. В альтернативном варианте Центральная платформа использует и адаптирует Существующий IoT-шлюз и TimescaleDB. Существующий IoT-шлюз предоставляет Центральный сервис синхронизации, а TimescaleDB расширяется данными и схемой свиноводческих ферм. Обе системы остаются внешними для Центральной платформы.

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

Основной вариант создаёт Центральную платформу. Её Центральный сервис синхронизации принимает, обрабатывает и сохраняет данные, синхронизированные Агентами фермы.

Агент фермы включает Локальный шлюз устройств и Интеграцию с камерами. Локальный шлюз устройств отправляет команды управления, а Агент фермы отправляет Локальные уведомления независимо от интернет-соединения.

Существующий Kafka остаётся асинхронным корпоративным каналом для Опубликованных метрик. Kafka не участвует в Локальных уведомлениях, управлении устройствами или работе Агента фермы при отсутствии интернет-соединения.

### Контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Диаграмма: MVP-система свиноферм — Основной вариант — System Context</summary>

![MVP-система свиноферм — Основной вариант — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRkF65RxdKn1fhnRTL3yKIRMR018btRG1WcoHNZH5GCjSHrXPSaawQxS8uBMJfeijiht8LGgq3GhqMhknKgsTjL-1V8K-ISyCX-AGCoI7bbSaDXnZROkSR-OxPyxvU3GYlSx3oUp0jCrzUTTwO1vEtTzOHsvlEjcRxLipLnx3DykM96VTxP5j7lnsikj-8hSQ3UhulddqdkKuvfvrV_97Isvs6ZJgrzQUDTiTpMUkCP_ERVad8zoO7drp_73FdX_ETeD-hctE7EW0DDY-StlNijb-L4VR6hlcR6zgtRitnPYURPdllYl_IPAXxQmd6pE3uFDu0lRSPzmfoA3tmL_Mt3q-i92rWIzQSSYkwPexfbC0yOT3wx10T_ZkJiQJ0nDMK1XaslRy4UbE5ad53H-XbYvnbC_0iftvhC3aOnU6pkgEpZZkCVZPV6GvytqB3S1VwXsqfAPzl5vepeWgvuJ1ZO8WpExZgNTDvnDhD-diyKE8-jkO2aFmH0TJSp8Bfi_82EI1hJH4HDZRLAWekENQ4RcPW-EvdysbiddW32sRH91rPRF0kPn9WfdeDRaZai6oVU9O422piRKpFoezI0AzecdaQLDEtkV1YNnaYGWkCmxiYMDzl47Si5xOG4ygGub8IsNuX2fJ6K94SCJuP6NORJY7rfpAU2WcFzK9f529k8nugX8EBguXMQE829DpxBZMVcL4gH9qY21LWaieHfKWGuGedyisHQWIdea9K3K29nGVXarXuIaqWeZeZ045x_yikpAwKnL6H7ggGYTKdwhm8G8aH6UR6bGL_yHag39152eHZr1XCIewo8qRT_OW8L8stiZ43LrfQVraD5976H-JpMB6nmsjtLU3uny4tMqmN6vryA71nDoRpymfETf8aRPIA7CqhNGUePYLJeIec0WFQbJ-qOqSGajOwLw9QLT9R5BptZe21f1h7uiGKtNqnNoi41qL9gRtqDU60b0kTzvEJsK8hRTwAiCetEifGkZsH8BdejY6JsNy5D_nAPcMbBsxaisdsLJr_Y39XpegpATjQ2EZiQ5Qb48vTVUbH1xndPSIYSHsNRQb13o7iBik9Ng5i-EoTJRvvO88iIvkIMBKqPBAXLAuPvKBfHDYoeNI6INBXUsAX72vKBgV-EL2IRJoxZZPR7zTz6Dq4OMuvSzkjdUVIYMuY7zsixsxLZR_RE9rtBnI2GyL_0ndcxUo27q2atj7I-5HyeHgypiIIXEhxL6_pRjVjYXQbI5KwEwNCiIhCfwwmLqmsnGnuVjNSeNiHYEHJtK0vbhoJyaDtBzJz2D5_tbJ_Jsufb6mtGv0wVODewUr107gS4HQloq0Q1eZgccvWEPm56jw7i0O9PgQ3T16J6jc0Pfi_r4t2w1o0ACiG9y7K380Gnv0om3m-YSyko5Wm0Ee6O1UKQDvRXrchlGmpuHDGuEpT9syy1vahHetu_fPutDJjBEUfDSK586srkltkshpzScCEA3LqNITqfQEhgjACnMW4Mi7NLLfjnKjz5TVuG8KRlOGI9C5QAEiDcWb0UeekmsRSK1xbDVwSK0_5n16YIGu0PX10xe2W56B0Wn402e54EbFSbLAqs2szOe2-jK3K3OqXY80ZGAe0lr9-95OqlCZenqFxwRIoMpjBQh6PybogfLVJzNLJbtjrDLENUq8LJku4adM8ibgH68g4ROMYLKZaa0b4gj6995AXAv6RdOvQ1UzfFM50C4gIi8Ja6f44gr4mcf44gr4mcILH2kHC9SaqQjuwAUA0NetDBguMzSBbr9WwCc5Qmi21apmvG0wJ7Xd0_fi0cK2ulMKbWk8foWHWGn0KWG2cKa5S6I64h9aImWPLK_PyvsvMuNTgHfGAg33r_XXbty5v1Sw0QJDlMIAl5Ec0G9l6MC09HDWi82Eo1jbSS1mCu3QxIeXm51QaRkY5m3eTdKYf3gkiZgH_rHwM2tybHHxKl268WJqw7S0kEflqO0E1WmeG3_gGz3jR02kUTSnqbxBU3fWrDEfgy_wHGOWrQrl5e1Uk96k9Ph10kZG636bRvSDZLZmQMgY6MpmRI4VRfRJsLif6WVG4T80MlaK8wq_hPM9TzlSLb-1tWX0ifhAsPnArcPv17hyhSDKW96LAHm0xztfK2cwnz7B0EX5jrrUDm2ypWN45aEA3Q-drErw3RYnMwiO6ji0yAuW1ZVZb40Y59r167fuZWOOqMcc6N4raCG-5eYCaLB49VNIB8bF4Er0r80QSEk0lCsf2WFM6PS3C9YSpW7eZ6fa0mn6bxIqlIQLqQM2vLJzN7pzN7pzNFoFzRdu-i7u-i7u-i7u-i7uea-wyV_OSlf39NKfKPSITIdnvfOIcSbVbnDrEL6N4tKvaLPEk1DtQidlVVZ-NV-nj_8V-wVUqll1MyZmyeMtybvvL_xdtj8_yNwG_tVojUpz0qx0GJY9lh-KuPS5d5rv2--LVuhxtCUFnSe6a0G39Oay6lRME-y0tBl6eorWeYl_vAFPHpFl6nW8i73qohi44YV-6VGvbM669SnsWGxxVqK_vP3BpsGq7_o3_jv_QSotqFkv_nI6N_dFrltzfxAtal5GCAjtZYsBKFuPiXpHGdEYKzvIzgune-Sm1xZaBPbIUDEHv_32HFy7ISoyhw7hzp3Q2HZq5Bm3zA_Gm33FEHxsWgA4ZVyNpFaSTOAVnEZ5ke-t14EliVrd_fTXZx_3WGjaEpW5JRC4_h0CIvZY2Z2NWGLWFPWCeooXwtTnjouIne5x8bf9In-OzlxWq3qUxXzCvyUMXVyMRVI_s22xXU87c_iVH8wu6lE7I7Y6RBn48G5Thx3T2xnSVmFUGNUQtzjf9qAZpd4SVGy3VeNClOovwyp_IiQpOwVxPm489aEl7iFfKsm4uWu7LsWIF3wAnLDav1Lsz2eC5EBcnxZh5tZ-Lr5C1WpESLm5IuMM4tfVebzVe5_nT0kIKz7Q8V_vpz0uRwC5MVzb3LYGWydU2DQ4f1cx6jw_IJH3VX9Z5YJy5oYcWDZJP1X-4pWU6Fy5kgr8sAs047v5Bsu0NY5_1Wlf_pbmrWla-tMcHLF5GthXVu4COlzcI1XgrgUJiUNqGCNCVSWnHsOD_1PuFa7YW8p3Ndu255xAlp8VF3InFMazW6iY076UWcISu301ZZXfiOey8Lhm4YtX9LhfHMAXYGPUOEzTu-z8sLQHF988yhwBF9LgldKqSTp9R0-R0GkxrfYxTw53VDb_8T-vawgSqUBUib32WUKwg01tvux4o4CQsNzkMST2RamytRkNJd699pedse5N3e-CPaGw0UeHXFeAk-Bgbp5z3jBz0jD7RaVP06olSRJ5ilkCJL70h81pd1Pc6e-BW2aC-nmlwaNe3hYqHWUncb2gipO0vJFCydy0)

</details>

[Исходный код основной C1-диаграммы](01-01-primary.c4.context.puml)

### Аргументация

| Критерий           | Обоснование                                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Изоляция           | Центральная платформа отделяет модель, API и эксплуатацию MVP от Существующего IoT-шлюза.                                                                                      |
| Автономность фермы | Локальный буфер сохраняет телеметрию, События фермы и состояние синхронизации. Локальные уведомления и управление устройствами выполняются при отсутствии интернет-соединения. |
| Интеграция         | Kafka передаёт Опубликованные метрики корпоративным потребителям. Он не участвует в Локальных уведомлениях и управлении устройствами.                                          |

#### Последствия

**✅ Положительные:**

- MVP не изменяет Существующий IoT-шлюз.
- Команда определяет Центральный сервис синхронизации, его интерфейс и транспорт, модель данных и правила хранения для фермерского домена.
- Отказ интернет-соединения не блокирует Локальные уведомления и управление устройствами.
- Центральная платформа изолирует домен MVP и поддерживает бесшовное развитие в SaaS-платформу.

**⚠️ Негативные:**

- MVP дублирует часть возможностей Существующего IoT-шлюза и TimescaleDB.

**↗️ Зависимости:**

- Существующий Kafka и контракты Опубликованных метрик.
- Локальная инфраструктура, устройства и камеры каждой фермы.

### Компромиссы

Основной вариант изолирует новый домен MVP от Существующего IoT-шлюза и позволяет самостоятельно определить Центральный сервис синхронизации, его интерфейс и транспорт, модель данных, правила хранения, правила доступа и операционный мониторинг. Он позволяет комфортно надстраивать будущую SaaS-платформу, но для этого приходится реализовать, развернуть, эксплуатировать, контролировать и защитить Центральный сервис синхронизации и Центральное хранилище.

## 4. Альтернатива

### Расширение Существующего IoT-шлюза и TimescaleDB

Альтернативный вариант использует и расширяет Существующий IoT-шлюз и TimescaleDB как внешние системы. Существующий IoT-шлюз предоставляет Центральный сервис синхронизации, а TimescaleDB расширяется данными и схемой свиноводческих ферм.

Агент фермы и Существующий Kafka работают так же, как в основном варианте.

### Контекстная диаграмма

> [!TIP]
> Установите в Chrome расширение <a href="https://chromewebstore.google.com/detail/svg-navigator/pefngfjmidahdaahgehodmfodhhhofkl" target="_blank">SVG Navigator</a>, чтобы просматривать диаграмму с масштабированием и панорамированием в отдельной вкладке.

<details>
<summary>Диаграмма: MVP-система свиноферм — Альтернативный вариант — System Context</summary>

![MVP-система свиноферм — Альтернативный вариант — System Context](https://puml.livingmodel.dev/a9f2k7xq/svg/xLtTRjl85RxdKn3RNWMRhVL7Kcrs0OcIW4MxmA89sejkiQ1bnYj4Fov5DtQB1UoutkqYYwGNUr6qG7zGOAzbnzeeTYA_0laAVPAU6Gx58MT83Yr7959Ea5YYvflvpfbplZaQKSxqqM2yPrZ6IDun7nhxG_k1UM2tXeFTySWSst8V_Z4jIPgkkZssZBrV3tRiB-JgrXPrVMGSV69EfyQkUN_m1rEkDBPer2_CNNEyqvkCRNqod5ZoZtguCJtQHl_HhZNP7-zu_MpB64-X0z1W-qwi7TDY-r4TBRDl6-FTeNdltX1ZMfPfVFIH_4T9XhQzcgpF389VnnEmhtt9dO8CUX_yPKximpqJMUlveXg6x1YsiMrCCo1-js_kP-WE_zg3_c3FW1KK1XcMDNbCkfD5Kd33LwYbYrpbCp2jwMISOVA-3KCdTKUlJEr3yBFns9nEHYOQWB_KssX93UjmjT2S4HNE2pet2hmmku-dtZ6cNvWxKMVt7q7KtyPK688dsXiQWx4tVK9680zif24YmjwcGgMEMweTWPivE9xxsLpAcmVEqBn9U5cNDmkSouaac8dUaZi26SpR9rCJ0cJSDxSd1xa7YQTND8qqRKh9-pGuaOya4S5bnfurc9gVhn4th1VMq9F2K2BIKXW-lieKXX0H734-IHbs6ywXDQSm7Wh9Jt42oLSYBYEUglY3Ysj8qYWYmZGzdDhcg32YL0WwH90AmSKNeqAG8KAKpcMJ8bG8puG4g1Y1uuiFmoQpy6GQGKHq-e22zt-cLHZTAGeZ8Zr5eUEhJr7u404IehD93IgAVq8oL9WWyfM8HoYp66KTvCQDExkN4347RsJYBQrULTlHQAIkCZucXyMCZxVKMblnhl_Iwsw1uNArWIzr1iRkP6mCoTMj66dBXJ97qtBduOjPxaI8Yedme4RbNrtB8RI4vUwLa7Rbn2Osxyq3O02vzha8CKL3VvYF5OAhmiIq5lgpfW3aovrtqbCOGgkZdi8myizwiX2wFP7WkIXqu5COFuLFV7Ac9IMVhYJpgLGKhTs9yg6k2lEfwMfFtrfJBNAX77lwaYEFyCbBZaH2fowRKWAUGzXJbnozmZbnsJYRz791X5WJjoWnwcfEvK8kt9FAXTm9CUL2xeoYvS9cHKAuNCZTJ_no8ITQQNUSh7U-BlenkeX2tF9dDvkxJwKGNCG_kzdSNIlhVpRnDcvUAOI7Cdw6iybRMOG-WKczeoNpA6b2jVuT2Rc9rUQet-RTBniKhS8GodJtInxYLHXFtU0kc4sAcF3zAwb2TgEHoASw0BDD-SVa1kxVrBM3klQJYl9RSCrMndOxW7gphUijjIe8KBezqjer0K15xr5DJGLKkhrGqz80UY_Ib6I0scDQ9GbGOVltcaa0XGFe9G7QF40z0T3b0TG40Az_nBDh0ZeyW980Q6KrckVMRk9ATzDCM3SqE4lNI0llJjAgSJEkdJG-DqKRgqbQ5K4HgAcjThkfLDwUp8W3gWrLqoXjQMYQKg_40bHYRQUfr6krkkhxgrtd0khSx26G2WjGUqcjKuq2b5vowrR2W5elhRN3W7Ogm8yIIN02C8C6D0K0lIe5w8W050eWqf_agXIdmMnhPGMqYmUWRDZgYW1K2g08z2VXHs99Iu-CMZYyA_L6OgjjK3M-Z9PJrVHwggnsocgdh7RAQaUesi6LIBGMYLOZ4bE9iBL8g1gH12gHK3KYYLGYT3Loi-sWbdLBQmi1l5MKX2SWrOWaMedurOWaMeduoIg9LY9-BacYkxZlfu81MZSqEhjRLmiNKs1eqOMh2G8wJF2b01fCU2S3scm29GB2zPIQ2WYdA161141K13oPYGLmP8OIicYB21bLJrcJxObTXDEf4f0heC7N-6wJlmFo2nqFKkDkcIBlb6c0m5l64A2U2D1PG4FaZR8ueBiUG6aswpv0hrS3TmLF0D1iQaH8DLnbDGB_AVImgl-G57kIzuR21xJeTm2uwg_IW0O6T2X0E-X3qBLa0AvvLp5IMybuFA3NqgZTP_Ka0H3hrdSBG2zSJrCZJMS1T6ZqcDAtoOP6AFXK9T9qDlWsa0yto-daenIL0sW8QG2jV7KzhZ-jbP5tsznMxa3l121QJQKSJaLhipG2BVxHOIn0JyeK3e3txhHRpth6qKe0Q5cFNTus0BpE1OGMGmeDhyNKxLe9kB5RYnWQcm3mTX2Tct5rJoBg6eFGDVyr6g07BpDD3wk19FQnG62C52CkiLkp93v1S09H0ah0hGFI3gSA35XbN0f0Pt8w1Q0ngf4Cq1bTKkFEcXH6bpAMK-NtuillnPVVY_z__Lvy-SNuyelnvHVZoo_6P_sc6_-NBST_gQGi9SfIeYmbtjrI8Z7votAYB2VASg8i9-BA2NjW3qtvarzzUjSzTXRkiVl4cJkldPaCJy-TXVF6MRf_SkRkaVDQ_k_HTxBp5-VA_HRrWajld1ayMa37D-vJvvKC3sPmUU7CuC8JtFy-_egih0DXC5QIoDV4tllmGzdvXtEDXq63BDsZpyQVZPs_EKk4XQjBvme87RadqEU937FDWT8bkkn-YtxAFg_tPJGV_8N-paiQys_eVUO-HHpTvwl-xbFPMSXuA9ZLkS1M1YZt17a1qK9pefUSkUnSOqPdC0UuvpsPKdjZAa_WYO7-FvAGUTz1rnzWj2FY7Q2_H0F3F1Tus4kA4ZR-Np3d6Ue4FudHirKVPmw6NcFxJzol_Hv_XmkNo7Pm2ffc3lnX6UOmnH8mLvu5O3sO3AFCeUkBi5jd4UF0FG6jgAKF3MlKsRSFkwEzuUJGDF7lfGt-HnjiTqo4mAFdOe66VV4zOaxM0Hk7yU20ApGaT5rYfyna7437xf-nLS_9meEBla410xESm-Ddy6m1I_3nvC5Tw7dbFdDU8eyy68pCQTyOcfssZDq8IuA8dsAdVUs57CBBCDqnueY6azsl679BSEiIsNA6YStmE9v34IMS5Gky2ATqQ26Jpl6wlCONO0dTKnaRWXRm_0FapXTajgUX4FOMxGBdoG_WWcV8yAjGE23YU3OSL40SWnOoVdOCBp_nF4IueKd8gW5bj4vB74WBckq26OcwdkBvtmGvvp6umCPveOZ25NfVeOVdw26URkRvaOfD509WCOpp0GgurMyHWO3R6-ps8EQ8Z72YxJyaMvVEAsBCZAJt3Bl-AEGsNsFminC8XKjdGT9g0OJmCpe-FV32YsWSQLzvpZf7lbyfIJ1LE6LdxYaoYFtzA7ugcRyVzCrf2nJR6866J6NMm1l4ChQIXqCfn0dUkCowxMX-dqbMxAFdA6gkq8_lENdrYg76A1JNl-R1O6eFnhlWfq-7XlrmOeqoUFVdniD7HgmVKG_MZzuUU8Q3yHBnmvAFzw1lo4xoQcNgB996nDHBREArv_QLTEANI7uwBu8uJ3KUcm5fizgXxjw53k6a-gbyvqxilXiu-fQtMRWmNwKiuExSaPZD9OxiFpUy2_hSc6ozTo-UuXnFT46KcKqWjAbW8LbWLon_5jhTFJ6znFIHsv7M0Djh76qXxJnX1H0m2-0SfeMPyf82y_qhXECrAQvU-cvPy3PNNnK4DG5hVu8c9FFqujVpAB0wmy5twVE30ZAuYFSKQky9M5rYR_mF)

</details>

[Исходный код альтернативной C1-диаграммы](01-02-alternative.c4.context.puml)

| Вариант                                          | Плюсы                                                                                                                                    | Минусы                                                                                                              | Причина отсутствия выбора на данном этапе                                                                     |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Расширение Существующего IoT-шлюза и TimescaleDB | Использует Существующий IoT-шлюз для Центрального сервиса синхронизации и TimescaleDB, расширенную данными и схемой свиноводческих ферм. | Требует оценки пригодности и изменения Существующего IoT-шлюза и TimescaleDB, а также согласования изоляции данных. | Данные не подтверждают поддержку Центрального сервиса синхронизации, изоляции данных и нужных правил доступа. |

## 5. Риски

1. **Несовместимость Существующего IoT-шлюза или TimescaleDB с требованиями MVP**  
   _Меры:_ до выбора альтернативы проверить пригодность Существующего IoT-шлюза для предоставления Центрального сервиса синхронизации и TimescaleDB для расширения данными и схемой свиноводческих ферм. Проверить изоляцию данных, правила хранения, правила доступа и влияние изменений на текущих пользователей. Убедиться, что изменения не нарушают работу Существующего IoT-шлюза для текущих пользователей.

2. **Недоступность интернет-соединения на ферме**  
   _Меры:_ хранить данные в Агенте фермы и синхронизировать их после восстановления связи.

3. **Нарушение Локальных уведомлений или управления устройствами из-за корпоративных интеграций**  
   _Меры:_ не использовать Kafka для Локальных уведомлений, управления устройствами и передачи видео или аудио в реальном времени.
