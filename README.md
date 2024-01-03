# How to create a .NET8 WebAPI CRUD MongoDB Microservice

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



## 5. Add the Controller



## 6. Modify Program.cs file



## 7. Modify appsettings.json file





