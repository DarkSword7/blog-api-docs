# Blog API Documentation Project

This repository contains comprehensive documentation for the Blog API, a production-ready RESTful API for managing blog posts, users, comments, and engagement features.

## 📁 Project Structure

```
blog-api-docs/
├── README.md                     # Main documentation homepage
├── SUMMARY.md                    # GitBook table of contents
├── LICENSE                       # Apache 2.0 license
├── openapi.yaml                  # OpenAPI 3.0 specification
├── authentication.md             # Legacy auth documentation
├── api-reference/                # Complete API reference
│   ├── README.md                 # API reference overview
│   ├── quick-start.md            # 5-minute getting started guide
│   ├── authentication.md         # JWT authentication & flows
│   ├── users.md                  # User management endpoints
│   ├── blogs.md                  # Blog post CRUD operations
│   ├── comments.md               # Comment system features
│   ├── likes.md                  # Engagement & likes management
│   ├── data-models.md            # Complete schema definitions
│   └── error-responses.md        # Error handling guide
└── guides/                       # Additional guides
    ├── rate-limiting.md          # API usage limits
    └── error-handling.md         # Error handling patterns
```

## 🎯 Documentation Features

### ✅ Complete API Coverage

- **6 Main Endpoints Groups**: Authentication, Users, Blogs, Comments, Likes, Root
- **20+ API Endpoints**: Fully documented with examples
- **Role-based Access**: Clear admin vs user permissions
- **Request/Response Examples**: JSON examples for every endpoint

### ✅ Developer Experience

- **Quick Start Guide**: Get running in 5 minutes
- **Code Examples**: JavaScript, Python, cURL samples
- **Error Handling**: Comprehensive error documentation
- **TypeScript Definitions**: Complete type definitions
- **OpenAPI Specification**: Machine-readable API spec

### ✅ Production Ready

- **Security Documentation**: JWT flows, rate limiting, CORS
- **Best Practices**: Error handling, retry logic, optimization
- **Monitoring & Logging**: Error tracking and analytics
- **Testing Examples**: Unit test patterns for API integration

### ✅ GitBook Integration

- **Auto-sync**: Documentation updates automatically sync to GitBook
- **Interactive Navigation**: Clean, searchable documentation
- **Mobile Friendly**: Responsive design for all devices
- **Version Control**: Git-based documentation versioning

## 🚀 Live Documentation

- **GitBook**: https://ashutosh-3.gitbook.io/blog-api/
- **API Endpoint**: https://blog-api-dc2g.onrender.com/api/v1
- **OpenAPI Viewer**: Import `openapi.yaml` into Swagger UI or Postman

## 📋 API Quick Facts

| Feature            | Details                                       |
| ------------------ | --------------------------------------------- |
| **Base URL**       | `https://blog-api-dc2g.onrender.com/api/v1`   |
| **Authentication** | JWT Bearer tokens + HTTP-only refresh cookies |
| **Rate Limiting**  | 60 requests/minute per IP                     |
| **Content Format** | JSON + multipart/form-data for uploads        |
| **Database**       | MongoDB with Mongoose ODM                     |
| **File Storage**   | Cloudinary integration                        |
| **Documentation**  | OpenAPI 3.0 + GitBook                         |

## 🛠️ Usage

### For Developers

1. **Read**: Start with [Quick Start Guide](api-reference/quick-start.md)
2. **Explore**: Browse the [API Reference](api-reference/README.md)
3. **Integrate**: Use code examples and OpenAPI spec
4. **Deploy**: Follow best practices and error handling

### For Technical Writers

1. **Edit**: Modify Markdown files in this repository
2. **Preview**: Use GitBook or Markdown preview
3. **Commit**: Changes auto-sync to GitBook
4. **Review**: Test with actual API endpoints

### For Project Managers

1. **Overview**: Check [README.md](README.md) for project summary
2. **Features**: Review [API Reference](api-reference/README.md) for capabilities
3. **Security**: Understand authentication and permissions
4. **Support**: Use documentation for user training

## 🔄 GitBook Sync Setup

This repository is configured to automatically sync with GitBook:

1. **GitHub Integration**: Repository connected to GitBook
2. **Auto-sync**: Changes to `main` branch trigger GitBook updates
3. **Bi-directional**: Edit in GitHub or GitBook, changes sync both ways
4. **Preview**: GitBook provides live preview and staging

### Sync Configuration

- **Source**: GitHub repository `blog-api-docs`
- **Branch**: `main` (production)
- **Path**: Root directory
- **Format**: GitBook Markdown

## 📝 Content Guidelines

### Writing Style

- **Clear & Concise**: Direct, actionable language
- **Code Examples**: Working examples for all languages
- **Error Scenarios**: Document common issues and solutions
- **User-Focused**: Write for developers consuming the API

### Structure Standards

- **Consistent Headers**: Use H1 for page titles, H2 for sections
- **Code Blocks**: Language-specific syntax highlighting
- **Tables**: For structured data like status codes
- **Links**: Cross-reference related documentation

### Maintenance

- **API Changes**: Update docs with every API modification
- **Code Examples**: Test all examples before committing
- **Screenshots**: Keep UI screenshots current
- **Links**: Verify all external links work

## 🔧 Local Development

### Prerequisites

- Git
- Markdown editor (VSCode recommended)
- GitBook CLI (optional)

### Setup

```bash
# Clone repository
git clone https://github.com/your-username/blog-api-docs.git
cd blog-api-docs

# Edit documentation
code .

# Preview changes (if GitBook CLI installed)
gitbook serve
```

### File Editing

1. **Edit Markdown files** in your preferred editor
2. **Test API examples** against live API
3. **Preview changes** using Markdown preview or GitBook
4. **Commit changes** to trigger GitBook sync

## 📊 Documentation Metrics

### Coverage

- ✅ **100% Endpoint Coverage**: All API endpoints documented
- ✅ **Complete Examples**: Request/response for every endpoint
- ✅ **Error Documentation**: All error codes and solutions
- ✅ **Authentication Flow**: Complete auth documentation

### Quality

- ✅ **Code Examples**: 3+ languages (JS, Python, cURL)
- ✅ **Interactive Elements**: Mermaid diagrams for flows
- ✅ **Search Optimization**: Keywords and cross-references
- ✅ **Mobile Responsive**: Works on all devices

### Maintenance

- 🔄 **Auto-sync**: GitBook integration active
- 📅 **Regular Updates**: Documentation updated with API changes
- 🧪 **Tested Examples**: All code examples verified
- 🔍 **Regular Reviews**: Monthly documentation audits

## 📞 Support & Contributing

### Documentation Issues

- **Typos/Errors**: Create GitHub issue or PR
- **Missing Content**: Request via GitHub issues
- **Improvements**: Submit pull requests
- **Questions**: Use GitHub discussions

### API Issues

- **Bug Reports**: Report to API repository
- **Feature Requests**: API repository issues
- **Technical Support**: Contact development team

### Contributing

1. **Fork** the repository
2. **Create branch** for your changes
3. **Edit documentation** and test examples
4. **Submit pull request** with clear description
5. **Review process** with maintainers

## 📄 License

This documentation is licensed under the Apache License 2.0. The Blog API itself is also Apache 2.0 licensed.

---

**Ready to explore?** Start with the [Quick Start Guide](api-reference/quick-start.md) or browse the [complete API reference](api-reference/README.md)!
