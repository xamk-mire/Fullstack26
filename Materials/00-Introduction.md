# How the Web Works (Introduction)

When you open a website, it can feel like “the internet just shows you a page.” But underneath, a lot happens in a few seconds: your device communicates with other computers, asks for resources, receives responses, and the browser turns those responses into the page you see and interact with.

Understanding these basics is one of the most important foundations for this course. Even when we later use modern tools like React, TypeScript, Python APIs, and PostgreSQL, the same underlying web mechanisms remain the same.

This topic gives you a clear mental model of what happens from the moment you type a URL into your browser to the moment you click a button in a web app.

---

## The Big Idea: The Web Is a Conversation

At its core, the web works through a **conversation** between two roles:

* **Client**: Your browser (Chrome, Firefox, Safari) or an app on your phone
* **Server**: A computer somewhere on the internet that stores or generates content

The client **requests** something, and the server **responds**.

This is called the **client–server model**.

---

## What Happens When You Open a Website?

Imagine you type:

`https://example.com`

A simplified version of what happens:

1. **Your browser finds the server**

   * It turns `example.com` into an IP address (a numeric address like `93.184.216.34`) using **DNS** (Domain Name System).
2. **Your browser connects to that server**

   * It opens a network connection (usually secured with HTTPS).
3. **Your browser sends an HTTP request**

   * A message like: “Give me the homepage.”
4. **The server sends an HTTP response**

   * It replies with content: HTML, CSS, JavaScript, images, etc.
5. **Your browser renders the page**

   * It parses HTML → builds the page structure
   * Applies CSS → styles it
   * Runs JavaScript → adds interactivity
6. **More requests happen**

   * The HTML often refers to other files, and the browser requests them too:

     * CSS files
     * JS files
     * images
     * API calls for data

Even a “simple” page can cause dozens of requests.

---

## The Browser’s Job: Turning Files Into a Page

The browser is more than a “viewer” — it’s an engine that:

* Downloads resources
* Interprets **HTML** (structure)
* Applies **CSS** (style)
* Executes **JavaScript** (behavior)

A useful way to remember this:

* **HTML**: what’s on the page
* **CSS**: what it looks like
* **JavaScript**: what it does

When we later use React, we’re still producing HTML/CSS/JS — just in a more structured way.

---

## URLs, Domains, and DNS (Quick but Important)

### URL (Uniform Resource Locator)

A URL tells the browser **where** to go and **what** to ask for.

Example:
`https://myapp.com/products?page=2`

It contains:

* `https` → protocol (how to communicate)
* `myapp.com` → domain name (server name)
* `/products` → path (resource)
* `?page=2` → query parameters (extra info)

### DNS (Domain Name System)

DNS is like the internet’s phone book:

* Domain names are human-friendly
* IP addresses are machine-friendly
  DNS translates one into the other.

---

## HTTP: Requests and Responses

The language of web communication is called **HTTP** (HyperText Transfer Protocol). Think of HTTP as a standard format for messages.

### Requests

A request typically includes:

* A **method** (what action you want)
* A **path** (what resource)
* Headers (extra info)
* Sometimes a body (data you send)

Common methods:

* **GET**: request data (read)
* **POST**: send data (create)
* **PUT/PATCH**: update data
* **DELETE**: remove data

### Responses

Responses include:

* A **status code**
* Headers
* A response body (HTML, JSON, etc.)

Common status codes:

* **200 OK**: success
* **201 Created**: something was created
* **400 Bad Request**: client error (often invalid input)
* **401 Unauthorized**: not logged in / no access
* **404 Not Found**: resource doesn’t exist
* **500 Server Error**: server crashed or malfunctioned

---

## Static vs Dynamic Websites

### Static website

The server returns files that already exist (HTML/CSS/JS/images).
Example: a simple portfolio site.

### Dynamic web application

The server generates responses based on logic and data.
Example: an app that shows your personal tasks after you log in.

Most modern web apps are dynamic:

* The frontend runs in the browser
* The backend provides data through APIs
* The database stores persistent information

---

## Frontend, Backend, and Database (The Full-Stack Model)

A typical full-stack app has three main parts:

### Frontend (Client)

* Runs in the browser
* Shows UI and handles interactions
* Built with React + TypeScript later in the course

### Backend (Server)

* Runs on a server computer
* Handles business logic and security
* Provides data via APIs (we’ll build this with Python)

### Database

* Stores data permanently
* Supports querying and relationships
* We’ll use PostgreSQL

**Important:**
The frontend usually *does not* talk to the database directly.
It talks to the backend API, and the backend talks to the database.

---

## APIs and JSON: How Apps Exchange Data

Web apps often load data without reloading the whole page. The frontend sends an HTTP request to an API endpoint, and the server responds with **JSON**.

Example response:

```json
{
  "id": 3,
  "title": "Finish exercises",
  "done": false
}
```

You’ll use this pattern constantly in the course:

* React fetches data from Python API
* Python API reads/writes PostgreSQL
* API returns JSON back to React

---

## A Useful Mental Model

When you build a feature like “Add a new task,” here’s what happens:

1. User fills a form in the browser (frontend)
2. Frontend sends POST request to backend API
3. Backend validates input
4. Backend stores task in PostgreSQL
5. Backend returns the created task as JSON
6. Frontend updates the UI

If you understand this flow, full-stack development becomes far less mysterious.

