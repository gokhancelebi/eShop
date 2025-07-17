# eShop API Yapısı - Kapsamlı Özet

## 🏗️ **Proje Genel Yapısı**

Bu proje Microsoft tarafından geliştirilmiş bir **referans e-ticaret uygulamasıdır** ve **.NET 9** ile **mikroservis mimarisi** kullanır.

### 📊 **Ana API Servisleri**
- **Basket.API** - Sepet işlemleri (gRPC)
- **Catalog.API** - Ürün kataloğu (REST + AI)
- **Identity.API** - Kimlik doğrulama (OpenID Connect)
- **Ordering.API** - Sipariş yönetimi (CQRS + MediatR)
- **Webhooks.API** - Webhook yönetimi

---

## 🎯 **Minimal API Sistemi**

### ✅ **Minimal API Özellikleri**
- **Controller sınıfları YOK** - Direkt static metodlar
- **MapGet/MapPost/MapPut/MapDelete** kullanımı
- **TypedResults** return tipleri
- **Endpoint handler'lar** static metodlar
- **Daha az boilerplate kod**

### 📋 **Geleneksel vs Minimal API**
```csharp
// ❌ Geleneksel Controller-based API
[ApiController]
[Route("api/[controller]")]
public class CatalogController : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<List<CatalogItem>>> GetItems()
    {
        return Ok(items);
    }
}

// ✅ Minimal API
public static class CatalogApi
{
    public static IEndpointRouteBuilder MapCatalogApi(this IEndpointRouteBuilder app)
    {
        api.MapGet("/items", GetAllItems);
        return app;
    }
    
    public static async Task<Ok<PaginatedItems<CatalogItem>>> GetAllItems(...)
    {
        return TypedResults.Ok(result);
    }
}
```

---

## 🔌 **Extension Method Çalışma Mantığı**

### 🎯 **Extension Method Nasıl Çalışır?**

```csharp
// 1. Extension method tanımı
public static IEndpointRouteBuilder MapCatalogApi(this IEndpointRouteBuilder app)
//                                                ^^^^                     ^^^
//                                                Bu   IEndpointRouteBuilder tipi
//                                                keyword      üzerinden
//                                                extend       çağrılabilir
//                                                eder         yapar
{
    // endpoint tanımları...
    return app;
}

// 2. WebApplication IEndpointRouteBuilder'ı implement eder
public sealed class WebApplication : IEndpointRouteBuilder // Framework kodu
{
    // ...
}

// 3. Program.cs'de kullanım
var app = builder.Build();  // app = WebApplication instance
app.MapCatalogApi();       // Extension method çağrısı

// 4. Derleyici dönüşümü
CatalogApi.MapCatalogApi(app);  // Static method çağrısı
```

### 🔄 **Type Uyumluluğu**
- **WebApplication** → `IEndpointRouteBuilder` interface'ini implement eder
- **Extension method** → `IEndpointRouteBuilder` tipini extend eder
- **Derleyici** → Otomatik dönüşüm yapar

---

## 🌐 **Global Using Sistemi**

### 📍 **Namespace Import Mekanizması**

```csharp
// GlobalUsings.cs
global using eShop.Catalog.API;  // 👈 NAMESPACE import
//           ^^^^^^^^^^^^^^^^^^
//           Namespace adı (sınıf adı değil!)

// CatalogApi.cs
namespace eShop.Catalog.API;      // 👈 AYNI NAMESPACE
public static class CatalogApi    // 👈 SINIF ADI
{
    public static IEndpointRouteBuilder MapCatalogApi(this IEndpointRouteBuilder app)
    {
        // ...
    }
}

// Program.cs (görünür using YOK!)
var app = builder.Build();
app.MapCatalogApi();  // ✅ Çalışır! Namespace import edildi
```

### 💡 **Neden Using Satırı Görünmez?**
- `global using` = Tüm dosyalara otomatik import
- **Derleyici** = Otomatik olarak namespace'i ekler
- **Sınıf adı** = Namespace'den otomatik bulunur

---

## 🛣️ **Endpoint Mapping ve Routing**

### 📋 **Catalog API Endpoint'leri**

```csharp
// CatalogApi.cs - MapCatalogApi() metodu
public static IEndpointRouteBuilder MapCatalogApi(this IEndpointRouteBuilder app)
{
    // API versiyonlaması
    var vApi = app.NewVersionedApi("Catalog");
    var api = vApi.MapGroup("api/catalog").HasApiVersion(1, 0).HasApiVersion(2, 0);
    var v1 = vApi.MapGroup("api/catalog").HasApiVersion(1, 0);
    var v2 = vApi.MapGroup("api/catalog").HasApiVersion(2, 0);

    // Endpoint tanımları
    v1.MapGet("/items", GetAllItemsV1)
        .WithName("ListItems")
        .WithSummary("List catalog items")
        .WithDescription("Get a paginated list of items in the catalog.")
        .WithTags("Items");
    
    v2.MapGet("/items", GetAllItems)
        .WithName("ListItems-V2")
        .WithTags("Items");
    
    api.MapGet("/items/{id:int}", GetItemById)
        .WithName("GetItem")
        .WithTags("Items");
    
    api.MapPost("/items", CreateItem)
        .WithName("CreateItem");
    
    api.MapPut("/items/{id:int}", UpdateItem)
        .WithName("UpdateItem-V2");
    
    api.MapDelete("/items/{id:int}", DeleteItemById)
        .WithName("DeleteItem");
    
    return app;
}
```

### 🎯 **Routing Yapısı**

| HTTP Metodu | URL | Handler Metodu | API Versiyonu |
|-------------|-----|----------------|---------------|
| `GET` | `/api/catalog/items` | `GetAllItemsV1` | v1.0 |
| `GET` | `/api/catalog/items` | `GetAllItems` | v2.0 |
| `GET` | `/api/catalog/items/{id}` | `GetItemById` | v1.0, v2.0 |
| `GET` | `/api/catalog/items/{id}/pic` | `GetItemPictureById` | v1.0, v2.0 |
| `POST` | `/api/catalog/items` | `CreateItem` | v1.0, v2.0 |
| `PUT` | `/api/catalog/items/{id}` | `UpdateItem` | v2.0 |
| `DELETE` | `/api/catalog/items/{id}` | `DeleteItemById` | v1.0, v2.0 |

### 🏷️ **OpenAPI Metadata**
```csharp
api.MapGet("/items/{id:int}", GetItemById)
    .WithName("GetItem")                    // Endpoint adı
    .WithSummary("Get catalog item")        // Kısa açıklama
    .WithDescription("Get an item from the catalog")  // Detaylı açıklama
    .WithTags("Items");                     // Swagger gruplandırma
```

---

## 🗄️ **Database Connection ve Configuration**

### 🔗 **Database Bağlantısı**

```csharp
// Extensions.cs - AddApplicationServices()
public static void AddApplicationServices(this IHostApplicationBuilder builder)
{
    // PostgreSQL + pgvector bağlantısı
    builder.AddNpgsqlDbContext<CatalogContext>("catalogdb", configureDbContextOptions: dbContextOptionsBuilder =>
    {
        dbContextOptionsBuilder.UseNpgsql(builder =>
        {
            builder.UseVector();  // AI embedding için pgvector
        });
    });

    // Database migration ve seed
    builder.Services.AddMigration<CatalogContext, CatalogContextSeed>();
}
```

### 📊 **Database Context**

```csharp
// CatalogContext.cs
public class CatalogContext : DbContext
{
    public DbSet<CatalogItem> CatalogItems { get; set; }
    public DbSet<CatalogBrand> CatalogBrands { get; set; }
    public DbSet<CatalogType> CatalogTypes { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.HasPostgresExtension("vector");  // pgvector extension
        builder.ApplyConfiguration(new CatalogBrandEntityTypeConfiguration());
        builder.ApplyConfiguration(new CatalogTypeEntityTypeConfiguration());
        builder.ApplyConfiguration(new CatalogItemEntityTypeConfiguration());
        builder.UseIntegrationEventLogs();  // Outbox pattern
    }
}
```

### ⚙️ **Configuration Dosyaları**

```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "CatalogDB": "Host=localhost;Database=CatalogDB;Username=postgres;Password=yourWeak(!)Password"
  }
}

// appsettings.json
{
  "ConnectionStrings": {
    "EventBus": "amqp://localhost"
  },
  "EventBus": {
    "SubscriptionClientName": "Catalog"
  },
  "CatalogOptions": {
    "UseCustomizationData": false
  }
}
```

---

## 🔧 **Dependency Injection**

### 📦 **Service Registration**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.AddServiceDefaults();        // Temel servisler
builder.AddApplicationServices();    // Uygulama servisleri
builder.Services.AddProblemDetails(); // Problem details support
builder.Services.AddApiVersioning();  // API versiyonlama

// Extensions.cs - AddApplicationServices()
public static void AddApplicationServices(this IHostApplicationBuilder builder)
{
    // Database services
    builder.AddNpgsqlDbContext<CatalogContext>("catalogdb");
    builder.Services.AddMigration<CatalogContext, CatalogContextSeed>();

    // Integration services
    builder.Services.AddTransient<IIntegrationEventLogService, IntegrationEventLogService<CatalogContext>>();
    builder.Services.AddTransient<ICatalogIntegrationEventService, CatalogIntegrationEventService>();

    // Event bus
    builder.AddRabbitMqEventBus("eventbus")
           .AddSubscription<OrderStatusChangedToAwaitingValidationIntegrationEvent, OrderStatusChangedToAwaitingValidationIntegrationEventHandler>()
           .AddSubscription<OrderStatusChangedToPaidIntegrationEvent, OrderStatusChangedToPaidIntegrationEventHandler>();

    // Configuration
    builder.Services.AddOptions<CatalogOptions>()
        .BindConfiguration(nameof(CatalogOptions));

    // AI services
    if (builder.Configuration["OllamaEnabled"] is string ollamaEnabled && bool.Parse(ollamaEnabled))
    {
        builder.AddOllamaApiClient("embedding").AddEmbeddingGenerator();
    }
    else if (!string.IsNullOrWhiteSpace(builder.Configuration.GetConnectionString("textEmbeddingModel")))
    {
        builder.AddOpenAIClientFromConfiguration("textEmbeddingModel").AddEmbeddingGenerator();
    }

    builder.Services.AddScoped<ICatalogAI, CatalogAI>();
}
```

### 🎯 **Service Kullanımı**

```csharp
// CatalogServices.cs - Endpoint handler'lara inject edilen servisler
public class CatalogServices(
    CatalogContext context,
    [FromServices] ICatalogAI catalogAI,
    IOptions<CatalogOptions> options,
    ILogger<CatalogServices> logger,
    [FromServices] ICatalogIntegrationEventService eventService)
{
    public CatalogContext Context { get; } = context;
    public ICatalogAI CatalogAI { get; } = catalogAI;
    public IOptions<CatalogOptions> Options { get; } = options;
    public ILogger<CatalogServices> Logger { get; } = logger;
    public ICatalogIntegrationEventService EventService { get; } = eventService;
}

// Endpoint handler'da kullanım
public static async Task<Ok<PaginatedItems<CatalogItem>>> GetAllItems(
    [AsParameters] PaginationRequest paginationRequest,
    [AsParameters] CatalogServices services)  // 👈 Dependency injection
{
    var root = (IQueryable<CatalogItem>)services.Context.CatalogItems;
    // ...
}
```

---

## 🔢 **API Versioning**

### 📋 **Versioning Yapısı**

```csharp
// CatalogApi.cs
public static IEndpointRouteBuilder MapCatalogApi(this IEndpointRouteBuilder app)
{
    var vApi = app.NewVersionedApi("Catalog");
    var api = vApi.MapGroup("api/catalog").HasApiVersion(1, 0).HasApiVersion(2, 0);
    var v1 = vApi.MapGroup("api/catalog").HasApiVersion(1, 0);
    var v2 = vApi.MapGroup("api/catalog").HasApiVersion(2, 0);

    // v1 endpoint
    v1.MapGet("/items", GetAllItemsV1)
        .WithName("ListItems");
    
    // v2 endpoint (gelişmiş filtreleme)
    v2.MapGet("/items", GetAllItems)
        .WithName("ListItems-V2");
}
```

### 🎯 **API Version Kullanımı**

```bash
# Version 1.0
GET /api/catalog/items?api-version=1.0

# Version 2.0 (filtreleme desteği)
GET /api/catalog/items?api-version=2.0&name=shirt&type=1&brand=2
```

---

## 📖 **OpenAPI/Swagger Integration**

### ⚙️ **OpenAPI Configuration**

```csharp
// Program.cs
builder.AddDefaultOpenApi(withApiVersioning);  // OpenAPI setup

var app = builder.Build();
app.UseDefaultOpenApi();  // OpenAPI middleware
```

### 📄 **OpenAPI Metadata**

```json
// appsettings.json
{
  "OpenApi": {
    "Endpoint": {
      "Name": "Catalog.API V1"
    },
    "Document": {
      "Description": "The Catalog Microservice HTTP API. This is a Data-Driven/CRUD microservice sample",
      "Title": "eShop - Catalog HTTP API",
      "Version": "v1"
    }
  }
}
```

### 🏷️ **Build-time OpenAPI Generation**

```xml
<!-- Catalog.API.csproj -->
<PackageReference Include="Microsoft.Extensions.ApiDescription.Server">
  <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  <PrivateAssets>all</PrivateAssets>
</PackageReference>

<PropertyGroup>
  <OpenApiDocumentsDirectory>$(MSBuildProjectDirectory)</OpenApiDocumentsDirectory>
</PropertyGroup>
```

---

## 📁 **Proje Dosya Yapısı**

```
src/Catalog.API/
├── Program.cs                 # 🚀 Ana başlangıç noktası
├── GlobalUsings.cs           # 🌐 Global namespace imports
├── Apis/
│   └── CatalogApi.cs         # 🔌 Endpoint tanımları (417 satır)
├── Extensions/
│   └── Extensions.cs         # ⚙️ Service registration
├── Infrastructure/
│   ├── CatalogContext.cs     # 🗄️ Database context
│   ├── CatalogContextSeed.cs # 🌱 Database seed data
│   └── EntityConfigurations/ # 📊 Entity configurations
├── Model/
│   ├── CatalogItem.cs        # 📦 Ürün modeli
│   ├── CatalogBrand.cs       # 🏷️ Marka modeli
│   ├── CatalogType.cs        # 📋 Kategori modeli
│   └── CatalogServices.cs    # 🔧 Service container
├── Services/
│   ├── CatalogAI.cs          # 🤖 AI servisleri
│   └── CatalogIntegrationEventService.cs  # 📨 Event servisleri
├── IntegrationEvents/        # 📬 Event handlers
├── appsettings.json          # ⚙️ Configuration
└── appsettings.Development.json  # 🔧 Dev configuration
```

---

## 🎯 **Diğer API Servisleri**

### 🛒 **Basket API (gRPC)**
```csharp
// Program.cs
builder.Services.AddGrpc();
app.MapGrpcService<BasketService>();

// BasketService.cs
public class BasketService : Basket.BasketBase
{
    public override async Task<CustomerBasketResponse> GetBasket(GetBasketRequest request, ServerCallContext context)
    {
        // gRPC implementation
    }
}
```

### 📋 **Ordering API (CQRS)**
```csharp
// OrdersApi.cs
public static RouteGroupBuilder MapOrdersApiV1(this IEndpointRouteBuilder app)
{
    var api = app.MapGroup("api/orders").HasApiVersion(1.0);
    
    api.MapPost("/", CreateOrderAsync);
    api.MapGet("/{orderId:int}", GetOrderAsync);
    api.MapPut("/cancel", CancelOrderAsync);
    api.MapPut("/ship", ShipOrderAsync);
    
    return api;
}
```

---

## 📚 **Önemli Bilgiler**

### 🔑 **Key Concepts**

1. **Minimal API** - Controller'sız API geliştirme
2. **Extension Methods** - `this` keyword ile tip genişletme
3. **Global Using** - Namespace'leri global import etme
4. **Dependency Injection** - Constructor injection ve `[AsParameters]`
5. **API Versioning** - Backward compatibility için versiyonlama
6. **OpenAPI** - Otomatik dokümantasyon
7. **PostgreSQL + pgvector** - AI embedding için vector database
8. **RabbitMQ** - Event-driven communication
9. **Outbox Pattern** - Reliable messaging
10. **Microservices** - Distributed system architecture

### 🚀 **Best Practices**

1. **Static Methods** - Endpoint handler'lar static olmalı
2. **TypedResults** - Tip güvenli return values
3. **AsParameters** - Multiple parameter binding
4. **Metadata** - WithName, WithSummary, WithDescription
5. **Versioning** - API evolution için versiyonlama
6. **Configuration** - appsettings.json hierarchy
7. **Logging** - Structured logging
8. **Health Checks** - Service health monitoring
9. **Security** - Authentication/Authorization
10. **Testing** - Functional ve unit tests

### 🔧 **Development Commands**

```bash
# Migration oluşturma
dotnet ef migrations add --context CatalogContext [migration-name]

# Projeyi çalıştırma
dotnet run --project src/Catalog.API/Catalog.API.csproj

# OpenAPI dokümantasyonu
GET /openapi/v1.json
GET /swagger
```

### 📈 **Performance Considerations**

- **Pagination** - Büyük data setler için sayfalama
- **Caching** - Redis ile response caching
- **Database Indexing** - Query optimization
- **Connection Pooling** - Database connection management
- **Async/Await** - Non-blocking operations
- **Vector Search** - AI embedding performansı

---

## 🎭 **Sonuç**

Bu proje modern .NET geliştirme pratiklerini gösteriyor:
- **Minimal API** ile temiz kod
- **Extension Methods** ile genişletilebilirlik
- **Dependency Injection** ile loose coupling
- **Microservices** ile scalability
- **Event-driven** architecture ile reliability
- **AI integration** ile modern features

**Bu yapı, enterprise-level e-ticaret uygulamaları için excellent bir referans!** 🚀 