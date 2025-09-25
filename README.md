# Local Passport Website - Postman Testing Guide

Dự án này là một **web application** sử dụng **Passport.js Local Strategy** với EJS templates. Khác với auth service, project này được thiết kế cho browser với form-based authentication và redirects.

## 📁 Cấu trúc Project

```
local_passport_website/
├── app.js              # Main web application
├── package.json        # Dependencies
├── config/
│   └── passport.js     # Passport Local Strategy configuration
├── models/
│   └── User.js         # User model with bcrypt hashing
├── routes/
│   └── auth.js         # Web authentication routes
└── views/              # EJS Templates
    ├── login.ejs       # Login form
    ├── register.ejs    # Registration form
    └── profile.ejs     # User profile page
```

## 🚀 Khởi chạy Server

```bash
cd src/local_passport_website
npm install
node app.js
```

**Server chạy tại:** `http://localhost:3000`
**Database:** MongoDB tại `mongodb://127.0.0.1:27017/passportAuth`

## 📋 Endpoints Overview

| Method | Endpoint | Mô tả | Content Type | Authentication |
|--------|----------|-------|--------------|----------------|
| GET | `/register` | Hiển thị form đăng ký | HTML | ❌ Public |
| POST | `/register` | Xử lý đăng ký | Form Data | ❌ Public |
| GET | `/login` | Hiển thị form login | HTML | ❌ Public |
| POST | `/login` | Xử lý login | Form Data | ❌ Credentials |
| GET | `/profile` | Trang profile (protected) | HTML | ✅ Session required |
| GET | `/logout` | Đăng xuất | Redirect | ✅ Session required |

---

## 🧪 Postman Test Cases

### ⚠️ Lưu ý quan trọng về Web Application Testing

Khác với API service, project này được thiết kế cho **browser interaction**:
- Responses trả về **HTML content** thay vì JSON
- Sử dụng **form-urlencoded** data thay vì JSON
- **Redirects** thay vì status responses
- **EJS templates** render thay vì JSON data

---

### 1. 📄 Get Register Page

**Request Configuration:**
- **Method:** `GET`
- **URL:** `http://localhost:3000/register`

**Expected Response (200 OK):**
```html
<h2>Register</h2>
<form action="/register" method="POST">
  <input type="text" name="username" placeholder="Username" required /><br>
  <input type="password" name="password" placeholder="Password" required /><br>
  <button type="submit">Register</button>
</form>
<a href="/login">Login</a>
```
![alt text](image.png)
---

### 2. 📝 Register User (Form Submission)

**Request Configuration:**
- **Method:** `POST`
- **URL:** `http://localhost:3000/register`
- **Headers:**
  ```
  Content-Type: application/x-www-form-urlencoded
  ```
- **Body (x-www-form-urlencoded):**
  ```
  username: testuser
  password: testpassword123
  ```

**Expected Response (302 Redirect):**
- **Status:** 302 Found
- **Location Header:** `/login`
- **Body:** Redirecting to `/login`
![alt text](image-2.png)

**Error Response (200 with error message):**
```html
Error: E11000 duplicate key error collection: passportAuth.users index: username_1 dup key: { username: "testuser" }
```
![alt text](image-1.png)
---

### 3. 📄 Get Login Page

**Request Configuration:**
- **Method:** `GET`
- **URL:** `http://localhost:3000/login`

**Expected Response (200 OK):**
```html
<h2>Login</h2>
<form action="/login" method="POST">
  <input type="text" name="username" placeholder="Username" required /><br>
  <input type="password" name="password" placeholder="Password" required /><br>
  <button type="submit">Login</button>
</form>
<a href="/register">Register</a>
```
![alt text](image-3.png)
---

### 4. 🔐 Login User (Form Submission)

**Request Configuration:**
- **Method:** `POST`
- **URL:** `http://localhost:3000/login`
- **Headers:**
  ```
  Content-Type: application/x-www-form-urlencoded
  ```
- **Body (x-www-form-urlencoded):**
  ```
  username: testuser
  password: testpassword123
  ```

**Success Response (302 Redirect):**
- **Status:** 302 Found
- **Location Header:** `/profile`
- **Set-Cookie:** `connect.sid=...`

![alt text](image-4.png)

**Failed Response (302 Redirect to login):**
- **Status:** 302 Found  
- **Location Header:** `/login`

![alt text](image-5.png)
---

### 5. 👤 Profile Page (Protected)

**Request Configuration:**
- **Method:** `GET`
- **URL:** `http://localhost:3000/profile`
- **Headers:** Cookie được gửi tự động sau login

**Success Response (200 OK) - Đã login:**
```html
<h2>Welcome testuser</h2>
<a href="/logout">Logout</a>
```
![alt text](image-6.png)

**Failed Response (302 Redirect) - Chưa login:**
- **Status:** 302 Found
- **Location Header:** `/login`
![alt text](image-8.png)
---

### 6. 🚪 Logout

**Request Configuration:**
- **Method:** `GET`
- **URL:** `http://localhost:3000/logout`

**Expected Response (302 Redirect):**
- **Status:** 302 Found
- **Location Header:** `/login`
- **Set-Cookie:** Cookie cleared
![alt text](image-7.png)
---

## 🔄 Complete Web Testing Flow

### Scenario 1: Happy Path (Web User Journey)
```
1. GET /register     → Show registration form
2. POST /register    → Create user, redirect to login
3. GET /login        → Show login form  
4. POST /login       → Authenticate, redirect to profile
5. GET /profile      → Show welcome page
6. GET /logout       → Clear session, redirect to login
7. GET /profile      → Redirect to login (unauthorized)
```

### Scenario 2: Error Handling
```
1. POST /register (duplicate) → Error message displayed
2. POST /login (wrong creds)  → Redirect back to login
3. GET /profile (no session)  → Redirect to login
4. Direct access protection   → All protected routes redirect
```

---

## 🔧 Postman Configuration for Web Testing

### 1. Environment Variables
```
passport_web_url = http://localhost:3000
test_username = testuser_{{$timestamp}}
test_password = testpassword123
```

### 2. Collection Structure
```
📁 Local Passport Website
  ├── 📄 1. Get Register Page
  ├── 📄 2. Register User (Success)
  ├── 📄 3. Register User (Duplicate - Error)
  ├── 📄 4. Get Login Page
  ├── 📄 5. Login User (Success)
  ├── 📄 6. Login User (Wrong Password)
  ├── 📄 7. Profile (After Login)
  ├── 📄 8. Profile (No Session)
  ├── 📄 9. Logout
  └── 📄 10. Profile (After Logout)
```

### 3. Pre-request Scripts

**For Registration:**
```javascript
// Generate unique username
const timestamp = Date.now();
pm.environment.set("unique_username", "user_" + timestamp);
```

**For Form Data:**
```javascript
// Set form data dynamically
pm.request.body.urlencoded.add({
    key: "username",
    value: pm.environment.get("unique_username")
});
```

### 4. Test Scripts for Web Responses

**For HTML Content Validation:**
```javascript
pm.test("Response contains register form", function () {
    pm.expect(pm.response.text()).to.include("<h2>Register</h2>");
    pm.expect(pm.response.text()).to.include('action="/register"');
    pm.expect(pm.response.text()).to.include('name="username"');
});

pm.test("Response contains login form", function () {
    pm.expect(pm.response.text()).to.include("<h2>Login</h2>");
    pm.expect(pm.response.text()).to.include('action="/login"');
});
```

**For Redirect Validation:**
```javascript
pm.test("Redirects to profile after login", function () {
    pm.response.to.have.status(302);
    pm.response.to.have.header("location", "/profile");
});

pm.test("Sets session cookie", function () {
    pm.expect(pm.cookies.has("connect.sid")).to.be.true;
});
```

**For Profile Content:**
```javascript
pm.test("Profile shows welcome message", function () {
    pm.response.to.have.status(200);
    pm.expect(pm.response.text()).to.include("<h2>Welcome");
    pm.expect(pm.response.text()).to.include("testuser");
});
```

---

## 📊 Expected Results Matrix

| Action | Status | Response Type | Cookie Status | Next Action |
|--------|--------|---------------|---------------|-------------|
| GET /register | 200 | HTML Form | No change | Fill & submit form |
| POST /register | 302 | Redirect to /login | No change | Navigate to login |
| GET /login | 200 | HTML Form | No change | Fill & submit form |
| POST /login (valid) | 302 | Redirect to /profile | Cookie set | Access profile |
| POST /login (invalid) | 302 | Redirect to /login | No cookie | Try again |
| GET /profile (auth) | 200 | HTML Welcome page | Cookie exists | User logged in |
| GET /profile (no auth) | 302 | Redirect to /login | No valid cookie | Need to login |
| GET /logout | 302 | Redirect to /login | Cookie cleared | Logged out |

---




## 🐛 Troubleshooting Web Application

### Issue 1: "Cannot GET /" error
**Cause:** No root route defined
**Solution:** All routes start from specific paths (/login, /register, /profile)

### Issue 2: Form submission returns 404
**Cause:** Incorrect Content-Type header
**Solution:** Use `application/x-www-form-urlencoded` for forms

### Issue 3: Always redirects to login
**Cause:** Session/cookie issues
**Solutions:**
- Check if MongoDB is running
- Verify session configuration
- Test cookie persistence in Postman

### Issue 4: Registration doesn't work
**Cause:** Database connection or validation errors
**Solutions:**
- Check MongoDB connection
- Verify User model and schema
- Check for duplicate username constraints

### Issue 5: HTML responses hard to read
**Solution:** Use Postman Preview tab to render HTML

---

