### Overview: `Codable` and Key Mapping

`Codable` is a type alias in the Swift standard library that combines two protocols: `Encodable` (for converting Swift types to external formats like JSON) and `Decodable` (for converting external formats back into Swift types).

It serves as the **compiler-synthesized bridge** between your type-safe Swift code and loosely typed data formats (JSON, Plist).

-----

### Derived Sub-Questions

1.  **What** is `Codable` really?
2.  **How** do you handle mismatched JSON keys (The `CodingKeys` Enum)?
3.  **How** can you handle this automatically (Key Decoding Strategy)?
4.  **What** happens if you need to map complex/nested structures?

-----

### 1\. What is `Codable` really?

#### **What**

`Codable` is simply:

```swift
typealias Codable = Encodable & Decodable
```

If all properties of your type are themselves `Codable` (like `String`, `Int`, `Date`, or other `Codable` structs), the Swift compiler automatically generates the serialization code (the `init(from:)` and `encode(to:)` methods) for you.

#### **Why**

Before `Codable` (Swift 4), developers relied on `JSONSerialization` (casting `Any` dictionaries) or third-party libraries (like SwiftyJSON). This was error-prone, untyped, and full of boilerplate. `Codable` provides a compile-time safe, native way to handle data.

-----

### 2\. Handling Mismatched Keys (The `CodingKeys` Enum)

#### **What**

Often, APIs use `snake_case` (e.g., `user_id`) while Swift uses `camelCase` (e.g., `userId`). To bridge this gap without renaming your Swift properties to ugly snake\_case, you use a special nested enum called `CodingKeys`.

#### **Why**

This decouples your internal variable naming from the external API contract. It allows you to maintain clean Swift style guides regardless of the backend's naming conventions.

#### **How**

You define an enum named `CodingKeys` that conforms to `String` and `CodingKey`.

```swift
struct User: Codable {
    let id: Int
    let firstName: String // Swift style
    let isActive: Bool

    // Map Swift names (cases) to JSON keys (raw values)
    enum CodingKeys: String, CodingKey {
        case id                 // Matches JSON "id"
        case firstName = "first_name" // Maps "first_name" -> firstName
        case isActive = "is_active"   // Maps "is_active" -> isActive
    }
}

// JSON: {"id": 1, "first_name": "Alice", "is_active": true}
```

**Critical Rule:** If you define `CodingKeys`, you must include **all** properties you want to encode/decode. If you omit a property from the enum, it will be ignored by the encoder/decoder.

-----

### 3\. Automatic Handling (Key Decoding Strategy)

#### **What**

If your key mapping is purely a standard conversion between `snake_case` and `camelCase`, you don't need `CodingKeys` at all. You can configure the `JSONDecoder` to do the work for you.

#### **Why**

This reduces boilerplate. If you have 50 structs that all need `snake_case` mapping, writing `CodingKeys` for every single one is tedious and prone to typos.

#### **How**

```swift
struct User: Codable {
    let firstName: String
    let lastName: String
}

let json = """
{ "first_name": "John", "last_name": "Doe" }
""".data(using: .utf8)!

let decoder = JSONDecoder()
// ✅ Automatically converts "first_name" -> firstName
decoder.keyDecodingStrategy = .convertFromSnakeCase

let user = try decoder.decode(User.self, from: json)
```

**Trade-off:** This incurs a small runtime performance cost (converting strings dynamically) compared to the compile-time static mapping of `CodingKeys`. For most apps, this is negligible, but in high-throughput parsing systems, `CodingKeys` is faster.

-----

### 4\. Advanced: Manual Mapping (Nested/Complex Keys)

#### **What**

Sometimes the API structure doesn't match your object structure at all (e.g., the JSON has a nested object but you want a flat struct). `CodingKeys` is insufficient here; you must implement `init(from: Decoder)` manually.

#### **How**

```swift
struct Product: Decodable {
    let name: String
    let price: Double
    
    // JSON: { "name": "MacBook", "meta": { "cost": 1200.0 } }
    
    enum CodingKeys: String, CodingKey {
        case name
        case meta // Key for the nested container
    }
    
    enum MetaKeys: String, CodingKey {
        case cost
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.name = try container.decode(String.self, forKey: .name)
        
        // Dig into the nested container
        let metaContainer = try container.nestedContainer(keyedBy: MetaKeys.self, forKey: .meta)
        self.price = try metaContainer.decode(Double.self, forKey: .cost)
    }
}
```

### Summary

  * **Codable** = Automagic serialization.
  * **CodingKeys**: The manual bridge for specific key renaming (`case myProp = "api_key"`).
  * **keyDecodingStrategy**: The "lazy" switch for global Snake -\> Camel conversion.
  * **Manual init**: Required when the *structure* of the data changes (flattening nested JSON), not just the names.