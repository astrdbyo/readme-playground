## API Layer Overview

### Controller-Based API Versioning

```sh
src/
├── controllers/ 
│   ├── v1/                
│   └── v2/   
```

&nbsp;

API versioning is required to ensure backward compatibility. It allows developers to deploy new logic safely and roll back API behavior by switching versions, rather than reverting Git commits.
