# Authentication and Authorization

## Authentication

- Verifies the identity of users of entities trying to access a system, application or network
- Grants access to only authorized users protecting sensitive information and resources

### Requirements

- Storage: Figure out the storage of username and password with encyption
- Login Management
- Session Management: How long the user can stay logged in
- Security: Plan against hacks, attempts to predict password, rate limiting

### Types

- Password: Passwords should be complex and stored securely using encypted hash
- Multi-Factor Authentication (MFA): Password and a code (sent as email, sms)
- Biometric: Fingerprints, facial recognition, voice recognition
- Token: Physical or digital token like a security key or smart card
- OAuth: Delegated authentication using third party applications

### Authentication Information

- Tokens are included in HTTP headers (generally Bearer token), URL Params, or Request Payload
- Session ID is stored in a cookie which is sent with each HTTP request
- Certificates and keys are verified like SSL/TLS certificates
- Encryption
    - SSL/TLS (Secure Sockets Layer / Transport Layer Security)
        - Encrypts data transmission between a user browser and a web server
        - Secures communication channel and prevents eavesdropping
    - End-to-End Encryption

## Authorization

- Determines what actions or operations a user, system or entity
    - Is allowed to perform within a system or network

### Requirements

- How the permissions will be structured
    - Roles & heirarchy, permission types, how users or roles get these roles
- Access Control Method: Access Control Lists (ACL) or Role-Based Access Control (RBAC)
- Protect resources according to the permissions
- Context & conditions on which access might change, like time of the day or user location
- Keep track of the access for security monitoring

### Models

- Role Based Access Control (RBAC)
    - Assigning roles to users or groups letting them access only what their role requires
    - For example, HR personnel can access HR data but not finance information
- Access Control List
    - Lists permissions associated with a system resource
    - Specifies which user or system processes are granted access to resources
        - And what operations are allowed
- Security Assertion Markup Language (SAML)
    - Using an XML based protocol for single sign on
    - Access permissions are communicated through digitally signed documents
- Mandatory Access Control
    - Controlling permissions at a deep level in the computer system usually managed by admin
