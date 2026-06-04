# Developer Quick Start Guide

A fast reference for developers working on the SRMS project.

## 🚀 Quick Setup (5 minutes)

### Backend
```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your credentials
npm run dev
```
Server runs on: `http://localhost:5000`

### Frontend
```bash
cd frontend
npm install
npm run dev
```
App runs on: `http://localhost:3000`

### With Docker
```bash
docker-compose -f backend/docker/docker-compose.yml up -d
```
Accesses:
- Backend: `http://localhost:5000`
- Frontend: `http://localhost:3000`
- MongoDB: `http://localhost:8081` (Mongo Express)

---

## 📁 Project Structure Overview

### Backend Layers
```
src/
├── api/
│   ├── controllers/    ← Handle HTTP requests
│   └── routes/         ← Map endpoints to controllers
├── services/           ← Business logic (reusable)
├── models/             ← Database schemas
├── middleware/         ← Auth, logging, errors
├── utils/              ← Helpers (JWT, password, response)
├── constants/          ← System constants (roles, grades)
├── config/             ← Configuration
└── server.js           ← Entry point
```

### Frontend Structure
```
src/
├── stores/             ← Zustand state (auth)
├── services/           ← API calls (axios)
├── components/         ← React components
├── pages/              ← Page-level components
├── hooks/              ← Custom React hooks
├── utils/              ← Helper functions
├── styles/             ← Global styles
└── App.jsx             ← Root component
```

---

## 🔧 Common Tasks

### Add New API Endpoint

1. **Create Controller Method** (`src/api/controllers/`)
```javascript
export const getStudents = asyncHandler(async (req, res) => {
  const students = await StudentService.getAll(req.query);
  sendSuccess(res, 200, 'Students fetched', students);
});
```

2. **Add Route** (`src/api/routes/`)
```javascript
router.get('/students', authenticate, authorize(['ADMIN']), getStudents);
```

3. **Test It**
```bash
curl -H "Authorization: Bearer <token>" http://localhost:5000/api/v1/students
```

### Add New Database Model

1. **Create Model** (`src/models/`)
```javascript
const schema = new Schema({
  name: { type: String, required: true },
  code: { type: String, unique: true },
});

export default mongoose.model('YourModel', schema);
```

2. **Import in Server** (`src/server.js`)
```javascript
import YourModel from './models/YourModel.js';
```

3. **Use in Service**
```javascript
const data = await YourModel.find().limit(10);
```

### Create New Service

1. **Create File** (`src/services/yourService.js`)
```javascript
class YourService {
  static async create(data) {
    return await YourModel.create(data);
  }
}
export default YourService;
```

2. **Use in Controller**
```javascript
import YourService from '../services/yourService.js';

export const create = asyncHandler(async (req, res) => {
  const result = await YourService.create(req.body);
  sendSuccess(res, 201, 'Created', result);
});
```

---

## 🔐 Authentication Flow

### Registration → Login → Use Token

```
User Registration
    ↓
Create User with hashed password
    ↓
Generate JWT token pair (access + refresh)
    ↓
Store in localStorage
    ↓
Include in API requests: Authorization: Bearer <token>
    ↓
Token expires? Use refresh token to get new access token
    ↓
Still expired? Logout and redirect to login
```

### Check if Authenticated
```javascript
import { useAuthStore } from './stores/authStore';

const { user, isAuthenticated } = useAuthStore();

if (!isAuthenticated) {
  // Show login page
}
```

---

## 📊 Database Operations

### Query Data
```javascript
// Find all
const users = await User.find();

// Find with filters
const activeUsers = await User.find({ isActive: true });

// Paginated
const page = 1, limit = 20;
const users = await User.find()
  .skip((page - 1) * limit)
  .limit(limit);

// Count
const total = await User.countDocuments();

// Search
const results = await User.find({
  $or: [
    { firstName: { $regex: 'john', $options: 'i' } },
    { email: { $regex: 'john', $options: 'i' } }
  ]
});
```

### Create/Update
```javascript
// Create
const user = await User.create({
  firstName: 'John',
  email: 'john@example.com',
  password: hashedPassword
});

// Update
const updated = await User.findByIdAndUpdate(
  id,
  { firstName: 'Jane' },
  { new: true }
);

// Delete (soft)
await User.findByIdAndUpdate(id, { isActive: false });
```

---

## 🎨 Frontend Component Template

### Create New Page Component

```javascript
import React, { useEffect, useState } from 'react';
import { Box, Card, CardHeader, CardContent, Button } from '@mui/material';
import api from '../../services/api';

export default function MyPage() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await api.get('/endpoint');
      setData(response.data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <Box>
      <Card>
        <CardHeader title="My Page" />
        <CardContent>
          {/* Your content here */}
        </CardContent>
      </Card>
    </Box>
  );
}
```

---

## 🔍 Debugging Tips

### Backend Debugging
```bash
# View logs
docker logs srms-backend -f

# Enable verbose logging
NODE_ENV=development npm run dev

# Check database
docker logs srms-mongodb
```

### Frontend Debugging
```bash
# Browser DevTools (F12)
- Console: Check for errors
- Network: Monitor API calls
- Storage: Check localStorage for tokens

# React DevTools Extension
- Inspect component props
- Check state changes
```

### Common Issues

| Issue | Solution |
|-------|----------|
| 401 Unauthorized | Token expired? Check localStorage, refresh page |
| 403 Forbidden | Wrong role? Check user.role vs required role |
| Database connection error | Check MONGODB_URI in .env |
| CORS error | Check CORS_ORIGIN in backend .env |
| Port already in use | Kill process: `sudo lsof -i :5000` |

---

## 📚 Key Files to Know

| File | Purpose |
|------|---------|
| `src/app.js` | Express setup + middleware chain |
| `src/constants/index.js` | All system constants (roles, grades) |
| `src/services/` | Business logic |
| `src/models/` | Database schemas |
| `src/middleware/auth.js` | Authentication/authorization |
| `src/utils/response.js` | Standardized responses |
| `frontend/src/stores/authStore.js` | Auth state management |
| `frontend/src/services/api.js` | API client with interceptors |

---

## 🧪 Testing

### Run Tests
```bash
# Backend tests
cd backend
npm test

# With coverage
npm run test:coverage

# Watch mode
npm run test:watch
```

### Test API Endpoint
```bash
# Using curl
curl -X POST http://localhost:5000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"pass"}'

# Using Postman
# 1. Import collection (coming soon)
# 2. Set environment variables
# 3. Run requests
```

---

## 🚀 Deployment

### Local Docker
```bash
docker-compose -f backend/docker/docker-compose.yml up -d
```

### AWS (Simple)
```bash
# SSH to EC2
ssh -i key.pem ec2-user@ip

# Clone and run
git clone <repo>
cd srms/backend
npm install
npm start
```

See [DEPLOYMENT.md](../docs/DEPLOYMENT.md) for detailed guide.

---

## 📝 Code Style

### Backend
- Use `asyncHandler` wrapper for all controller methods
- Import services at top, use in controller
- Always return standardized response (use `sendSuccess`/`sendError`)
- Add JSDoc comments to functions

### Frontend
- Use functional components with hooks
- Extract reusable components
- Use Material-UI components consistently
- Follow camelCase naming

---

## 🎯 Useful Commands

```bash
# Backend
npm install          # Install dependencies
npm run dev         # Start dev server
npm test            # Run tests
npm run lint        # Check code quality
npm start           # Production start

# Frontend
npm install         # Install dependencies
npm run dev        # Start dev server
npm run build      # Production build
npm run preview    # Preview build

# Docker
docker-compose up -d          # Start all services
docker-compose logs -f        # View logs
docker-compose ps             # Show containers
docker-compose down           # Stop all services
docker ps                     # List running containers
docker exec <id> bash         # SSH into container
```

---

## 📞 Getting Help

1. **Check Documentation**
   - [ARCHITECTURE.md](../ARCHITECTURE.md) - System design
   - [API.md](../docs/API.md) - Complete API reference
   - [README.md](../README.md) - Overview

2. **Check Code Comments**
   - Look for JSDoc comments above functions
   - Check constants for valid values

3. **Search Codebase**
   - Find similar patterns in existing code
   - Look for examples in services/controllers

4. **Test Endpoints**
   - Use curl or Postman
   - Check response format in documentation

---

## 🔄 Workflow Tips

1. **Before coding**: Check existing patterns
2. **While coding**: Follow established conventions
3. **After coding**: Test thoroughly
4. **Before committing**: Check for errors with ESLint
5. **After committing**: Push to feature branch

---

**Last Updated**: April 2026
**For Issues**: Check docs or search codebase for similar implementations
