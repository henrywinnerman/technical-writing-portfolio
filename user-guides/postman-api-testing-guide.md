# Getting Started with Postman for API Testing

A beginner-friendly guide to testing APIs using Postman.

## Introduction

Postman is a popular tool for testing and developing APIs. Whether you're a developer building APIs or a technical writer documenting them, Postman helps you send requests, inspect responses, and automate testing workflows.

**What you'll learn:**
- How to send your first API request
- Organizing requests with Collections
- Using Variables and Environments
- Writing basic tests
- Running collections automatically

**Prerequisites:**
- Basic understanding of what an API is
- A computer with internet access
- No coding experience required

**Time to complete:** 30-45 minutes

---

## Table of Contents

1. [Installing Postman](#1-installing-postman)
2. [Understanding the Interface](#2-understanding-the-interface)
3. [Sending Your First Request](#3-sending-your-first-request)
4. [Understanding HTTP Methods](#4-understanding-http-methods)
5. [Working with Request Components](#5-working-with-request-components)
6. [Organizing Requests with Collections](#6-organizing-requests-with-collections)
7. [Using Variables](#7-using-variables)
8. [Managing Environments](#8-managing-environments)
9. [Writing Tests](#9-writing-tests)
10. [Running Collections](#10-running-collections)
11. [Sharing and Exporting](#11-sharing-and-exporting)
12. [Next Steps](#12-next-steps)

---

## 1. Installing Postman

### Download and Install

1. Visit [postman.com/downloads](https://www.postman.com/downloads/)
2. Download the version for your operating system (Windows, macOS, or Linux)
3. Run the installer and follow the prompts
4. Launch Postman

### Create an Account (Recommended)

While you can use Postman without an account, creating one allows you to:
- Sync your work across devices
- Collaborate with team members
- Access your collections from anywhere

Click **Create Account** and sign up with your email or Google account.

---

## 2. Understanding the Interface

When you open Postman, you'll see several key areas:

```
┌─────────────────────────────────────────────────────────────────┐
│  [New] [Import]                              [Environment ▼]    │
├──────────────────┬──────────────────────────────────────────────┤
│                  │                                              │
│   SIDEBAR        │              REQUEST BUILDER                 │
│                  │  ┌────────────────────────────────────────┐  │
│   Collections    │  │ GET ▼ │ https://api.example.com/users │  │
│   ├─ My API      │  └────────────────────────────────────────┘  │
│   │  ├─ Users    │                                              │
│   │  └─ Orders   │  [Params] [Auth] [Headers] [Body]            │
│   └─ Tests       │                                              │
│                  │                                              │
│   Environments   ├──────────────────────────────────────────────┤
│   History        │              RESPONSE VIEWER                 │
│                  │                                              │
│                  │  Status: 200 OK    Time: 245ms    Size: 1.2KB│
│                  │                                              │
│                  │  [Body] [Cookies] [Headers] [Test Results]   │
│                  │                                              │
└──────────────────┴──────────────────────────────────────────────┘
```

**Key areas:**

| Area | Purpose |
|------|---------|
| Sidebar | Browse collections, environments, and history |
| Request Builder | Configure and send API requests |
| Response Viewer | View the API's response |
| Environment Selector | Switch between different configurations |

---

## 3. Sending Your First Request

Let's send a request to a free, public API to see how Postman works.

### Step-by-step:

1. Click the **+** button to open a new request tab

2. In the URL field, enter:
   ```
   https://jsonplaceholder.typicode.com/users/1
   ```

3. Make sure **GET** is selected (it's the default)

4. Click **Send**

### Understanding the Response

You should see a response like this:

```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "suite": "Apt. 556",
    "city": "Gwenborough"
  },
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org"
}
```

**Key response information:**

| Element | What it means |
|---------|---------------|
| Status: 200 OK | The request was successful |
| Time: ~200ms | How long the request took |
| Size: ~500B | Size of the response |
| Body | The actual data returned |

🎉 **Congratulations!** You've just made your first API request.

---

## 4. Understanding HTTP Methods

APIs use different HTTP methods for different actions:

| Method | Purpose | Example Use |
|--------|---------|-------------|
| **GET** | Retrieve data | Get user profile |
| **POST** | Create new data | Create new account |
| **PUT** | Update existing data (full) | Replace user details |
| **PATCH** | Update existing data (partial) | Change just the email |
| **DELETE** | Remove data | Delete a post |

### Selecting a Method

Click the dropdown next to the URL field to change the HTTP method:

```
┌─────────┬──────────────────────────────┐
│ GET   ▼ │ https://api.example.com/users│
└─────────┴──────────────────────────────┘
    │
    ├─ GET
    ├─ POST
    ├─ PUT
    ├─ PATCH
    └─ DELETE
```

---

## 5. Working with Request Components

Most API requests require more than just a URL. Let's explore each component.

### 5.1 Query Parameters

Query parameters add filters or options to your request. They appear after `?` in the URL.

**Example:** Get posts by a specific user

1. Enter: `https://jsonplaceholder.typicode.com/posts`
2. Click the **Params** tab
3. Add a parameter:
   - Key: `userId`
   - Value: `1`

The URL automatically updates to:
```
https://jsonplaceholder.typicode.com/posts?userId=1
```

### 5.2 Headers

Headers send additional information with your request.

**Common headers:**

| Header | Purpose |
|--------|---------|
| Content-Type | Format of data you're sending |
| Authorization | Authentication credentials |
| Accept | Format you want back |

**Adding headers:**

1. Click the **Headers** tab
2. Add a new row:
   - Key: `Content-Type`
   - Value: `application/json`

### 5.3 Request Body

For POST, PUT, and PATCH requests, you often need to send data.

**Example: Create a new post**

1. Change method to **POST**
2. Enter: `https://jsonplaceholder.typicode.com/posts`
3. Click the **Body** tab
4. Select **raw**
5. Choose **JSON** from the dropdown
6. Enter:

```json
{
  "title": "My First Post",
  "body": "This is the content of my post.",
  "userId": 1
}
```

7. Click **Send**

You should receive a `201 Created` response with the new post data.

### 5.4 Authorization

Many APIs require authentication. Postman supports multiple auth types.

**Setting up Bearer Token auth:**

1. Click the **Authorization** tab
2. Select **Bearer Token** from the Type dropdown
3. Enter your token in the Token field

Postman automatically adds the header:
```
Authorization: Bearer your_token_here
```

**Other auth types:**
- API Key
- Basic Auth (username/password)
- OAuth 2.0
- AWS Signature

---

## 6. Organizing Requests with Collections

Collections help you group related requests together.

### Creating a Collection

1. Click **Collections** in the sidebar
2. Click the **+** icon or **Create Collection**
3. Name your collection (e.g., "JSONPlaceholder API")
4. Click **Create**

### Adding Requests to a Collection

**Method 1: Save a new request**
1. Configure your request
2. Click **Save** (or Ctrl/Cmd + S)
3. Enter a name (e.g., "Get User")
4. Select your collection
5. Click **Save**

**Method 2: Drag existing requests**
- Drag requests from History into your collection

### Organizing with Folders

For larger APIs, create folders within collections:

```
📁 My API Collection
├── 📁 Users
│   ├── GET All Users
│   ├── GET User by ID
│   ├── POST Create User
│   └── DELETE User
├── 📁 Posts
│   ├── GET All Posts
│   └── POST Create Post
└── 📁 Authentication
    └── POST Login
```

**Creating folders:**
1. Right-click your collection
2. Select **Add Folder**
3. Name the folder
4. Drag requests into the folder

---

## 7. Using Variables

Variables let you reuse values across multiple requests and avoid repetition.

### Variable Syntax

Use double curly braces to reference variables:

```
{{variable_name}}
```

**Example:**
```
https://{{base_url}}/users/{{user_id}}
```

### Types of Variables

| Scope | Use Case |
|-------|----------|
| **Global** | Values used across all collections |
| **Collection** | Values specific to one collection |
| **Environment** | Values that change between environments |
| **Local** | Temporary values within a single request |

### Creating Collection Variables

1. Click on your collection name
2. Go to the **Variables** tab
3. Add variables:

| Variable | Initial Value | Current Value |
|----------|---------------|---------------|
| base_url | https://jsonplaceholder.typicode.com | https://jsonplaceholder.typicode.com |
| user_id | 1 | 1 |

4. Click **Save**

### Using Variables in Requests

Now update your requests to use variables:

**Before:**
```
https://jsonplaceholder.typicode.com/users/1
```

**After:**
```
{{base_url}}/users/{{user_id}}
```

**Benefits:**
- Change the base URL once, update all requests
- Easily switch between user IDs
- No hardcoded values

---

## 8. Managing Environments

Environments let you switch between different configurations (development, staging, production) with one click.

### Creating an Environment

1. Click the **Environment** dropdown (top right)
2. Select **Manage Environments** (or click the gear icon)
3. Click **Add** to create a new environment
4. Name it (e.g., "Development")
5. Add variables:

| Variable | Type | Initial Value | Current Value |
|----------|------|---------------|---------------|
| base_url | default | http://localhost:3000 | http://localhost:3000 |
| api_key | secret | dev_key_12345 | dev_key_12345 |

6. Click **Save**

### Creating Multiple Environments

Create separate environments for each stage:

**Development:**
```
base_url: http://localhost:3000
api_key: dev_key_12345
```

**Staging:**
```
base_url: https://staging-api.myapp.com
api_key: staging_key_67890
```

**Production:**
```
base_url: https://api.myapp.com
api_key: prod_key_xxxxx
```

### Switching Environments

1. Click the environment dropdown (top right)
2. Select the environment you want to use
3. Your requests automatically use the new values

```
┌────────────────────────┐
│ No Environment       ▼ │
├────────────────────────┤
│ ○ No Environment       │
│ ● Development          │  ← Currently selected
│ ○ Staging              │
│ ○ Production           │
└────────────────────────┘
```

---

## 9. Writing Tests

Postman lets you write JavaScript tests that run automatically after each request.

### Accessing the Tests Tab

1. Open a request
2. Click the **Tests** tab (next to Body)

### Basic Test Examples

**Test 1: Check status code**

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

**Test 2: Check response time**

```javascript
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});
```

**Test 3: Check response contains data**

```javascript
pm.test("Response has user name", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.name).to.exist;
});
```

**Test 4: Validate specific value**

```javascript
pm.test("User ID is 1", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.id).to.eql(1);
});
```

### Using Test Snippets

Postman provides pre-written test snippets:

1. Look for the **Snippets** panel on the right side of the Tests tab
2. Click a snippet to insert it
3. Modify as needed

**Popular snippets:**
- Status code: Code is 200
- Response body: Contains string
- Response body: JSON value check
- Response time is less than 200ms

### Viewing Test Results

After sending a request:

1. Look at the **Test Results** tab in the response area
2. See which tests passed (✓) or failed (✗)

```
Test Results (3/3)
✓ Status code is 200
✓ Response time is less than 500ms
✓ Response has user name
```

---

## 10. Running Collections

The Collection Runner lets you execute all requests in a collection automatically.

### Opening the Collection Runner

1. Click on your collection
2. Click the **Run** button (or right-click → Run Collection)

### Configuring the Run

**Options:**

| Option | Description |
|--------|-------------|
| Environment | Select which environment to use |
| Iterations | Number of times to run the collection |
| Delay | Wait time between requests (ms) |
| Data File | CSV/JSON file for data-driven testing |

### Running the Collection

1. Select your environment
2. Set iterations to 1 (for now)
3. Click **Run [Collection Name]**

### Viewing Results

After the run completes, you'll see:

```
Run Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total Requests: 5
Passed Tests: 12
Failed Tests: 0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GET Get All Users         ✓ 200 OK    156ms    3/3 tests
GET Get User by ID        ✓ 200 OK    142ms    2/2 tests
POST Create User          ✓ 201 Created 198ms  4/4 tests
PUT Update User           ✓ 200 OK    167ms    2/2 tests
DELETE Delete User        ✓ 200 OK    134ms    1/1 tests
```

### Data-Driven Testing

Run the same requests with different data using a CSV or JSON file.

**Example CSV file (users.csv):**
```csv
user_id,expected_name
1,Leanne Graham
2,Ervin Howell
3,Clementine Bauch
```

**Using data in tests:**
```javascript
pm.test("Name matches expected", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.name).to.eql(pm.iterationData.get("expected_name"));
});
```

---

## 11. Sharing and Exporting

### Exporting a Collection

Share your collection with teammates or back it up:

1. Right-click your collection
2. Select **Export**
3. Choose format (Collection v2.1 recommended)
4. Save the JSON file

### Exporting an Environment

1. Click the gear icon (Manage Environments)
2. Click the download icon next to your environment
3. Save the JSON file

### Importing

To import a collection or environment:

1. Click **Import** (top left)
2. Drag and drop the JSON file, or click to browse
3. The collection/environment appears in your sidebar

### Sharing via Workspace

For team collaboration:

1. Create a **Team Workspace**
2. Move collections to the shared workspace
3. Invite team members

---

## 12. Next Steps

You've learned the basics of Postman! Here are some areas to explore next:

### Advanced Features

| Feature | Description |
|---------|-------------|
| **Pre-request Scripts** | Run JavaScript before a request sends |
| **Newman** | Run collections from command line |
| **Monitors** | Schedule collections to run automatically |
| **Mock Servers** | Simulate API responses |
| **API Documentation** | Generate docs from collections |

### Practice Projects

1. **Test a public API:** Try the GitHub API, OpenWeather API, or Spotify API
2. **Create a full CRUD collection:** Build requests for Create, Read, Update, Delete
3. **Set up environments:** Create dev/staging/prod configurations
4. **Write comprehensive tests:** Validate status codes, response structure, and data

### Learning Resources

- [Postman Learning Center](https://learning.postman.com/)
- [Postman YouTube Channel](https://www.youtube.com/postman)
- [Postman Community Forum](https://community.postman.com/)

---

## Quick Reference

### Keyboard Shortcuts

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| New Request | Ctrl + N | Cmd + N |
| Send Request | Ctrl + Enter | Cmd + Enter |
| Save | Ctrl + S | Cmd + S |
| Find | Ctrl + F | Cmd + F |
| Open Console | Ctrl + Alt + C | Cmd + Option + C |

### Common Test Assertions

```javascript
// Status codes
pm.response.to.have.status(200);
pm.response.to.have.status("OK");
pm.response.to.be.success; // 2xx

// Response body
pm.response.to.have.body("expected text");
pm.response.to.have.jsonBody("key", "value");

// Headers
pm.response.to.have.header("Content-Type");

// Response time
pm.expect(pm.response.responseTime).to.be.below(500);
```

### Variable Reference

```javascript
// Get variables
pm.variables.get("variable_name");
pm.environment.get("env_variable");
pm.collectionVariables.get("collection_var");
pm.globals.get("global_var");

// Set variables
pm.environment.set("token", "abc123");
pm.collectionVariables.set("user_id", 5);

// Use in URL/body
{{variable_name}}
```

---

## Troubleshooting

### Request not sending?
- Check your internet connection
- Verify the URL is correct
- Look for typos in variable names

### Getting unexpected errors?
- Check the Postman Console (View → Show Postman Console)
- Verify headers are correct
- Ensure body format matches Content-Type

### Variables not working?
- Confirm the environment is selected
- Check variable scope (collection vs environment)
- Verify variable names match exactly (case-sensitive)

---

*Last updated: January 2026*
