# Technical Architecture - MeetLines Backend

## Overview

MeetLines Backend is an **ASP.NET Core** application implementing **Clean Architecture** (Hexagonal Architecture) to provide a scalable and maintainable RESTful API for the meeting and appointment management system.

## Architecture Principles

### Clean Architecture (Hexagonal Architecture)

The architecture is divided into independent layers that follow the Dependency Inversion Principle:

```
┌─────────────────────────────────────────────┐
│  MeetLines.API (Controllers & Endpoints)    │ ← HTTP Entry Point
└──────────────┬──────────────────────────────┘
               │ (depends on)
┌──────────────▼──────────────────────────────┐
│  MeetLines.Application (UseCases & DTOs)    │ ← Business Logic Orchestration
└──────────────┬──────────────────────────────┘
               │ (depends on)
┌──────────────▼──────────────────────────────┐
│  MeetLines.Domain (Entities & Rules)        │ ← Pure Domain Logic
└──────────────────────────────────────────────┘
       ▲              │              ▲
       │ (implements) │ (depends on) │ (implements)
       │              ▼              │
    ┌──────────────────────────────────────┐
    │  MeetLines.Infrastructure            │ ← Concrete Implementations
    │  (Repos, DB, Services, Config)       │
    └──────────────────────────────────────┘
       ▲
       │ (uses)
       │
┌──────────────────────────────────────────┐
│  SharedKernel (Common Utilities)         │
└──────────────────────────────────────────┘
```

## Application Layers

### 1. **API Layer** (`MeetLines.API`)

**Responsibilities:**
- Expose HTTP endpoints
- Validate requests
- Map DTOs to domain entities
- Handle authentication/authorization
- Return appropriate HTTP responses

**Main Components:**
```
MeetLines.API/
├── Controllers/           # REST endpoints
├── Filters/              # Global filters (exceptions, validation)
├── Middleware/           # HTTP pipeline (auth, CORS, logging)
├── GeoIp/               # Geolocation services
├── Jobs/                # Scheduled tasks
├── DTOs/                # Data Transfer Objects
├── appsettings.json     # Configuration
└── Program.cs           # Entry point
```

**Example controller flow:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class AppointmentsController : ControllerBase
{
    private readonly ICreateAppointmentUseCase _createUseCase;
    
    public AppointmentsController(ICreateAppointmentUseCase createUseCase)
    {
        _createUseCase = createUseCase;
    }
    
    [HttpPost]
    public async Task<IActionResult> Create(CreateAppointmentDto dto)
    {
        var result = await _createUseCase.Execute(dto);
        return Ok(result);
    }
}
```

### 2. **Application Layer** (`MeetLines.Application`)

**Responsibilities:**
- Define use cases
- Orchestrate business logic
- Define repository interfaces
- Map between DTOs and domain entities
- Handle transactions

**Main Components:**
```
MeetLines.Application/
├── UseCases/            # Use cases (create, update, delete)
├── Services/            # Application services
├── DTOs/                # Data transfer models
├── Validators/          # Input validators
├── Common/              # Shared utilities
└── Interfaces/          # Repository contracts
```

**Use case structure example:**
```csharp
public class CreateAppointmentUseCase : ICreateAppointmentUseCase
{
    private readonly IAppointmentRepository _repository;
    private readonly INotificationService _notificationService;
    
    public async Task<AppointmentDto> Execute(CreateAppointmentRequest request)
    {
        // 1. Validate input
        var validator = new CreateAppointmentValidator();
        var validation = await validator.ValidateAsync(request);
        
        if (!validation.IsValid)
            throw new ValidationException(validation.Errors);
        
        // 2. Create domain entity
        var appointment = Appointment.Create(
            request.ServiceId,
            request.CustomerPhone,
            request.ScheduledTime
        );
        
        // 3. Persist
        await _repository.Add(appointment);
        await _repository.UnitOfWork.SaveChangesAsync();
        
        // 4. Notify
        await _notificationService.SendConfirmation(appointment);
        
        // 5. Return DTO
        return new AppointmentDto { /* mapping */ };
    }
}
```

### 3. **Domain Layer** (`MeetLines.Domain`)

**Responsibilities:**
- Define domain entities
- Implement business rules
- Define specifications and aggregates
- Define domain events
- **NO** external dependencies

**Main Components:**
```
MeetLines.Domain/
├── Entities/            # Aggregate roots (Project, Appointment, Service)
├── ValueObjects/        # Value objects (Money, Schedule, etc.)
├── Aggregates/          # Aggregates and relationships
├── Enums/               # Domain enumerations
├── Repositories/        # Repository interfaces
├── Events/              # Domain events
└── Specifications/      # Query specifications
```

**Domain entity example:**
```csharp
public class Appointment : AggregateRoot
{
    public Guid Id { get; private set; }
    public Guid ProjectId { get; private set; }
    public string CustomerPhone { get; private set; }
    public DateTime ScheduledTime { get; private set; }
    public AppointmentStatus Status { get; private set; }
    
    public static Appointment Create(
        Guid projectId,
        string customerPhone,
        DateTime scheduledTime)
    {
        // Domain validations
        if (string.IsNullOrWhiteSpace(customerPhone))
            throw new DomainException("Phone cannot be empty");
        
        if (scheduledTime < DateTime.UtcNow)
            throw new DomainException("Cannot schedule in the past");
        
        var appointment = new Appointment
        {
            Id = Guid.NewGuid(),
            ProjectId = projectId,
            CustomerPhone = customerPhone,
            ScheduledTime = scheduledTime,
            Status = AppointmentStatus.Pending
        };
        
        // Publish domain event
        appointment.AddDomainEvent(new AppointmentCreatedDomainEvent(appointment));
        
        return appointment;
    }
}
```

### 4. **Infrastructure Layer** (`MeetLines.Infrastructure`)

**Responsibilities:**
- Implement repositories (data access)
- Integrate with database (Entity Framework)
- Implement external services (Email, SMS, WhatsApp)
- Configure dependency injection
- Handle caching, logging, etc.

**Main Components:**
```
MeetLines.Infrastructure/
├── Data/                # DbContext, migrations, seeds
├── Repositories/        # Repository implementations
├── Services/            # External services
│   ├── NotificationService  # Email, SMS
│   ├── WhatsappService      # WhatsApp integration
│   ├── PaymentService       # Payment processing
│   └── AuthService          # OAuth, JWT
├── IoC/                 # Dependency injection configuration
└── Configuration/       # Entity Framework configuration
```

**Repository example:**
```csharp
public class AppointmentRepository : IAppointmentRepository
{
    private readonly MeetLinesDbContext _context;
    
    public async Task<Appointment> GetByIdAsync(Guid id)
    {
        return await _context.Appointments
            .Include(a => a.Service)
            .Include(a => a.Customer)
            .FirstOrDefaultAsync(a => a.Id == id);
    }
    
    public async Task<IEnumerable<Appointment>> GetByProjectAsync(Guid projectId)
    {
        return await _context.Appointments
            .Where(a => a.ProjectId == projectId)
            .ToListAsync();
    }
    
    public async Task Add(Appointment appointment)
    {
        await _context.Appointments.AddAsync(appointment);
    }
    
    public void Update(Appointment appointment)
    {
        _context.Appointments.Update(appointment);
    }
}
```

### 5. **Shared Kernel** (`SharedKernel`)

**Responsibilities:**
- Reusable common utilities
- Base classes for entities and aggregates
- Event handling
- Extensions

```csharp
public abstract class AggregateRoot : Entity
{
    private List<DomainEvent> _domainEvents = new();
    
    public IReadOnlyList<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();
    
    public void AddDomainEvent(DomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }
    
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

## Key Patterns

### Unit of Work Pattern

Coordinates database operations:

```csharp
public interface IUnitOfWork
{
    IAppointmentRepository Appointments { get; }
    IServiceRepository Services { get; }
    IProjectRepository Projects { get; }
    
    Task SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}
```

### Repository Pattern

Abstracts data access:

```csharp
public interface IAppointmentRepository : IRepository<Appointment>
{
    Task<Appointment> GetByIdAsync(Guid id);
    Task<IEnumerable<Appointment>> GetByProjectAsync(Guid projectId);
    Task<IEnumerable<Appointment>> GetAvailableSlotsAsync(Guid serviceId);
}
```

### Dependency Injection

Configuration in `Program.cs`:

```csharp
builder.Services
    .AddApplicationServices()        // UseCases, Validators
    .AddInfrastructureServices()     // Repositories, Services
    .AddDomainServices();            // Domain logic services
```

### CQRS (Command Query Responsibility Segregation)

Separation of read and write operations:

```csharp
// Command (Write)
public class CreateAppointmentCommand : IRequest<AppointmentDto>
{
    public Guid ProjectId { get; set; }
    public string CustomerPhone { get; set; }
    public DateTime ScheduledTime { get; set; }
}

// Query (Read)
public class GetAppointmentsQuery : IRequest<IEnumerable<AppointmentDto>>
{
    public Guid ProjectId { get; set; }
}
```

## Key Technologies

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Framework | ASP.NET Core 8 | Web API |
| ORM | Entity Framework Core | Data access |
| Database | PostgreSQL | Persistent storage |
| Validation | FluentValidation | Model validation |
| Mapping | AutoMapper | DTO ↔ Entity mapping |
| Authentication | JWT + OAuth2 | Security |
| Logging | Serilog | Event logging |
| Testing | xUnit + Moq | Automated tests |

## Typical Request Flow

```
HTTP Request
    ↓
[Middleware] → Auth, CORS, Logging
    ↓
[Controller] → Validate input
    ↓
[UseCase] → Orchestrate logic
    ↓
[Domain] → Apply business rules
    ↓
[Repository] → Access/persist data
    ↓
[Infrastructure] → Database, external services
    ↓
[Response] → DTO + Status Code
    ↓
[Middleware] → Logging, Compression
    ↓
HTTP Response
```

## Architecture Benefits

✅ **Testability**: Each layer can be tested independently  
✅ **Maintainability**: Changes are localized to specific layers  
✅ **Scalability**: Easy to add new features  
✅ **Flexibility**: Swap implementations without affecting business logic  
✅ **Framework Independence**: Domain doesn't depend on specific technologies  

## References

- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design - Eric Evans](https://www.domainlanguage.com/ddd/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
