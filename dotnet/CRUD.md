
### The CRUD Pattern (Works for ANY resource)

**1. GET ALL Pattern:**
**For GET all:**
- Do I need to filter/sort? (add parameters)
- Return OK with list
```csharp
[HttpGet]
public IActionResult GetAll()
{
    var items = _repository.GetAll();
    return Ok(items);
}
```

**2. GET BY ID Pattern:**
**For GET by id:**
- Does it exist? If no → 404
- Return OK with item
```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var item = _repository.GetById(id);
    if (item == null)
        return NotFound();
    
    return Ok(item);
}
```

**3. CREATE Pattern (the one you just read):**
**For POST:**
- Is the data valid? If no → 400
- Save it
- Return 201 with location
```csharp
[HttpPost]
public IActionResult Create([FromBody] Item item)
{
    // 1. Validate
    if (item == null)
        return BadRequest();
    
    // 2. Process (save to DB, etc.)
    _repository.Add(item);
    
    // 3. Return success with location
    return CreatedAtAction(nameof(GetById), new { id = item.Id }, item);
}
```

**4. UPDATE Pattern:**
**For PUT:**
- Is the data valid? If no → 400
- Does it exist? If no → 404
- Update it
- Return 204
```csharp
[HttpPut("{id}")]
public IActionResult Update(int id, [FromBody] Item item)
{
    // 1. Validate
    if (item == null || id != item.Id)
        return BadRequest();
    
    // 2. Check exists
    var existing = _repository.GetById(id);
    if (existing == null)
        return NotFound();
    
    // 3. Update
    _repository.Update(item);
    
    // 4. Return success
    return NoContent();  // 204 No Content (standard for PUT)
}
```

**5. DELETE Pattern:**

**For DELETE:**
- Does it exist? If no → 404
- Delete it
- Return 204
```csharp
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    // 1. Check exists
    var item = _repository.GetById(id);
    if (item == null)
        return NotFound();
    
    // 2. Delete
    _repository.Delete(id);
    
    // 3. Return success
    return NoContent();  // 204 No Content
}
```









