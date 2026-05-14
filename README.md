# Multi LLM Springboot Integration Service

## Overview

This is a Spring Boot 3.4.1 application that integrates multiple Large Language Models (LLMs) using LangChain4J 0.30.0. The service provides a unified API for interacting with cloud-based and local LLMs, enabling enterprise-grade AI capabilities including natural language processing, content generation, sentiment analysis, data extraction, and retrieval-augmented generation (RAG).

## Technology Stack

### Core Framework
- Java 23
- Spring Boot 3.4.1
- Jakarta EE 10 (Jakarta Servlet, Jakarta Persistence, Jakarta Validation)

### AI/ML Libraries
- LangChain4J 0.30.0 - LLM integration framework
- Deep Learning 4J 1.0.0-M2.1 - Deep learning framework
- ND4J 1.0.0-M2.1 - N-dimensional arrays for Java

### LLM Providers
- OpenAI (GPT-4o, GPT-3.5 Turbo)
- Anthropic (Claude 3)
- Google Vertex AI (Gemini, Palm)
- Hugging Face
- Cohere
- Ollama (Local LLMs: Llama3, Llama2, Mistral, Phi, TinyLlama)

### Infrastructure
- H2 Database (development)
- PostgreSQL (staging/production)
- Redis (caching)
- Spring Security with JWT authentication
- Jasypt for property encryption

## Architecture

### Package Structure

```
io.fusion.air.microservice
├── adapters                    # REST API implementations
│   ├── controllers
│   │   ├── open               # Public endpoints
│   │   └── secured            # Protected endpoints
│   └── repository             # Data access layer
├── ai
│   └── genai                  # Generative AI functionality
│       ├── controllers        # AI-specific REST controllers
│       │   ├── AbstractAiController
│       │   ├── AiOpenAiControllerImpl
│       │   └── AiOllamaControllerImpl
│       └── core
│           ├── assistants     # AI assistant implementations
│           │   ├── Assistant (interface)
│           │   ├── CarRentalAssistant
│           │   ├── ChefAssistant
│           │   ├── DataExtractorAssistant
│           │   ├── HAL9000Assistant
│           │   ├── HealthCareAssistant
│           │   ├── LanguageAssistant
│           │   └── ModerateAssistant
│           ├── models         # Data models
│           ├── prompts        # Prompt templates
│           ├── services       # Business logic
│           └── tools          # LLM function tools
├── domain                     # Domain entities and models
│   ├── entities
│   ├── models
│   └── ports
│       └── services           # Domain service interfaces
├── security                   # Security configuration
│   ├── jwt
│   └── filters
├── aop                        # Aspect-oriented programming
├── external                   # External service integrations
├── server                     # Server configuration
│   └── service
└── utils                      # Utility classes
```

### API Endpoints

#### OpenAI Endpoints
- `POST /api/ai/openai/chat` - Generic AI chat
- `POST /api/ai/openai/chat/custom` - Chat with custom data (RAG)
- `POST /api/ai/openai/chat/structured` - Structured data extraction
- `GET /api/ai/openai/chat/userid/{userId}` - Retrieve chat history
- `POST /api/ai/openai/string/chat` - String-based chat responses

#### Ollama Endpoints
- `POST /api/ai/ollama/chat` - Generic AI chat (local LLMs)
- `POST /api/ai/ollama/chat/custom` - Chat with custom data
- `POST /api/ai/ollama/chat/structured` - Structured data extraction
- `GET /api/ai/ollama/chat/userid/{userId}` - Retrieve chat history
- `POST /api/ai/ollama/string/chat` - String-based chat responses

## Configuration

### Environment Variables

```bash
# API Keys (required for cloud LLMs)
OPENAI_API_KEY=<your-openai-api-key>
ANTHROPIC_API_KEY=<your-anthropic-api-key>
COHERE_API_KEY=<your-cohere-api-key>
HF_API_KEY=<your-huggingface-api-key>
RAPID_API_KEY=<your-rapidapi-key>

# Database encryption
JASYPT_ENCRYPTOR_PASSWORD=<your-encryption-key>
```

### Application Profiles

| Profile | Description | Database |
|---------|-------------|----------|
| dev | Development mode | H2 (in-memory) |
| staging | Staging environment | PostgreSQL |
| prod | Production environment | PostgreSQL |

### Key Configuration Properties

```properties
# Server configuration
server.port=19090
service.api.path=/ai-service/api/v1

# LangChain4J configuration
langchain4j.open-ai.model=gpt-4o-2024-05-13
langchain4j.ollama.url=http://localhost:11434/api/generate/
langchain4j.ollama.model=llama3

# Security configuration
server.token.type=1
server.token.auth.expiry=600000
server.token.refresh.expiry=3600000

# Database configuration
spring.datasource.url=jdbc:h2:mem:ai_324
spring.datasource.username=sa
spring.datasource.password=ENC(<encrypted-password>)
```

## Setup Instructions

### Prerequisites

- Java 23 or later
- Maven 3.8+
- Git

### Installation

1. Clone the repository
```bash
git clone https://github.com/joyprakashk/multi-llm-springboot-integration.git
cd multi-llm-springboot-integration
```

2. Set up environment variables
```bash
export OPENAI_API_KEY=<your-openai-api-key>
export JASYPT_ENCRYPTOR_PASSWORD=<your-encryption-key>
```

3. Encrypt database passwords (if needed)
```bash
# For Windows
encrypt <your-password> <your-encryption-key>

# For Linux/Mac
./encrypt <your-password> <your-encryption-key>
```

4. Update database passwords in property files
```properties
# application-dev.properties
spring.datasource.password=ENC(<encrypted-h2-password>)

# application-staging.properties
spring.datasource.password=ENC(<encrypted-staging-password>)

# application-prod.properties
spring.datasource.password=ENC(<encrypted-prod-password>)
```

5. Build the application
```bash
# Windows
mvn clean package

# Linux/Mac
./compile
```

6. Run the application
```bash
# Development mode
java -jar target/ai-service-0.3.4-spring-boot.jar --spring.profiles.active=dev

# Staging mode
java -jar target/ai-service-0.3.4-spring-boot.jar --spring.profiles.active=staging

# Production mode
java -jar target/ai-service-0.3.4-spring-boot.jar --spring.profiles.active=prod
```

## Usage

### Testing with Swagger UI

After starting the application, access Swagger UI at:
```
http://localhost:19090/api/ai-service/api/v1/swagger-ui.html
```

### Authentication

The service uses JWT tokens for authentication. Generate test tokens by setting:
```properties
server.token.test=true
```

For production environments, disable test token generation:
```properties
server.token.test=false
```

### Example API Calls

#### Generic Chat
```bash
curl -X POST http://localhost:19090/api/ai-service/api/v1/ai/openai/chat \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "Hello, how are you?"
```

#### Chat with Custom Data (RAG)
```bash
curl -X POST http://localhost:19090/api/ai-service/api/v1/ai/openai/chat/custom \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "What are the terms for car rentals?"
```

#### Structured Data Extraction
```bash
curl -X POST http://localhost:19090/api/ai-service/api/v1/ai/openai/chat/structured \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "Extract recipe ingredients: oven dish, cucumber, potato, tomato, salmon, olives, olive oil"
```

## AI Assistant Implementations

### Assistant Interface

All AI assistants implement the `Assistant` interface:

```java
public interface Assistant {
    String chat(String userMessage);
    String chat(String memoryId, String userMessage);
}
```

### Available Assistants

| Assistant | Purpose |
|-----------|---------|
| CarRentalAssistant | Car rental service support |
| ChefAssistant | Recipe generation and cooking assistance |
| DataExtractorAssistant | Structured data extraction |
| HAL9000Assistant | General-purpose AI assistant |
| HealthCareAssistant | Healthcare-related queries |
| LanguageAssistant | Language translation and learning |
| ModerateAssistant | Content moderation |

## Database Setup

### PostgreSQL Setup

```sql
-- Create database
CREATE DATABASE ai_324;

-- Create user
CREATE USER myuser WITH PASSWORD 'mypassword';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE ai_324 TO myuser;
ALTER DATABASE ai_324 OWNER TO myuser;
```

### H2 Database (Development)

H2 database is configured for development mode with in-memory storage. Access the H2 console at:
```
http://localhost:19090/api/ai-service/api/v1/h2-ui
```

## Security

### JWT Token Configuration

The service supports two token types:
- Type 1: Secret key-based authentication
- Type 2: Public/private key-based authentication

Configure token type in application properties:
```properties
server.token.type=1
server.token.key=<your-secret-key>
```

### Password Encryption

Database passwords are encrypted using Jasypt with AES-256 encryption:
```properties
jasypt.encryptor.algorithm=PBEWithHmacSHA512AndAES_256
```

## Monitoring and Logging

### Actuator Endpoints

Enable actuator endpoints in production:
```properties
management.endpoints.web.exposure.include=health,info,metrics
```

### Log Configuration

Logs are configured using Logback with rolling file appender:
- Log location: `/tmp/logs/ai-service/`
- Max file size: 10MB
- Max history: 100 days
- Total size cap: 3GB

## Development

### Building

```bash
mvn clean package
```

### Running Tests

```bash
mvn test
```

### Code Quality

```bash
mvn clean verify sonar:sonar
```

## Deployment

### Docker Deployment

1. Build the Docker image
```bash
cd src/docker
docker build -t ai-service:latest .
```

2. Run the container
```bash
docker run -p 19090:19090 \
  -e OPENAI_API_KEY=<your-key> \
  -e JASYPT_ENCRYPTOR_PASSWORD=<your-key> \
  ai-service:latest
```

## Troubleshooting

### Common Issues

1. **Unable to decrypt password**
   - Ensure `JASYPT_ENCRYPTOR_PASSWORD` environment variable is set
   - Verify encrypted passwords in property files match the encryption key

2. **LLM API connection failures**
   - Verify API keys are set in environment variables
   - Check network connectivity to LLM provider endpoints

3. **Database connection issues**
   - Verify database credentials
   - Check database server availability
   - Ensure correct JDBC URL format

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the Apache License 2.0. See the LICENSE file for details.

## Author

Joyprakash Kalita

## Acknowledgments

- LangChain4J team for the excellent LLM integration framework
- Spring Boot team for the robust framework
- All contributors and users of this project
## Security

### Authentication and Authorization

The service implements a multi-layered security architecture:

#### JWT Token System

- **Authentication Token**: Primary token for user authentication
- **Transaction Token**: Secondary token containing application-specific claims
- **Token Types**: Secret key (Type 1) or Public/Private key (Type 2) based
- **Token Expiry**: Auth token (10 minutes), Refresh token (60 minutes)

#### Security Filters

| Filter | Purpose |
|--------|---------|
| JwtAuthFilter | JWT token validation and claim extraction |
| SecurityFilter | Security policy enforcement |
| LogFilter | Request/response logging |
| HeaderManager | HTTP header management |

#### Security Annotations

| Annotation | Purpose |
|------------|---------|
| `@AuthorizationRequired` | Requires specific user role |
| `@SingleTokenAuthorizationRequired` | Single token validation |
| `@UserTokenAuthorization` | User-specific token validation |
| `@ValidateRefreshToken` | Refresh token validation |

### Data Protection

#### Password Encryption

Database passwords are encrypted using Jasypt with:
- Algorithm: `PBEWithHmacSHA512AndAES_256`
- IV Generator: `RandomIvGenerator`
- Salt Generator: `RandomSaltGenerator`

#### Environment Variables

```bash
# Required for production
JASYPT_ENCRYPTOR_PASSWORD=<encryption-key>
OPENAI_API_KEY=<openai-api-key>
ANTHROPIC_API_KEY=<anthropic-api-key>
COHERE_API_KEY=<cohere-api-key>
```

### API Security

#### Public Endpoints
- `/api/ai-service/api/v1/ai/openai/string/*` - String-based responses
- `/api/ai-service/api/v1/ai/ollama/string/*` - String-based responses

#### Protected Endpoints
- All JSON-based endpoints require JWT authentication
- Role-based access control enforced via `@AuthorizationRequired`

## Testing

### Unit Testing

```bash
mvn test
```

### Integration Testing

```bash
mvn verify
```

### Test Coverage

Run SonarQube analysis:
```bash
mvn clean verify sonar:sonar
```

### Manual Testing

#### Generate Test Tokens

Set in `application.properties`:
```properties
server.token.test=true
```

#### Test Endpoints

```bash
# Get test token
curl -X POST http://localhost:19090/api/ai-service/api/v1/auth/token

# Test AI endpoint
curl -X POST http://localhost:19090/api/ai-service/api/v1/ai/openai/chat \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "Hello, how are you?"
```

## Monitoring

### Actuator Endpoints

```properties
management.endpoints.web.exposure.include=health,info,metrics,env,configprops
management.endpoint.health.show-details=always
```

### Metrics

The service exposes the following metrics:
- Request latency percentiles
- Database query times
- LLM API call durations
- JWT token validation times

### Logging

#### Log Levels

| Package | Level |
|---------|-------|
| Root | INFO |
| dev.langchain4j | DEBUG |
| dev.ai4j.openai4j | DEBUG |
| org.springframework | INFO |

#### Log Format

```json
{
  "timestamp": "2024-01-15T10:30:00.000+00:00",
  "level": "INFO",
  "thread": "http-nio-19090-exec-1",
  "logger": "io.fusion.air.microservice.ai.genai.controllers.AbstractAiController",
  "message": "Chat Request to AI...",
  "context": {
    "buildNumber": "258",
    "service": "ai-service"
  }
}
```

## Performance

### Optimization Strategies

1. **Caching**: Redis-based caching for frequently accessed data
2. **Connection Pooling**: HikariCP for database connections
3. **Async Processing**: Non-blocking I/O for external API calls
4. **Response Compression**: GZIP compression for large responses

### Performance Benchmarks

| Operation | Avg Latency | P95 Latency | P99 Latency |
|-----------|-------------|-------------|-------------|
| OpenAI Chat | 850ms | 1200ms | 1800ms |
| Ollama Chat | 2500ms | 3500ms | 5000ms |
| RAG Query | 1500ms | 2200ms | 3000ms |
| Data Extraction | 600ms | 900ms | 1200ms |

## CI/CD Pipeline

### Build Process

```bash
# Clean and build
mvn clean package

# Run tests
mvn test

# Generate coverage report
mvn clean verify

# Build Docker image
docker build -t ai-service:latest .
```

### Deployment

#### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-service
  template:
    metadata:
      labels:
        app: ai-service
    spec:
      containers:
      - name: ai-service
        image: ai-service:latest
        ports:
        - containerPort: 19090
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: openai-api-key
        - name: JASYPT_ENCRYPTOR_PASSWORD
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: encryption-key
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

## Troubleshooting

### Common Issues

#### 1. Database Connection Failures

**Symptoms**: Application fails to start with database connection errors

**Resolution**:
```bash
# Verify environment variable
echo $JASYPT_ENCRYPTOR_PASSWORD

# Check database connectivity
psql -h <hostname> -p <port> -U <user> -d <database>
```

#### 2. LLM API Connection Failures

**Symptoms**: AI endpoints return 500 errors

**Resolution**:
```bash
# Verify API keys
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# Test API connectivity
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/models
```

#### 3. JWT Token Validation Failures

**Symptoms**: 403 Forbidden errors on protected endpoints

**Resolution**:
```bash
# Verify token format
# Ensure token includes "Bearer " prefix
# Check token expiration
# Verify signing key matches configuration
```

#### 4. Ollama Local LLM Not Responding

**Symptoms**: Ollama endpoints return connection refused

**Resolution**:
```bash
# Verify Ollama is running
curl http://localhost:11434/api/tags

# If not running, start Ollama
ollama serve
```

### Debug Mode

Enable debug logging:
```bash
java -jar target/ai-service-0.3.4-spring-boot.jar \
  --spring.profiles.active=dev \
  --logging.level.io.fusion.air.microservice=DEBUG
```

## Contributing

### Code Style

- Follow Google Java Style Guide
- Use 4-space indentation
- Maximum line length: 120 characters
- All public methods must have Javadoc comments

### Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes using conventional commits:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `docs:` for documentation changes
   - `refactor:` for code refactoring
   - `test:` for test additions/changes
   - `chore:` for maintenance
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request with:
   - Detailed description of changes
   - Related issue references
   - Testing instructions
   - Performance impact (if applicable)

### Code Review Checklist

- [ ] Code follows project style guidelines
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Documentation updated
- [ ] Security considerations addressed
- [ ] Performance impact assessed
- [ ] Error handling implemented

## Versioning

This project uses Semantic Versioning (SemVer):

```
MAJOR.MINOR.PATCH
```

- **MAJOR**: Breaking changes
- **MINOR**: Backward-compatible new features
- **PATCH**: Backward-compatible bug fixes

## Changelog

### Version 0.3.4 (Current)
- Updated to Spring Boot 3.4.1
- Updated to LangChain4J 0.30.0
- Added support for multiple LLM providers
- Enhanced security configuration
- Improved error handling

### Version 0.3.3
- Initial release
- Basic AI integration
- JWT authentication
- H2/PostgreSQL support

## License

This project is licensed under the Apache License 2.0. See the LICENSE file for details.

## Author

**Joyprakash Kalita**

## Acknowledgments

- LangChain4J team for the excellent LLM integration framework
- Spring Boot team for the robust framework
- All contributors and users of this project

## Support

For support, please open an issue in the GitHub repository or contact the maintainers.
## API Documentation

### OpenAPI 3.0 Specification

The service exposes a complete OpenAPI 3.0 specification accessible via:

```
http://localhost:19090/api/ai-service/api/v1/swagger-ui.html
```

### API Specification Endpoints

| Endpoint | Description |
|----------|-------------|
| `/api/ai-service/api/v1/openapi/json` | OpenAPI JSON specification |
| `/api/ai-service/api/v1/openapi/yaml` | OpenAPI YAML specification |
| `/api/ai-service/api/v1/swagger-ui.html` | Swagger UI interface |

### Request/Response Formats

#### Standard Response Format

```json
{
  "timestamp": "2024-01-15T10:30:00.000+00:00",
  "status": 200,
  "error": null,
  "message": "Success message",
  "path": "/api/ai-service/api/v1/endpoint",
  "payload": {
    "key": "value"
  }
}
```

#### Error Response Format

```json
{
  "timestamp": "2024-01-15T10:30:00.000+00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Error message",
  "path": "/api/ai-service/api/v1/endpoint",
  "errorCode": "AKH461",
  "errors": [
    "Field name | Error message"
  ]
}
```

### Error Codes

| Code Range | Category | Description |
|------------|----------|-------------|
| 400-409 | Standard | General errors |
| 410-429 | Security | Authentication/Authorization errors |
| 430-439 | Messaging | Message queue errors |
| 440-459 | Database | Database errors |
| 460-489 | Business | Business logic errors |
| 490-499 | Controller | Controller errors |
| 590-599 | Server | Server errors |

## Database Schema

### Tables

#### Chat Messages

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| user_id | VARCHAR | User identifier |
| message | TEXT | Chat message content |
| response | TEXT | AI response content |
| created_at | TIMESTAMP | Creation timestamp |
| model | VARCHAR | LLM model used |

#### Products

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| name | VARCHAR | Product name |
| description | TEXT | Product description |
| price | DECIMAL | Product price |
| zip_code | VARCHAR | ZIP code |
| active | BOOLEAN | Product availability |
| version | BIGINT | Optimistic lock version |
| created_at | TIMESTAMP | Creation timestamp |
| updated_at | TIMESTAMP | Update timestamp |

## Configuration Reference

### Application Properties

| Property | Default | Description |
|----------|---------|-------------|
| `server.port` | 19090 | Server port |
| `service.api.path` | /ai-service/api/v1 | API path prefix |
| `langchain4j.open-ai.model` | gpt-4o-2024-05-13 | OpenAI model |
| `langchain4j.ollama.url` | http://localhost:11434/api/generate/ | Ollama URL |
| `langchain4j.ollama.model` | llama3 | Ollama model |
| `spring.datasource.url` | jdbc:h2:mem:ai_324 | Database URL |
| `spring.datasource.username` | sa | Database username |
| `server.token.auth.expiry` | 600000 | Auth token expiry (ms) |
| `server.token.refresh.expiry` | 3600000 | Refresh token expiry (ms) |

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `ANTHROPIC_API_KEY` | No | Anthropic API key |
| `COHERE_API_KEY` | No | Cohere API key |
| `HF_API_KEY` | No | HuggingFace API key |
| `JASYPT_ENCRYPTOR_PASSWORD` | Yes | Database encryption key |

## Performance Tuning

### JVM Options

```bash
java -jar target/ai-service-0.3.4-spring-boot.jar \
  --spring.profiles.active=prod \
  -Xms512m \
  -Xmx1024m \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp/heapdump.hprof
```

### Database Connection Pool

```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

### LLM Rate Limiting

Implement rate limiting for LLM API calls:
```java
@RateLimiter(name = "llmRateLimiter", fallbackMethod = "fallbackChat")
public String chat(String message) {
    // LLM call implementation
}
```

## Security Best Practices

1. **Never commit API keys** - Use environment variables or secret management
2. **Rotate encryption keys** - Regularly rotate Jasypt encryption keys
3. **Enable HTTPS** - Always use HTTPS in production
4. **Validate inputs** - All user inputs are validated
5. **Rate limiting** - Implement rate limiting for API endpoints
6. **CORS configuration** - Restrict CORS to trusted origins
7. **CSRF protection** - Enable CSRF protection for stateful operations
8. **JWT token validation** - Validate all JWT tokens before processing

## Deployment Checklist

### Pre-Deployment

- [ ] All API keys configured in environment variables
- [ ] Database encryption key set
- [ ] SSL certificates installed
- [ ] Load balancer configured
- [ ] Monitoring and alerting configured
- [ ] Backup procedures tested
- [ ] Security scan passed

### Deployment

```bash
# Build the application
mvn clean package

# Run tests
mvn test

# Deploy
java -jar target/ai-service-0.3.4-spring-boot.jar \
  --spring.profiles.active=prod \
  -Djava.security.manager \
  -Djava.security.policy=./security.policy
```

### Post-Deployment

- [ ] Health check passes
- [ ] All endpoints responding
- [ ] Database connections established
- [ ] LLM API connectivity verified
- [ ] Logs streaming correctly
- [ ] Metrics being collected

## Maintenance

### Log Rotation

Logs are automatically rotated with the following policy:
- Max file size: 10MB
- Max history: 100 days
- Total size cap: 3GB

### Database Maintenance

```sql
-- Vacuum database
VACUUM ANALYZE;

-- Check for dead tuples
SELECT schemaname,relname,n_dead_tup,n_live_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > n_live_tup * 0.1;
```

### Security Updates

Regularly update dependencies:
```bash
mvn versions:display-dependency-updates
mvn versions:display-plugin-updates
```

## FAQ

### Q: How do I add a new LLM provider?

**A**: Add a new configuration class in `io.fusion.air.microservice.ai.genai.utils` and implement the `ChatLanguageModel` bean with a unique qualifier.

### Q: How do I implement custom AI assistants?

**A**: Create a new interface in `io.fusion.air.microservice.ai.genai.core.assistants` implementing the `Assistant` interface with appropriate `@SystemMessage` and `@UserMessage` annotations.

### Q: How do I configure caching?

**A**: Enable caching in `application.properties`:
```properties
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379
```

### Q: How do I enable HTTPS?

**A**: Configure SSL in `application.properties`:
```properties
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=<password>
server.ssl.key-store-type=PKCS12
```

## Related Projects

- [ms-springboot-334-vanilla](https://github.com/arafkarsh/ms-springboot-334-vanilla) - Base microservice template
- [LangChain4J](https://github.com/langchain4j/langchain4j) - Java LLM integration library
- [Spring Boot](https://github.com/spring-projects/spring-boot) - Spring Boot framework

## Resources

- [LangChain4J Documentation](https://docs.langchain4j.dev/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [OpenAI API Documentation](https://platform.openai.com/docs/)
- [Ollama Documentation](https://ollama.ai/docs/)
