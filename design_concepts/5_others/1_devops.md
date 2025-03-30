## Devops Life Cycle
- Documenting and determining project structure and schedule
- Analyzing project requirements and collecting resources
- Designing software model with architecture and interface designing
- Developing an actual product by considering design & requirements
- Testing the build, fixing errors & bugs, refactoring the code
- Software deployment and monitoring for further enhancements

## CI/CD
- Practices about how an integrated code on a shared repository is used to release software to production
  - Deployment can be done multiple times a day with the help of automation
  - Error detection and response are fast due to short cycle and iterations
- Continuous Integration (CI)
  - Regularly build, test, and merge changes to a shared staging repository
  - Using version control to manage conflicts from too many branches at once
- Continuous Deployment (CD)
  - Continuous Delivery
    - Automatically testing changes for bugs and uploading to staging repository
  - Automatically releasing changes from staging repository to production
  - Automatically deploying application after testing and build stages

## Container Management Platforms
- Manages creation, deployment, scaling, availability, and destruction of software containers
- Benefits
  - Solves the problem of moving software from one computing environment or OS to another
  - Serves as a self isolated unit that can run anywhere that supports it, regardless of the host OS
  - Packages application code and associated dependencies and configurations into a virtual container
- Containers like Docker encapsulate the mircoservice and its dependencies
  - Allows to package applications into lightweight & portable containers
  - Encapsulates everything required to run the application: code, runtime, libraries, system tools
  - Ensures consistency across different environments
- Orchestration tools like Kubernetes manage the deployment, scaling, operation of containers
  - Also provides features for container scheduling, service discovery, load balancing, etc.
