# How to create a .NET8 WebAPI CRUD MongoDB Microservice

## 0. Prerequisites

- Install Docker Desktop

- Install Visual Studio 2022 Community Edition version 17.8

- Install Studio 3T Free for MongoDB

## 1. Create .NET8 WebAPI in Visual Studio 2022 Community Edition



## 2. Add the MongoDB.Driver dependency

Select the menu option Tools->Nuget Package Manager->Manage Nuget Packages for Solution...

Then browse Mongo.DB.Driver and install it in your solution

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/c793e87e-eb6a-4464-b9d5-d50a2d5b71b3)

## 3. Add the Models

Create the Models folder and inside include the following two files:

**Book.cs**

```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
using System.Text.Json.Serialization;

namespace BookStoreApi.Models
{
    public class Book
    {
        [BsonId]
        [BsonRepresentation(BsonType.ObjectId)]
        public string? Id { get; set; }

        [BsonElement("Name")]
        [JsonPropertyName("Name")]
        public string BookName { get; set; } = null!;

        public decimal Price { get; set; }

        public string Category { get; set; } = null!;

        public string Author { get; set; } = null!;
    }
}
```

**BookStoreDatabaseSettings.cs**

```csharp
namespace BookStoreApi.Models
{
    public class BookStoreDatabaseSettings
    {
        public string ConnectionString { get; set; } = null!;

        public string DatabaseName { get; set; } = null!;

        public string BooksCollectionName { get; set; } = null!;
    }
}
```

## 4. Add the Service

Create a Services folder with the following file:

**BookService.cs**

```csharp
using BookStoreApi.Models;
using Microsoft.Extensions.Options;
using MongoDB.Driver;

namespace BookStoreApi.Services
{
    public class BooksService
    {
        private readonly IMongoCollection<Book> _booksCollection;

        public BooksService(
            IOptions<BookStoreDatabaseSettings> bookStoreDatabaseSettings)
        {
            var mongoClient = new MongoClient(
                bookStoreDatabaseSettings.Value.ConnectionString);

            var mongoDatabase = mongoClient.GetDatabase(
                bookStoreDatabaseSettings.Value.DatabaseName);

            _booksCollection = mongoDatabase.GetCollection<Book>(
                bookStoreDatabaseSettings.Value.BooksCollectionName);
        }

        public async Task<List<Book>> GetAsync() =>
            await _booksCollection.Find(_ => true).ToListAsync();

        public async Task<Book?> GetAsync(string id) =>
            await _booksCollection.Find(x => x.Id == id).FirstOrDefaultAsync();

        public async Task CreateAsync(Book newBook) =>
            await _booksCollection.InsertOneAsync(newBook);

        public async Task UpdateAsync(string id, Book updatedBook) =>
            await _booksCollection.ReplaceOneAsync(x => x.Id == id, updatedBook);

        public async Task RemoveAsync(string id) =>
            await _booksCollection.DeleteOneAsync(x => x.Id == id);
    }
}
```

## 5. Add the Controller

In the Controllers folder include the following file:

**BooksController.cs**

```csharp
using BookStoreApi.Models;
using BookStoreApi.Services;
using Microsoft.AspNetCore.Mvc;
using System.Data;

namespace BookStoreApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly BooksService _booksService;

    public BooksController(BooksService booksService) =>
        _booksService = booksService;

    [HttpGet]
    public async Task<List<Book>> Get() =>
        await _booksService.GetAsync();

    [HttpGet("{id:length(24)}")]
    public async Task<ActionResult<Book>> Get(string id)
    {
        var book = await _booksService.GetAsync(id);

        if (book is null)
        {
            return NotFound();
        }

        return book;
    }

    [HttpPost]
    public async Task<IActionResult> Post(Book newBook)
    {
        await _booksService.CreateAsync(newBook);

        return CreatedAtAction(nameof(Get), new { id = newBook.Id }, newBook);
    }

    [HttpPut("{id:length(24)}")]
    public async Task<IActionResult> Update(string id, Book updatedBook)
    {
        var book = await _booksService.GetAsync(id);

        if (book is null)
        {
            return NotFound();
        }

        updatedBook.Id = book.Id;

        await _booksService.UpdateAsync(id, updatedBook);

        return NoContent();
    }

    [HttpDelete("{id:length(24)}")]
    public async Task<IActionResult> Delete(string id)
    {
        var book = await _booksService.GetAsync(id);

        if (book is null)
        {
            return NotFound();
        }

        await _booksService.RemoveAsync(id);

        return NoContent();
    }
}
```

## 6. Modify Program.cs file

In the Program.cs file include the following code:

```csharp
using BookStoreApi.Models;
using BookStoreApi.Services;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

ConfigurationManager Configuration = builder.Configuration;

// Add services to the container.
builder.Services.Configure<BookStoreDatabaseSettings>(
    builder.Configuration.GetSection("BookStoreDatabase"));

builder.Services.AddSingleton<BooksService>();

//builder.Services.AddHttpContextAccessor();

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

## 7. Modify appsettings.json file

In the appsettings.json file include the following code:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "BookStoreDatabase": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "BookStore",
    "BooksCollectionName": "Books"
  },
  "AllowedHosts": "*"
}
```

## 8. How to run the application

### 8.1. First, we need to pull and run the MondoDB database Docker container image

For pulling the MondoDB docker image from Docker hub we run these commands

```
docker login
```

```
docker pull mongo
```

For running the Mongodb docker image we type the command:

```

```

To enter in the Mongodb database we type this command:

```
docker exec -it mongodb-container mongosh
```

And also we create a new database with a collections and two documents inside, see this picture:

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/29417925-b773-4350-b2f3-e0d982ac62e1)

### 8.2. Second, we build and run the WebAPI application with HTTP protocol



### 8.3. Third, we verify the data with 3T Studio Free for MongoDb

We connect to the MongoDB running container from 3T Studio setting the connection string: **mongodb://localhost:27017**

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/2fe753e7-66e4-461b-96fb-824c46a9e032)

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/729fbbef-5c62-4ced-8026-86119d45bfd9)

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/c4b3e323-763b-467d-8abe-5f01418260e2)

We set the connection name

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/9e6a36b0-aeb3-4a88-b526-466297751eec)

And we connect to the database

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/dea5a270-256b-4f5e-93ad-96a546e61965)

See the data inside the new database and collection

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-MongoDB/assets/32194879/4b031a72-e331-4e52-8f49-31b79cc68655)


