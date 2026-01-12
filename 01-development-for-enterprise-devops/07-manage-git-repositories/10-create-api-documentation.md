# Create API Documentation

## Key Concepts
- **API Documentation**: Explains how API works, inputs, outputs, and endpoints
- **Documentation Tools**: Swagger/OpenAPI, API Blueprint, MkDocs, Docusaurus
- **Automation**: Keep documentation synchronized with code changes
- **Publishing**: Central location accessible to team and stakeholders

## Importance of API Documentation

Good documentation provides:
- **Understanding**: How API works and what it does
- **Inputs/Outputs**: What data API accepts and returns
- **Usage Examples**: How to call API endpoints
- **Error Handling**: How to handle errors and edge cases
- **Authentication**: How to authenticate requests

### Documentation Quality Checklist
- [ ] Choose appropriate format (OpenAPI, Markdown, etc.)
- [ ] Include examples and usage scenarios
- [ ] Keep updated when code changes
- [ ] Gather feedback from API users
- [ ] Automate generation where possible

## Creating API Documentation in Azure DevOps

### Documentation Tools

| Tool | Best For |
|------|----------|
| **Swagger (OpenAPI)** | REST APIs with standard spec |
| **API Blueprint** | API-first design approach |
| **MkDocs** | Markdown-based documentation |
| **Docusaurus** | Modern documentation websites |

### Integration Approach

```yaml
# Azure Pipeline example
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - 'src/**'

stages:
  - stage: GenerateDocs
    jobs:
      - job: BuildAPIDocs
        steps:
          # Generate OpenAPI spec from code
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run generate-spec'
          
          # Build documentation site
          - task: Npm@1
            inputs:
              command: 'custom'
              customCommand: 'run build-docs'
          
          # Publish to Azure DevOps wiki or website
          - task: PublishBuildArtifacts@1
            inputs:
              PathtoPublish: 'docs/build'
              ArtifactName: 'api-docs'
```

### Code Annotations

Use decorators/annotations to generate documentation:

```csharp
// C# with Swagger annotations
/// <summary>
/// Gets user by ID
/// </summary>
/// <param name="id">User identifier</param>
/// <returns>User object</returns>
[HttpGet("{id}")]
[ProducesResponseType(typeof(User), 200)]
[ProducesResponseType(404)]
public async Task<IActionResult> GetUser(int id)
{
    var user = await _userService.GetUserAsync(id);
    return user != null ? Ok(user) : NotFound();
}
```

```python
# Python with OpenAPI annotations
@app.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    """
    Get user by ID
    
    - **user_id**: Unique identifier for user
    """
    user = await user_service.get_user(user_id)
    return user
```

### Tools for Auto-Generation

| Tool | Language/Framework |
|------|-------------------|
| **Swagger Codegen** | Multiple languages |
| **Springfox** | Spring Boot (Java) |
| **Swashbuckle** | ASP.NET Core (C#) |
| **FastAPI** | Python |
| **NSwag** | .NET |

### Publishing Options

| Location | Use Case |
|----------|----------|
| **Azure DevOps Wiki** | Internal team documentation |
| **Static Website** | Public or internal docs |
| **Azure Web App** | Hosted documentation portal |
| **Azure Storage** | Static site hosting |

## Creating API Documentation in GitHub

### Documentation with Markdown

**Structure**:
```
docs/
├── api/
│   ├── authentication.md
│   ├── endpoints/
│   │   ├── users.md
│   │   ├── orders.md
│   │   └── products.md
│   ├── errors.md
│   └── examples.md
├── getting-started.md
└── README.md
```

**Template for Endpoint Documentation**:
```markdown
# GET /api/users

## Description
Retrieves list of users with optional filtering.

## Authentication
Requires Bearer token in Authorization header.

## Request

### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | integer | No | Page number (default: 1) |
| limit | integer | No | Items per page (default: 20) |
| role | string | No | Filter by role |

### Example Request
\`\`\`bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.example.com/v1/users?page=1&limit=20"
\`\`\`

## Response

### Success Response (200 OK)
\`\`\`json
{
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "role": "admin"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
\`\`\`

### Error Responses
| Status Code | Description |
|-------------|-------------|
| 401 | Unauthorized - Invalid or missing token |
| 403 | Forbidden - Insufficient permissions |
| 500 | Server Error |

## Example Error Response (401)
\`\`\`json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid authentication token"
  }
}
\`\`\`
```

### Static Site Generators

| Generator | Language | Best For |
|-----------|----------|----------|
| **Jekyll** | Ruby | GitHub Pages (native) |
| **MkDocs** | Python | Technical documentation |
| **Docusaurus** | JavaScript | Modern docs with React |
| **Hugo** | Go | Very fast builds |
| **VuePress** | JavaScript | Vue.js ecosystem |

### Setup Example (MkDocs)

```bash
# Install MkDocs
pip install mkdocs mkdocs-material

# Initialize documentation
mkdocs new api-docs
cd api-docs

# Configure
cat > mkdocs.yml << EOF
site_name: My API Documentation
theme:
  name: material
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - Authentication: api/authentication.md
  - Endpoints:
    - Users: api/endpoints/users.md
    - Orders: api/endpoints/orders.md
EOF

# Build documentation
mkdocs build

# Serve locally for preview
mkdocs serve
```

### Publishing to GitHub Pages

**Manual Deployment**:
```bash
# Build docs
mkdocs build

# Deploy to GitHub Pages
mkdocs gh-deploy
```

**Automated with GitHub Actions**:
```yaml
# .github/workflows/docs.yml
name: Deploy Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
      - 'mkdocs.yml'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.x
      
      - name: Install dependencies
        run: |
          pip install mkdocs-material
      
      - name: Deploy to GitHub Pages
        run: mkdocs gh-deploy --force
```

### OpenAPI/Swagger Integration

```yaml
# .github/workflows/openapi-docs.yml
name: Generate API Docs

on:
  push:
    branches: [ main ]
    paths:
      - 'src/**'

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Generate OpenAPI spec from code
      - name: Generate OpenAPI Spec
        run: npm run generate-openapi
      
      # Generate HTML docs from spec
      - name: Generate HTML Docs
        uses: Legion2/swagger-ui-action@v1
        with:
          output: docs-output
          spec-file: openapi.json
      
      # Deploy to GitHub Pages
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs-output
```

## Automation Best Practices

### CI/CD Integration

```yaml
# Documentation in build pipeline
- Generate API spec from code annotations
- Build documentation site
- Run link checker to find broken links
- Deploy to hosting platform
- Notify team of documentation updates
```

### Keep Documentation Current

| Trigger | Action |
|---------|--------|
| Code change in API | Regenerate documentation |
| PR merged | Update published docs |
| New release | Version documentation |
| Breaking change | Highlight in docs |

### Documentation Versioning

```
docs/
├── v1/
│   └── api/
├── v2/
│   └── api/
└── latest/
    └── api/
```

## Critical Notes
- 🎯 Good API documentation is essential for developer experience
- 💡 Use code annotations to keep docs synchronized with implementation
- ⚠️ Automate documentation generation in CI/CD pipeline
- 📊 Publish to central location accessible to all stakeholders
- 🔄 Version documentation alongside API versions
- ✨ Include examples and error scenarios for every endpoint

[Learn More](https://learn.microsoft.com/en-us/training/modules/manage-git-repositories/10-create-api-documentation)
