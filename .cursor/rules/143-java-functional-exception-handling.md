---
author: Juan Antonio Breña Moral
version: 0.10.0
---
# Java Exceptions Best Practices

## Role

You are a Senior software engineer with extensive experience in Java software development

## Goal

Modern Java error handling emphasizes functional programming approaches over traditional exception-based patterns.
Prefer monads like Optional<T> for nullable values and VAVR's Either<L,R> for error handling in predictable failure scenarios.
Reserve exceptions only for truly exceptional circumstances like system failures and programming errors.
Design comprehensive error types with clear codes, messages, and context for business logic failures.
Implement structured logging and monitoring that works effectively with functional error handling patterns.
Use monadic composition to create maintainable, testable, and performant error handling flows.

**Required Dependencies:**
- VAVR library (io.vavr:vavr) for Either<L,R> and other functional programming constructs
- SLF4J for structured logging compatible with functional error handling

### Consultative Interaction Technique

This technique emphasizes **analyzing before acting** and **proposing options before implementing**. Instead of immediately making changes, the assistant:

1. **Analyzes** the current state and identifies specific issues
2. **Categorizes** problems by impact (CRITICAL, MAINTAINABILITY, etc.)
3. **Proposes** multiple solution options with clear trade-offs
4. **Asks** the user to choose their preferred approach
5. **Implements** based on user selection

**Benefits:**
- Builds user understanding of the codebase
- Ensures changes align with user preferences and constraints
- Teaches best practices through explanation
- Prevents unwanted modifications
- Encourages informed decision-making

**Example interaction:**
```
🔍 I found 3 Maven best practices improvements in this POM:

1. **CRITICAL: Hardcoded Dependency Versions**
- Problem: Dependencies have hardcoded versions scattered throughout the POM
- Solutions: A) Move to properties section B) Use dependencyManagement C) Import BOM files

2. **MAINTAINABILITY: Missing Plugin Version Management**
- Problem: Maven plugins lack explicit version declarations
- Solutions: A) Add pluginManagement section B) Define plugin versions in properties C) Use parent POM approach

3. **ORGANIZATION: Inconsistent POM Structure**
- Problem: Elements are not in logical order, affecting readability
- Solutions: A) Reorganize sections B) Add descriptive comments C) Use consistent naming conventions

Which would you like to implement? (1A, 1B, 1C, 2A, 2B, 2C, 3A, 3B, 3C, or 'show more details')
```

Focus on being consultative rather than prescriptive - analyze, propose, ask, then implement based on user choice.

### Implementing These Principles

These guidelines are built upon the following core principles:

1. **Minimize Exception Usage**: Reserve exceptions for truly exceptional circumstances. Use monads like `Optional<T>` for nullable values, VAVR's `Either<L,R>` for error handling, and `CompletableFuture<T>` for asynchronous operations instead of throwing exceptions for predictable failure cases.
2. **Favor Functional Error Handling**: Prefer returning functional types that encapsulate success/failure states (e.g., `Optional.empty()`, `Either.left(error)`, `Either.right(value)`) over throwing exceptions for business logic errors or validation failures.
3. **Use Specific Exception Types When Necessary**: When exceptions are unavoidable, differentiate between different types of errors by using specific exception classes (e.g., `IOException` for I/O errors, `SQLException` for database errors, `IllegalArgumentException` for invalid method arguments).
4. **Checked vs. Unchecked Exceptions**: Use checked exceptions sparingly for truly recoverable errors that callers must handle, and unchecked exceptions for programming errors that indicate bugs in the code.
5. **Exception Naming Conventions**: Follow a consistent naming convention for exception classes when they are necessary (e.g., `ResourceNotFoundException`, `InvalidInputException`).

## Constraints

Before applying these guidelines, ensure the project is in a valid state by running Maven validation. This helps identify any existing configuration issues that need to be resolved first.

- **MANDATORY**: Run `./mvnw validate` or `mvn validate` before applying any Maven best practices recommendations
- **VERIFY**: Ensure all validation errors are resolved before proceeding with POM modifications

## Examples

### Table of contents

- Example 1: Functional Error Handling with Monads
- Example 2: When to Use Exceptions vs. Functional Error Handling
- Example 3: Designing Error Types for Functional Error Handling
- Example 4: Composable Error Handling with Monads
- Example 5: Logging and Monitoring with Functional Error Handling
- Example 6: Clear Error Communication in Functional Design
- Example 7: Functional Control Flow Patterns

### Example 1: Functional Error Handling with Monads

Title: Use Optional and VAVR Either types instead of exceptions for predictable failures
Description: Prefer functional error handling using monads like Optional for nullable values and VAVR's Either<L,R> for operations that can fail. Reserve exceptions only for truly exceptional circumstances like system failures or programming errors.

**Good example:**

```java
// GOOD: Using functional error handling with monads
import io.vavr.control.Either;
import io.vavr.control.Try;
import java.util.Objects;
import java.util.Optional;
import java.util.concurrent.CompletableFuture;

public class UserService {

    // Use Optional for nullable values
    public Optional<User> findUserById(Long id) {
        if (Objects.isNull(id) || id <= 0) {
            return Optional.empty();
        }

        return userRepository.findById(id);
    }

    // Use Either type for operations that can fail
    public Either<TransferError, TransferSuccess> transferMoney(
            Account from, Account to, BigDecimal amount) {

        // Validate inputs
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            return Either.left(new TransferError("INVALID_AMOUNT",
                "Transfer amount must be positive"));
        }

        if (Objects.isNull(from)) {
            return Either.left(new TransferError("ACCOUNT_NOT_FOUND",
                "Source account not found"));
        }

        if (Objects.isNull(to)) {
            return Either.left(new TransferError("ACCOUNT_NOT_FOUND",
                "Destination account not found"));
        }

        if (from.getBalance().compareTo(amount) < 0) {
            return Either.left(new TransferError("INSUFFICIENT_FUNDS",
                String.format("Insufficient funds: required %s, available %s",
                    amount, from.getBalance())));
        }

        // Perform transfer
        TransferSuccess success = performTransfer(from, to, amount);
        return Either.right(success);
    }

    // Use CompletableFuture for asynchronous operations
    public CompletableFuture<Optional<User>> findUserByEmailAsync(String email) {
        return CompletableFuture.supplyAsync(() -> {
            if (Objects.isNull(email) || email.trim().isEmpty()) {
                return Optional.empty();
            }
            return userRepository.findByEmail(email);
        });
    }
}

// VAVR Either usage examples
public class EitherExamples {

    // Handling Either results
    public void handleTransferResult(Either<TransferError, TransferSuccess> result) {
        result
            .peek(success -> logger.info("Transfer completed: {}", success.getTransactionId()))
            .peekLeft(error -> logger.warn("Transfer failed: {} - {}", error.getCode(), error.getMessage()));
    }

    // Chaining Either operations
    public Either<String, Integer> parseAndDouble(String input) {
        return Try.of(() -> Integer.parseInt(input))
            .toEither()
            .mapLeft(throwable -> "Invalid number: " + input)
            .map(number -> number * 2);
    }

    // Combining multiple Either results
    public Either<String, String> combineResults(Either<String, String> first, Either<String, String> second) {
        return first.flatMap(firstValue ->
            second.map(secondValue -> firstValue + " " + secondValue));
    }
}
```

**Bad example:**

```java
// AVOID: Overusing exceptions for predictable failures
public class UserService {

    // Bad: Using exceptions for normal "not found" cases
    public User findUserById(Long id) throws UserNotFoundException {
        if (Objects.isNull(id)) {
            throw new IllegalArgumentException("User ID cannot be null");
        }
        if (id <= 0) {
            throw new IllegalArgumentException("User ID must be positive");
        }

        User user = userRepository.findById(id);
        if (Objects.isNull(user)) {
            throw new UserNotFoundException("User with ID " + id + " not found");
        }
        return user;
    }

    // Bad: Multiple checked exceptions for business logic failures
    public void transferMoney(Account from, Account to, BigDecimal amount)
            throws InsufficientFundsException, AccountNotFoundException,
                   InvalidAmountException {

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException("Transfer amount must be positive");
        }

        if (Objects.isNull(from)) {
            throw new AccountNotFoundException("Source account not found");
        }

        if (Objects.isNull(to)) {
            throw new AccountNotFoundException("Destination account not found");
        }

        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(
                "Insufficient funds: required " + amount + ", available " + from.getBalance());
        }

        // Perform transfer
    }

    // Bad: Using exceptions in asynchronous operations
    public CompletableFuture<User> findUserByEmailAsync(String email) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                if (Objects.isNull(email) || email.trim().isEmpty()) {
                    throw new IllegalArgumentException("Email cannot be null or empty");
                }
                User user = userRepository.findByEmail(email);
                if (Objects.isNull(user)) {
                    throw new UserNotFoundException("User not found with email: " + email);
                }
                return user;
            } catch (Exception e) {
                throw new RuntimeException(e); // Wrapping exceptions in async context
            }
        });
    }
}
```

### Example 2: When to Use Exceptions vs. Functional Error Handling

Title: Reserve exceptions for truly exceptional circumstances, use functional approaches for business logic
Description: Use exceptions only for truly exceptional circumstances like system failures, programming errors, or unrecoverable conditions. For business logic failures, validation errors, and recoverable conditions, prefer functional error handling with Optional or Either types.

**Good example:**

```java
// GOOD: Appropriate separation of concerns

public class DocumentProcessor {

    // Use Optional for "not found" scenarios
    public Optional<Document> findDocument(String documentId) {
        return documentRepository.findById(documentId);
    }

        // Use Either type for business logic that can fail
    public Either<ProcessingError, Document> processDocument(Path filePath) {
        if (!Files.exists(filePath)) {
            return Either.left(new ProcessingError("FILE_NOT_FOUND",
                "Document file does not exist: " + filePath));
        }

        if (!Files.isReadable(filePath)) {
            return Either.left(new ProcessingError("ACCESS_DENIED",
                "Cannot read document file: " + filePath));
        }

        try {
            String content = Files.readString(filePath);
            Document document = parseContent(content);
            return Either.right(document);
        } catch (IOException e) {
            // I/O exceptions are truly exceptional - system level issues
            throw new DocumentProcessingException("System error reading file: " + filePath, e);
        } catch (OutOfMemoryError e) {
            // System errors should remain as exceptions
            throw new DocumentProcessingException("Insufficient memory to process file: " + filePath, e);
        }
    }

    // Use validation without exceptions
    public ValidationResult validateDocumentContent(String content) {
        List<String> errors = new ArrayList<>();

        if (Objects.isNull(content) || content.trim().isEmpty()) {
            errors.add("Content cannot be empty");
        }

        if (content.length() > MAX_CONTENT_LENGTH) {
            errors.add("Content exceeds maximum length of " + MAX_CONTENT_LENGTH);
        }

        return errors.isEmpty() ?
            ValidationResult.valid() :
            ValidationResult.invalid(errors);
    }

    // Programming errors should still use unchecked exceptions
    public void setMaxRetries(int maxRetries) {
        if (maxRetries < 0) {
            throw new IllegalArgumentException("Max retries cannot be negative: " + maxRetries);
        }
        this.maxRetries = maxRetries;
    }

    // System configuration errors use unchecked exceptions
    private void validateConfiguration() {
        if (Objects.isNull(apiKey) || apiKey.isEmpty()) {
            throw new IllegalStateException("API key must be configured before processing");
        }
    }
}

public static class ProcessingError {
    private final String code;
    private final String message;

    public ProcessingError(String code, String message) {
        this.code = code;
        this.message = message;
    }

    // getters...
}
```

**Bad example:**

```java
// AVOID: Using exceptions for business logic and normal failure cases

public class DocumentProcessor {

    // Bad: Using exceptions for normal "not found" cases
    public Document findDocument(String documentId) throws DocumentNotFoundException {
        Document doc = documentRepository.findById(documentId);
        if (Objects.isNull(doc)) {
            throw new DocumentNotFoundException("Document not found: " + documentId);
        }
        return doc;
    }

    // Bad: Multiple checked exceptions for predictable business failures
    public Document processDocument(Path filePath)
            throws FileNotFoundException, AccessDeniedException, InvalidContentException,
                   FileTooLargeException, DocumentProcessingException {

        if (!Files.exists(filePath)) {
            throw new FileNotFoundException("Document file does not exist: " + filePath);
        }

        if (!Files.isReadable(filePath)) {
            throw new AccessDeniedException("Cannot read document file: " + filePath);
        }

        try {
            String content = Files.readString(filePath);

            if (content.trim().isEmpty()) {
                throw new InvalidContentException("Document content cannot be empty");
            }

            if (content.length() > MAX_CONTENT_LENGTH) {
                throw new FileTooLargeException("Document exceeds maximum size");
            }

            return parseContent(content);
        } catch (IOException e) {
            throw new DocumentProcessingException("Failed to process document", e);
        }
    }

    // Bad: Using checked exceptions for validation
    public void validateDocumentContent(String content)
            throws EmptyContentException, ContentTooLongException {
        if (Objects.isNull(content) || content.trim().isEmpty()) {
            throw new EmptyContentException("Content cannot be empty");
        }

        if (content.length() > MAX_CONTENT_LENGTH) {
            throw new ContentTooLongException("Content exceeds maximum length");
        }
    }

    // Bad: Using checked exception for programming error
    public void setMaxRetries(int maxRetries) throws InvalidArgumentException {
        if (maxRetries < 0) {
            throw new InvalidArgumentException("Max retries cannot be negative");
        }
        this.maxRetries = maxRetries;
    }
}
```

### Example 3: Designing Error Types for Functional Error Handling

Title: Create well-structured error types for Either patterns and minimal exception design
Description: Design comprehensive error types for functional error handling using the appropriate approach based on complexity: - Use enums for simple error cases that don't require additional data or complex context - Use sealed classes and records for complex error hierarchies that need pattern matching and rich data - Use builder patterns for errors requiring extensive contextual information When exceptions are necessary, follow clear naming conventions and include rich contextual information.

**Good example:**

```java
// GOOD: Modern error type design with sealed classes and pattern matching

// Enum-based error types for simple business errors (when no additional data is needed)
public enum SimpleValidationError {
    REQUIRED_FIELD_MISSING("Required field is missing"),
    INVALID_EMAIL_FORMAT("Email format is invalid"),
    PASSWORD_TOO_SHORT("Password must be at least 8 characters"),
    USERNAME_ALREADY_EXISTS("Username already exists"),
    INVALID_PHONE_NUMBER("Phone number format is invalid");

    private final String description;

    SimpleValidationError(String description) {
        this.description = description;
    }

    public String getDescription() { return description; }

    // Simple pattern matching with enums
    public String getHelpText() {
        return switch (this) {
            case REQUIRED_FIELD_MISSING -> "Please fill in all required fields marked with *";
            case INVALID_EMAIL_FORMAT -> "Please enter a valid email address (e.g., user@example.com)";
            case PASSWORD_TOO_SHORT -> "Password should contain at least 8 characters";
            case USERNAME_ALREADY_EXISTS -> "Please choose a different username";
            case INVALID_PHONE_NUMBER -> "Please enter phone number in format: +1-555-123-4567";
        };
    }

    public ErrorSeverity getSeverity() {
        return switch (this) {
            case REQUIRED_FIELD_MISSING, INVALID_EMAIL_FORMAT, INVALID_PHONE_NUMBER -> ErrorSeverity.HIGH;
            case PASSWORD_TOO_SHORT -> ErrorSeverity.MEDIUM;
            case USERNAME_ALREADY_EXISTS -> ErrorSeverity.LOW;
        };
    }

    public enum ErrorSeverity { LOW, MEDIUM, HIGH }
}

// Enum for HTTP-style status errors
public enum ApiError {
    UNAUTHORIZED(401, "Authentication required"),
    FORBIDDEN(403, "Access denied"),
    NOT_FOUND(404, "Resource not found"),
    RATE_LIMITED(429, "Too many requests"),
    SERVER_ERROR(500, "Internal server error");

    private final int code;
    private final String message;

    ApiError(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() { return code; }
    public String getMessage() { return message; }

    // Pattern matching for different response strategies
    public boolean isRetryable() {
        return switch (this) {
            case RATE_LIMITED, SERVER_ERROR -> true;
            case UNAUTHORIZED, FORBIDDEN, NOT_FOUND -> false;
        };
    }
}

// Usage examples for enum-based error handling
public class SimpleErrorHandling {

    // Using enums with Either for simple validation
    public Either<SimpleValidationError, User> validateUser(UserRequest request) {
        if (Objects.isNull(request.email()) || request.email().trim().isEmpty()) {
            return Either.left(SimpleValidationError.REQUIRED_FIELD_MISSING);
        }

        if (!isValidEmail(request.email())) {
            return Either.left(SimpleValidationError.INVALID_EMAIL_FORMAT);
        }

        if (request.password().length() < 8) {
            return Either.left(SimpleValidationError.PASSWORD_TOO_SHORT);
        }

        if (userExists(request.username())) {
            return Either.left(SimpleValidationError.USERNAME_ALREADY_EXISTS);
        }

        return Either.right(new User(request.username(), request.email()));
    }

    // Simple error formatting with enums
    public String formatValidationError(SimpleValidationError error) {
        return "❌ " + error.getDescription() + "\n💡 " + error.getHelpText();
    }
}

// Sealed class for domain-specific error hierarchies with exhaustive pattern matching
public sealed class PaymentError
    permits PaymentError.InsufficientFunds,
            PaymentError.AccountNotFound,
            PaymentError.InvalidAmount,
            PaymentError.TransferLimitExceeded {

    private final String message;
    private final String transactionId;
    private final Instant timestamp;

    protected PaymentError(String message, String transactionId) {
        this.message = message;
        this.transactionId = transactionId;
        this.timestamp = Instant.now();
    }

    public String getMessage() { return message; }
    public String getTransactionId() { return transactionId; }
    public Instant getTimestamp() { return timestamp; }

    public static final class InsufficientFunds extends PaymentError {
        private final BigDecimal availableAmount;
        private final BigDecimal requestedAmount;

        public InsufficientFunds(String transactionId, BigDecimal available, BigDecimal requested) {
            super(String.format("Insufficient funds: available %s, requested %s", available, requested),
                  transactionId);
            this.availableAmount = available;
            this.requestedAmount = requested;
        }

        public BigDecimal getAvailableAmount() { return availableAmount; }
        public BigDecimal getRequestedAmount() { return requestedAmount; }
    }

    public static final class AccountNotFound extends PaymentError {
        private final String accountId;

        public AccountNotFound(String transactionId, String accountId) {
            super("Account not found: " + accountId, transactionId);
            this.accountId = accountId;
        }

        public String getAccountId() { return accountId; }
    }

    public static final class InvalidAmount extends PaymentError {
        private final BigDecimal amount;

        public InvalidAmount(String transactionId, BigDecimal amount) {
            super("Invalid amount: " + amount, transactionId);
            this.amount = amount;
        }

        public BigDecimal getAmount() { return amount; }
    }

    public static final class TransferLimitExceeded extends PaymentError {
        private final BigDecimal limit;
        private final BigDecimal attempted;

        public TransferLimitExceeded(String transactionId, BigDecimal limit, BigDecimal attempted) {
            super(String.format("Transfer limit exceeded: limit %s, attempted %s", limit, attempted),
                  transactionId);
            this.limit = limit;
            this.attempted = attempted;
        }

        public BigDecimal getLimit() { return limit; }
        public BigDecimal getAttempted() { return attempted; }
    }
}

// Sealed class for validation errors with pattern matching
public sealed interface ValidationError
    permits ValidationError.RequiredFieldMissing,
            ValidationError.InvalidFormat,
            ValidationError.ValueOutOfRange,
            ValidationError.DuplicateValue {

    String getMessage();
    String getFieldName();

    record RequiredFieldMissing(String fieldName) implements ValidationError {
        @Override
        public String getMessage() {
            return "Required field is missing: " + fieldName;
        }

        @Override
        public String getFieldName() {
            return fieldName;
        }
    }

    record InvalidFormat(String fieldName, String expectedFormat) implements ValidationError {
        @Override
        public String getMessage() {
            return String.format("Invalid format for field '%s', expected: %s", fieldName, expectedFormat);
        }

        @Override
        public String getFieldName() {
            return fieldName;
        }
    }

    record ValueOutOfRange(String fieldName, Object value, Object min, Object max) implements ValidationError {
        @Override
        public String getMessage() {
            return String.format("Value '%s' for field '%s' is out of range [%s, %s]",
                value, fieldName, min, max);
        }

        @Override
        public String getFieldName() {
            return fieldName;
        }
    }

    record DuplicateValue(String fieldName, Object value) implements ValidationError {
        @Override
        public String getMessage() {
            return String.format("Duplicate value '%s' for field '%s'", value, fieldName);
        }

        @Override
        public String getFieldName() {
            return fieldName;
        }
    }
}

// Error handling with pattern matching
public class ErrorHandler {

    // Pattern matching for payment errors with exhaustive handling
    public String formatPaymentErrorMessage(PaymentError error) {
        return switch (error) {
            case PaymentError.InsufficientFunds funds ->
                String.format("Payment failed: Insufficient funds. Available: %s, Required: %s. " +
                    "Transaction ID: %s",
                    funds.getAvailableAmount(), funds.getRequestedAmount(), funds.getTransactionId());

            case PaymentError.AccountNotFound notFound ->
                String.format("Payment failed: Account '%s' not found. Transaction ID: %s",
                    notFound.getAccountId(), notFound.getTransactionId());

            case PaymentError.InvalidAmount invalid ->
                String.format("Payment failed: Invalid amount '%s'. Transaction ID: %s",
                    invalid.getAmount(), invalid.getTransactionId());

            case PaymentError.TransferLimitExceeded limitExceeded ->
                String.format("Payment failed: Transfer limit (%s) exceeded. Attempted: %s. Transaction ID: %s",
                    limitExceeded.getLimit(), limitExceeded.getAttempted(), limitExceeded.getTransactionId());
        };
    }

    // Pattern matching for validation errors
    public String formatValidationErrorMessage(ValidationError error) {
        return switch (error) {
            case ValidationError.RequiredFieldMissing missing ->
                "❌ Required field '" + missing.fieldName() + "' is missing";

            case ValidationError.InvalidFormat format ->
                "❌ Invalid format for '" + format.fieldName() +
                "', expected: " + format.expectedFormat();

            case ValidationError.ValueOutOfRange range ->
                "❌ Value '" + range.value() + "' for '" + range.fieldName() +
                "' must be between " + range.min() + " and " + range.max();

            case ValidationError.DuplicateValue duplicate ->
                "❌ Value '" + duplicate.value() + "' for '" + duplicate.fieldName() +
                "' already exists";
        };
    }

    // Determine error severity using pattern matching
    public ErrorSeverity getErrorSeverity(PaymentError error) {
        return switch (error) {
            case PaymentError.InsufficientFunds ignored -> ErrorSeverity.MEDIUM;
            case PaymentError.AccountNotFound ignored -> ErrorSeverity.HIGH;
            case PaymentError.InvalidAmount ignored -> ErrorSeverity.LOW;
            case PaymentError.TransferLimitExceeded ignored -> ErrorSeverity.MEDIUM;
        };
    }

    public enum ErrorSeverity { LOW, MEDIUM, HIGH }
}

// Rich error types with context for complex scenarios
public static class BusinessError {
    private final String code;
    private final String message;
    private final Map<String, Object> context;
    private final Instant timestamp;

    public BusinessError(String code, String message) {
        this(code, message, Collections.emptyMap());
    }

    public BusinessError(String code, String message, Map<String, Object> context) {
        this.code = code;
        this.message = message;
        this.context = Collections.unmodifiableMap(context);
        this.timestamp = Instant.now();
    }

    // Builder pattern for complex errors
    public static BusinessErrorBuilder builder(String code) {
        return new BusinessErrorBuilder(code);
    }

    public static class BusinessErrorBuilder {
        private final String code;
        private String message;
        private final Map<String, Object> context = new HashMap<>();

        public BusinessErrorBuilder(String code) {
            this.code = code;
        }

        public BusinessErrorBuilder message(String message) {
            this.message = message;
            return this;
        }

        public BusinessErrorBuilder context(String key, Object value) {
            this.context.put(key, value);
            return this;
        }

        public BusinessError build() {
            return new BusinessError(code, message, context);
        }
    }

    public String getCode() { return code; }
    public String getMessage() { return message; }
    public Map<String, Object> getContext() { return context; }
    public Instant getTimestamp() { return timestamp; }
}

// Exceptions only for system-level issues (rare cases)
public class SystemException extends RuntimeException {
    private final String errorCode;
    private final Map<String, Object> context;

    public SystemException(String errorCode, String message, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.context = new HashMap<>();
    }

    public SystemException withContext(String key, Object value) {
        this.context.put(key, value);
        return this;
    }

    public String getErrorCode() { return errorCode; }
    public Map<String, Object> getContext() { return context; }
}
```

**Bad example:**

```java
// AVOID: Poor error type design and excessive exceptions

// Bad: Using exceptions for business validation
public class RequiredFieldMissingException extends Exception {
    public RequiredFieldMissingException(String field) {
        super("Field missing: " + field);
    }
}

public class InvalidFormatException extends Exception {
    public InvalidFormatException(String field) {
        super("Bad format: " + field);
    }
}

public class ValueOutOfRangeException extends Exception {
    public ValueOutOfRangeException(String field) {
        super("Out of range: " + field);
    }
}

// Bad: Generic error types without context
public class BusinessError {
    private String message;

    public BusinessError(String message) {
        this.message = message;
    }
}

// Bad: Vague exception naming
public class BadThingHappened extends Exception {
    public BadThingHappened(String message) {
        super(message);
    }
}

// Bad: Inconsistent naming (doesn't end with Exception)
public class InvalidEmail extends RuntimeException {
    public InvalidEmail(String email) {
        super("Bad email: " + email);
    }
}

// Bad: Too generic exception types
public class ServiceError extends Exception {
    public ServiceError(String message) {
        super(message);
    }
}

// Bad: Unclear abbreviations
public class DBConnEx extends Exception {
    public DBConnEx(String msg) {
        super(msg);
    }
}

// Bad: No context about what failed
public class ProcessingFailed extends Exception {
    public ProcessingFailed() {
        super("Something went wrong");
    }
}

// Bad: Using many exceptions for normal business logic
public class PaymentService {
    public void processPayment(PaymentRequest request)
            throws InsufficientFundsException, AccountNotFoundException,
                   InvalidAmountException, PaymentLimitExceededException {
        // Multiple business logic exceptions
    }
}
```

### Example 4: Composable Error Handling with Monads

Title: Chain operations safely using flatMap, map, and other monadic operations
Description: Use monadic composition to chain operations that can fail without throwing exceptions. This approach makes error handling explicit and composable, leading to more maintainable code.

**Good example:**

```java
// GOOD: Composable error handling with monads

public class FileProcessor {
    private static final Logger logger = LoggerFactory.getLogger(FileProcessor.class);

    // Using Optional for chaining operations
    public Optional<ProcessedDocument> processFileChain(Path filePath) {
        return validateFile(filePath)
            .flatMap(this::readFileContent)
            .flatMap(this::parseContent)
            .flatMap(this::validateContent)
            .map(this::createDocument);
    }

    // Using Either type for more detailed error handling
    public Either<ProcessingError, ProcessedDocument> processFileWithEither(Path filePath) {
        return validateFileExists(filePath)
            .flatMap(this::readFileContentSafely)
            .flatMap(this::parseContentSafely)
            .flatMap(this::validateContentSafely)
            .map(this::createDocument);
    }

    // Composing multiple async operations
    public CompletableFuture<Optional<ProcessedDocument>> processFileAsync(Path filePath) {
        return CompletableFuture
            .supplyAsync(() -> validateFile(filePath))
            .thenCompose(result -> result
                .map(path -> readFileContentAsync(path))
                .orElse(CompletableFuture.completedFuture(Optional.empty())))
            .thenCompose(content -> content
                .map(this::parseContentAsync)
                .orElse(CompletableFuture.completedFuture(Optional.empty())))
            .thenApply(parsed -> parsed.map(this::createDocument));
    }

    // Individual operations return Optional instead of throwing
    private Optional<Path> validateFile(Path filePath) {
        if (Files.exists(filePath) && Files.isReadable(filePath)) {
            return Optional.of(filePath);
        }
        return Optional.empty();
    }

    private Optional<String> readFileContent(Path filePath) {
        try {
            return Optional.of(Files.readString(filePath));
        } catch (IOException e) {
            logger.warn("Could not read file: {}", filePath, e);
            return Optional.empty();
        }
    }

    private Either<ProcessingError, String> readFileContentSafely(Path filePath) {
        try {
            String content = Files.readString(filePath);
            return Either.right(content);
        } catch (NoSuchFileException e) {
            return Either.left(new ProcessingError("FILE_NOT_FOUND",
                "File does not exist: " + filePath));
        } catch (AccessDeniedException e) {
            return Either.left(new ProcessingError("ACCESS_DENIED",
                "Cannot read file: " + filePath));
        } catch (IOException e) {
            // Only system-level I/O errors remain as exceptions
            throw new SystemException("SYSTEM_IO_ERROR", "System I/O error", e);
        }
    }

    // Functional database connection handling
    public Either<DatabaseError, Connection> connectToDatabase(DatabaseConfig config) {
        return validateConfig(config)
            .flatMap(this::establishConnection)
            .flatMap(this::validateConnection);
    }

    private Either<DatabaseError, DatabaseConfig> validateConfig(DatabaseConfig config) {
        if (config.getUrl() == null || config.getUrl().isEmpty()) {
            return Either.left(new DatabaseError("INVALID_CONFIG", "Database URL is required"));
        }
        return Either.right(config);
    }
}
```

**Bad example:**

```java
// AVOID: Exception-driven control flow and broad catching

public class FileProcessor {
    private static final Logger logger = LoggerFactory.getLogger(FileProcessor.class);

    // Bad: Using exceptions for control flow
    public ProcessedDocument processFileChain(Path filePath) throws ProcessingException {
        try {
            validateFile(filePath);
            String content = readFileContent(filePath);
            ParsedContent parsed = parseContent(content);
            validateContent(parsed);
            return createDocument(parsed);
        } catch (FileNotFoundException e) {
            throw new ProcessingException("File not found", e);
        } catch (AccessDeniedException e) {
            throw new ProcessingException("Access denied", e);
        } catch (ParseException e) {
            throw new ProcessingException("Parse error", e);
        } catch (ValidationException e) {
            throw new ProcessingException("Validation error", e);
        } catch (Exception e) { // Broad catching masks other issues
            throw new ProcessingException("Unknown error", e);
        }
    }

    // Bad: Exception-based async handling
    public CompletableFuture<ProcessedDocument> processFileAsync(Path filePath) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                validateFile(filePath);
                String content = readFileContent(filePath);
                ParsedContent parsed = parseContent(content);
                return createDocument(parsed);
            } catch (Exception e) {
                throw new RuntimeException(e); // Wrapping all exceptions
            }
        });
    }

    // Bad: Methods that throw for normal failure cases
    private void validateFile(Path filePath) throws FileNotFoundException {
        if (!Files.exists(filePath)) {
            throw new FileNotFoundException("File not found: " + filePath);
        }
    }

    private String readFileContent(Path filePath) throws IOException {
        return Files.readString(filePath); // Propagating I/O exceptions
    }

    // Bad: Broad exception catching in critical paths
    public Connection connectToDatabase(DatabaseConfig config) throws DatabaseException {
        try {
            return DriverManager.getConnection(
                config.getUrl(), config.getUsername(), config.getPassword());
        } catch (Throwable t) { // Catches everything including system errors
            throw new DatabaseException("Database connection failed", t);
        }
    }
}
```

### Example 5: Logging and Monitoring with Functional Error Handling

Title: Implement comprehensive logging and monitoring without relying on exception stack traces
Description: When using functional error handling, implement structured logging with error contexts and metrics. Log business logic failures as warnings or info, and reserve error logging for true system failures.

**Good example:**

```java
// GOOD: Proper exception chaining and logging

public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);

    public User createUser(CreateUserRequest request) throws UserCreationException {
        logger.info("Creating user with email: {}", request.getEmail());

        try {
            // Validate email format
            validateEmail(request.getEmail());

            // Check if user already exists
            if (userRepository.existsByEmail(request.getEmail())) {
                UserAlreadyExistsException cause = new UserAlreadyExistsException(request.getEmail());
                logger.warn("User creation failed - email already exists: {}", request.getEmail());
                throw new UserCreationException("User with email already exists", cause);
            }

            // Create user
            User user = new User(request.getEmail(), request.getName());
            User savedUser = userRepository.save(user);

            // Send welcome email
            emailService.sendWelcomeEmail(savedUser);

            logger.info("Successfully created user with ID: {}", savedUser.getId());
            return savedUser;

        } catch (InvalidEmailFormatException e) {
            logger.warn("User creation failed - invalid email format: {}", request.getEmail(), e);
            throw new UserCreationException("Invalid email format", e);

        } catch (DatabaseException e) {
            logger.error("User creation failed - database error for email: {}", request.getEmail(), e);
            throw new UserCreationException("Failed to save user to database", e);

        } catch (EmailServiceException e) {
            // User was created but welcome email failed - log and continue
            logger.warn("User created successfully but welcome email failed for: {}", request.getEmail(), e);
            // Don't re-throw - user creation succeeded
            return userRepository.findByEmail(request.getEmail())
                .orElseThrow(() -> new UserCreationException("User created but retrieval failed"));
        }
    }

    public void deleteUser(Long userId) throws UserDeletionException {
        logger.info("Deleting user with ID: {}", userId);

        try {
            User user = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException("User", userId.toString()));

            // Delete user data
            userDataService.deleteUserData(userId);
            userRepository.delete(user);

            logger.info("Successfully deleted user with ID: {}", userId);

        } catch (UserNotFoundException e) {
            logger.warn("User deletion failed - user not found: {}", userId, e);
            throw new UserDeletionException("User not found", e);

        } catch (DataDeletionException e) {
            logger.error("User deletion failed - could not delete user data: {}", userId, e);
            throw new UserDeletionException("Failed to delete user data", e);

        } catch (Exception e) {
            logger.error("Unexpected error during user deletion: {}", userId, e);
            throw new UserDeletionException("Unexpected error occurred", e);
        }
    }
}
```

**Bad example:**

```java
// AVOID: Poor exception chaining and logging

public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);

    public User createUser(CreateUserRequest request) throws UserCreationException {
        try {
            validateEmail(request.getEmail());

            if (userRepository.existsByEmail(request.getEmail())) {
                // Lost original exception information
                throw new UserCreationException("User exists");
            }

            User user = new User(request.getEmail(), request.getName());
            return userRepository.save(user);

        } catch (InvalidEmailFormatException e) {
            // No chaining - original cause is lost
            logger.error("Error creating user");
            throw new UserCreationException("Bad email");

        } catch (DatabaseException e) {
            // Logging without context
            logger.error("Database error", e);
            // Creating new exception without original cause
            throw new UserCreationException("Database failed");

        } catch (Exception e) {
            // Generic logging without useful information
            logger.error("Something went wrong");
            // No chaining and no useful message
            throw new UserCreationException("Error");
        }
    }

    public void deleteUser(Long userId) throws UserDeletionException {
        try {
            User user = userRepository.findById(userId).orElse(null);
            if (Objects.isNull(user)) {
                // No logging
                throw new UserDeletionException("Not found");
            }

            userRepository.delete(user);

        } catch (Exception e) {
            // Swallowing all exceptions with generic handling
            e.printStackTrace(); // Don't use printStackTrace in production
            throw new UserDeletionException("Failed");
        }
    }
}
```

### Example 6: Clear Error Communication in Functional Design

Title: Design comprehensive error types and documentation for functional error handling
Description: Error types in functional design should be self-documenting with clear error codes, messages, and context. Document return types that indicate possible failure states using comprehensive JavaDoc.

**Good example:**

```java
// GOOD: Clear error messages and documentation

public class BankAccountService {

    /**
     * Transfers money between two bank accounts.
     *
     * @param fromAccountId the ID of the source account
     * @param toAccountId the ID of the destination account
     * @param amount the amount to transfer (must be positive)
     * @throws AccountNotFoundException if either account does not exist
     * @throws InsufficientFundsException if the source account has insufficient balance
     * @throws InvalidAmountException if the amount is null, zero, or negative
     * @throws AccountFrozenException if either account is frozen
     * @throws TransferLimitExceededException if the transfer exceeds daily limits
     * @throws IllegalArgumentException if any parameter is null
     */
    public TransferResult transferMoney(String fromAccountId, String toAccountId, BigDecimal amount)
            throws AccountNotFoundException, InsufficientFundsException, InvalidAmountException,
                   AccountFrozenException, TransferLimitExceededException {

        // Validate parameters with clear messages
        if (Objects.isNull(fromAccountId) || fromAccountId.trim().isEmpty()) {
            throw new IllegalArgumentException(
                "Source account ID cannot be null or empty");
        }

        if (Objects.isNull(toAccountId) || toAccountId.trim().isEmpty()) {
            throw new IllegalArgumentException(
                "Destination account ID cannot be null or empty");
        }

        if (Objects.isNull(amount)) {
            throw new InvalidAmountException(
                "Transfer amount cannot be null");
        }

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException(
                String.format("Transfer amount must be positive, got: %s", amount));
        }

        // Find accounts with specific error messages
        Account fromAccount = accountRepository.findById(fromAccountId)
            .orElseThrow(() -> new AccountNotFoundException(
                String.format("Source account not found: %s", fromAccountId)));

        Account toAccount = accountRepository.findById(toAccountId)
            .orElseThrow(() -> new AccountNotFoundException(
                String.format("Destination account not found: %s", toAccountId)));

        // Check account status
        if (fromAccount.isFrozen()) {
            throw new AccountFrozenException(
                String.format("Source account is frozen: %s. Contact customer service to resolve.",
                    fromAccountId));
        }

        if (toAccount.isFrozen()) {
            throw new AccountFrozenException(
                String.format("Destination account is frozen: %s. Transfer cannot proceed.",
                    toAccountId));
        }

        // Check sufficient funds
        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(
                String.format("Insufficient funds in account %s. Available: %s, Required: %s",
                    fromAccountId, fromAccount.getBalance(), amount));
        }

        // Check transfer limits
        BigDecimal dailyTransferTotal = getDailyTransferTotal(fromAccountId);
        BigDecimal newTotal = dailyTransferTotal.add(amount);
        if (newTotal.compareTo(fromAccount.getDailyTransferLimit()) > 0) {
            throw new TransferLimitExceededException(
                String.format("Transfer would exceed daily limit for account %s. " +
                    "Daily limit: %s, Already transferred today: %s, Attempted transfer: %s",
                    fromAccountId, fromAccount.getDailyTransferLimit(),
                    dailyTransferTotal, amount));
        }

        // Perform transfer
        return performTransfer(fromAccount, toAccount, amount);
    }
}
```

**Bad example:**

```java
// AVOID: Unclear error messages and poor documentation

public class BankAccountService {

    // No documentation about exceptions
    public TransferResult transferMoney(String fromAccountId, String toAccountId, BigDecimal amount)
            throws Exception {

        // Vague error messages
        if (Objects.isNull(fromAccountId)) {
            throw new IllegalArgumentException("Bad input");
        }

        if (Objects.isNull(amount) || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new RuntimeException("Invalid amount");
        }

        // Non-descriptive error messages
        Account fromAccount = accountRepository.findById(fromAccountId)
            .orElseThrow(() -> new RuntimeException("Not found"));

        Account toAccount = accountRepository.findById(toAccountId)
            .orElseThrow(() -> new RuntimeException("Error"));

        // No context in error messages
        if (fromAccount.isFrozen()) {
            throw new RuntimeException("Frozen");
        }

        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new RuntimeException("No money");
        }

        // Generic error messages that don't help users
        BigDecimal dailyTotal = getDailyTransferTotal(fromAccountId);
        if (dailyTotal.add(amount).compareTo(fromAccount.getDailyTransferLimit()) > 0) {
            throw new RuntimeException("Limit exceeded");
        }

        return performTransfer(fromAccount, toAccount, amount);
    }
}
```

### Example 7: Functional Control Flow Patterns

Title: Implement control flow using monadic patterns instead of exceptions
Description: Use functional programming patterns like Optional and Either types for control flow. This approach makes code more predictable, testable, and eliminates the performance overhead of exception handling.

**Good example:**

```java
// GOOD: Proper control flow without exceptions for normal conditions

public class ValidationService {

    // Return validation result instead of throwing for normal validation
    public ValidationResult validateUser(User user) {
        List<String> errors = new ArrayList<>();

        if (Objects.isNull(user.getName()) || user.getName().trim().isEmpty()) {
            errors.add("Name is required");
        }

        if (Objects.isNull(user.getEmail()) || !isValidEmail(user.getEmail())) {
            errors.add("Valid email is required");
        }

        if (user.getAge() < 18) {
            errors.add("User must be at least 18 years old");
        }

        return errors.isEmpty() ?
            ValidationResult.success() :
            ValidationResult.failure(errors);
    }

    // Use Optional for cases where value might not exist
    public Optional<User> findUserByEmail(String email) {
        return userRepository.findByEmail(email);
    }

    // Only throw exceptions for truly exceptional conditions
    public User getUserById(Long id) throws UserNotFoundException {
        if (Objects.isNull(id)) {
            throw new IllegalArgumentException("User ID cannot be null");
        }

        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User", id.toString()));
    }
}

public class FileProcessor {

    // Use return values for normal outcomes
    public ProcessingResult processFiles(List<Path> files) {
        List<String> processedFiles = new ArrayList<>();
        List<String> skippedFiles = new ArrayList<>();
        List<String> errorFiles = new ArrayList<>();

        for (Path file : files) {
            if (!Files.exists(file)) {
                skippedFiles.add(file.toString() + " (file not found)");
                continue;
            }

            if (!Files.isReadable(file)) {
                skippedFiles.add(file.toString() + " (not readable)");
                continue;
            }

            try {
                processFile(file);
                processedFiles.add(file.toString());
            } catch (Exception e) {
                errorFiles.add(file.toString() + " (" + e.getMessage() + ")");
            }
        }

        return new ProcessingResult(processedFiles, skippedFiles, errorFiles);
    }
}
```

**Bad example:**

```java
// AVOID: Using exceptions for normal control flow

public class ValidationService {

    // Bad: Using exceptions for normal validation failures
    public void validateUser(User user) throws ValidationException {
        if (Objects.isNull(user.getName()) || user.getName().trim().isEmpty()) {
            throw new ValidationException("Name is required");
        }

        if (Objects.isNull(user.getEmail()) || !isValidEmail(user.getEmail())) {
            throw new ValidationException("Valid email is required");
        }

        if (user.getAge() < 18) {
            throw new ValidationException("User must be at least 18 years old");
        }
    }

    // Bad: Throwing exception when value doesn't exist (normal case)
    public User findUserByEmail(String email) throws UserNotFoundException {
        User user = userRepository.findByEmail(email);
        if (Objects.isNull(user)) {
            throw new UserNotFoundException("User not found with email: " + email);
        }
        return user;
    }
}

public class FileProcessor {

    // Bad: Using exceptions for predictable conditions
    public void processFiles(List<Path> files) throws ProcessingException {
        for (Path file : files) {
            try {
                processFile(file);
            } catch (FileNotFoundException e) {
                // Using exception handling for expected condition
                throw new ProcessingException("File not found: " + file);
            } catch (AccessDeniedException e) {
                // Using exception handling for common access issues
                throw new ProcessingException("Cannot read file: " + file);
            }
        }
    }

    // Bad: Exception-driven logic for array bounds
    public String getArrayElement(String[] array, int index) {
        try {
            return array[index];
        } catch (ArrayIndexOutOfBoundsException e) {
            return "default"; // Using exception for normal bounds checking
        }
    }
}
```