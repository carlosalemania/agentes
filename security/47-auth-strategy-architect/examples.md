# Auth Strategy Examples

## Example 1: Complete Auth System

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

class AuthService {
  async register(email, password) {
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await db.users.insert({ email, password: hashedPassword });
    return { id: user.id, email: user.email };
  }

  async login(email, password) {
    const user = await db.users.findOne({ email });
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new Error('Invalid credentials');
    }

    const tokens = this.generateTokens(user);
    await this.saveRefreshToken(user.id, tokens.refreshToken);
    return tokens;
  }

  generateTokens(user) {
    const accessToken = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    return { accessToken, refreshToken };
  }
}
```

## Example 2: RBAC with Database

```javascript
// permissions table
await db.permissions.insertMany([
  { id: 1, name: 'users:read' },
  { id: 2, name: 'users:write' },
  { id: 3, name: 'posts:write' },
]);

// role_permissions table
await db.rolePermissions.insertMany([
  { roleId: 1, permissionId: 1 },
  { roleId: 1, permissionId: 2 },
  { roleId: 2, permissionId: 3 },
]);

async function getUserPermissions(userId) {
  const user = await db.users.findById(userId);
  const permissions = await db.query(`
    SELECT p.name
    FROM permissions p
    JOIN role_permissions rp ON p.id = rp.permission_id
    WHERE rp.role_id = ?
  `, [user.roleId]);

  return permissions.map(p => p.name);
}
```
