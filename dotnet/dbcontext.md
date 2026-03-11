# 📘 **DbContext Blueprint - The Ultimate Reference**

Save this as your go-to guide for every DbContext you'll ever create!

## 🏗️ **The Basic Blueprint Template**

```csharp
using Microsoft.EntityFrameworkCore;
using YourProject.Models;  // Change to your models namespace

namespace YourProject.Data;  // Change to your project name

public class AppDbContext : DbContext
{
    // 1️⃣ Constructor - ALWAYS looks like this
    public AppDbContext(DbContextOptions<AppDbContext> options) 
        : base(options)
    {
    }

    // 2️⃣ DbSets - ONE for each table in your database
    public DbSet<ModelName> ModelNames { get; set; }  // Replace ModelName
    public DbSet<AnotherModel> AnotherModels { get; set; }
    // Add as many as you need

    // 3️⃣ OPTIONAL: Configure relationships and rules
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);  // Always call base first
        
        // Your configurations go here
        ConfigureRelationships(modelBuilder);
        ConfigureIndexes(modelBuilder);
        ConfigureDefaultValues(modelBuilder);
    }

    // 4️⃣ OPTIONAL: Configure database connection (rarely used with DI)
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        base.OnConfiguring(optionsBuilder);
        // Only use if NOT using Program.cs configuration
    }
}
```

## 📋 **Real Examples You Can Copy**

### Example 1: **Simple Blog Database**
```csharp
using Microsoft.EntityFrameworkCore;
using BlogAPI.Models;

namespace BlogAPI.Data;

public class BlogDbContext : DbContext
{
    public BlogDbContext(DbContextOptions<BlogDbContext> options) 
        : base(options)
    {
    }

    // Tables
    public DbSet<Post> Posts { get; set; }
    public DbSet<Comment> Comments { get; set; }
    public DbSet<Author> Authors { get; set; }
    public DbSet<Category> Categories { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // One-to-Many: Author → Posts
        modelBuilder.Entity<Post>()
            .HasOne(p => p.Author)
            .WithMany(a => a.Posts)
            .HasForeignKey(p => p.AuthorId);

        // Index for faster searches
        modelBuilder.Entity<Post>()
            .HasIndex(p => p.PublishedDate);
    }
}
```

### Example 2: **E-Commerce Database**
```csharp
using Microsoft.EntityFrameworkCore;
using StoreAPI.Models;

namespace StoreAPI.Data;

public class StoreDbContext : DbContext
{
    public StoreDbContext(DbContextOptions<StoreDbContext> options) 
        : base(options)
    {
    }

    // Core tables
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Category> Categories { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Composite primary key for OrderItem (OrderId + ProductId)
        modelBuilder.Entity<OrderItem>()
            .HasKey(oi => new { oi.OrderId, oi.ProductId });

        // Prevent decimal truncation issues
        modelBuilder.Entity<Product>()
            .Property(p => p.Price)
            .HasPrecision(18, 2);

        // Index for frequent searches
        modelBuilder.Entity<Product>()
            .HasIndex(p => p.CategoryId);
    }
}
```

### Example 3: **Your Finance App**
```csharp
using Microsoft.EntityFrameworkCore;
using FinanceAPI.Models;

namespace FinanceAPI.Data;

public class FinanceDbContext : DbContext
{
    public FinanceDbContext(DbContextOptions<FinanceDbContext> options) 
        : base(options)
    {
    }

    // Your tables
    public DbSet<User> Users { get; set; }
    public DbSet<Account> Accounts { get; set; }
    public DbSet<Transaction> Transactions { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // User → Accounts (one-to-many)
        modelBuilder.Entity<User>()
            .HasMany(u => u.Accounts)
            .WithOne(a => a.User)
            .HasForeignKey(a => a.UserId)
            .OnDelete(DeleteBehavior.Cascade);  // Delete accounts if user deleted

        // Account → Transactions (one-to-many)
        modelBuilder.Entity<Account>()
            .HasMany(a => a.Transactions)
            .WithOne(t => t.Account)
            .HasForeignKey(t => t.AccountId)
            .OnDelete(DeleteBehavior.Cascade);

        // Unique constraints
        modelBuilder.Entity<User>()
            .HasIndex(u => u.Email)
            .IsUnique();

        modelBuilder.Entity<User>()
            .HasIndex(u => u.Username)
            .IsUnique();
    }
}
```

## 🔧 **Program.cs Registration (Always the Same)**

```csharp
// For In-Memory Database (Development/Testing)
builder.Services.AddDbContext<YourDbContext>(options =>
    options.UseInMemoryDatabase("YourDBName"));

// For SQL Server
builder.Services.AddDbContext<YourDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// For PostgreSQL
builder.Services.AddDbContext<YourDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

// For SQLite
builder.Services.AddDbContext<YourDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

## 📁 **Folder Structure Pattern**

```
YourProject/
├── Data/
│   ├── YourDbContext.cs           # Your DbContext
│   ├── Configurations/             # (Optional) Separate config files
│   │   ├── UserConfiguration.cs
│   │   └── AccountConfiguration.cs
│   └── Migrations/                  # Auto-generated by EF Core
├── Models/
│   ├── User.cs
│   ├── Account.cs
│   └── Transaction.cs
└── Program.cs
```

## 🎯 **The "Always the Same" Parts**

### ✅ **Always have this constructor:**
```csharp
public YourDbContext(DbContextOptions<YourDbContext> options) : base(options) { }
```

### ✅ **Always have DbSet for each model:**
```csharp
public DbSet<ModelName> ModelNames { get; set; }
```

### ✅ **Always register in Program.cs:**
```csharp
builder.Services.AddDbContext<YourDbContext>(...);
```

### ✅ **Always call base.OnModelCreating:**
```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    // your config
}
```

## 📝 **DbContext Lifecycle Cheat Sheet**

```
NEW DbContext Created
        ↓
📦 Gets options from constructor
        ↓
🔌 Opens connection (when needed)
        ↓
📊 Tracks entities you query
        ↓
✏️ You modify entities
        ↓
💾 SaveChanges() called
        ↓
📝 Generates SQL from changes
        ↓
⚡ Executes SQL in transaction
        ↓
🔒 Closes connection
        ↓
🗑️ DbContext Disposed (end of request)
```

## 🚨 **Common Mistakes to Avoid**

```csharp
// ❌ WRONG - Missing constructor
public class AppDbContext : DbContext
{
    public DbSet<TaskItem> Tasks { get; set; }
}

// ✅ CORRECT - Always include constructor
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<TaskItem> Tasks { get; set; }
}
```

```csharp
// ❌ WRONG - Forgetting DbSet
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    // No DbSet - EF won't create tables!
}

// ✅ CORRECT - Include DbSet for each entity
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<TaskItem> Tasks { get; set; }
}
```

## 🎓 **The Golden Rule**

**90% of DbContext code is copy-paste with renamed:**
- Class name (`AppDbContext` → `YourDbContext`)
- Namespace (`YourProject.Data`)
- DbSet types (`TaskItem` → `YourModel`)
- Connection string in Program.cs

**Save this blueprint and just fill in the blanks for every project!**