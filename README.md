# Smart Campus API — 5COSC022W Coursework

# 📡 Smart Campus API (JAX-RS)

## 👨‍💻 Author
W.K.T. Dulara  
Module: 5COSC022W – Client-Server Architectures

## Overview
A RESTful JAX-RS API for managing campus rooms and IoT sensors.
Built with Jersey 2.41 deployed on Apache Tomcat 9.

**Base URL:** `http://localhost:8080/SmartCampusAPI/api/v1`

## Tech Stack
- Java 11
- JAX-RS (Jersey 2.41)
- Apache Tomcat 9
- Maven
- In-memory storage (ConcurrentHashMap)

## How to Build and Run

1. Clone the repository:
   git clone https://github.com/WKTDulara/SmartCampusAPI.git

2. Open in NetBeans IDE

3. Ensure Apache Tomcat 9 is configured in NetBeans

4. Right-click project → Clean and Build

5. Right-click project → Run

6. API is available at:
   http://localhost:8080/SmartCampusAPI/api/v1

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/v1 | Discovery endpoint |
| GET | /api/v1/rooms | Get all rooms |
| POST | /api/v1/rooms | Create a room |
| GET | /api/v1/rooms/{id} | Get room by ID |
| DELETE | /api/v1/rooms/{id} | Delete a room |
| GET | /api/v1/sensors | Get all sensors |
| GET | /api/v1/sensors?type=X | Filter sensors by type |
| POST | /api/v1/sensors | Create a sensor |
| GET | /api/v1/sensors/{id} | Get sensor by ID |
| GET | /api/v1/sensors/{id}/readings | Get sensor readings |
| POST | /api/v1/sensors/{id}/readings | Add a reading |

## Sample curl Commands

1. Discovery endpoint:
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1

2. Get all rooms:
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/rooms

3. Create a room:
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/rooms \
  -H "Content-Type: application/json" \
  -d "{\"id\":\"CS-201\",\"name\":\"CS Lab\",\"capacity\":40}"

4. Filter sensors by type:
curl -X GET "http://localhost:8080/SmartCampusAPI/api/v1/sensors?type=Temperature"

5. Add a sensor reading:
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/sensors/TEMP-001/readings \
  -H "Content-Type: application/json" \
  -d "{\"value\":24.5}"

## Report: Answers to Coursework Questions

### Part 1.1
Question - In your report, explain the default lifecycle of a JAX-RS Resource class. Is a
new instance instantiated for every incoming request, or does the runtime treat it as a
singleton? Elaborate on how this architectural decision impacts the way you manage and
synchronize your in-memory data structures (maps/lists) to prevent data loss or race conditions.

Answer - JAX-RS Resource Lifecycle
By default, JAX-RS creates a new instance of each Resource class for every
incoming HTTP request (per-request scope). This means instance variables are
not shared between requests. In this project, shared state is managed through
a Singleton DataStore using ConcurrentHashMap, which is thread-safe and
prevents race conditions when multiple requests arrive simultaneously.

### Part 1.2
Question - Why is the provision of ”Hypermedia” (links and navigation within responses)
considered a hallmark of advanced RESTful design (HATEOAS)? How does this approach
benefit client developers compared to static documentation?

Answer - HATEOAS
HATEOAS (Hypermedia as the Engine of Application State) means API responses
include links to related actions and resources. This benefits client developers
because they do not need to hardcode URLs — they navigate the API dynamically
by following embedded links, similar to browsing a website. It reduces coupling
between client and server and makes the API self-documenting.

### Part 2.1
Question - When returning a list of rooms, what are the implications of returning only
IDs versus returning the full room objects? Consider network bandwidth and client side
processing

Answer - IDs vs Full Objects
Returning only IDs is bandwidth-efficient but forces clients to make additional
requests for each room (N+1 problem). Returning full objects increases payload
size but reduces round trips. For a large campus API, returning summary objects
for lists and full detail only for individual GET requests is the best approach.

### Part 2.2
Question - Is the DELETE operation idempotent in your implementation? Provide a detailed
justification by describing what happens if a client mistakenly sends the exact same DELETE
request for a room multiple times.

Answer - DELETE Idempotency
Yes, DELETE is idempotent in this implementation. The first DELETE on a room
returns 204 No Content. Subsequent DELETE requests for the same room return
404 Not Found. The server state does not change after the first deletion, so
repeated calls produce consistent outcomes, satisfying the HTTP idempotency
requirement.

### Part 3.1
Question - We explicitly use the @Consumes (MediaType.APPLICATION_JSON) annotation on
the POST method. Explain the technical consequences if a client attempts to send data in
a different format, such as text/plain or application/xml. How does JAX-RS handle this
mismatch?

Answer - @Consumes Mismatch
The @Consumes(MediaType.APPLICATION_JSON) annotation tells JAX-RS the endpoint
only accepts application/json. If a client sends text/plain or application/xml,
JAX-RS automatically returns 415 Unsupported Media Type before the method body
executes, protecting the endpoint from unparseable data.

### Part 3.2
Question - You implemented this filtering using @QueryParam. Contrast this with an alternative design where the type is part of the URL path (e.g., /api/vl/sensors/type/CO2). Why
is the query parameter approach generally considered superior for filtering and searching
collections?

Answer - @QueryParam vs Path Parameter
Using @QueryParam for filtering (/sensors?type=CO2) is superior because query
parameters are optional — the endpoint works with or without them. Path
parameters (/sensors/type/CO2) imply a fixed resource hierarchy and make
filtering mandatory. Query parameters are the standard convention for
filtering, searching, and pagination in REST APIs.

### Part 4.1
Question - Discuss the architectural benefits of the Sub-Resource Locator pattern. How
does delegating logic to separate classes help manage complexity in large APIs compared
to defining every nested path (e.g., sensors/{id}/readings/{rid}) in one massive controller class?

Answer - Sub-Resource Locator Pattern
The Sub-Resource Locator pattern delegates nested path handling to dedicated
classes. Instead of defining all sensor reading paths in SensorResource, a
method returns a SensorReadingResource instance. This separates concerns,
prevents god classes, makes code easier to test and maintain, and allows
sub-resources to be independently versioned or reused.

### Part 5.2
Question - Why is HTTP 422 often considered more semantically accurate than a standard
404 when the issue is a missing reference inside a valid JSON payload?

Answer - HTTP 422 vs 404
A 404 means the requested URL does not exist. When a sensor POST references a
non-existent roomId, the URL /api/v1/sensors is valid — the problem is inside
the request data. HTTP 422 Unprocessable Entity precisely communicates that
the server understands the format but cannot process it due to a logical
validation failure, giving clients clearer diagnostic information.

### Part 5.4
Question - From a cybersecurity standpoint, explain the risks associated with exposing
internal Java stack traces to external API consumers. What specific information could an
attacker gather from such a trace?

Answer - Stack Trace Security Risk
Exposing Java stack traces reveals: package and class names (exposing internal
architecture), library versions (enabling targeted CVE exploits), file paths
and line numbers (aiding injection attacks), and business logic structure
(helping attackers find weaknesses). A global exception mapper returns a
generic 500 message instead, keeping internal details hidden.

### Part 5.5
Question - Why is it advantageous to use JAX-RS filters for cross-cutting concerns like
logging, rather than manually inserting Logger.info() statements inside every single resource method?

Answer - Filters vs Manual Logging
Using JAX-RS filters follows the DRY principle. Manual Logger.info() calls in
every method are error-prone, inconsistent, and hard to maintain. A single
filter class automatically intercepts every request and response, ensuring
uniform logging coverage that can be toggled or replaced without touching
any resource class.
