# Диаграмма контейнеров в модели С4

Для просмотра диаграммы в markdown использую плагин для Visual Studio Code
https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml

```puml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

System_Boundary(c1, "Система Будущее 2.0") {
    Container(portal, "Портал самообслуживания", "React/Angular", "Витрина данных")
    Container(bi, "BI-система", "Power BI", "Аналитика и отчеты")
    Container(dwh, "Облачный DWH", "Snowflake", "Агрегированные данные")
    Container(datalake, "Data Lake", "Azure Blob Storage", "Сырые данные")
    Container(legacy_dwh, "Легаси DWH", "SQL Server 2008", "Deprecated") #red
    
    Container(fintech, "Финтех-сервисы", "Golang/Java")
    Container(ai, "ИИ-сервисы", "Python")
    Container(internal, "Внутренние сервисы", "Java")
    
    Container(gateway, "API Gateway", "Kong", "Единая точка входа") #FF5722
    Container(bus, "Интеграционная шина", "Kafka")
    Container(ui, "Клиентский интерфейс", "React", "Для операторов")
}

Rel(portal, gateway, "API-запросы", "HTTPS")
Rel(gateway, dwh, "Запросы данных", "gRPC")
Rel(gateway, fintech, "Финансовые транзакции", "REST")
Rel(gateway, internal, "Управление клиниками", "REST")
Rel(gateway, ai, "Метаданные ИИ", "REST (read-only)")

Rel(bi, gateway, "Запросы для отчетов", "HTTPS")
Rel(ui, gateway, "Операции операторов", "WebSockets")

Rel(ai, datalake, "Чтение мед.данных", "Apache Arrow")
Rel(fintech, bus, "События платежей", "Avro")
Rel(bus, dwh, "Синхронизация", "Kafka Connect")
Rel_L(legacy_dwh, dwh, "Миграция данных", "Batch") #red
@enduml
```

см. файл Task1/c4.png

# Проблемные места

Проблемные места и их приоритизация (MoSCoW)

## Must Have (критичные проблемы):

- Устаревший DWH (SQL Server 2008).
- Низкая производительность, сложность масштабирования.

**Решение:** Миграция на облачное хранилище (Snowflake/BigQuery).

- Медленные отчёты.
- Трансформации данных занимают часы.

**Решение:** Оптимизация ETL-процессов, кэширование, предрасчёты.

- Текущая шина (Apache Camel) не справляется с нагрузкой.

**Решение:** Переход на Kafka или облачную шину.

## Should Have (важные, но не критические):

- Разрозненность данных
- Данные распределены по доменам без единой точки доступа.

**Решение:** Внедрение Data Lake для сырых данных и DWH для аналитики.

- Устаревший клиентский интерфейс (Power Builder)
- Неудобство для пользователей, высокая стоимость поддержки.

**Решение:** Разработка веб-интерфейса.

## Could Have (желательные улучшения):

- Кастомизации BI-системы
- Трудности с поддержкой.

**Решение:** Стандартизация отчётов и дашбордов.

- Безопасность данных
- Риски из-за устаревших технологий.

**Решение:** Внедрение RBAC и аудита доступа.

## Won't Have (не приоритетные):

- Полный отказ от легаси-систем
- Постепенная миграция допустима в рамках годового плана.

## Аргументы для бизнеса

- Отказ от легаси (Power Builder, SQL Server 2008) уменьшит расходы на поддержку.
- Современный DWH и BI-система сократят время построения отчётов с часов до минут.
- Облачные решения позволяют масштабировать ресурсы и инфраструктуру тогда, когда нужно, а не после покупки новых серверов.