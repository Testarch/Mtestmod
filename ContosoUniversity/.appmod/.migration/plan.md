# .NET App Migration Plan: Local SQL Server to Azure SQL MI Instance

## Migration Overview

**Source Technology**: Local SQL Server (LocalDB)
**Target Technology**: Azure SQL Managed Instance
**Project**: ContosoUniversity (.NET Framework 4.8.2 + ASP.NET MVC 5)

## Current Architecture Analysis

### Database Layer
- **Current**: SQL Server LocalDB with connection string using `(LocalDb)\MSSQLLocalDB`
- **ORM**: Entity Framework Core 3.1.32
- **Provider**: Mixed usage of `System.Data.SqlClient` and `Microsoft.Data.SqlClient`

### Connection Configuration
- **Location**: `Web.config` 
- **Current String**: `Data Source=(LocalDb)\MSSQLLocalDB;Initial Catalog=ContosoUniversityNoAuthEFCore;Integrated Security=True;MultipleActiveResultSets=True`

## Migration Strategy

### Phase 1: Package Dependencies Update
1. **Replace System.Data.SqlClient with Microsoft.Data.SqlClient 6.0.2**
   - Update all references in .csproj and packages.config
   - Update using statements in code files

2. **Add Azure Identity Support**
   - Add Azure.Identity package version 1.14.0
   - Enable Managed Identity authentication

3. **Update Entity Framework References**
   - Update to latest compatible EF Core version
   - Configure EF provider for Microsoft.Data.SqlClient

### Phase 2: Connection String Migration
1. **Update Web.config**
   - Replace LocalDB connection string with Azure SQL MI format
   - Add Managed Identity authentication
   - Format: `Server=tcp:<managed-instance-name>.<dns-zone>.database.windows.net,3342;Database=<database-name>;Authentication=Active Directory Default;TrustServerCertificate=True`

2. **Environment Configuration**
   - Support multiple environments (dev/staging/production)
   - Use app settings for different Azure SQL MI instances

### Phase 3: Code Updates
1. **Update Using Statements**
   - `using System.Data.SqlClient;` → `using Microsoft.Data.SqlClient;`
   - `using Microsoft.SqlServer.Server;` → `using Microsoft.Data.SqlClient.Server;`

2. **Update Entity Framework Configuration**
   - Configure provider factory for Microsoft.Data.SqlClient
   - Update any EF-specific configurations

### Phase 4: Authentication & Security
1. **Implement Managed Identity**
   - Configure application to use Azure Managed Identity
   - Remove hardcoded credentials
   - Ensure secure connection to Azure SQL MI

## Required Package Updates

### Packages to Add:
- `Azure.Identity` version `1.14.0`
- `Microsoft.Data.SqlClient` version `6.0.2`

### Packages to Update:
- Entity Framework Core packages to latest compatible version
- Remove old `System.Data.SqlClient` references

## Files to Modify

### Configuration Files:
- `Web.config` - Update connection strings
- `packages.config` - Update package references
- `ContosoUniversity.csproj` - Update project references

### Code Files:
- `Data/SchoolContext.cs` - Update using statements
- Any controllers or services using direct SQL connections
- Global.asax.cs - Add provider factory registration if needed

## Target Azure SQL MI Configuration

### Connection String Format:
```
Server=tcp:<managed-instance-name>.<dns-zone>.database.windows.net,3342;Database=ContosoUniversityNoAuthEFCore;Authentication=Active Directory Default;TrustServerCertificate=True;MultipleActiveResultSets=True
```

### Key Features:
- Port 3342 for public endpoint
- DNS zone specific FQDN
- Managed Identity authentication
- Maintains database name: `ContosoUniversityNoAuthEFCore`

## Success Criteria

1. ✅ All `System.Data.SqlClient` references replaced with `Microsoft.Data.SqlClient`
2. ✅ Azure Identity packages successfully integrated
3. ✅ Connection string updated to Azure SQL MI format with Managed Identity
4. ✅ Application compiles successfully
5. ✅ Entity Framework context works with new provider
6. ✅ All SQL operations function correctly with Azure SQL MI
7. ✅ No security vulnerabilities in new packages (CVE check)

## Rollback Plan

- Maintain backup of original `Web.config`
- Keep original packages.config as reference
- Git branch for easy rollback: `appmod/dotnet-migration-local-sql-server-to-Azure-SQL-MI-instance-[timestamp]`

## Notes

- This migration maintains the existing .NET Framework 4.8.2 architecture
- Database schema and data migration to Azure SQL MI handled separately
- Application changes focus only on connectivity and authentication
- No business logic changes required