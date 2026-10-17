# ai-engineering-course

Курс «ИИ в ПромТехе».

## Модуль 1. Интеграция LLM, параметры запроса и надёжность интеграции (fallback)

В модуле реализованы:
- Клиент для отправки запросов к LLM (`module_01/llm_client.py`) с настройкой параметров генерации (`temperature`, `max_tokens`, `system_prompt`).
- Подсчёт расхода токенов и мониторинг квоты (`module_01/token_tracker.py`).
- Отказоустойчивый механизм fallback на резервного провайдера HuggingFace при недоступности основного провайдера GigaChat (TODO 5).

---

## Troubleshooting

### 1. `'ascii' codec can't encode characters`
**Что видел:** Ошибка `'ascii' codec can't encode characters in position 7-15: ordinal not in range(128)` при вызове `call_gigachat`.
**Почему:** В переменной окружения `GIGACHAT_CREDENTIALS` оказались символы кириллицы (значение `сломанный_ключ`), в то время как значение ключа передается в HTTP-заголовок `Authorization: Basic ...`, допускающий только ASCII-символы.
**Что сделал:** Благодаря реализованному механизму `try / except Exception as e` скрипт перехватил ошибку и переключился на резервного провайдера. После проверки вернул валидный ключ (Base64 строка из латинских букв и цифр) в файл `.env`.

### 2. `Error code: 402 - You have no remaining credits`
**Что видел:** Ошибка `openai.APIStatusError: Error code: 402 - {'error': 'You have no remaining credits. Purchase pre-paid credits to continue using Inference Providers. Alternatively, subscribe to PRO to get monthly included credits.'}` при обращении к роутеру HuggingFace (`router.huggingface.co/v1`).
**Почему:** На бесплатном аккаунте Hugging Face исчерпан месячный лимит включённых кредитов на использование Inference Providers для сторонних моделей.
**Что сделал:** Для восстановления работоспособности резервного провайдера необходимо перевыпустить токен на новом аккаунте либо пополнить баланс в настройках биллинга Hugging Face. В коде fallback ошибка корректно обрабатывается блоком перехвата исключений, предотвращая аварийное завершение программы без вывода пояснения.

### 3. `401 Unauthorized`
**Что видел:** Ответ `401 Unauthorized` от API провайдера при отправке запроса.
**Почему:** Неверный, устаревший или отозванный ключ доступа в `.env` (например, `HF_API_KEY` или `GIGACHAT_CREDENTIALS`).
**Что сделал:** Проверил формат и корректность ключей в личном кабинете провайдера (для GigaChat — Studio SberDevices, для Hugging Face — Settings → Access Tokens с правом `Inference Providers`) и обновил соответствующие переменные в локальном `.env`.