# SRMS — Student Result Management System

A full-stack web application for managing student results, exams, and marks following AKTU (Dr. A.P.J. Abdul Kalam Technical University) standards.

## Tech Stack

**Backend:** Node.js, Express.js, MongoDB, Mongoose, JWT  
**Frontend:** React.js, Vite, Material UI, Zustand

## Features

- Role-based access: Super Admin, Institution Admin, Teacher, Student
- AKTU B.Tech CSE subjects (Sem 1–8)
- Exam scheduling and marks upload by teachers
- Semester-wise results with SGPA/CGPA (10-point scale)
- Backlog tracking
- Student roll number generation (year-wise prefix)

## Setup

### Backend
```bash
cd backend
npm install
cp .env.example .env   # fill in your MongoDB URI and JWT secrets
npm run dev
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

### Seed Data
```bash
cd backend
npm run seed:admin      # create admin user
npm run seed:students   # generate 80 students (4 years × 20)
npm run seed:subjects   # seed AKTU subjects
npm run seed:results    # generate semester results
```

## Default Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | atulkumarsingh208@gmail.com | Atul208@ |
| Student | (see Users section in admin) | Student@123 |

## Environment Variables

See `backend/.env.example` for required variables.
