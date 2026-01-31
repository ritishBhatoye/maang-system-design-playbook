# gRPC vs REST

## 1. Problem Statement

When should we choose gRPC over REST for service-to-service communication, and what are the trade-offs?

---

## 2. REST (Representational State Transfer)

### Characteristics
* **HTTP-based**: Uses HTTP methods (GET, POST, PUT, DELETE)
* **Resource-oriented**: URLs represent resources
* **Stateless**: Each request contains all information
* **JSON/XML**: Human-readable formats

### Pros
* **Simple**: Easy to understand, debug
* **Universal**: Works with any HTTP client
* **Caching**: HTTP caching works out of the box
* **Browser-friendly**: Works directly from browsers

### Cons
* **Overhead**: JSON is verbose, HTTP headers add overhead
* **No Streaming**: Request-response only (HTTP/2 helps)
* **No Strong Typing**: JSON schema validation needed
* **Performance**: Slower than binary protocols

---

## 3. gRPC (Google Remote Procedure Call)

### Characteristics
* **HTTP/2-based**: Multiplexed, binary protocol
* **Protocol Buffers**: Binary serialization (smaller, faster)
* **Strong Typing**: Schema-defined contracts
* **Streaming**: Supports bidirectional streaming

### Pros
* **Performance**: Binary format, HTTP/2 multiplexing
* **Strong Typing**: Compile-time type checking
* **Streaming**: Real-time bidirectional communication
* **Code Generation**: Auto-generate client/server code

### Cons
* **Complexity**: More setup, learning curve
* **Browser Support**: Limited (needs gRPC-Web)
* **Less Human-Readable**: Binary format harder to debug
* **Tooling**: Less tooling than REST

---

## 4. Performance Comparison

### Latency
* **REST**: ~5-10ms overhead (JSON parsing, HTTP)
* **gRPC**: ~1-2ms overhead (binary, HTTP/2)
* **Winner**: gRPC (2-5x faster)

### Throughput
* **REST**: Limited by JSON size, HTTP/1.1 connection limits
* **gRPC**: HTTP/2 multiplexing, binary format
* **Winner**: gRPC (higher throughput)

### Payload Size
* **REST**: JSON is verbose (larger payloads)
* **gRPC**: Protocol Buffers are compact (smaller payloads)
* **Winner**: gRPC (30-50% smaller)

---

## 5. Use Cases

### Choose REST When:
* **Public APIs**: External-facing, browser clients
* **Simple CRUD**: Standard resource operations
* **Human Readable**: Need to debug, inspect requests
* **Caching**: Need HTTP caching
* **Wide Compatibility**: Need to work with any client

### Choose gRPC When:
* **Internal Services**: Service-to-service communication
* **High Performance**: Need low latency, high throughput
* **Streaming**: Real-time bidirectional communication
* **Strong Contracts**: Need type safety, code generation
* **Microservices**: Many small services communicating

---

## 6. Streaming

### REST Streaming
* **Server-Sent Events (SSE)**: Server → Client only
* **Chunked Transfer**: Streaming responses
* **WebSockets**: Bidirectional (not REST, but HTTP-based)

### gRPC Streaming
* **Unary**: Request → Response (like REST)
* **Server Streaming**: Client request → Server stream
* **Client Streaming**: Client stream → Server response
* **Bidirectional**: Both sides stream simultaneously

**Use Case**: Real-time updates, large file transfers, chat systems

---

## 7. Schema Evolution

### REST
* **Versioning**: URL versioning (/v1/, /v2/) or headers
* **Backward Compatibility**: Manual, no enforcement
* **Breaking Changes**: Can break clients

### gRPC
* **Protocol Buffers**: Built-in versioning support
* **Backward Compatibility**: Rules enforced (add optional fields)
* **Breaking Changes**: Compile-time errors prevent breaks

**Winner**: gRPC (better schema management)

---

## 8. Tooling & Ecosystem

### REST
* **Tools**: Postman, curl, browser DevTools
* **Documentation**: OpenAPI/Swagger
* **Libraries**: Available in all languages
* **Monitoring**: Standard HTTP monitoring

### gRPC
* **Tools**: grpcurl, BloomRPC (limited)
* **Documentation**: Protocol Buffer definitions
* **Libraries**: Good support, but less universal
* **Monitoring**: Need gRPC-specific tools

**Winner**: REST (better tooling, more universal)

---

## 9. How to Explain This in an Interview

> "I use REST for public APIs and browser clients because it's simple, universal, and human-readable. For internal service-to-service communication, I use gRPC because it's 2-5x faster due to binary serialization and HTTP/2 multiplexing. I also use gRPC when I need streaming capabilities, like real-time updates or large file transfers. The trade-off is complexity and tooling - REST is simpler to debug and has better tooling, but gRPC provides better performance and type safety."

**Key Points:**
* REST for external APIs, gRPC for internal services
* gRPC for performance-critical, high-throughput scenarios
* gRPC for streaming use cases
* Trade-off: Simplicity vs Performance

---

## 10. Common Mistakes

* **gRPC for everything**: "We use gRPC for all APIs"
  * Fix: Use REST for public APIs, gRPC for internal services
* **REST for high-performance**: "REST is fine for everything"
  * Fix: Use gRPC when performance matters (internal services)
* **Ignoring streaming**: "We don't need streaming"
  * Fix: Consider streaming for real-time use cases
* **No schema management**: "We change APIs freely"
  * Fix: Version APIs, maintain backward compatibility
* **Not considering browser support**: "gRPC from browser"
  * Fix: Use gRPC-Web or REST for browser clients
