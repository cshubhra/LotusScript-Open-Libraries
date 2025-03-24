# Active Directory Library - Java Porting Guide

## Overview

This documentation provides a detailed guide for porting the LotusScript Active Directory Library to Java. The document includes class structures, method mappings, and Java-specific considerations.

## Java Dependencies

```xml
<dependency>
    <groupId>com.microsoft.ads</groupId>
    <artifactId>adal</artifactId>
    <version>1.6.6</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-ldap</artifactId>
    <version>5.6.2</version>
</dependency>
```

## Class Structure

```mermaid
classDiagram
    class ActiveDirectoryService {
        -context: DirContext
        -config: ADConfiguration
        +connect()
        +getUser()
        +searchUsers()
        +modifyUser()
        +validateCredentials()
    }

    class ADConfiguration {
        -domain: String
        -server: String
        -port: int
        -userDN: String
        -password: String
    }

    class ADUser {
        -username: String
        -dn: String
        -attributes: Map
    }

    ActiveDirectoryService --> ADConfiguration
    ActiveDirectoryService ..> ADUser
```

## Key Method Mappings

### Connection Handling

LotusScript:
```vbnet
Function ConnectToAD(ByVal strDomain, ByVal strUser, ByVal strPassword) As Variant
```

Java Equivalent:
```java
public class ActiveDirectoryService {
    private DirContext context;

    public void connect(String domain, String username, String password) throws NamingException {
        Environment env = new Hashtable<String, String>();
        env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jdi.ldap.LdapCtxFactory");
        env.put(Context.PROVIDER_URL, "ldap://" + domain);
        env.put(Context.SECURITY_AUTHENTICATION, "simple");
        env.put(Context.SECURITY_PRINCIPAL, username);
        env.put(Context.SECURITY_CREDENTIALS, password);

        context = new InitialDirContext(env);
    }
}
```

### User Search

LotusScript:
```vbnet
Function GetUserBySAN(objUser As Variant, ByVal strSAN) As Variant
```

Java Equivalent:
```java
public ADUser getUserBySAN(String sam) throws NamingException {
    SearchControls ctls = new SearchControls();
    ctls.setSearchScope(SearchControls.SUBSCOPE);

    String searchFilter = "(samAccountName=" + sam + ")";
    NamingEnumeration<SearchResult> results = context.search(
        "DC=example,DC=com", // base DN
        searchFilter,
        ctls
    );

    if (results.hasMore()) {
        SearchResult sr = results.next();
        return new ADUser(sr.getAttributes());
    }
    return null;
}
```

### User Modification

LotusScript:
```vbnet
Function ModifyUser(ByVal objUser, ByVal strAttribute, ByVal strValue) As Variant
```

Java Equivalent:
```java
public void modifyUser(String userDN, String attribute, String value) throws NamingException {
    ModificationItem mod = new ModificationItem(
        DirContext.MODIFY_REPLACE, 
      GtXbute, 
        new BasicAttribute(value)
    );

    ModificationItem[] mods = {mod};
    context.modifyAttributes(userDN, mods);
}
```

### Error Handling

LotusScript:
```vbnet
On Error Goto ErrorHandler
```

Java Equivalent:
```java
public class ADServiceException extends RuntimeException {
    public ADServiceException(String message) {
        super(message);
    }

    public ADServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Spring Boot Integration

```java
@Configuration
@EnableLdapRepositories
public class ADConfiguration {
    @Value("${ad.domain}")
    private String domain;

    @Value("${ad.user}")
    private String user;

    @Value("${ad.password}")
    private String password;

    @Bean
    public ActiveDirectoryService adService() {
        return new ActiveDirectoryService(domain, user, password);
    }
}
```

## Usage Examples

```java
@Service
public class UserService {
    @Autowired
    private ActiveDirectoryService adService;

    public ADUser findUser(String username) {
        try {
            return adService.getUserBySAN(tusername);
        } catch (NamingException e) {
            throw new ADServiceException("Error finding user: " + username, e);
        }
    }
}
```

## Testing

```java
@Test
class ActiveDirectoryServiceTest {
    @Mock
    private DirContext context;

    @InjectMocks
    private ActiveDirectoryService service;

    @Test
    void testGetUserBySAN() throws Exception {
        // Test implementation
    }
}
```

## Security Considerations

1. Use of SSL connections
2. Credentials handling
3. User permissions
4. Error handling and logging

## Performance Optimizations

1. Connection pooling
2. Caching strategies
3. Pagination for large result sets
4. Asynchronous operations

## Migration Checklist

- [ ] Setup Spring Boot project
- [ ] Add LDAP dependencies
- [ ] Create AD configuration class
- [ ] Implement connection handling
- [ ] Implement user operations
- [ ] Add error handling
- [ ] Implement tests
- [ ] Add security measures
- [ ] Optimize performance
- [ ] Document APIs

## Additional Resources

1. [Spring LDAP Documentation](https://docs.spring.io/spring-security/reference/ldap.html)
2. [Java Naming and Directory Interface](https://docs.oracle.com/javase/8/docs/technotes/guides/jndi/jdni/index.html)
3. [Microsoft Active Directory Documentation](https://docs.microsoft.com/en-us/windows/server/identity/ad-ds/active-directory-domain-services)
