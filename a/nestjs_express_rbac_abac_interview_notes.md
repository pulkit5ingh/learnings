# Interview Revision Notes: JWT Authentication, RBAC, and ABAC

## Node.js, Express.js, and NestJS

---

https://roadmap.sh/questions/nodejs

# Part 1: Authentication and Authorization

## Interview Question

**How do you implement authentication and authorization in Node.js or NestJS applications?**

## Interview-Ready Answer

I implement JWT-based authentication and authorization using roles and permissions in my Node.js and NestJS applications.

During registration, I securely hash the user's password using a password-hashing algorithm such as bcrypt or Argon2 before storing it in the database.

When the user logs in, I verify their credentials and generate a signed JWT access token. The frontend sends this token with protected API requests in the `Authorization` header using the Bearer scheme.

On the backend, I use authentication middleware in Express.js or authentication guards in NestJS to verify the token's signature and expiration. Once verified, I make the authenticated user's claims available to the request handler.

For authorization, I use role-based or permission-based access control. For example, Admin, Manager, and Customer roles can have different permissions. I also perform resource-level authorization checks to ensure users can access or modify only the resources they are permitted to use.

This separation of authentication and authorization makes the application more secure, maintainable, and scalable.

## Authentication vs. Authorization

| Authentication                           | Authorization                                  |
| ---------------------------------------- | ---------------------------------------------- |
| Verifies who the user is                 | Determines what the user is allowed to do      |
| Validates identity or credentials        | Checks roles, permissions, and resource access |
| Happens before authorization             | Follows successful authentication              |
| Example: Is this a valid logged-in user? | Example: Can this user delete this record?     |

- Authentication = Who are you?
- Authorization = What are you allowed to do?

A valid JWT does not automatically grant access to every endpoint or resource.

## JWT Authentication Flow

1. User registers with an email and password.
2. Backend hashes the password using bcrypt or Argon2.
3. Backend stores the password hash, not the original password.
4. User logs in with their credentials.
5. Backend compares the submitted password with the stored hash.
6. If valid, backend generates a signed JWT.
7. Frontend sends the token with requests to protected endpoints.
8. Backend verifies the token's signature and expiration.
9. The request continues as an authenticated request.

Example request header:

```http
Authorization: Bearer <JWT_TOKEN>
```

---

# Part 2: RBAC — Role-Based Access Control

## What is RBAC?

RBAC stands for **Role-Based Access Control**. In RBAC, permissions are assigned to roles, and roles are assigned to users.

Instead of writing conditions for every individual user, we define roles such as:

- Admin
- Manager
- Editor
- Customer

Then we assign permissions to each role.

### Example: E-commerce Application

| Role     | Permissions                                         |
| -------- | --------------------------------------------------- |
| Admin    | Create, read, update, and delete products and users |
| Manager  | Create, read, and update products                   |
| Customer | View products and manage their own orders           |

The relationship looks like this:

```text
User
  |
  v
Role
  |
  v
Permissions
  |
  v
Allowed API actions
```

For example, a Manager may have `product:create`, `product:read`, and `product:update`, but not `product:delete`.

## How I would design RBAC

I would model the system around users, roles, and permissions.

A simple relational data model could be:

```text
users
- id
- name
- email

roles
- id
- name

permissions
- id
- resource
- action

user_roles
- user_id
- role_id

role_permissions
- role_id
- permission_id
```

For a small application, a user's role might be stored directly on the user record. For a larger application, separate mapping tables make it easier to support multiple roles per user and manage permissions centrally.

Example permission names:

```text
user:read
user:create
user:update
user:delete
product:read
product:create
product:update
product:delete
order:read
order:update
```

The backend should enforce permissions. Hiding a button in the frontend is useful for user experience, but it is not security.

## RBAC in NestJS

NestJS provides Guards, custom decorators, and metadata through its execution context. A common implementation is:

1. An authentication guard verifies the JWT and populates `request.user`.
2. A custom `@Roles()` decorator attaches required roles to an endpoint.
3. A roles guard reads that metadata and compares it with the authenticated user's roles.
4. The guard allows or rejects the request.

### Step 1: Define roles

```typescript
export enum Role {
  ADMIN = 'admin',
  MANAGER = 'manager',
  CUSTOMER = 'customer',
}
```

### Step 2: Create a Roles decorator

```typescript
import { SetMetadata } from '@nestjs/common';
import { Role } from './role.enum';

export const ROLES_KEY = 'roles';

export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);
```

### Step 3: Create the RolesGuard

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from './roles.decorator';
import { Role } from './role.enum';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    // No role metadata means this guard imposes no role restriction.
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      return false;
    }

    const userRoles: Role[] = user.roles ?? [user.role];

    return requiredRoles.some((role) => userRoles.includes(role));
  }
}
```

This example assumes an earlier authentication guard has already verified the JWT and populated `request.user`. Register the authentication guard before the roles guard.

### Step 4: Protect a controller endpoint

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { Roles } from './roles.decorator';
import { RolesGuard } from './roles.guard';
import { JwtAuthGuard } from './jwt-auth.guard';
import { Role } from './role.enum';

@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)
export class UsersController {
  @Get()
  @Roles(Role.ADMIN)
  findAllUsers() {
    return 'Only admins can view all users';
  }
}
```

A request from a valid Admin can pass both guards. A valid Customer passes authentication but fails the role check, so the endpoint is forbidden.

In a real application, register guards through dependency injection and apply them globally or at controller/route level as appropriate. If a guard is global, ensure public routes and routes with no role restriction are handled deliberately.

## RBAC in Express.js

Express does not provide a built-in authorization framework like NestJS Guards, but we can implement authorization using middleware.

### Step 1: Authentication middleware

```javascript
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
  const authorization = req.headers.authorization;
  const [scheme, token] = authorization?.split(' ') ?? [];

  if (scheme !== 'Bearer' || !token) {
    return res.status(401).json({
      message: 'Authentication required',
    });
  }

  try {
    const user = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ['HS256'],
    });

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({
      message: 'Invalid or expired token',
    });
  }
}
```

### Step 2: Reusable role authorization middleware

```javascript
function authorizeRoles(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        message: 'Authentication required',
      });
    }

    const userRoles = req.user.roles ?? [req.user.role];
    const hasRequiredRole = allowedRoles.some((role) =>
      userRoles.includes(role)
    );

    if (!hasRequiredRole) {
      return res.status(403).json({
        message: 'You do not have permission to access this resource',
      });
    }

    next();
  };
}
```

### Step 3: Apply middleware to routes

```javascript
const express = require('express');
const router = express.Router();

router.get('/users', authenticate, authorizeRoles('admin'), (req, res) => {
  res.json({ message: 'Admin can view all users' });
});

router.post(
  '/products',
  authenticate,
  authorizeRoles('admin', 'manager'),
  (req, res) => {
    res.json({ message: 'Product created' });
  }
);
```

The order matters: authenticate first, then authorize, then execute the route handler.

---

# Part 3: ABAC — Attribute-Based Access Control

## What is ABAC?

ABAC stands for **Attribute-Based Access Control**. Instead of deciding access only from a user's role, ABAC evaluates attributes about the user, the resource, the requested action, and sometimes the environment.

A useful way to remember it is:

- **Subject:** Who is making the request? Example: user, department, clearance level.
- **Resource:** What are they trying to access? Example: document owner, department, classification.
- **Action:** What are they trying to do? Example: read, update, delete.
- **Environment:** Under what conditions? Example: time, device, network, location or risk level.

An authorization policy evaluates these attributes and returns allow or deny.

### Example: Company Document Management System

Suppose employees can access documents belonging to their own department.

A policy might say:

```text
Allow access if:
- user is authenticated
- user's department matches the document's department
- user has permission for the requested action
- document is not archived
```

Example:

| User  | Department | Document department | Action | Result                        |
| ----- | ---------- | ------------------- | ------ | ----------------------------- |
| Alice | Finance    | Finance             | Read   | Allow                         |
| Bob   | Sales      | Finance             | Read   | Deny                          |
| Carol | Finance    | Finance             | Update | Allow, if policy permits      |
| Dave  | Finance    | Finance             | Delete | Deny, if delete is restricted |

The important point is that access depends on attributes and policy conditions, not merely on a role name.

## RBAC vs. ABAC

| RBAC                                    | ABAC                                                                  |
| --------------------------------------- | --------------------------------------------------------------------- |
| Access is based on roles                | Access is based on attributes and policies                            |
| Easier to implement and understand      | More flexible for complex rules                                       |
| Example: Admin can delete users         | Example: A manager can update a document only in their own department |
| Good for broad job-function permissions | Good for contextual and resource-specific decisions                   |

They are not mutually exclusive. A production application can use RBAC for broad permissions and ABAC for additional contextual checks.

---

# Part 4: ABAC in NestJS

NestJS does not include a complete ABAC policy engine by default. We can implement simple policies in a service or guard, or integrate a policy library such as CASL for more expressive rules.

A clean design is:

1. JWT guard authenticates the user.
2. The controller loads the target resource.
3. An authorization service evaluates user, action, resource, and relevant context.
4. The endpoint proceeds only if the policy allows the action.

### Example: Department-based document access

Assume the authenticated user has `id` and `departmentId`, and a document has `ownerId`, `departmentId`, and `archived`.

#### Step 1: Define the document shape

```typescript
export interface DocumentRecord {
  id: string;
  ownerId: string;
  departmentId: string;
  archived: boolean;
}
```

#### Step 2: Create a policy service

```typescript
import { Injectable } from '@nestjs/common';

type Action = 'read' | 'update' | 'delete';

interface AuthenticatedUser {
  id: string;
  departmentId: string;
  role: string;
}

@Injectable()
export class DocumentPolicy {
  can(
    user: AuthenticatedUser,
    action: Action,
    document: DocumentRecord
  ): boolean {
    if (user.role === 'admin') {
      return true;
    }

    if (document.archived) {
      return false;
    }

    const sameDepartment = user.departmentId === document.departmentId;

    if (!sameDepartment) {
      return false;
    }

    if (action === 'read') {
      return true;
    }

    if (action === 'update') {
      return document.ownerId === user.id;
    }

    // Only admins can delete in this example.
    return false;
  }
}
```

#### Step 3: Use the policy in a service or controller

```typescript
import {
  ForbiddenException,
  Injectable,
  NotFoundException,
} from '@nestjs/common';

@Injectable()
export class DocumentsService {
  constructor(
    private readonly policy: DocumentPolicy,
    private readonly repository: DocumentsRepository
  ) {}

  async updateDocument(
    user: AuthenticatedUser,
    id: string,
    dto: UpdateDocumentDto
  ) {
    const document = await this.repository.findById(id);

    if (!document) {
      throw new NotFoundException('Document not found');
    }

    if (!this.policy.can(user, 'update', document)) {
      throw new ForbiddenException('You cannot update this document');
    }

    return this.repository.update(id, dto);
  }
}
```

`DocumentsRepository`, `UpdateDocumentDto`, and the authenticated-user type here represent application-specific classes/types that you would define. The key idea is to load the resource and evaluate the policy before changing it.

For list endpoints, do not fetch every record and filter only after returning data. Apply the access policy in the database query where possible, so unauthorized records are never exposed.

---

# Part 5: ABAC in Express.js

In Express, the same policy can be implemented as reusable middleware. The route should authenticate the user, load the resource, evaluate the policy, and then execute the handler.

### Step 1: Define a reusable policy function

```javascript
function canAccessDocument(user, action, document) {
  if (user.role === 'admin') {
    return true;
  }

  if (document.archived) {
    return false;
  }

  if (user.departmentId !== document.departmentId) {
    return false;
  }

  if (action === 'read') {
    return true;
  }

  if (action === 'update') {
    return document.ownerId === user.id;
  }

  // Only admins can delete in this example.
  return false;
}
```

### Step 2: Create resource authorization middleware

```javascript
function authorizeDocument(action, documentRepository) {
  return async (req, res, next) => {
    try {
      const document = await documentRepository.findById(req.params.id);

      if (!document) {
        return res.status(404).json({
          message: 'Document not found',
        });
      }

      const allowed = canAccessDocument(req.user, action, document);

      if (!allowed) {
        return res.status(403).json({
          message: 'You cannot perform this action on this document',
        });
      }

      req.document = document;
      next();
    } catch (error) {
      next(error);
    }
  };
}
```

### Step 3: Apply it to routes

```javascript
router.patch(
  '/documents/:id',
  authenticate,
  authorizeDocument('update', documentRepository),
  async (req, res, next) => {
    try {
      const updated = await documentRepository.update(
        req.document.id,
        req.body
      );

      res.json(updated);
    } catch (error) {
      next(error);
    }
  }
);
```

This example assumes the repository methods and `authenticate` middleware are implemented in the application. Validate the request body separately before updating the record.

---

# Part 6: How to Choose RBAC, ABAC, or Both

### Use RBAC when:

- Permissions map cleanly to job roles.
- Rules are relatively stable.
- You need straightforward administration.
- Example: Only Admins can manage users.

### Use ABAC when:

- Access depends on ownership, department, classification, status, time, or other context.
- Two users with the same role may have different access to different records.
- Example: A Manager can approve requests only for their own department and only below a specified amount.

### Use both in a real application

A practical pattern is:

```text
Request
  |
  v
JWT Authentication
  |
  v
RBAC check: Does the user have the broad permission?
  |
  v
Load requested resource
  |
  v
ABAC check: Does this user meet the resource/context rules?
  |
  v
Allow or deny
```

Example: A Manager has the broad `invoice:update` permission (RBAC), but can update only invoices belonging to their own department (ABAC).

---

# Part 7: Important Security and Design Considerations

1. **Never rely on frontend checks.** The backend must enforce every permission.
2. **Authenticate before authorizing.** Do not trust `req.user` or user attributes unless they come from verified server-side authentication or trusted data.
3. **Do not trust client-supplied attributes.** A client must not be able to change their role, department, ownership, or approval status in the request body and gain access.
4. **Check resource ownership and tenant boundaries.** In multi-tenant systems, scope database queries by tenant and enforce access at the server.
5. **Default to deny.** If a policy cannot be evaluated or required attributes are missing, deny access rather than allowing it.
6. **Avoid stale permissions.** JWT role claims can become outdated. Use short-lived access tokens, refresh strategies, or server-side permission checks where needed.
7. **Keep policies testable.** Write unit tests for allowed and denied combinations, including edge cases.
8. **Avoid leaking resource existence where appropriate.** Depending on the application, return 404 instead of 403 when revealing that a resource exists would expose sensitive information.
9. **Audit sensitive decisions.** Log important administrative and access decisions without logging tokens or secrets.
10. **Keep policy logic centralized.** Avoid duplicating authorization conditions across many controllers and routes.

---

# Part 8: Common Interview Follow-Up Questions

## 1. Can RBAC and ABAC be used together?

Yes. I use RBAC for broad permissions and ABAC for contextual, resource-level decisions. For example, a Manager may have permission to update invoices, but ABAC ensures they can update only invoices from their own department.

## 2. Where should authorization logic live?

I keep authorization rules in reusable guards, middleware, or dedicated policy/authorization services rather than scattering conditions throughout controllers. The controller or service invokes the policy after authentication and, when needed, after loading the resource.

## 3. What happens if a user changes their role in the frontend?

It must have no effect. The backend must derive identity and trusted claims from a verified token and/or server-side records. It must never accept a role or permission from an untrusted request body as proof of access.

## 4. How do you handle permissions changing after a JWT is issued?

A JWT is valid until its expiration unless the application adds revocation or server-side checks. I would use short-lived access tokens and a secure refresh-token strategy. For sensitive or frequently changing permissions, I would check current permissions from a server-side cache or database.

## 5. What is the difference between a NestJS Guard and middleware?

Middleware runs earlier in the request pipeline and is useful for general request processing, such as logging or basic token handling. Guards run in Nest's execution context and can access route/controller metadata, making them a natural place for authentication and authorization decisions.

## 6. How would you test an authorization system?

I would test:

- A user with the required role is allowed.
- A user without the role is denied.
- A user cannot access another user's resource.
- A user from another tenant or department is denied.
- Missing or invalid attributes result in denial.
- Admin exceptions work only where explicitly intended.

---

# Part 9: Interview-Ready Summary Answer

**Question: How would you implement RBAC and ABAC in NestJS or Express.js?**

I would first implement authentication using JWT, so every protected request has a verified user identity.

For RBAC, I would define roles such as Admin, Manager, and Customer, and map permissions to those roles. In NestJS, I would use custom decorators to attach required roles to routes and Guards to read that metadata and authorize the request. In Express.js, I would create reusable authorization middleware that checks the authenticated user's roles before allowing the route handler to execute.

For ABAC, I would evaluate attributes of the user, resource, action, and request context. For example, in a document-management application, a user may read a document only if they belong to the same department as the document. They may update it only if they also own it, while an Admin may have broader access.

I would keep these rules in a centralized policy or authorization service, apply them on the backend, and default to denying access when required information is missing. In a real application, I would often combine both: RBAC for broad permissions and ABAC for ownership, department, tenant, or other contextual restrictions.

## Quick Memory Trick

- **RBAC:** Role decides what actions you can generally perform.
- **ABAC:** Attributes and policy decide whether you can perform this action on this resource in this context.
- **NestJS:** Guards + decorators + policy service.
- **Express.js:** Authentication middleware + authorization middleware + policy function.
- **Production rule:** Authenticate → broad permission check → load resource → contextual policy check → allow or deny.
