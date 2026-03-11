# 📝 **C# Async/Await - Complete Reference Guide**

## 🎯 **What is Async/Await?**

A way to write non-blocking code that **pauses** method execution without **blocking** the thread.

---

## 🔑 **Keywords**

| Keyword | Purpose |
|---------|---------|
| `async` | Marks a method as asynchronous (can use `await`) |
| `await` | Pauses method until task completes, but frees the thread |
| `Task` | Represents an async operation that returns **nothing** |
| `Task<T>` | Represents an async operation that returns **T** |

---

## 📋 **Basic Syntax Patterns**

### 1. No Return Value
```csharp
public async Task SaveDataAsync()
{
    await _context.SaveChangesAsync();
    // Method continues here after SaveChangesAsync completes
}
```

### 2. Returns a Value
```csharp
public async Task<User> GetUserAsync(int id)
{
    return await _context.Users.FindAsync(id);
}
```

### 3. Multiple Awaits
```csharp
public async Task ProcessUserAsync(int id)
{
    var user = await GetUserAsync(id);      // Wait for user
    var orders = await GetOrdersAsync(user); // Wait for orders
    await SaveLogAsync();                    // Wait for log
}
```

---

## ⚡ **The Golden Rule**

**Async ALL THE WAY** or **NOT AT ALL** - Never mix!

```csharp
// ❌ WRONG - Don't do this!
public async Task<User> GetUserAsync(int id)
{
    return _context.Users.Find(id);  // Sync method in async method
}

// ❌ WRONG - Don't do this!
public User GetUser(int id)
{
    return await _context.Users.FindAsync(id);  // Can't await in non-async
}

// ✅ CORRECT
public async Task<User> GetUserAsync(int id)
{
    return await _context.Users.FindAsync(id);
}
```

---

## 🚫 **Common Mistakes**

### 1. Forgetting `await`
```csharp
// Returns Task<User>, NOT User!
public Task<User> GetUserAsync(int id)
{
    return _context.Users.FindAsync(id);  // No await - returns Task directly
}

// Caller must handle Task
Task<User> task = GetUserAsync(1);  // Not awaited - method runs but you can't access result yet
```

### 2. Blocking Async Code (DEADLOCK RISK!)
```csharp
// ❌ DANGEROUS - Can cause deadlock!
User user = GetUserAsync(1).Result;      // Blocks thread - BAD!
User user = GetUserAsync(1).GetAwaiter().GetResult(); // Also BAD

// ✅ CORRECT
User user = await GetUserAsync(1);       // Async all the way
```

### 3. Async Void (Except Event Handlers)
```csharp
// ❌ BAD - Can't be awaited, exceptions crash app!
public async void SaveUserAsync(User user)
{
    await _context.SaveChangesAsync();
}

// ✅ GOOD
public async Task SaveUserAsync(User user)
{
    await _context.SaveChangesAsync();
}
```

---

## 🏗️ **Method Signatures**

```csharp
// Synchronous versions
public void DoWork() { }
public string GetData() { }

// Asynchronous versions
public async Task DoWorkAsync() { }           // Returns nothing
public async Task<string> GetDataAsync() { }  // Returns string
```

---

## 🎭 **What Happens During Await**

```csharp
public async Task<string> GetDataAsync()
{
    Console.WriteLine("1. Starting on thread: " + Thread.CurrentThread.ManagedThreadId);
    
    string result = await FetchFromDatabaseAsync();  // 🟡 PAUSES HERE
    
    // 🔵 RESUMES HERE (possibly on different thread!)
    Console.WriteLine("2. Resumed on thread: " + Thread.CurrentThread.ManagedThreadId);
    return result;
}
```

**Flow:**
1. Method starts on Thread 1
2. Hits `await`, Thread 1 returns to pool (free!)
3. Database works
4. When done, method resumes on ANY available thread (maybe Thread 2)
5. Continues execution

---

## 📊 **When to Use Async**

| DO Use Async For | DON'T Use Async For |
|-----------------|-------------------|
| Database queries (EF Core) | CPU-intensive calculations |
| Web API calls (HttpClient) | Simple math operations |
| File I/O operations | In-memory list operations |
| Network requests | Quick validation logic |

---

## 🔄 **Running Tasks in Parallel**

### Wait for all:
```csharp
var task1 = GetUserAsync(1);
var task2 = GetUserAsync(2);
var task3 = GetUserAsync(3);

await Task.WhenAll(task1, task2, task3);
// All three run at the same time!
```

### Wait for first to complete:
```csharp
var task1 = GetFromDbAsync();
var task2 = GetFromCacheAsync();
var task3 = GetFromApiAsync();

var first = await Task.WhenAny(task1, task2, task3);
// Returns as soon as ANY one completes
```

---

## 🎯 **Async in Web API Controllers**

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _context.Users.FindAsync(id);  // ✅ GOOD
    // Thread is FREE while waiting for database!
    
    if (user == null)
        return NotFound();
    
    return user;
}
```

**Why this matters:**
- Without async: Thread blocks for each database call
- With async: Thread handles other requests while waiting
- **Your API can handle 100x more users!**

---

## 📝 **Cheat Sheet**

```csharp
// SYNCHRONOUS (blocking)
public User GetUser(int id) => _users.Find(id);

// ASYNCHRONOUS (non-blocking)
public async Task<User> GetUserAsync(int id) 
    => await _context.Users.FindAsync(id);

// Call async method
User user = await GetUserAsync(1);

// Chain async calls
public async Task<User> GetUserWithOrdersAsync(int id)
{
    var user = await GetUserAsync(id);
    user.Orders = await GetOrdersAsync(id);
    return user;
}

// Error handling
try
{
    await SaveUserAsync(user);
}
catch (Exception ex)
{
    _logger.LogError(ex, "Save failed");
}
```

---

## 🎓 **Key Takeaways**

1. **Async is about SCALABILITY**, not speed
2. **Never block** on async code (no .Result, no .Wait())
3. **Async all the way** - once async, everything should be async
4. **Await** tells the compiler "pause here, don't block"
5. **Task** is a promise of future value
6. Use **async/await** for I/O operations (DB, files, network)
7. Use **regular methods** for CPU work

---

**Want to practice?** Try converting this sync method to async:
```csharp
public List<Transaction> GetLargeTransactions(decimal minAmount)
{
    return _context.Transactions
        .Where(t => t.Amount > minAmount)
        .ToList();
}
```

**Your async version:**
```csharp
public async Task<List<Transaction>> GetLargeTransactionsAsync(decimal minAmount)
{
    return await _context.Transactions
        .Where(t => t.Amount > minAmount)
        .ToListAsync();  // Note: ToListAsync() not ToList()
}
```

Save this note and refer back to it whenever you're confused! 📚