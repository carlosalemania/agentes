# Spring Boot Architect - Examples

---

## Example: Complete CRUD Application

```java
// DTO
@Data
public class UserDTO {
    private Long id;
    private String email;
    private String name;
    private LocalDateTime createdAt;
}

// Request DTOs
@Data
public class CreateUserRequest {
    @NotBlank
    @Email
    private String email;

    @NotBlank
    @Size(min = 8)
    private String password;

    @NotBlank
    private String name;
}

// Mapper
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDTO toDTO(User user);
    User toEntity(CreateUserRequest request);
}

// Custom Exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

---

**Versión:** 1.0.0
