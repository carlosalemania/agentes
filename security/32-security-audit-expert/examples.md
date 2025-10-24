# Security Audit Expert - Examples

---

## Example 1: Secure API Endpoint

```javascript
import express from 'express';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import csrf from 'csurf';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { body, validationResult } from 'express-validator';

const app = express();

// ✅ Security middleware
app.use(helmet());
app.use(express.json({ limit: '10mb' })); // Limit payload size

// ✅ Rate limiting
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});
app.use('/api/', apiLimiter);

// ✅ CSRF protection
const csrfProtection = csrf({ cookie: true });

// ✅ Input validation + sanitization
app.post('/api/users',
  [
    body('email').isEmail().normalizeEmail(),
    body('password').isLength({ min: 8 }).trim().escape(),
    body('name').trim().escape().isLength({ max: 100 })
  ],
  async (req, res) => {
    // Validate input
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    const { email, password, name } = req.body;

    try {
      // ✅ Hash password
      const hashedPassword = await bcrypt.hash(password, 10);

      // ✅ Parameterized query (using ORM)
      const user = await User.create({
        email,
        password: hashedPassword,
        name
      });

      // ✅ Generate JWT
      const token = jwt.sign(
        { userId: user.id },
        process.env.JWT_SECRET,
        { expiresIn: '1h' }
      );

      res.status(201).json({ token });
    } catch (error) {
      // ✅ Don't leak error details
      console.error(error);
      res.status(500).json({ error: 'Internal server error' });
    }
  }
);

// ✅ Authentication middleware
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) return res.sendStatus(401);

  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
}

// ✅ Secure endpoint with authorization check
app.get('/api/users/:id', authenticateToken, async (req, res) => {
  const requestedId = parseInt(req.params.id, 10);

  // ✅ Check authorization (prevent IDOR)
  if (requestedId !== req.user.userId) {
    return res.sendStatus(403);
  }

  const user = await User.findByPk(requestedId, {
    attributes: { exclude: ['password'] } // ✅ Don't leak password hash
  });

  if (!user) return res.sendStatus(404);
  res.json(user);
});
```

## Example 2: Secure File Upload

```javascript
import multer from 'multer';
import path from 'path';
import crypto from 'crypto';

// ✅ Whitelist allowed file types
const allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];
const maxFileSize = 5 * 1024 * 1024; // 5MB

const storage = multer.diskStorage({
  destination: './uploads/',
  filename: (req, file, cb) => {
    // ✅ Generate random filename to prevent path traversal
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname);
    cb(null, `${randomName}${ext}`);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: maxFileSize },
  fileFilter: (req, file, cb) => {
    // ✅ Validate MIME type
    if (!allowedMimeTypes.includes(file.mimetype)) {
      return cb(new Error('Invalid file type'), false);
    }
    cb(null, true);
  }
});

app.post('/upload', authenticateToken, upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }

  res.json({
    filename: req.file.filename,
    size: req.file.size
  });
});
```

---

**Versión:** 1.0.0
