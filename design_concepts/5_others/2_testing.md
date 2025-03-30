## Quality Assurance
- Multi-testing strategy
- Automation and script based testing
- Measurement and reporting mechanism
- Formal techical reviews

## Unit Testing
- Focuses on individual units (functions, procedures)
  - Typically performed or automated by developers
- Isolates sections of code and verifies its correctness
- Helps to fix bugs early in the development cycle and enables to make changes quickly
- Types
  - Black Box Testing: Covers iput, user interface & output
  - White Box Testing: Tests the functional behavior and internal design structure
  - Gray Box Testing: Uses partial knowledge of the internal structure
- Cannot cover non functional parameters like scalability & performance
- Not sufficient for testing interactions between units

## Integration Testing
- Focuses on interfaces and interactions between two units or modules
- Types
  - Big Bang IT
    - All the modules are put together and tested
    - Useful only for small systems, becomes complex for large systems
  - Bottom Up IT
    - Lower level modules are tested with higher level modules until all the modules are tested
  - Top Down IT
    - Simulates the behavior of the lower level modules that are not yet integrated
    - Needs many stubs and difficult to observe outputs

## Regression Testing
- Used to ensure that the new changes do not introduce new bugs or breaks functionality
- Types
  - Retesting: Testing entire application or specific functionality affected by the changes
  - Re-execution: Running a previously executed test suite
    - To ensure that the changes did not break any existing functionality
  - Comparision: Comparing the current version with a previous version

## Security Testing
- To identify and protect against vulnerabilities that can be exploited by attackers
- Authentication & Authorization: Unauthorized access, weak passwords, data breaches
- Application: Unpatched software, misconfigured systems, data encryption & leakage
- Network: Firewalls, routers, denial of service (DoS), man in the middle
- Database: SQL injection, cross-site scripting

## Other Types
- Smoke Testing
  - Initial testing to check that the system is ready and stable for further testing
  - Checks that the system did not catch fire and there is no smoke
- Alpha Testing: Validation testing within the organization
- Beta Testing: Testing conducted at customer sites
- System Testing: Security testing, stress & performance testing, recovery testing
