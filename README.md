# Дизайн-документ: ML-система анализа тональности комментариев в социальной сети

## Информация о проекте

**Кейс:** Автоматизация мониторинга пользовательских реакций на контент в социальной сети

## Состав команды

| Имя | Роль(и) |
|-----|---------|
| Берулава Леон | Product Owner, Data Analyst |
| Кристина Фролова | Software Architect, Data Engineer |
| Мария Космач | Data Scientist, Data Engineer |

---

## 1. Цели и предпосылки

### Зачем разрабатываем продукт?

Существующий процесс анализа комментариев в социальной сети выполняется преимущественно вручную или с минимальным уровнем автоматизации. Это создает ряд критических проблем:

- **Высокая нагрузка на модераторов** при обработке 500,000+ комментариев ежедневно
- **Значительные задержки** в реакции на опасный и токсичный контент
- **Регуляторные риски** - потенциальные санкции от РКН за несвоевременную модерацию
- **Субъективность оценок** при ручном анализе тональности

**Цели разработки ML-системы:**
- Автоматизация процесса анализа комментариев с использованием машинного обучения
- Существенное сокращение объема ручного труда модераторов
- Обеспечение быстрой реакции на негативные и токсичные комментарии  
- Гарантия соблюдения регуляторных требований по модерации контента
- Улучшение пользовательского опыта через проактивную модерацию

### Почему ML улучшит ситуацию?

Переход от выборочного ручного анализа к автоматизированной ML-системе обеспечит:

- **Полный охват:** автоматическая оценка всех комментариев в режиме реального времени
- **Консистентность:** выявление скрытого негатива, троллинга и оскорблений по единым критериям
- **Масштабируемость:** обработка растущих объемов без пропорционального увеличения штата
- **Адаптивность:** непрерывное улучшение через retraining на новых данных
- **Гибкость:** настройка под специфику различных сообществ и контекстов

### Бизнес-требования

- Обработка минимум 500,000 комментариев в сутки
- Полноценная поддержка русского языка, включая интернет-сленг и эмодзи
- Время реакции системы не должно превышать 5 минут с момента публикации комментария
- Интеграция с существующими системами модерации без нарушения рабочих процессов

### Функциональные требования

- **Многоклассовая классификация** комментариев по тональности (позитивная/нейтральная/негативная)
- **Детекция токсичного контента:** выявление оскорбительных, угрожающих и неприемлемых сообщений
- **Автоматизированные действия:** уведомление модераторов или автоматическое скрытие/удаление контента
- **Настраиваемые пороги:** гибкая конфигурация чувствительности модели под специфику сообщества
- **Аналитическая отчетность:** формирование детальных отчетов по уровню токсичности и трендам

### Нефункциональные требования

- **Масштабируемость:** возможность расширения на различные каналы и платформы
- **Точность:** Precision ≥ 0.90 для негативного класса
- **Интеграция:** предоставление REST API для взаимодействия с внешними системами
- **Мониторинг:** полноценная поддержка логирования, мониторинга и отладки
- **Производительность:** обработка пиковых нагрузок без деградации качества

### MVP (Минимально жизнеспособный продукт)

**Основные компоненты MVP:**
- Классификация тональности на базе предобученной модели (например, BERT для русского языка)
- Веб-дашборд для отображения аналитики по негативным комментариям
- REST API для приема комментариев и возврата результатов классификации
- Механизм обратной связи для модераторов (ручная корректировка решений системы)

### Технический долг

**Планируемые улучшения после MVP:**
- Разработка кастомной модели с учетом доменной специфики платформы
- Реализация supervised learning на основе накопленной обратной связи от модераторов
- Расширение интеграций: поддержка Telegram, VK, YouTube и других платформ
- Внедрение продвинутых техник обработки: анализ контекста, детекция сарказма

---

## 2. Методология

### ML-парадигма

Система базируется на комбинации двух подходов:
- **Многоклассовая классификация текста** для анализа тональности
- **Бинарная классификация** для детекции токсичного контента

Для MVP применяется **transfer learning** с использованием предобученных моделей (DeepPavlov/rubert-base-cased или sberbank-ai/ruBert), что обеспечивает быстрый старт и высокое качество без необходимости обучения с нуля.

### Данные

**Источники данных:**
- **Реальные комментарии пользователей** социальной сети (основной источник)
- **Открытые датасеты:** RuSentiment, Taiga, TData, RuModeration
- **Ручная разметка:** минимум 10,000 записей с привлечением опытных модераторов (semi-supervised labeling)

**Требования к данным:**
- Сбалансированная выборка по всем классам тональности
- Представительность различных тематик и стилей общения
- Качественная разметка с контролем межэкспертного согласия

### Метрики качества

**Основные ML-метрики:**
- **F1-score по негативному классу** — основная метрика (приоритет на balance precision/recall)
- **ROC-AUC** — общий показатель способности модели различать классы
- **Accuracy** — вспомогательная метрика для общей оценки

**Связь с бизнес-результатом:**
При достижении precision ≥ 0.9 и recall ≥ 0.8 ожидается сокращение объема ручной модерации на 60-70%, что напрямую влияет на операционные расходы и скорость реакции.

### Риски и методы митигации

**Основные риски:**

1. **Class Imbalance** (менее 10% негативных сообщений)
   - *Решение:* техники балансировки (oversampling, SMOTE, focal loss)

2. **Семантическая неоднозначность** (сарказм, ирония, контекстные шутки)
   - *Решение:* data augmentation (Synonym Replacement, Back Translation)

3. **Cold Start проблемы** для новых тематик и языковых конструкций
   - *Решение:* поэтапное тестирование на фокус-группах, постепенное развертывание

4. **Этические риски** (false positives, цензура)
   - *Решение:* детальный анализ ошибок, система обратной связи, прозрачность решений

---

## 3. Подготовка пилота

### Подготовительные мероприятия

**Подготовка данных:**
- Разметка репрезентативной выборки из 100,000 комментариев
- Валидация качества разметки с привлечением нескольких экспертов
- Создание тестовых наборов данных для оценки производительности

**Техническая подготовка:**
- Настройка end-to-end pipeline: ingestion → preprocessing → inference → alert
- Разработка mock-интерфейса для интеграции с UI модератора
- Подготовка инфраструктуры для A/B тестирования

### Критерии успеха пилота

**Количественные показатели:**
- F1-score по негативному классу ≥ 0.85
- Доля ложных срабатываний (false positives) ≤ 10%
- Сокращение времени обработки инцидента на 60% и более

**Качественные показатели:**
- Положительная обратная связь от команды модераторов
- Отсутствие значимого роста пользовательских жалоб
- Стабильная работа системы без критических сбоев

### Система обратной связи

**Механизм оценки модераторами:**
- Трехуровневая шкала оценки: "корректно" / "частично корректно" / "ошибка"
- Детальное логирование ошибок с анализом первопричин
- Возможность предложения корректных меток для ошибочно классифицированных примеров
- Регулярный анализ паттернов ошибок для улучшения модели

**Цикл обратной связи:**
- Еженедельное review качества с командой модерации
- Ежемесячное обновление модели на основе накопленной обратной связи
- Квартальная оценка влияния на бизнес-метрики

---

## 4. Диаграммы и архитектура

- [Диаграмма BPMN - текущий процесс модерации](bpmn.png)
- [Диаграмма BPMN - процесс с ML-системой](bpmn.png) 
- [ER-диаграмма структуры данных](er.jpg)
- [Диаграмма компонентов](Components.png)
- [Диаграмма последовательности](sequences.jpg)  
- [Диаграмма прецедентов](usecase.jpg)

---

## Заключение

Разрабатываемая ML-система для автоматизации анализа тональности комментариев представляет стратегически важное решение для масштабирования процессов модерации. Система обеспечит обработку 500,000+ комментариев ежедневно, значительно снизит нагрузку на модераторов и гарантирует соблюдение регуляторных требований.

Поэтапный подход с тщательно спланированным пилотом минимизирует риски внедрения и обеспечивает возможность корректировки решения на основе реальной обратной связи от пользователей системы.
