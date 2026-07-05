# Guía de los 23 patrones de diseño: aplicación práctica con Python, C, C++ y C#

## Objetivo

Esta guía explica los 23 patrones de diseño clásicos, conocidos como patrones GoF, y cómo aplicarlos en proyectos reales. Está orientada principalmente a Python, pero cada patrón incluye ejemplos compactos en:

```text
1. Python
2. C
3. C++
4. C#
```

Los ejemplos usan temas distintos para que el patrón no quede asociado a un único dominio.

La guía está organizada en tres familias:

```text
1. Patrones creacionales:
   Cómo crear objetos.

2. Patrones estructurales:
   Cómo componer clases, objetos y módulos.

3. Patrones de comportamiento:
   Cómo coordinar responsabilidades, algoritmos y comunicación.
```

---

# Parte I: fundamentos

## 1. Qué es un patrón de diseño

Un patrón de diseño es una solución reutilizable a un problema recurrente de diseño de software.

No es:

```text
Una librería.
Una receta obligatoria.
Una clase que se copia igual en todos los proyectos.
Una solución universal.
```

Sí es:

```text
Un vocabulario común.
Una forma de organizar responsabilidades.
Una guía para reducir acoplamiento.
Un mecanismo para hacer el código más extensible.
```

Ejemplo:

```text
Problema:
Necesitas crear objetos de varias familias sin acoplar el código cliente a clases concretas.

Patrón:
Abstract Factory.
```

---

## 2. Patrones y Python

Python es dinámico y flexible. Esto cambia cómo se aplican algunos patrones.

En Python, a veces no necesitas una estructura pesada porque puedes usar:

```text
Funciones.
Closures.
Duck typing.
Protocolos.
Dataclasses.
Decoradores.
Context managers.
Módulos.
Diccionarios de funciones.
```

Ejemplo:

```python
# Strategy simple en Python usando funciones

def normal_price(amount: float) -> float:
    return amount

def vip_price(amount: float) -> float:
    return amount * 0.9

def calculate_total(amount: float, pricing_strategy) -> float:
    return pricing_strategy(amount)

print(calculate_total(100, vip_price))
```

En Java, C# o C++ tal vez usarías interfaces/clases. En Python, muchas veces basta pasar una función.

Regla:

```text
Aplica patrones para aclarar diseño, no para complicarlo.
```

---

## 3. Los 23 patrones

| Familia | Patrones |
|---|---|
| Creacionales | Factory Method, Abstract Factory, Builder, Prototype, Singleton |
| Estructurales | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| Comportamiento | Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor |

---

## 4. Cómo leer esta guía

Cada patrón incluye:

```text
Intención.
Problema que resuelve.
Cuándo usarlo.
Cuándo evitarlo.
Ejemplo principal en Python.
Ejemplo en C.
Ejemplo en C++.
Ejemplo en C#.
```

Los ejemplos son breves. Su objetivo es mostrar la idea, no cubrir todos los detalles de producción.

---

# Parte II: patrones creacionales

Los patrones creacionales tratan sobre cómo crear objetos sin acoplar demasiado el código a clases concretas.

---

# 1. Factory Method

## Intención

Definir una interfaz para crear objetos, dejando que subclases o funciones especializadas decidan qué clase concreta instanciar.

## Problema

El código cliente necesita crear objetos, pero no debería depender directamente de clases concretas.

## Cuándo usarlo

Úsalo cuando:

```text
La clase exacta depende de configuración.
El objeto concreto depende del entorno.
Quieres evitar if/elif repetidos por todo el código.
Quieres aislar la creación de objetos.
```

Evítalo cuando:

```text
Solo hay una clase concreta.
La creación es trivial.
El factory agrega indirección sin beneficio.
```

---

## Python: conectores de almacenamiento

```python
from abc import ABC, abstractmethod


class Storage(ABC):
    @abstractmethod
    def save(self, key: str, value: bytes) -> None:
        pass


class LocalStorage(Storage):
    def save(self, key: str, value: bytes) -> None:
        print(f"Saving {key} locally")


class S3Storage(Storage):
    def save(self, key: str, value: bytes) -> None:
        print(f"Saving {key} to S3")


def create_storage(kind: str) -> Storage:
    if kind == "local":
        return LocalStorage()

    if kind == "s3":
        return S3Storage()

    raise ValueError(f"Unknown storage kind: {kind}")


storage = create_storage("s3")
storage.save("report.pdf", b"content")
```

Aplicación:

```text
El resto del sistema usa Storage.
La creación concreta se concentra en create_storage.
```

---

## C: parser de archivos con function pointers

```c
#include <stdio.h>
#include <string.h>

typedef struct Parser {
    void (*parse)(const char *path);
} Parser;

void parse_csv(const char *path) {
    printf("Parsing CSV: %s\n", path);
}

void parse_json(const char *path) {
    printf("Parsing JSON: %s\n", path);
}

Parser create_parser(const char *kind) {
    Parser parser;

    if (strcmp(kind, "csv") == 0) {
        parser.parse = parse_csv;
    } else {
        parser.parse = parse_json;
    }

    return parser;
}

int main(void) {
    Parser parser = create_parser("csv");
    parser.parse("data.csv");
    return 0;
}
```

---

## C++: notificaciones

```cpp
#include <iostream>
#include <memory>
#include <string>

class Notifier {
public:
    virtual ~Notifier() = default;
    virtual void send(const std::string& message) = 0;
};

class EmailNotifier : public Notifier {
public:
    void send(const std::string& message) override {
        std::cout << "Email: " << message << "\n";
    }
};

class SmsNotifier : public Notifier {
public:
    void send(const std::string& message) override {
        std::cout << "SMS: " << message << "\n";
    }
};

std::unique_ptr<Notifier> create_notifier(const std::string& kind) {
    if (kind == "email") {
        return std::make_unique<EmailNotifier>();
    }

    return std::make_unique<SmsNotifier>();
}

int main() {
    auto notifier = create_notifier("email");
    notifier->send("Server restarted");
}
```

---

## C#: exportadores de reportes

```csharp
using System;

public interface IReportExporter
{
    void Export(string reportName);
}

public class PdfExporter : IReportExporter
{
    public void Export(string reportName) =>
        Console.WriteLine($"Exporting {reportName} as PDF");
}

public class CsvExporter : IReportExporter
{
    public void Export(string reportName) =>
        Console.WriteLine($"Exporting {reportName} as CSV");
}

public static class ReportExporterFactory
{
    public static IReportExporter Create(string kind)
    {
        return kind switch
        {
            "pdf" => new PdfExporter(),
            "csv" => new CsvExporter(),
            _ => throw new ArgumentException("Unknown exporter")
        };
    }
}

var exporter = ReportExporterFactory.Create("pdf");
exporter.Export("sales");
```

---

# 2. Abstract Factory

## Intención

Crear familias de objetos relacionados sin especificar sus clases concretas.

## Problema

Necesitas crear varios objetos que deben ser compatibles entre sí.

## Cuándo usarlo

Úsalo cuando:

```text
Tienes familias de productos.
Quieres cambiar una familia completa por configuración.
Necesitas consistencia entre objetos creados.
```

Evítalo cuando:

```text
Solo creas un tipo de objeto.
Las combinaciones no necesitan consistencia.
Agrega demasiadas interfaces para un caso simple.
```

---

## Python: UI para temas claro/oscuro

```python
from abc import ABC, abstractmethod


class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


class LightButton(Button):
    def render(self) -> str:
        return "Light button"


class LightCheckbox(Checkbox):
    def render(self) -> str:
        return "Light checkbox"


class DarkButton(Button):
    def render(self) -> str:
        return "Dark button"


class DarkCheckbox(Checkbox):
    def render(self) -> str:
        return "Dark checkbox"


class UIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass


class LightUIFactory(UIFactory):
    def create_button(self) -> Button:
        return LightButton()

    def create_checkbox(self) -> Checkbox:
        return LightCheckbox()


class DarkUIFactory(UIFactory):
    def create_button(self) -> Button:
        return DarkButton()

    def create_checkbox(self) -> Checkbox:
        return DarkCheckbox()


def build_screen(factory: UIFactory) -> None:
    print(factory.create_button().render())
    print(factory.create_checkbox().render())


build_screen(DarkUIFactory())
```

---

## C: familia de drivers

```c
#include <stdio.h>

typedef struct {
    void (*connect)(void);
} Database;

typedef struct {
    void (*publish)(const char *);
} Queue;

typedef struct {
    Database (*create_database)(void);
    Queue (*create_queue)(void);
} CloudFactory;

void aws_db_connect(void) { printf("AWS RDS connected\n"); }
void aws_queue_publish(const char *msg) { printf("AWS SQS: %s\n", msg); }

Database create_aws_database(void) {
    Database db = { aws_db_connect };
    return db;
}

Queue create_aws_queue(void) {
    Queue q = { aws_queue_publish };
    return q;
}

CloudFactory create_aws_factory(void) {
    CloudFactory factory = { create_aws_database, create_aws_queue };
    return factory;
}

int main(void) {
    CloudFactory factory = create_aws_factory();
    Database db = factory.create_database();
    Queue queue = factory.create_queue();

    db.connect();
    queue.publish("job-created");
    return 0;
}
```

---

## C++: widgets por plataforma

```cpp
#include <iostream>
#include <memory>

class Menu {
public:
    virtual ~Menu() = default;
    virtual void draw() = 0;
};

class Dialog {
public:
    virtual ~Dialog() = default;
    virtual void draw() = 0;
};

class WindowsMenu : public Menu {
public:
    void draw() override { std::cout << "Windows menu\n"; }
};

class WindowsDialog : public Dialog {
public:
    void draw() override { std::cout << "Windows dialog\n"; }
};

class MacMenu : public Menu {
public:
    void draw() override { std::cout << "Mac menu\n"; }
};

class MacDialog : public Dialog {
public:
    void draw() override { std::cout << "Mac dialog\n"; }
};

class UIFactory {
public:
    virtual ~UIFactory() = default;
    virtual std::unique_ptr<Menu> create_menu() = 0;
    virtual std::unique_ptr<Dialog> create_dialog() = 0;
};

class WindowsFactory : public UIFactory {
public:
    std::unique_ptr<Menu> create_menu() override {
        return std::make_unique<WindowsMenu>();
    }

    std::unique_ptr<Dialog> create_dialog() override {
        return std::make_unique<WindowsDialog>();
    }
};

int main() {
    WindowsFactory factory;
    auto menu = factory.create_menu();
    auto dialog = factory.create_dialog();

    menu->draw();
    dialog->draw();
}
```

---

## C#: familia de pagos

```csharp
using System;

public interface IPaymentGateway
{
    void Pay(decimal amount);
}

public interface IRefundGateway
{
    void Refund(decimal amount);
}

public class StripePayment : IPaymentGateway
{
    public void Pay(decimal amount) => Console.WriteLine($"Stripe pay {amount}");
}

public class StripeRefund : IRefundGateway
{
    public void Refund(decimal amount) => Console.WriteLine($"Stripe refund {amount}");
}

public interface IPaymentFactory
{
    IPaymentGateway CreatePayment();
    IRefundGateway CreateRefund();
}

public class StripeFactory : IPaymentFactory
{
    public IPaymentGateway CreatePayment() => new StripePayment();
    public IRefundGateway CreateRefund() => new StripeRefund();
}

IPaymentFactory factory = new StripeFactory();
factory.CreatePayment().Pay(100);
factory.CreateRefund().Refund(20);
```

---

# 3. Builder

## Intención

Construir objetos complejos paso a paso, separando el proceso de construcción de la representación final.

## Problema

Un constructor tiene demasiados parámetros o existen muchas configuraciones opcionales.

## Cuándo usarlo

Úsalo cuando:

```text
El objeto requiere muchos parámetros.
Hay pasos de construcción.
Quieres validar antes de construir.
Quieres crear variantes del mismo objeto.
```

Evítalo cuando:

```text
El objeto tiene pocos campos.
Un dataclass o constructor simple basta.
```

---

## Python: configuración de pipeline

```python
from dataclasses import dataclass, field


@dataclass(frozen=True)
class PipelineConfig:
    source: str
    destination: str
    batch_size: int = 1000
    retries: int = 3
    validations: list[str] = field(default_factory=list)


class PipelineConfigBuilder:
    def __init__(self) -> None:
        self._source = ""
        self._destination = ""
        self._batch_size = 1000
        self._retries = 3
        self._validations: list[str] = []

    def source(self, value: str) -> "PipelineConfigBuilder":
        self._source = value
        return self

    def destination(self, value: str) -> "PipelineConfigBuilder":
        self._destination = value
        return self

    def batch_size(self, value: int) -> "PipelineConfigBuilder":
        self._batch_size = value
        return self

    def add_validation(self, value: str) -> "PipelineConfigBuilder":
        self._validations.append(value)
        return self

    def build(self) -> PipelineConfig:
        if not self._source or not self._destination:
            raise ValueError("source and destination are required")

        return PipelineConfig(
            source=self._source,
            destination=self._destination,
            batch_size=self._batch_size,
            retries=self._retries,
            validations=self._validations.copy(),
        )


config = (
    PipelineConfigBuilder()
    .source("s3://raw")
    .destination("postgres://warehouse")
    .batch_size(500)
    .add_validation("not_null")
    .build()
)
```

---

## C: builder de consulta SQL

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char query[512];
} QueryBuilder;

void qb_init(QueryBuilder *qb) {
    strcpy(qb->query, "SELECT ");
}

void qb_columns(QueryBuilder *qb, const char *columns) {
    strcat(qb->query, columns);
}

void qb_from(QueryBuilder *qb, const char *table) {
    strcat(qb->query, " FROM ");
    strcat(qb->query, table);
}

void qb_where(QueryBuilder *qb, const char *condition) {
    strcat(qb->query, " WHERE ");
    strcat(qb->query, condition);
}

int main(void) {
    QueryBuilder qb;
    qb_init(&qb);
    qb_columns(&qb, "id, name");
    qb_from(&qb, "users");
    qb_where(&qb, "active = true");

    printf("%s\n", qb.query);
    return 0;
}
```

---

## C++: construcción de email

```cpp
#include <iostream>
#include <string>
#include <vector>

class Email {
public:
    std::string to;
    std::string subject;
    std::string body;
    std::vector<std::string> attachments;
};

class EmailBuilder {
    Email email;

public:
    EmailBuilder& to(const std::string& value) {
        email.to = value;
        return *this;
    }

    EmailBuilder& subject(const std::string& value) {
        email.subject = value;
        return *this;
    }

    EmailBuilder& body(const std::string& value) {
        email.body = value;
        return *this;
    }

    EmailBuilder& attach(const std::string& file) {
        email.attachments.push_back(file);
        return *this;
    }

    Email build() {
        return email;
    }
};

int main() {
    Email email = EmailBuilder()
        .to("client@example.com")
        .subject("Report")
        .body("Attached report.")
        .attach("report.pdf")
        .build();

    std::cout << email.subject << "\n";
}
```

---

## C#: construcción de request HTTP

```csharp
using System;
using System.Collections.Generic;

public class ApiRequest
{
    public string Url { get; init; } = "";
    public string Method { get; init; } = "GET";
    public Dictionary<string, string> Headers { get; init; } = new();
}

public class ApiRequestBuilder
{
    private readonly ApiRequest _request = new();

    public ApiRequestBuilder Url(string url)
    {
        _request.Url = url;
        return this;
    }

    public ApiRequestBuilder Method(string method)
    {
        _request.Method = method;
        return this;
    }

    public ApiRequestBuilder Header(string key, string value)
    {
        _request.Headers[key] = value;
        return this;
    }

    public ApiRequest Build()
    {
        if (string.IsNullOrWhiteSpace(_request.Url))
            throw new InvalidOperationException("Url is required");

        return _request;
    }
}

var request = new ApiRequestBuilder()
    .Url("https://api.example.com/tasks")
    .Method("POST")
    .Header("Content-Type", "application/json")
    .Build();
```

---

# 4. Prototype

## Intención

Crear objetos nuevos copiando prototipos existentes.

## Problema

Crear un objeto desde cero es costoso o complejo, pero puedes clonar uno existente y modificarlo.

## Cuándo usarlo

Úsalo cuando:

```text
La creación es costosa.
Hay objetos base preconfigurados.
Quieres evitar clases concretas repetidas.
Necesitas snapshots editables.
```

Evítalo cuando:

```text
Copiar estado puede causar errores.
Hay recursos no clonables.
El constructor simple basta.
```

---

## Python: plantillas de documentos

```python
import copy
from dataclasses import dataclass, field


@dataclass
class Document:
    title: str
    sections: list[str] = field(default_factory=list)

    def clone(self) -> "Document":
        return copy.deepcopy(self)


base = Document(
    title="Base contract",
    sections=["Parties", "Terms", "Signatures"],
)

client_contract = base.clone()
client_contract.title = "Contract for Client A"
client_contract.sections.append("Special conditions")

print(base.sections)
print(client_contract.sections)
```

---

## C: clonar configuración

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char env[32];
    int retries;
    int timeout;
} Config;

Config clone_config(Config original) {
    return original;
}

int main(void) {
    Config base = {"production", 3, 30};
    Config custom = clone_config(base);

    strcpy(custom.env, "staging");
    custom.timeout = 10;

    printf("%s %d\n", custom.env, custom.timeout);
    return 0;
}
```

---

## C++: clonar enemigos en juego

```cpp
#include <iostream>
#include <memory>
#include <string>

class Enemy {
public:
    virtual ~Enemy() = default;
    virtual std::unique_ptr<Enemy> clone() const = 0;
    virtual void info() const = 0;
};

class RobotEnemy : public Enemy {
    std::string weapon;

public:
    RobotEnemy(std::string weapon) : weapon(std::move(weapon)) {}

    std::unique_ptr<Enemy> clone() const override {
        return std::make_unique<RobotEnemy>(*this);
    }

    void info() const override {
        std::cout << "Robot with " << weapon << "\n";
    }
};

int main() {
    RobotEnemy prototype("laser");
    auto copy = prototype.clone();
    copy->info();
}
```

---

## C#: clonar campaña

```csharp
using System;
using System.Collections.Generic;

public class Campaign
{
    public string Name { get; set; } = "";
    public List<string> Channels { get; set; } = new();

    public Campaign Clone()
    {
        return new Campaign
        {
            Name = Name,
            Channels = new List<string>(Channels)
        };
    }
}

var baseCampaign = new Campaign
{
    Name = "Base Launch",
    Channels = new List<string> { "Email", "LinkedIn" }
};

var newCampaign = baseCampaign.Clone();
newCampaign.Name = "Product Launch";
newCampaign.Channels.Add("Webinar");

Console.WriteLine(string.Join(", ", newCampaign.Channels));
```

---

# 5. Singleton

## Intención

Garantizar que exista una sola instancia de una clase y proporcionar un punto global de acceso.

## Problema

Algunos recursos deben ser compartidos y únicos, como configuración, logger o conexión centralizada.

## Cuándo usarlo

Úsalo con cuidado cuando:

```text
Debe existir exactamente una instancia.
El estado compartido está justificado.
Quieres centralizar acceso a recurso único.
```

Evítalo cuando:

```text
Solo buscas una variable global cómoda.
Dificulta tests.
Introduce estado oculto.
Rompe inyección de dependencias.
```

En Python, muchas veces un módulo simple reemplaza Singleton.

---

## Python: configuración de aplicación

```python
class Settings:
    _instance: "Settings | None" = None

    def __new__(cls) -> "Settings":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.env = "production"
            cls._instance.log_level = "INFO"

        return cls._instance


a = Settings()
b = Settings()

print(a is b)
print(a.env)
```

Alternativa Python más simple:

```python
# settings.py
APP_ENV = "production"
LOG_LEVEL = "INFO"
```

---

## C: singleton de logger

```c
#include <stdio.h>

typedef struct {
    FILE *output;
} Logger;

Logger *get_logger(void) {
    static Logger logger = { NULL };

    if (logger.output == NULL) {
        logger.output = stdout;
    }

    return &logger;
}

int main(void) {
    Logger *logger = get_logger();
    fprintf(logger->output, "Application started\n");
    return 0;
}
```

---

## C++: singleton thread-safe local static

```cpp
#include <iostream>

class MetricsRegistry {
public:
    static MetricsRegistry& instance() {
        static MetricsRegistry registry;
        return registry;
    }

    void increment(const std::string& name) {
        std::cout << "Increment " << name << "\n";
    }

private:
    MetricsRegistry() = default;
};

int main() {
    MetricsRegistry::instance().increment("requests_total");
}
```

---

## C#: singleton con Lazy

```csharp
using System;

public sealed class AppConfig
{
    private static readonly Lazy<AppConfig> _instance = new(() => new AppConfig());

    public static AppConfig Instance => _instance.Value;

    public string Environment { get; } = "Production";

    private AppConfig()
    {
    }
}

Console.WriteLine(AppConfig.Instance.Environment);
```

---

# Parte III: patrones estructurales

Los patrones estructurales organizan clases y objetos para formar estructuras flexibles.

---

# 6. Adapter

## Intención

Convertir la interfaz de una clase en otra interfaz esperada por el cliente.

## Problema

Tienes una clase o servicio útil, pero su interfaz no coincide con la que tu código espera.

## Cuándo usarlo

Úsalo cuando:

```text
Integras librerías externas.
Migras de una API antigua a una nueva.
Quieres aislar dependencias externas.
Necesitas compatibilidad entre interfaces.
```

Evítalo cuando:

```text
Puedes cambiar directamente la interfaz original.
El adapter solo oculta un mal diseño sin simplificarlo.
```

---

## Python: adaptar pasarela de pagos externa

```python
class PaymentProcessor:
    def charge(self, amount: float) -> None:
        raise NotImplementedError


class ExternalStripeClient:
    def create_payment(self, cents: int) -> None:
        print(f"Stripe charged {cents} cents")


class StripeAdapter(PaymentProcessor):
    def __init__(self, client: ExternalStripeClient) -> None:
        self.client = client

    def charge(self, amount: float) -> None:
        self.client.create_payment(int(amount * 100))


processor = StripeAdapter(ExternalStripeClient())
processor.charge(19.99)
```

---

## C: adaptar sensor antiguo

```c
#include <stdio.h>

typedef struct {
    int (*read_celsius)(void);
} TemperatureSensor;

int legacy_read_fahrenheit(void) {
    return 77;
}

int adapted_read_celsius(void) {
    int f = legacy_read_fahrenheit();
    return (f - 32) * 5 / 9;
}

int main(void) {
    TemperatureSensor sensor = { adapted_read_celsius };
    printf("%d C\n", sensor.read_celsius());
    return 0;
}
```

---

## C++: adaptar logger legacy

```cpp
#include <iostream>
#include <string>

class Logger {
public:
    virtual ~Logger() = default;
    virtual void info(const std::string& message) = 0;
};

class LegacyLogger {
public:
    void write(const char* level, const char* text) {
        std::cout << "[" << level << "] " << text << "\n";
    }
};

class LegacyLoggerAdapter : public Logger {
    LegacyLogger legacy;

public:
    void info(const std::string& message) override {
        legacy.write("INFO", message.c_str());
    }
};

int main() {
    LegacyLoggerAdapter logger;
    logger.info("Adapter working");
}
```

---

## C#: adaptar proveedor de clima

```csharp
using System;

public interface IWeatherService
{
    decimal GetTemperatureCelsius(string city);
}

public class ExternalWeatherApi
{
    public decimal GetTempFahrenheit(string city) => 77m;
}

public class WeatherApiAdapter : IWeatherService
{
    private readonly ExternalWeatherApi _api;

    public WeatherApiAdapter(ExternalWeatherApi api)
    {
        _api = api;
    }

    public decimal GetTemperatureCelsius(string city)
    {
        var f = _api.GetTempFahrenheit(city);
        return (f - 32m) * 5m / 9m;
    }
}

IWeatherService weather = new WeatherApiAdapter(new ExternalWeatherApi());
Console.WriteLine(weather.GetTemperatureCelsius("Santiago"));
```

---

# 7. Bridge

## Intención

Separar una abstracción de su implementación para que ambas puedan variar independientemente.

## Problema

Tienes combinaciones crecientes de abstracciones e implementaciones.

## Cuándo usarlo

Úsalo cuando:

```text
Hay múltiples dimensiones de variación.
No quieres una explosión de subclases.
Quieres cambiar implementación en runtime.
```

Evítalo cuando:

```text
Solo tienes una dimensión simple.
La separación agrega complejidad innecesaria.
```

---

## Python: reportes y canales de salida

```python
from abc import ABC, abstractmethod


class ReportOutput(ABC):
    @abstractmethod
    def write(self, content: str) -> None:
        pass


class ConsoleOutput(ReportOutput):
    def write(self, content: str) -> None:
        print(content)


class FileOutput(ReportOutput):
    def write(self, content: str) -> None:
        print(f"Writing to file: {content}")


class Report:
    def __init__(self, output: ReportOutput) -> None:
        self.output = output

    def render(self) -> None:
        self.output.write("Generic report")


class SalesReport(Report):
    def render(self) -> None:
        self.output.write("Sales report")


report = SalesReport(ConsoleOutput())
report.render()
```

---

## C: alarma y canal

```c
#include <stdio.h>

typedef struct {
    void (*send)(const char *);
} Channel;

void email_send(const char *msg) {
    printf("Email: %s\n", msg);
}

void sms_send(const char *msg) {
    printf("SMS: %s\n", msg);
}

typedef struct {
    Channel channel;
} Alert;

void alert_trigger(Alert *alert, const char *msg) {
    alert->channel.send(msg);
}

int main(void) {
    Channel sms = { sms_send };
    Alert alert = { sms };

    alert_trigger(&alert, "CPU high");
    return 0;
}
```

---

## C++: controles remotos y dispositivos

```cpp
#include <iostream>
#include <memory>

class Device {
public:
    virtual ~Device() = default;
    virtual void turn_on() = 0;
};

class Tv : public Device {
public:
    void turn_on() override { std::cout << "TV on\n"; }
};

class Remote {
protected:
    std::shared_ptr<Device> device;

public:
    Remote(std::shared_ptr<Device> device) : device(std::move(device)) {}
    virtual void power() { device->turn_on(); }
};

class AdvancedRemote : public Remote {
public:
    using Remote::Remote;

    void power() override {
        std::cout << "Advanced remote: ";
        device->turn_on();
    }
};

int main() {
    auto tv = std::make_shared<Tv>();
    AdvancedRemote remote(tv);
    remote.power();
}
```

---

## C#: mensajes y transportes

```csharp
using System;

public interface IMessageTransport
{
    void Send(string payload);
}

public class HttpTransport : IMessageTransport
{
    public void Send(string payload) => Console.WriteLine($"HTTP: {payload}");
}

public abstract class Message
{
    protected readonly IMessageTransport Transport;

    protected Message(IMessageTransport transport)
    {
        Transport = transport;
    }

    public abstract void Dispatch();
}

public class OrderMessage : Message
{
    public OrderMessage(IMessageTransport transport) : base(transport) {}

    public override void Dispatch()
    {
        Transport.Send("order-created");
    }
}

var message = new OrderMessage(new HttpTransport());
message.Dispatch();
```

---

# 8. Composite

## Intención

Componer objetos en estructuras de árbol y tratar objetos individuales y composiciones de forma uniforme.

## Problema

Necesitas manipular elementos individuales y grupos con la misma interfaz.

## Cuándo usarlo

Úsalo cuando:

```text
Tienes jerarquías tipo árbol.
Carpetas y archivos.
Menús y submenús.
Organizaciones.
Componentes UI anidados.
```

Evítalo cuando:

```text
La estructura no es jerárquica.
Las operaciones de hoja y contenedor son muy diferentes.
```

---

## Python: menú anidado

```python
from abc import ABC, abstractmethod


class MenuItem(ABC):
    @abstractmethod
    def render(self, indent: int = 0) -> None:
        pass


class Link(MenuItem):
    def __init__(self, label: str) -> None:
        self.label = label

    def render(self, indent: int = 0) -> None:
        print(" " * indent + f"- {self.label}")


class Menu(MenuItem):
    def __init__(self, label: str) -> None:
        self.label = label
        self.children: list[MenuItem] = []

    def add(self, item: MenuItem) -> None:
        self.children.append(item)

    def render(self, indent: int = 0) -> None:
        print(" " * indent + self.label)
        for child in self.children:
            child.render(indent + 2)


root = Menu("Admin")
root.add(Link("Users"))
settings = Menu("Settings")
settings.add(Link("Security"))
root.add(settings)
root.render()
```

---

## C: árbol de archivos simple

```c
#include <stdio.h>

typedef struct Node Node;

struct Node {
    const char *name;
    Node **children;
    int child_count;
};

void print_node(Node *node, int indent) {
    for (int i = 0; i < indent; i++) printf(" ");
    printf("%s\n", node->name);

    for (int i = 0; i < node->child_count; i++) {
        print_node(node->children[i], indent + 2);
    }
}

int main(void) {
    Node file = {"report.pdf", NULL, 0};
    Node *children[] = {&file};
    Node folder = {"docs", children, 1};

    print_node(&folder, 0);
    return 0;
}
```

---

## C++: organización empresarial

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

class OrgUnit {
public:
    virtual ~OrgUnit() = default;
    virtual void print(int indent = 0) const = 0;
};

class Employee : public OrgUnit {
    std::string name;

public:
    Employee(std::string name) : name(std::move(name)) {}

    void print(int indent = 0) const override {
        std::cout << std::string(indent, ' ') << name << "\n";
    }
};

class Department : public OrgUnit {
    std::string name;
    std::vector<std::unique_ptr<OrgUnit>> children;

public:
    Department(std::string name) : name(std::move(name)) {}

    void add(std::unique_ptr<OrgUnit> child) {
        children.push_back(std::move(child));
    }

    void print(int indent = 0) const override {
        std::cout << std::string(indent, ' ') << name << "\n";
        for (const auto& child : children) {
            child->print(indent + 2);
        }
    }
};

int main() {
    Department engineering("Engineering");
    engineering.add(std::make_unique<Employee>("Ana"));
    engineering.print();
}
```

---

## C#: componentes UI

```csharp
using System;
using System.Collections.Generic;

public interface IComponent
{
    void Render(int indent = 0);
}

public class Text : IComponent
{
    private readonly string _value;

    public Text(string value)
    {
        _value = value;
    }

    public void Render(int indent = 0)
    {
        Console.WriteLine(new string(' ', indent) + _value);
    }
}

public class Panel : IComponent
{
    private readonly List<IComponent> _children = new();

    public void Add(IComponent child) => _children.Add(child);

    public void Render(int indent = 0)
    {
        Console.WriteLine(new string(' ', indent) + "Panel");
        foreach (var child in _children)
            child.Render(indent + 2);
    }
}

var panel = new Panel();
panel.Add(new Text("Hello"));
panel.Render();
```

---

# 9. Decorator

## Intención

Agregar responsabilidades a un objeto dinámicamente sin modificar su clase.

## Problema

Quieres extender comportamiento sin crear muchas subclases.

## Cuándo usarlo

Úsalo cuando:

```text
Quieres agregar funcionalidades combinables.
Necesitas envolver comportamiento.
Quieres respetar Open/Closed Principle.
```

Evítalo cuando:

```text
Las capas se vuelven difíciles de seguir.
Hay orden de decoradores muy frágil.
```

---

## Python: middleware de repositorio

```python
from abc import ABC, abstractmethod


class TaskRepository(ABC):
    @abstractmethod
    def save(self, title: str) -> None:
        pass


class PostgresTaskRepository(TaskRepository):
    def save(self, title: str) -> None:
        print(f"Saving task: {title}")


class LoggingTaskRepository(TaskRepository):
    def __init__(self, wrapped: TaskRepository) -> None:
        self.wrapped = wrapped

    def save(self, title: str) -> None:
        print(f"Before save: {title}")
        self.wrapped.save(title)
        print("After save")


repo = LoggingTaskRepository(PostgresTaskRepository())
repo.save("Write guide")
```

---

## C: decorar impresión

```c
#include <stdio.h>

typedef struct Printer {
    void (*print)(const char *);
} Printer;

void plain_print(const char *text) {
    printf("%s\n", text);
}

void uppercase_print(const char *text) {
    printf("[UPPERCASE DECORATOR] %s\n", text);
}

int main(void) {
    Printer printer = { plain_print };
    printer.print("hello");

    printer.print = uppercase_print;
    printer.print("hello");

    return 0;
}
```

---

## C++: stream cifrado

```cpp
#include <iostream>
#include <memory>
#include <string>

class Writer {
public:
    virtual ~Writer() = default;
    virtual void write(const std::string& data) = 0;
};

class FileWriter : public Writer {
public:
    void write(const std::string& data) override {
        std::cout << "File: " << data << "\n";
    }
};

class EncryptionWriter : public Writer {
    std::unique_ptr<Writer> wrapped;

public:
    EncryptionWriter(std::unique_ptr<Writer> wrapped)
        : wrapped(std::move(wrapped)) {}

    void write(const std::string& data) override {
        wrapped->write("encrypted(" + data + ")");
    }
};

int main() {
    auto writer = std::make_unique<EncryptionWriter>(
        std::make_unique<FileWriter>()
    );
    writer->write("secret");
}
```

---

## C#: descuentos acumulables

```csharp
using System;

public interface IPriceCalculator
{
    decimal Calculate(decimal amount);
}

public class BasePriceCalculator : IPriceCalculator
{
    public decimal Calculate(decimal amount) => amount;
}

public class CouponDecorator : IPriceCalculator
{
    private readonly IPriceCalculator _wrapped;

    public CouponDecorator(IPriceCalculator wrapped)
    {
        _wrapped = wrapped;
    }

    public decimal Calculate(decimal amount)
    {
        return _wrapped.Calculate(amount) * 0.9m;
    }
}

IPriceCalculator calculator = new CouponDecorator(new BasePriceCalculator());
Console.WriteLine(calculator.Calculate(100));
```

---

# 10. Facade

## Intención

Proporcionar una interfaz simple a un subsistema complejo.

## Problema

Un cliente debe interactuar con muchas clases o pasos para lograr una operación común.

## Cuándo usarlo

Úsalo cuando:

```text
Quieres simplificar una API interna.
Necesitas ocultar complejidad.
Quieres reducir acoplamiento con subsistemas.
```

Evítalo cuando:

```text
La fachada se convierte en objeto gigante.
Oculta errores importantes.
Mezcla demasiadas responsabilidades.
```

---

## Python: procesamiento de video

```python
class VideoDecoder:
    def decode(self, path: str) -> str:
        return f"decoded:{path}"


class Compressor:
    def compress(self, video: str) -> str:
        return f"compressed:{video}"


class Uploader:
    def upload(self, video: str) -> None:
        print(f"Uploaded {video}")


class VideoService:
    def __init__(self) -> None:
        self.decoder = VideoDecoder()
        self.compressor = Compressor()
        self.uploader = Uploader()

    def process_and_upload(self, path: str) -> None:
        decoded = self.decoder.decode(path)
        compressed = self.compressor.compress(decoded)
        self.uploader.upload(compressed)


VideoService().process_and_upload("input.mp4")
```

---

## C: fachada para red

```c
#include <stdio.h>

void socket_open(void) { printf("socket open\n"); }
void socket_send(const char *data) { printf("send: %s\n", data); }
void socket_close(void) { printf("socket close\n"); }

void send_message(const char *data) {
    socket_open();
    socket_send(data);
    socket_close();
}

int main(void) {
    send_message("hello");
    return 0;
}
```

---

## C++: fachada de render

```cpp
#include <iostream>

class Shader {
public:
    void compile() { std::cout << "compile shader\n"; }
};

class Mesh {
public:
    void load() { std::cout << "load mesh\n"; }
};

class RendererFacade {
    Shader shader;
    Mesh mesh;

public:
    void render_scene() {
        shader.compile();
        mesh.load();
        std::cout << "render scene\n";
    }
};

int main() {
    RendererFacade renderer;
    renderer.render_scene();
}
```

---

## C#: fachada de onboarding

```csharp
using System;

public class UserCreator
{
    public void Create(string email) => Console.WriteLine($"Create {email}");
}

public class EmailSender
{
    public void SendWelcome(string email) => Console.WriteLine($"Welcome {email}");
}

public class AuditLogger
{
    public void Log(string eventName) => Console.WriteLine($"Audit {eventName}");
}

public class OnboardingFacade
{
    private readonly UserCreator _users = new();
    private readonly EmailSender _emails = new();
    private readonly AuditLogger _audit = new();

    public void Register(string email)
    {
        _users.Create(email);
        _emails.SendWelcome(email);
        _audit.Log("user_registered");
    }
}

new OnboardingFacade().Register("user@example.com");
```

---

# 11. Flyweight

## Intención

Compartir objetos pequeños e inmutables para reducir memoria.

## Problema

Tienes muchos objetos similares que repiten estado común.

## Cuándo usarlo

Úsalo cuando:

```text
Hay miles o millones de objetos.
Mucho estado se repite.
Puedes separar estado intrínseco y extrínseco.
```

Evítalo cuando:

```text
La cantidad de objetos es pequeña.
La optimización no es necesaria.
Complica el código sin beneficio medible.
```

---

## Python: estilos de texto compartidos

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class TextStyle:
    font: str
    size: int
    color: str


class StyleFactory:
    _cache: dict[tuple[str, int, str], TextStyle] = {}

    @classmethod
    def get(cls, font: str, size: int, color: str) -> TextStyle:
        key = (font, size, color)

        if key not in cls._cache:
            cls._cache[key] = TextStyle(font, size, color)

        return cls._cache[key]


style_a = StyleFactory.get("Inter", 14, "black")
style_b = StyleFactory.get("Inter", 14, "black")

print(style_a is style_b)
```

---

## C: símbolos compartidos

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char symbol[8];
} Currency;

Currency usd = {"USD"};
Currency eur = {"EUR"};

Currency *get_currency(const char *code) {
    if (strcmp(code, "USD") == 0) return &usd;
    return &eur;
}

int main(void) {
    Currency *a = get_currency("USD");
    Currency *b = get_currency("USD");

    printf("%d\n", a == b);
    return 0;
}
```

---

## C++: partículas de juego

```cpp
#include <iostream>
#include <map>
#include <memory>
#include <string>

class ParticleType {
public:
    std::string texture;

    explicit ParticleType(std::string texture)
        : texture(std::move(texture)) {}
};

class ParticleFactory {
    std::map<std::string, std::shared_ptr<ParticleType>> cache;

public:
    std::shared_ptr<ParticleType> get(const std::string& texture) {
        if (!cache.contains(texture)) {
            cache[texture] = std::make_shared<ParticleType>(texture);
        }

        return cache[texture];
    }
};

int main() {
    ParticleFactory factory;
    auto smoke = factory.get("smoke.png");
    auto smoke2 = factory.get("smoke.png");

    std::cout << (smoke == smoke2) << "\n";
}
```

---

## C#: iconos compartidos

```csharp
using System;
using System.Collections.Generic;

public record Icon(string Name, string Svg);

public class IconFactory
{
    private readonly Dictionary<string, Icon> _cache = new();

    public Icon Get(string name)
    {
        if (!_cache.ContainsKey(name))
            _cache[name] = new Icon(name, $"<svg>{name}</svg>");

        return _cache[name];
    }
}

var factory = new IconFactory();
var a = factory.Get("save");
var b = factory.Get("save");

Console.WriteLine(ReferenceEquals(a, b));
```

---

# 12. Proxy

## Intención

Proporcionar un sustituto o representante de otro objeto para controlar acceso.

## Problema

Quieres agregar control, caching, lazy loading, seguridad o acceso remoto sin cambiar el objeto real.

## Cuándo usarlo

Úsalo cuando:

```text
Necesitas lazy loading.
Quieres cachear resultados.
Quieres controlar permisos.
Quieres registrar llamadas.
Quieres representar un servicio remoto.
```

Evítalo cuando:

```text
Solo agrega una capa innecesaria.
Oculta latencias o errores importantes.
```

---

## Python: proxy con cache

```python
class WeatherApi:
    def get_temperature(self, city: str) -> float:
        print("Calling external API")
        return 21.5


class CachedWeatherProxy:
    def __init__(self, api: WeatherApi) -> None:
        self.api = api
        self.cache: dict[str, float] = {}

    def get_temperature(self, city: str) -> float:
        if city not in self.cache:
            self.cache[city] = self.api.get_temperature(city)

        return self.cache[city]


weather = CachedWeatherProxy(WeatherApi())
print(weather.get_temperature("Santiago"))
print(weather.get_temperature("Santiago"))
```

---

## C: proxy de acceso

```c
#include <stdio.h>
#include <string.h>

void delete_file_real(const char *path) {
    printf("Deleting %s\n", path);
}

void delete_file_proxy(const char *user, const char *path) {
    if (strcmp(user, "admin") != 0) {
        printf("Access denied\n");
        return;
    }

    delete_file_real(path);
}

int main(void) {
    delete_file_proxy("guest", "/tmp/a.txt");
    delete_file_proxy("admin", "/tmp/a.txt");
    return 0;
}
```

---

## C++: imagen lazy

```cpp
#include <iostream>
#include <memory>
#include <string>

class Image {
public:
    virtual ~Image() = default;
    virtual void display() = 0;
};

class RealImage : public Image {
    std::string path;

public:
    explicit RealImage(std::string path) : path(std::move(path)) {
        std::cout << "Loading " << this->path << "\n";
    }

    void display() override {
        std::cout << "Display " << path << "\n";
    }
};

class ImageProxy : public Image {
    std::string path;
    std::unique_ptr<RealImage> real;

public:
    explicit ImageProxy(std::string path) : path(std::move(path)) {}

    void display() override {
        if (!real) {
            real = std::make_unique<RealImage>(path);
        }

        real->display();
    }
};

int main() {
    ImageProxy image("large.png");
    image.display();
}
```

---

## C#: proxy de autorización

```csharp
using System;

public interface IReportService
{
    void Download(string reportId);
}

public class ReportService : IReportService
{
    public void Download(string reportId) =>
        Console.WriteLine($"Downloading {reportId}");
}

public class AuthorizedReportProxy : IReportService
{
    private readonly IReportService _service;
    private readonly string _role;

    public AuthorizedReportProxy(IReportService service, string role)
    {
        _service = service;
        _role = role;
    }

    public void Download(string reportId)
    {
        if (_role != "admin")
            throw new UnauthorizedAccessException();

        _service.Download(reportId);
    }
}

IReportService service = new AuthorizedReportProxy(new ReportService(), "admin");
service.Download("sales");
```

---

# Parte IV: patrones de comportamiento

Los patrones de comportamiento organizan comunicación, responsabilidades y algoritmos.

---

# 13. Chain of Responsibility

## Intención

Pasar una solicitud por una cadena de manejadores hasta que alguno la procese.

## Problema

Varias partes pueden manejar una solicitud, pero no quieres acoplar el emisor al receptor concreto.

## Cuándo usarlo

Úsalo cuando:

```text
Hay una cadena de validaciones.
Hay middleware.
Hay autorización por capas.
Hay procesamiento secuencial extensible.
```

Evítalo cuando:

```text
El flujo debe ser muy explícito.
Es difícil saber quién manejó la solicitud.
```

---

## Python: validaciones de request

```python
from abc import ABC, abstractmethod


class Handler(ABC):
    def __init__(self, next_handler: "Handler | None" = None) -> None:
        self.next_handler = next_handler

    def handle(self, request: dict) -> None:
        self.process(request)

        if self.next_handler:
            self.next_handler.handle(request)

    @abstractmethod
    def process(self, request: dict) -> None:
        pass


class AuthHandler(Handler):
    def process(self, request: dict) -> None:
        if not request.get("user"):
            raise PermissionError("Missing user")


class ValidationHandler(Handler):
    def process(self, request: dict) -> None:
        if not request.get("payload"):
            raise ValueError("Missing payload")


chain = AuthHandler(ValidationHandler())
chain.handle({"user": "ana", "payload": {"x": 1}})
```

---

## C: pipeline de filtros

```c
#include <stdio.h>

typedef int (*Handler)(int);

int positive_filter(int value) {
    if (value <= 0) {
        printf("Rejected: not positive\n");
        return 0;
    }
    return 1;
}

int max_filter(int value) {
    if (value > 100) {
        printf("Rejected: too large\n");
        return 0;
    }
    return 1;
}

int main(void) {
    Handler chain[] = { positive_filter, max_filter };

    int value = 50;
    for (int i = 0; i < 2; i++) {
        if (!chain[i](value)) return 1;
    }

    printf("Accepted\n");
    return 0;
}
```

---

## C++: soporte técnico

```cpp
#include <iostream>
#include <memory>
#include <string>

class SupportHandler {
protected:
    std::shared_ptr<SupportHandler> next;

public:
    virtual ~SupportHandler() = default;

    void set_next(std::shared_ptr<SupportHandler> handler) {
        next = handler;
    }

    virtual void handle(const std::string& issue) {
        if (next) {
            next->handle(issue);
        }
    }
};

class BillingSupport : public SupportHandler {
public:
    void handle(const std::string& issue) override {
        if (issue == "billing") {
            std::cout << "Billing handled\n";
        } else {
            SupportHandler::handle(issue);
        }
    }
};

int main() {
    auto billing = std::make_shared<BillingSupport>();
    billing->handle("billing");
}
```

---

## C#: middleware HTTP

```csharp
using System;

public abstract class Middleware
{
    private Middleware? _next;

    public Middleware Link(Middleware next)
    {
        _next = next;
        return next;
    }

    public void Handle(string request)
    {
        Process(request);
        _next?.Handle(request);
    }

    protected abstract void Process(string request);
}

public class LoggingMiddleware : Middleware
{
    protected override void Process(string request) =>
        Console.WriteLine($"Log {request}");
}

public class AuthMiddleware : Middleware
{
    protected override void Process(string request) =>
        Console.WriteLine("Auth OK");
}

var log = new LoggingMiddleware();
log.Link(new AuthMiddleware());
log.Handle("/tasks");
```

---

# 14. Command

## Intención

Encapsular una solicitud como objeto.

## Problema

Quieres parametrizar acciones, encolarlas, registrarlas, deshacerlas o ejecutarlas más tarde.

## Cuándo usarlo

Úsalo cuando:

```text
Necesitas undo/redo.
Quieres colas de trabajos.
Quieres invocar acciones de forma uniforme.
Quieres registrar comandos.
```

Evítalo cuando:

```text
Una función simple basta.
Hay demasiadas clases para acciones triviales.
```

---

## Python: cola de tareas

```python
from abc import ABC, abstractmethod
from collections import deque


class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass


class SendEmailCommand(Command):
    def __init__(self, to: str) -> None:
        self.to = to

    def execute(self) -> None:
        print(f"Sending email to {self.to}")


queue: deque[Command] = deque()
queue.append(SendEmailCommand("user@example.com"))

while queue:
    queue.popleft().execute()
```

---

## C: comandos de robot

```c
#include <stdio.h>

typedef struct {
    void (*execute)(void);
} Command;

void move_forward(void) {
    printf("Robot moves forward\n");
}

void turn_left(void) {
    printf("Robot turns left\n");
}

int main(void) {
    Command commands[] = {
        { move_forward },
        { turn_left }
    };

    for (int i = 0; i < 2; i++) {
        commands[i].execute();
    }

    return 0;
}
```

---

## C++: editor con undo

```cpp
#include <iostream>
#include <memory>
#include <stack>
#include <string>

class Editor {
public:
    std::string text;

    void append(const std::string& value) {
        text += value;
    }
};

class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class AppendCommand : public Command {
    Editor& editor;
    std::string value;

public:
    AppendCommand(Editor& editor, std::string value)
        : editor(editor), value(std::move(value)) {}

    void execute() override {
        editor.append(value);
    }

    void undo() override {
        editor.text.erase(editor.text.size() - value.size());
    }
};

int main() {
    Editor editor;
    AppendCommand cmd(editor, "hello");
    cmd.execute();
    std::cout << editor.text << "\n";
    cmd.undo();
}
```

---

## C#: botón con comando

```csharp
using System;

public interface ICommand
{
    void Execute();
}

public class SaveCommand : ICommand
{
    public void Execute() => Console.WriteLine("Saving document");
}

public class Button
{
    private readonly ICommand _command;

    public Button(ICommand command)
    {
        _command = command;
    }

    public void Click() => _command.Execute();
}

var button = new Button(new SaveCommand());
button.Click();
```

---

# 15. Interpreter

## Intención

Definir una representación para una gramática simple y un intérprete para evaluarla.

## Problema

Necesitas evaluar expresiones de un lenguaje pequeño o reglas configurables.

## Cuándo usarlo

Úsalo cuando:

```text
Tienes una gramática pequeña.
Quieres reglas configurables.
Necesitas expresiones evaluables.
```

Evítalo cuando:

```text
La gramática es compleja.
Necesitas performance alta.
Conviene usar parser formal.
```

---

## Python: filtros de tareas

```python
from abc import ABC, abstractmethod


class Expression(ABC):
    @abstractmethod
    def evaluate(self, task: dict) -> bool:
        pass


class Completed(Expression):
    def evaluate(self, task: dict) -> bool:
        return bool(task.get("completed"))


class HasTag(Expression):
    def __init__(self, tag: str) -> None:
        self.tag = tag

    def evaluate(self, task: dict) -> bool:
        return self.tag in task.get("tags", [])


class And(Expression):
    def __init__(self, left: Expression, right: Expression) -> None:
        self.left = left
        self.right = right

    def evaluate(self, task: dict) -> bool:
        return self.left.evaluate(task) and self.right.evaluate(task)


expr = And(Completed(), HasTag("work"))
print(expr.evaluate({"completed": True, "tags": ["work"]}))
```

---

## C: expresión booleana simple

```c
#include <stdio.h>

typedef int (*Expr)(int);

int is_even(int value) {
    return value % 2 == 0;
}

int is_positive(int value) {
    return value > 0;
}

int and_expr(Expr a, Expr b, int value) {
    return a(value) && b(value);
}

int main(void) {
    printf("%d\n", and_expr(is_even, is_positive, 10));
    return 0;
}
```

---

## C++: reglas de descuento

```cpp
#include <iostream>
#include <memory>

class Expr {
public:
    virtual ~Expr() = default;
    virtual bool eval(double amount) const = 0;
};

class GreaterThan : public Expr {
    double threshold;

public:
    explicit GreaterThan(double threshold) : threshold(threshold) {}

    bool eval(double amount) const override {
        return amount > threshold;
    }
};

int main() {
    std::unique_ptr<Expr> rule = std::make_unique<GreaterThan>(100.0);
    std::cout << rule->eval(150.0) << "\n";
}
```

---

## C#: reglas de autorización

```csharp
using System;

public interface IExpression
{
    bool Evaluate(User user);
}

public record User(string Role, bool Active);

public class IsAdmin : IExpression
{
    public bool Evaluate(User user) => user.Role == "admin";
}

public class IsActive : IExpression
{
    public bool Evaluate(User user) => user.Active;
}

public class AndExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public AndExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }

    public bool Evaluate(User user) =>
        _left.Evaluate(user) && _right.Evaluate(user);
}

var rule = new AndExpression(new IsAdmin(), new IsActive());
Console.WriteLine(rule.Evaluate(new User("admin", true)));
```

---

# 16. Iterator

## Intención

Acceder secuencialmente a elementos de una colección sin exponer su representación interna.

## Problema

Quieres recorrer una estructura sin acoplarte a cómo almacena sus datos.

## Cuándo usarlo

Úsalo cuando:

```text
Tienes colecciones personalizadas.
Quieres ocultar estructura interna.
Necesitas recorridos perezosos.
```

Evítalo cuando:

```text
Una lista estándar basta.
No hay lógica de recorrido especial.
```

---

## Python: iterador de páginas API

```python
class PaginatedTasks:
    def __init__(self, pages: list[list[str]]) -> None:
        self.pages = pages

    def __iter__(self):
        for page in self.pages:
            for task in page:
                yield task


tasks = PaginatedTasks([
    ["task-1", "task-2"],
    ["task-3"],
])

for task in tasks:
    print(task)
```

---

## C: iterador de array

```c
#include <stdio.h>

typedef struct {
    int *items;
    int count;
    int index;
} IntIterator;

int has_next(IntIterator *it) {
    return it->index < it->count;
}

int next(IntIterator *it) {
    return it->items[it->index++];
}

int main(void) {
    int values[] = {1, 2, 3};
    IntIterator it = { values, 3, 0 };

    while (has_next(&it)) {
        printf("%d\n", next(&it));
    }

    return 0;
}
```

---

## C++: colección personalizada

```cpp
#include <iostream>
#include <vector>

class Numbers {
    std::vector<int> values;

public:
    void add(int value) {
        values.push_back(value);
    }

    auto begin() { return values.begin(); }
    auto end() { return values.end(); }
};

int main() {
    Numbers numbers;
    numbers.add(1);
    numbers.add(2);

    for (int value : numbers) {
        std::cout << value << "\n";
    }
}
```

---

## C#: iterador con yield

```csharp
using System;
using System.Collections.Generic;

public class TaskCollection
{
    private readonly List<string> _tasks = new() { "A", "B", "C" };

    public IEnumerable<string> PendingTasks()
    {
        foreach (var task in _tasks)
            yield return task;
    }
}

var collection = new TaskCollection();

foreach (var task in collection.PendingTasks())
    Console.WriteLine(task);
```

---

# 17. Mediator

## Intención

Reducir acoplamiento haciendo que objetos se comuniquen a través de un mediador.

## Problema

Muchos objetos se conocen entre sí directamente y la comunicación se vuelve caótica.

## Cuándo usarlo

Úsalo cuando:

```text
Hay muchos componentes colaborando.
La comunicación directa crea acoplamiento.
Quieres centralizar coordinación.
```

Evítalo cuando:

```text
El mediador se convierte en objeto dios.
La comunicación es simple.
```

---

## Python: chat room

```python
class ChatRoom:
    def __init__(self) -> None:
        self.users: list["User"] = []

    def register(self, user: "User") -> None:
        self.users.append(user)
        user.room = self

    def send(self, sender: "User", message: str) -> None:
        for user in self.users:
            if user is not sender:
                user.receive(sender.name, message)


class User:
    def __init__(self, name: str) -> None:
        self.name = name
        self.room: ChatRoom | None = None

    def send(self, message: str) -> None:
        self.room.send(self, message)

    def receive(self, sender: str, message: str) -> None:
        print(f"{self.name} received from {sender}: {message}")


room = ChatRoom()
ana = User("Ana")
bob = User("Bob")
room.register(ana)
room.register(bob)
ana.send("Hello")
```

---

## C: mediador de componentes

```c
#include <stdio.h>

typedef struct Mediator Mediator;

struct Mediator {
    void (*notify)(const char *event);
};

void app_notify(const char *event) {
    printf("Mediator received: %s\n", event);
}

int main(void) {
    Mediator mediator = { app_notify };
    mediator.notify("button_clicked");
    return 0;
}
```

---

## C++: diálogo UI

```cpp
#include <iostream>
#include <string>

class DialogMediator {
public:
    void notify(const std::string& sender, const std::string& event) {
        std::cout << sender << " -> " << event << "\n";

        if (event == "login_clicked") {
            std::cout << "Validate form and submit\n";
        }
    }
};

class Button {
    DialogMediator& mediator;

public:
    Button(DialogMediator& mediator) : mediator(mediator) {}

    void click() {
        mediator.notify("button", "login_clicked");
    }
};

int main() {
    DialogMediator mediator;
    Button button(mediator);
    button.click();
}
```

---

## C#: coordinación de formulario

```csharp
using System;

public class FormMediator
{
    public void Notify(string component, string ev)
    {
        if (component == "Country" && ev == "Changed")
            Console.WriteLine("Reload city dropdown");
    }
}

public class CountryDropdown
{
    private readonly FormMediator _mediator;

    public CountryDropdown(FormMediator mediator)
    {
        _mediator = mediator;
    }

    public void Change()
    {
        _mediator.Notify("Country", "Changed");
    }
}

var mediator = new FormMediator();
new CountryDropdown(mediator).Change();
```

---

# 18. Memento

## Intención

Capturar y restaurar el estado interno de un objeto sin exponer sus detalles.

## Problema

Necesitas undo, checkpoints o snapshots.

## Cuándo usarlo

Úsalo cuando:

```text
Necesitas deshacer cambios.
Quieres snapshots de edición.
Quieres checkpoints.
```

Evítalo cuando:

```text
El estado es enorme.
Copiar estado es costoso.
La restauración puede romper invariantes.
```

---

## Python: editor con snapshots

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class EditorState:
    text: str


class Editor:
    def __init__(self) -> None:
        self.text = ""

    def type(self, value: str) -> None:
        self.text += value

    def save(self) -> EditorState:
        return EditorState(self.text)

    def restore(self, state: EditorState) -> None:
        self.text = state.text


editor = Editor()
editor.type("hello")
snapshot = editor.save()
editor.type(" world")
editor.restore(snapshot)

print(editor.text)
```

---

## C: snapshot de configuración

```c
#include <stdio.h>

typedef struct {
    int volume;
    int brightness;
} Settings;

Settings save(Settings current) {
    return current;
}

void restore(Settings *current, Settings snapshot) {
    *current = snapshot;
}

int main(void) {
    Settings settings = {50, 80};
    Settings snapshot = save(settings);

    settings.volume = 10;
    restore(&settings, snapshot);

    printf("%d\n", settings.volume);
    return 0;
}
```

---

## C++: juego con checkpoint

```cpp
#include <iostream>
#include <string>

class GameState {
public:
    int level;
    int health;
};

class Game {
    int level = 1;
    int health = 100;

public:
    GameState save() const {
        return {level, health};
    }

    void restore(const GameState& state) {
        level = state.level;
        health = state.health;
    }

    void damage() {
        health -= 50;
    }

    void print() const {
        std::cout << "health=" << health << "\n";
    }
};

int main() {
    Game game;
    auto checkpoint = game.save();
    game.damage();
    game.restore(checkpoint);
    game.print();
}
```

---

## C#: documento con historial

```csharp
using System;
using System.Collections.Generic;

public record DocumentMemento(string Text);

public class Document
{
    public string Text { get; private set; } = "";

    public void Write(string value) => Text += value;

    public DocumentMemento Save() => new(Text);

    public void Restore(DocumentMemento memento) => Text = memento.Text;
}

var doc = new Document();
var history = new Stack<DocumentMemento>();

doc.Write("A");
history.Push(doc.Save());
doc.Write("B");

doc.Restore(history.Pop());
Console.WriteLine(doc.Text);
```

---

# 19. Observer

## Intención

Definir una dependencia uno-a-muchos para notificar automáticamente a objetos cuando otro cambia.

## Problema

Varios componentes necesitan reaccionar a un evento sin acoplarse directamente al emisor.

## Cuándo usarlo

Úsalo cuando:

```text
Hay eventos de dominio.
Hay suscriptores múltiples.
Quieres desacoplar emisor y receptores.
Necesitas notificaciones.
```

Evítalo cuando:

```text
El flujo debe ser estrictamente secuencial y explícito.
Las notificaciones ocultas dificultan depuración.
```

---

## Python: eventos de dominio

```python
from collections.abc import Callable


class EventBus:
    def __init__(self) -> None:
        self.subscribers: dict[str, list[Callable[[dict], None]]] = {}

    def subscribe(self, event: str, handler: Callable[[dict], None]) -> None:
        self.subscribers.setdefault(event, []).append(handler)

    def publish(self, event: str, payload: dict) -> None:
        for handler in self.subscribers.get(event, []):
            handler(payload)


def send_email(payload: dict) -> None:
    print(f"Email to {payload['email']}")


bus = EventBus()
bus.subscribe("user_registered", send_email)
bus.publish("user_registered", {"email": "user@example.com"})
```

---

## C: observadores de sensor

```c
#include <stdio.h>

typedef void (*Observer)(int);

void display_update(int value) {
    printf("Display: %d\n", value);
}

void alarm_update(int value) {
    if (value > 80) printf("Alarm!\n");
}

int main(void) {
    Observer observers[] = { display_update, alarm_update };
    int temperature = 90;

    for (int i = 0; i < 2; i++) {
        observers[i](temperature);
    }

    return 0;
}
```

---

## C++: stock price

```cpp
#include <iostream>
#include <vector>

class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(double price) = 0;
};

class PriceDisplay : public Observer {
public:
    void update(double price) override {
        std::cout << "Price: " << price << "\n";
    }
};

class Stock {
    std::vector<Observer*> observers;

public:
    void subscribe(Observer* observer) {
        observers.push_back(observer);
    }

    void set_price(double price) {
        for (auto* observer : observers) {
            observer->update(price);
        }
    }
};

int main() {
    Stock stock;
    PriceDisplay display;

    stock.subscribe(&display);
    stock.set_price(123.45);
}
```

---

## C#: eventos

```csharp
using System;

public class OrderService
{
    public event Action<string>? OrderCreated;

    public void Create(string orderId)
    {
        Console.WriteLine($"Order {orderId} created");
        OrderCreated?.Invoke(orderId);
    }
}

var service = new OrderService();
service.OrderCreated += id => Console.WriteLine($"Notify warehouse: {id}");
service.OrderCreated += id => Console.WriteLine($"Send email: {id}");

service.Create("order-1");
```

---

# 20. State

## Intención

Permitir que un objeto cambie su comportamiento cuando cambia su estado interno.

## Problema

Tienes muchos `if estado == ...` dispersos.

## Cuándo usarlo

Úsalo cuando:

```text
El comportamiento depende fuertemente del estado.
Hay transiciones entre estados.
Quieres encapsular reglas por estado.
```

Evítalo cuando:

```text
Solo hay dos estados simples.
Una máquina de estados declarativa sería más adecuada.
```

---

## Python: flujo de ticket

```python
from abc import ABC, abstractmethod


class TicketState(ABC):
    @abstractmethod
    def close(self, ticket: "Ticket") -> None:
        pass


class OpenState(TicketState):
    def close(self, ticket: "Ticket") -> None:
        ticket.state = ClosedState()
        print("Ticket closed")


class ClosedState(TicketState):
    def close(self, ticket: "Ticket") -> None:
        print("Ticket already closed")


class Ticket:
    def __init__(self) -> None:
        self.state: TicketState = OpenState()

    def close(self) -> None:
        self.state.close(self)


ticket = Ticket()
ticket.close()
ticket.close()
```

---

## C: estado de conexión

```c
#include <stdio.h>

typedef enum {
    DISCONNECTED,
    CONNECTED
} State;

typedef struct {
    State state;
} Connection;

void send_data(Connection *conn) {
    if (conn->state == DISCONNECTED) {
        printf("Cannot send: disconnected\n");
    } else {
        printf("Sending data\n");
    }
}

int main(void) {
    Connection conn = { DISCONNECTED };
    send_data(&conn);

    conn.state = CONNECTED;
    send_data(&conn);

    return 0;
}
```

---

## C++: reproductor

```cpp
#include <iostream>

class Player;

class State {
public:
    virtual ~State() = default;
    virtual void press(Player& player) = 0;
};

class PlayingState;
class PausedState;

class Player {
public:
    State* state;

    explicit Player(State* state) : state(state) {}

    void press() {
        state->press(*this);
    }
};

class PausedState : public State {
public:
    void press(Player& player) override;
};

class PlayingState : public State {
public:
    void press(Player& player) override;
};

PausedState paused;
PlayingState playing;

void PausedState::press(Player& player) {
    std::cout << "Play\n";
    player.state = &playing;
}

void PlayingState::press(Player& player) {
    std::cout << "Pause\n";
    player.state = &paused;
}

int main() {
    Player player(&paused);
    player.press();
    player.press();
}
```

---

## C#: pedido

```csharp
using System;

public interface IOrderState
{
    void Cancel(Order order);
}

public class PendingState : IOrderState
{
    public void Cancel(Order order)
    {
        order.State = new CancelledState();
        Console.WriteLine("Order cancelled");
    }
}

public class CancelledState : IOrderState
{
    public void Cancel(Order order)
    {
        Console.WriteLine("Already cancelled");
    }
}

public class Order
{
    public IOrderState State { get; set; } = new PendingState();

    public void Cancel() => State.Cancel(this);
}

var order = new Order();
order.Cancel();
order.Cancel();
```

---

# 21. Strategy

## Intención

Definir una familia de algoritmos, encapsularlos y hacerlos intercambiables.

## Problema

Tienes variantes de un algoritmo y no quieres llenar el código con condicionales.

## Cuándo usarlo

Úsalo cuando:

```text
Hay varias formas de calcular, ordenar, validar o procesar.
Quieres cambiar algoritmo en runtime.
Quieres testear estrategias por separado.
```

Evítalo cuando:

```text
Solo hay una variante.
Las estrategias son triviales y no se reutilizan.
```

---

## Python: estrategia de precios

```python
from collections.abc import Callable


def regular_price(amount: float) -> float:
    return amount


def vip_price(amount: float) -> float:
    return amount * 0.9


def calculate_total(amount: float, strategy: Callable[[float], float]) -> float:
    return strategy(amount)


print(calculate_total(100, vip_price))
```

Versión con clases:

```python
class PricingStrategy:
    def calculate(self, amount: float) -> float:
        raise NotImplementedError
```

En Python, pasar funciones suele ser suficiente.

---

## C: algoritmo de ordenamiento elegido

```c
#include <stdio.h>

typedef int (*Compare)(int, int);

int ascending(int a, int b) {
    return a > b;
}

int descending(int a, int b) {
    return a < b;
}

void sort_two(int *a, int *b, Compare compare) {
    if (compare(*a, *b)) {
        int temp = *a;
        *a = *b;
        *b = temp;
    }
}

int main(void) {
    int a = 1;
    int b = 5;

    sort_two(&a, &b, descending);
    printf("%d %d\n", a, b);
    return 0;
}
```

---

## C++: compresión

```cpp
#include <iostream>
#include <memory>
#include <string>

class CompressionStrategy {
public:
    virtual ~CompressionStrategy() = default;
    virtual void compress(const std::string& file) = 0;
};

class ZipCompression : public CompressionStrategy {
public:
    void compress(const std::string& file) override {
        std::cout << "ZIP " << file << "\n";
    }
};

class Compressor {
    std::unique_ptr<CompressionStrategy> strategy;

public:
    explicit Compressor(std::unique_ptr<CompressionStrategy> strategy)
        : strategy(std::move(strategy)) {}

    void run(const std::string& file) {
        strategy->compress(file);
    }
};

int main() {
    Compressor compressor(std::make_unique<ZipCompression>());
    compressor.run("data.csv");
}
```

---

## C#: validación intercambiable

```csharp
using System;

public interface IValidationStrategy
{
    bool IsValid(string value);
}

public class EmailValidation : IValidationStrategy
{
    public bool IsValid(string value) => value.Contains("@");
}

public class PhoneValidation : IValidationStrategy
{
    public bool IsValid(string value) => value.Length >= 8;
}

public class Validator
{
    private readonly IValidationStrategy _strategy;

    public Validator(IValidationStrategy strategy)
    {
        _strategy = strategy;
    }

    public bool Validate(string value) => _strategy.IsValid(value);
}

Console.WriteLine(new Validator(new EmailValidation()).Validate("a@b.com"));
```

---

# 22. Template Method

## Intención

Definir el esqueleto de un algoritmo en una clase base, permitiendo que subclases redefinan pasos específicos.

## Problema

Varios procesos comparten una secuencia, pero cambian algunos pasos.

## Cuándo usarlo

Úsalo cuando:

```text
Hay flujo fijo con pasos variables.
Quieres evitar duplicación.
Quieres controlar orden del algoritmo.
```

Evítalo cuando:

```text
La herencia genera rigidez.
La composición con Strategy sería más flexible.
```

---

## Python: pipeline ETL

```python
from abc import ABC, abstractmethod


class ETLPipeline(ABC):
    def run(self) -> None:
        data = self.extract()
        transformed = self.transform(data)
        self.load(transformed)

    @abstractmethod
    def extract(self) -> list[dict]:
        pass

    @abstractmethod
    def transform(self, data: list[dict]) -> list[dict]:
        pass

    @abstractmethod
    def load(self, data: list[dict]) -> None:
        pass


class CsvToPostgresPipeline(ETLPipeline):
    def extract(self) -> list[dict]:
        return [{"name": "Ana"}]

    def transform(self, data: list[dict]) -> list[dict]:
        return [{**row, "active": True} for row in data]

    def load(self, data: list[dict]) -> None:
        print(f"Loading {data}")


CsvToPostgresPipeline().run()
```

---

## C: flujo fijo con callbacks

```c
#include <stdio.h>

typedef struct {
    void (*extract)(void);
    void (*transform)(void);
    void (*load)(void);
} Pipeline;

void run_pipeline(Pipeline *p) {
    p->extract();
    p->transform();
    p->load();
}

void csv_extract(void) { printf("extract csv\n"); }
void clean_transform(void) { printf("clean data\n"); }
void db_load(void) { printf("load db\n"); }

int main(void) {
    Pipeline p = { csv_extract, clean_transform, db_load };
    run_pipeline(&p);
    return 0;
}
```

---

## C++: importador

```cpp
#include <iostream>

class Importer {
public:
    virtual ~Importer() = default;

    void run() {
        connect();
        read();
        save();
    }

protected:
    virtual void connect() = 0;
    virtual void read() = 0;
    virtual void save() = 0;
};

class CsvImporter : public Importer {
protected:
    void connect() override { std::cout << "open file\n"; }
    void read() override { std::cout << "read csv\n"; }
    void save() override { std::cout << "save rows\n"; }
};

int main() {
    CsvImporter importer;
    importer.run();
}
```

---

## C#: procesamiento de pagos

```csharp
using System;

public abstract class PaymentProcess
{
    public void Execute()
    {
        Validate();
        Charge();
        Notify();
    }

    protected abstract void Validate();
    protected abstract void Charge();

    protected virtual void Notify()
    {
        Console.WriteLine("Default notification");
    }
}

public class CardPaymentProcess : PaymentProcess
{
    protected override void Validate() => Console.WriteLine("Validate card");
    protected override void Charge() => Console.WriteLine("Charge card");
}

new CardPaymentProcess().Execute();
```

---

# 23. Visitor

## Intención

Separar operaciones de los objetos sobre los que operan.

## Problema

Tienes una estructura de objetos estable, pero necesitas agregar operaciones nuevas sin modificar cada clase constantemente.

## Cuándo usarlo

Úsalo cuando:

```text
La estructura de clases es estable.
Agregas operaciones frecuentemente.
Necesitas recorrer árboles de objetos.
Quieres separar lógica transversal.
```

Evítalo cuando:

```text
Agregas nuevos tipos de elementos frecuentemente.
La doble dispatch complica demasiado.
En Python, una función con singledispatch puede ser suficiente.
```

---

## Python: exportar nodos de documento

```python
from functools import singledispatch
from dataclasses import dataclass


@dataclass
class Heading:
    text: str


@dataclass
class Paragraph:
    text: str


@singledispatch
def render(node) -> str:
    raise TypeError(f"Unsupported node: {type(node)}")


@render.register
def _(node: Heading) -> str:
    return f"# {node.text}"


@render.register
def _(node: Paragraph) -> str:
    return node.text


nodes = [Heading("Title"), Paragraph("Content")]
print("\n".join(render(node) for node in nodes))
```

Python puede implementar Visitor con `accept`, pero `singledispatch` suele ser más idiomático para este caso.

---

## C: visitor con tagged union

```c
#include <stdio.h>

typedef enum {
    NODE_NUMBER,
    NODE_TEXT
} NodeType;

typedef struct {
    NodeType type;
    union {
        int number;
        const char *text;
    } data;
} Node;

void visit_print(Node *node) {
    switch (node->type) {
        case NODE_NUMBER:
            printf("Number: %d\n", node->data.number);
            break;
        case NODE_TEXT:
            printf("Text: %s\n", node->data.text);
            break;
    }
}

int main(void) {
    Node n = { NODE_TEXT, .data.text = "hello" };
    visit_print(&n);
    return 0;
}
```

---

## C++: visitor de shapes

```cpp
#include <iostream>

class Circle;
class Rectangle;

class Visitor {
public:
    virtual ~Visitor() = default;
    virtual void visit(Circle& circle) = 0;
    virtual void visit(Rectangle& rectangle) = 0;
};

class Shape {
public:
    virtual ~Shape() = default;
    virtual void accept(Visitor& visitor) = 0;
};

class Circle : public Shape {
public:
    void accept(Visitor& visitor) override {
        visitor.visit(*this);
    }
};

class Rectangle : public Shape {
public:
    void accept(Visitor& visitor) override {
        visitor.visit(*this);
    }
};

class AreaVisitor : public Visitor {
public:
    void visit(Circle&) override {
        std::cout << "Area circle\n";
    }

    void visit(Rectangle&) override {
        std::cout << "Area rectangle\n";
    }
};

int main() {
    Circle circle;
    AreaVisitor visitor;
    circle.accept(visitor);
}
```

---

## C#: visitor de AST

```csharp
using System;

public interface IExpressionVisitor
{
    void Visit(NumberExpression number);
    void Visit(AddExpression add);
}

public interface IExpression
{
    void Accept(IExpressionVisitor visitor);
}

public class NumberExpression : IExpression
{
    public int Value { get; }

    public NumberExpression(int value)
    {
        Value = value;
    }

    public void Accept(IExpressionVisitor visitor) => visitor.Visit(this);
}

public class AddExpression : IExpression
{
    public IExpression Left { get; }
    public IExpression Right { get; }

    public AddExpression(IExpression left, IExpression right)
    {
        Left = left;
        Right = right;
    }

    public void Accept(IExpressionVisitor visitor) => visitor.Visit(this);
}

public class PrintVisitor : IExpressionVisitor
{
    public void Visit(NumberExpression number) =>
        Console.Write(number.Value);

    public void Visit(AddExpression add)
    {
        Console.Write("(");
        add.Left.Accept(this);
        Console.Write(" + ");
        add.Right.Accept(this);
        Console.Write(")");
    }
}

var expr = new AddExpression(new NumberExpression(1), new NumberExpression(2));
expr.Accept(new PrintVisitor());
```

---

# Parte V: cómo aplicar patrones en proyectos reales

## 1. No empieces por el patrón

No diseñes así:

```text
Necesito usar Strategy.
Necesito usar Abstract Factory.
Necesito usar Visitor.
```

Diseña así:

```text
¿Qué problema de cambio, acoplamiento o complejidad tengo?
¿Qué parte del sistema varía?
¿Qué parte debe permanecer estable?
¿Qué costo tendrá agregar esta abstracción?
```

---

## 2. Preguntas para elegir un patrón

```text
¿El problema es de creación de objetos?
Revisa Factory Method, Abstract Factory, Builder, Prototype, Singleton.

¿El problema es de estructura o compatibilidad?
Revisa Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.

¿El problema es de comportamiento o coordinación?
Revisa Chain, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor.
```

---

## 3. Patrones frecuentes en Python real

Muy frecuentes:

```text
Factory Method:
Crear clientes, conectores, parsers.

Builder:
Configuraciones complejas, requests, pipelines.

Adapter:
Integración con APIs externas.

Facade:
Servicios de aplicación.

Decorator:
Middlewares, logging, caching, autorización.

Proxy:
Cache, autorización, lazy loading.

Strategy:
Algoritmos intercambiables.

Observer:
Eventos de dominio.

Command:
Jobs, colas, acciones.

Template Method:
Pipelines.
```

Menos frecuentes o reemplazados por idioms Python:

```text
Singleton:
Muchas veces se reemplaza por módulos o inyección de dependencias.

Visitor:
Puede reemplazarse por singledispatch, pattern matching o funciones.

Iterator:
Está integrado en el protocolo iterable de Python.

Abstract Factory:
Útil, pero a menudo basta una función factory.
```

---

## 4. Señales de sobreingeniería

```text
Hay más clases que comportamiento.
Nadie entiende el flujo.
El patrón existe “por si acaso”.
No hay variación real.
Los tests son más difíciles.
El debugging requiere saltar por muchas capas.
```

Regla:

```text
Un patrón debe hacer el cambio más barato.
Si lo hace más caro, probablemente está mal aplicado.
```

---

## 5. Checklist para aplicar un patrón

```text
[ ] El problema está identificado.
[ ] Hay variación real o esperada.
[ ] El patrón reduce acoplamiento.
[ ] El patrón no oculta reglas críticas.
[ ] El equipo entiende la solución.
[ ] Hay tests.
[ ] El nombre del patrón aparece en documentación si ayuda.
[ ] No se usa solo por moda.
```

---

# Parte VI: resumen rápido de los 23 patrones

| Patrón | Familia | Úsalo para |
|---|---|---|
| Factory Method | Creacional | Crear objetos concretos sin acoplar cliente |
| Abstract Factory | Creacional | Crear familias de objetos compatibles |
| Builder | Creacional | Construir objetos complejos paso a paso |
| Prototype | Creacional | Clonar objetos base preconfigurados |
| Singleton | Creacional | Garantizar instancia única, con cuidado |
| Adapter | Estructural | Compatibilizar interfaces |
| Bridge | Estructural | Separar abstracción e implementación |
| Composite | Estructural | Árboles de objetos uniformes |
| Decorator | Estructural | Agregar comportamiento dinámicamente |
| Facade | Estructural | Simplificar subsistemas complejos |
| Flyweight | Estructural | Compartir estado para ahorrar memoria |
| Proxy | Estructural | Controlar acceso a otro objeto |
| Chain of Responsibility | Comportamiento | Pasar requests por una cadena |
| Command | Comportamiento | Encapsular acciones como objetos |
| Interpreter | Comportamiento | Evaluar lenguajes o reglas simples |
| Iterator | Comportamiento | Recorrer colecciones sin exponer estructura |
| Mediator | Comportamiento | Coordinar objetos desacoplados |
| Memento | Comportamiento | Guardar/restaurar estado |
| Observer | Comportamiento | Notificar cambios a suscriptores |
| State | Comportamiento | Cambiar comportamiento según estado |
| Strategy | Comportamiento | Intercambiar algoritmos |
| Template Method | Comportamiento | Fijar flujo y variar pasos |
| Visitor | Comportamiento | Agregar operaciones sobre estructura estable |

---

# Parte VII: cierre

Los patrones de diseño son herramientas de diseño, no metas. En Python, muchos patrones pueden implementarse con menos ceremonia que en lenguajes más rígidos. En C, suelen expresarse con structs y punteros a función. En C++, aparecen naturalmente con RAII, polimorfismo, templates y smart pointers. En C#, suelen expresarse con interfaces, delegates, eventos, genéricos e inyección de dependencias.

Regla final:

```text
Primero diseña para claridad.
Luego para cambio.
Después para reutilización.
Nunca al revés.
```
