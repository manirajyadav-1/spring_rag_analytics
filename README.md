#  Spring AI RAG Demo

A **Spring Boot** demo showcasing **Retrieval-Augmented Generation (RAG)** using **Spring AI** and **OpenAI GPT models**. This project enables intelligent querying of local PDF documents by combining the power of **LLMs** with semantic search.

---

##  Features

* Ingest and vectorize PDF documents
* Perform semantic search with Spring AI
* Use GPT models with document context (RAG)
* Expose an API endpoint for intelligent querying

---

##  Requirements

* **Java 23**
* **Maven**
* **Docker Desktop** (for PostgreSQL with PGVector)
* **OpenAI API Key**

---

##  Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pdf-document-reader</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

---

##  Getting Started

### 1. Configure Environment Variables

```bash
OPENAI_API_KEY=your_api_key_here
```

### 2. Update `application.properties`

```properties
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.model=gpt-4
spring.ai.vectorstore.pgvector.initialize-schema=true
```

### 3. Add PDF Documents

Place your documents in:

```
src/main/resources/docs/
```

---

##  Run the Application

1. Start Docker Desktop (for PostgreSQL + PGVector).
2. Launch the Spring Boot app:

```bash
./mvnw spring-boot:run
```

---

##  Key Components

###  IngestionService

Handles PDF ingestion and vector store population:

```java
@Component
public class IngestionService implements CommandLineRunner {

    private final VectorStore vectorStore;

    @Value("classpath:/docs/your-document.pdf")
    private Resource marketPDF;

    @Override
    public void run(String... args) {
        var pdfReader = new ParagraphPdfDocumentReader(marketPDF);
        TextSplitter textSplitter = new TokenTextSplitter();
        vectorStore.accept(textSplitter.apply(pdfReader.get()));
    }
}
```

---

###  ChatController

REST controller for document-aware queries:

```java
@RestController
public class ChatController {

    private final ChatClient chatClient;

    public ChatController(ChatClient.Builder builder, VectorStore vectorStore) {
        this.chatClient = builder
                .defaultAdvisors(new QuestionAnswerAdvisor(vectorStore))
                .build();
    }

    @GetMapping("/")
    public String chat() {
        return chatClient.prompt()
                .user("Your question here")
                .call()
                .content();
    }
}
```

---

##  Making Requests

Send a query:

```bash
curl http://localhost:8080/
```

Response will include GPT-generated content enhanced with document context.

---

##  Architecture

| Component          | Description                                          |
| ------------------ | ---------------------------------------------------- |
| Document Ingestion | Reads & splits PDF documents                         |
| Vector Store       | Stores embeddings using PGVector                     |
| Semantic Search    | Retrieves relevant document chunks                   |
| GPT Integration    | Generates responses with embedded document knowledge |

---

##  Best Practices

###  Document Ingestion

* Avoid reinitializing vector store unnecessarily
* Schedule periodic ingestion for updated documents

###  Query Optimization

* Monitor API token usage
* Cache frequent queries
* Rate-limit endpoint to avoid misuse

###  Security

* Secure API endpoints with authentication
* Encrypt and protect document content
* Store API keys securely (avoid hardcoding)

---

