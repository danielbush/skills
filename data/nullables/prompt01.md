Summarize James Shore's nullables and A-Frame and create a first draft of a skill.
The skill should be created in `nullable-skill/`.

The main content to summarize is here:

https://www.jamesshore.com/v2/projects/nullables/testing-without-mocks
https://www.jamesshore.com/v2/projects/nullables/a-light-introduction-to-nullables

Don't exhaustively document everything from the above urls.
Instead, (1) focus on the big things and (2) additionally ensure the following specific things are included:

- Include value objects
  - the value object pattern is considered to be immutable
  - but I'm also fine with "mutable value objects"
    - a mutable value object is a value object that we can call .create on to instantiate and modify using methods
    - the methods do only algorithmic in-memory operations eg some form of manipulation
    - any operations that require the network (infrastructure) should be handled by infrastructure code that takes and/or returns either a mutable or immutable value object
- most code should fit into the A-frame architecture; either it is infrastructure, or it is application or its logic or value objects;
- application code is high-level infrastructure code that orchestrates infrastructure code and logic/value code
- low-level infrastructure code is called an infrastructure wrapper; these wrap calls that directly interface with the outside world eg the network or the file system etc
- every class should have a static create method which should be used instead of the constructor
- where possible we should be able to call create with minimal arguments and get a reasonable production-ready instance
- the static create methods will often call .create on other classes and then pass the created instances to the constructor of the class creating a simple dependency injection
- most classes including all application and infrastructure classes should provide a static createNull method; these have to mimic what .create does but always using createNull
- the .create/.createNull static methods on application or top-level infrastructure code will eventually end up calling .create/.createNull on infrastructure wrapper code potentially via intermediate application or infrastructure code
- event emitters for state based tests
- no mocks; we use configurable responses on infrastructure code; unit tests are narrow, but sociable, potentially hitting nullable versions of infrastructure code
- separate to unit tests we also want a small number of live narrow integration tests that test or smoke test the live (.create) version of infrastructure code
