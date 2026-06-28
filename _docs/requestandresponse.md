---
module: 2
lesson: 5
slug: requestandresponse
title: "API Requests and Responses: A Tutorial with Examples"
description: "How API requests and responses work, with examples: HTTP methods, status codes, headers, and request and response bodies, explained for technical writers."
keywords: API request, API response, API status codes, HTTP methods, GET request, POST request, PUT request, DELETE request, REST API communication, HTTP headers, API parameters, request body, response body, JSON response, status code 200, status code 404, API testing tools, cURL commands, Postman API testing, API authentication, API errors, API troubleshooting, API documentation best practices, API examples
permalink: /apirequestsresponses.html
next_page:
  url: /pagination-idempotency-rate-limits.html
  title: "Pagination, idempotency, rate limits"
previous_page:
  url: /visualizingdatatypes.html
  title: "Visualizing Data Types"
image: /assets/images/api-request-response.svg
last_modified_at: 2023-10-15T09:00:00+00:00
author_name: Technical Writing Expert
author_description: Senior technical writer with 8+ years of experience documenting APIs and developer platforms
author_expertise: 
  - "API Documentation"
  - "HTTP Protocol"
  - "REST API Design"
  - "Technical Communication"
author_image: /assets/images/gaurav.svg
reading_time: 15
level: Intermediate
speakable: true
speakable_selectors:
  - ".doc-content h1" 
  - ".doc-content h2"
  - ".doc-content p:first-of-type"
  - "#what-is-an-api-request"
  - "#what-is-an-api-response"
  - "#http-methods"
schema_markup: true
---

So, you've met the [data types](/apidocumentation/visualizingdatatypes.html)—Boolean, Strings, Numbers, and their pals. Now, let's see them in action! APIs are all about communication: You send a **request**, and you get a **response**. Think of it like texting a friend—you ask a question (request), and they reply (response). Simple, right? 

Well, APIs work in the same way but with a bit more structure. Let's break it down!

{% include enhanced_note.html 
  title="Note" 
  type="info" 
  collapsible=true 
  expanded=true 
  content="New to API Communication?" content="If you're feeling a bit lost with all the technical jargon, don't worry! This chapter will guide you through API requests and responses step by step. By the end, you'll be making API calls like a pro!" 
%}

{% include optimized-image.html 
  src="/assets/images/api-request-response-flow.svg" 
  alt="API request and response flow diagram showing client, server, and data exchange" 
  width="800" 
  height="450" 
  loading="lazy" 
  class="centered"
  caption="The complete request-response cycle in API communication" 
%}

<div class="interactive-api-demo">
  <h3>Try It: Make an API Request</h3>
  <div class="demo-controls">
    <select id="api-method" class="api-control">
      <option value="GET">GET</option>
      <option value="POST">POST</option>
      <option value="PUT">PUT</option>
      <option value="DELETE">DELETE</option>
    </select>
    <input type="text" id="api-endpoint" value="https://api.example.com/books" class="api-control" readonly>
    <button id="send-api-btn" class="api-btn">Send Request</button>
  </div>
  <div class="demo-panels">
    <div class="request-panel">
      <h4>Request</h4>
      <pre id="request-display" class="code-display">GET https://api.example.com/books
Accept: application/json</pre>
    </div>
    <div class="response-panel">
      <h4>Response</h4>
      <pre id="response-display" class="code-display">// Click "Send Request" to see the response</pre>
    </div>
  </div>
  <div id="api-status" class="status-message">Ready to send request</div>
</div>

<style>
.interactive-api-demo {
  background: #f8f9fb;
  border-radius: 8px;
  padding: 20px;
  margin: 30px 0;
  border: 1px solid #e2e8f0;
}
.demo-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}
.api-control {
  padding: 8px 12px;
  border-radius: 4px;
  border: 1px solid #cbd5e0;
  font-family: monospace;
}
#api-method {
  width: 100px;
  background: #2c3e50;
  color: white;
  font-weight: bold;
}
#api-endpoint {
  flex: 1;
  min-width: 200px;
}
.api-btn {
  padding: 8px 16px;
  border-radius: 4px;
  background: #4a6ef5;
  color: white;
  border: none;
  cursor: pointer;
  font-weight: 500;
}
.demo-panels {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
}
.request-panel, .response-panel {
  flex: 1;
  background: #1e293b;
  border-radius: 6px;
  padding: 15px;
  min-width: 45%;
}
.request-panel h4, .response-panel h4 {
  margin-top: 0;
  color: #e2e8f0;
  margin-bottom: 10px;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}
.code-display {
  color: #e2e8f0;
  margin: 0;
  font-family: monospace;
  white-space: pre-wrap;
  font-size: 14px;
}
.status-message {
  background: #e2e8f0;
  padding: 10px;
  border-radius: 4px;
  text-align: center;
  font-style: italic;
}
@media (max-width: 768px) {
  .demo-panels {
    flex-direction: column;
  }
}
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const methodSelect = document.getElementById('api-method');
  const endpointInput = document.getElementById('api-endpoint');
  const sendButton = document.getElementById('send-api-btn');
  const requestDisplay = document.getElementById('request-display');
  const responseDisplay = document.getElementById('response-display');
  const statusMessage = document.getElementById('api-status');
  
  // Sample responses for different methods
  const responses = {
    GET: {
      status: 200,
      headers: "Content-Type: application/json\nCache-Control: max-age=3600",
      body: '{\n  "books": [\n    {\n      "id": 1,\n      "title": "The Great Gatsby",\n      "author": "F. Scott Fitzgerald"\n    },\n    {\n      "id": 2,\n      "title": "To Kill a Mockingbird",\n      "author": "Harper Lee"\n    }\n  ],\n  "total": 2\n}'
    },
    POST: {
      status: 201,
      headers: "Content-Type: application/json\nLocation: /books/3",
      body: '{\n  "id": 3,\n  "title": "1984",\n  "author": "George Orwell",\n  "created": true\n}'
    },
    PUT: {
      status: 200,
      headers: "Content-Type: application/json",
      body: '{\n  "id": 1,\n  "title": "The Great Gatsby",\n  "author": "F. Scott Fitzgerald",\n  "updated": true\n}'
    },
    DELETE: {
      status: 204,
      headers: "",
      body: ""
    }
  };
  
  // Sample request bodies
  const requestBodies = {
    GET: "",
    POST: 'Content-Type: application/json\n\n{\n  "title": "1984",\n  "author": "George Orwell"\n}',
    PUT: 'Content-Type: application/json\n\n{\n  "title": "The Great Gatsby",\n  "author": "F. Scott Fitzgerald"\n}',
    DELETE: ""
  };
  
  methodSelect.addEventListener('change', function() {
    const method = methodSelect.value;
    const endpoint = method === 'GET' ? 'https://api.example.com/books' : 
                    (method === 'POST' ? 'https://api.example.com/books' : 
                    'https://api.example.com/books/1');
    
    endpointInput.value = endpoint;
    
    // Update request display
    requestDisplay.textContent = `${method} ${endpoint}\nAccept: application/json`;
    
    if (requestBodies[method]) {
      requestDisplay.textContent += `\n\n${requestBodies[method]}`;
    }
    
    // Reset response
    responseDisplay.textContent = "// Click \"Send Request\" to see the response";
    statusMessage.textContent = "Ready to send request";
  });
  
  sendButton.addEventListener('click', function() {
    // Show loading state
    statusMessage.textContent = "Sending request...";
    
    // Simulate network delay
    setTimeout(() => {
      const method = methodSelect.value;
      const response = responses[method];
      
      // Build response text
      let responseText = `HTTP/1.1 ${response.status}`;
      if (response.status === 200) responseText += " OK";
      else if (response.status === 201) responseText += " Created";
      else if (response.status === 204) responseText += " No Content";
      
      if (response.headers) {
        responseText += `\n${response.headers}`;
      }
      
      if (response.body) {
        responseText += `\n\n${response.body}`;
      }
      
      responseDisplay.textContent = responseText;
      statusMessage.textContent = `Request complete! Received ${response.status} response`;
    }, 800);
  });
});
</script>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## What is an API Request? {#what-is-an-api-request}

Imagine you're at a restaurant. You tell the waiter, "I'd like a margherita pizza, please!" That's a **request**. The waiter (the API) takes your order, processes it, and brings back a pizza (the **response**).

An API request has a few main ingredients:
- **The Endpoint (URL):** This is where the request goes, like in [REST APIs](/apidocumentation/restAPI.html).
- **The HTTP Method:** Think of it as the type of request you're making (like asking for a menu vs. placing an order).
- **Headers:** Extra info you send along with your request (like saying "No olives, please").
- **Query Parameters:** These refine your request (like asking for extra cheese).
- **Request Body (for some requests):** If you're sending data (like placing an order), it goes here.

### Example: Making a GET Request to an API Endpoint {#making-get-request}
Let's say we want to find a book by its title.

```sh
   GET https://api.example.com/books?title=harry%20potter
   ```

- The **endpoint** is `https://api.example.com/books`
- The **method** is `GET` (because we just want to retrieve data)
- The **query parameter** is `title=harry%20potter` (we're looking for books titled *Harry Potter*)

  <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>


## What is an API Response? {#what-is-an-api-response}

Great, you placed your order. Now the waiter brings back your pizza (hopefully the right one!). 

{% include optimized-image.html 
  src="/assets/images/api-response-structure.svg" 
  alt="Structure of an API response showing status code, headers, and body" 
  width="750" 
  height="400" 
  loading="lazy" 
  class="centered"
  caption="Anatomy of an API response with key components" 
%}

An API response usually includes:
- **The Status Code:** A quick signal of whether the request was successful.
- **Headers:** More details about the response.
- **The Body:** The actual data you asked for (your pizza!).

### Example: A JSON Response from an API {#json-response-example}
Here's what the API might return for our Harry Potter book request:

```json
{
  "title": "Harry Potter and the Sorcerer's Stone",
  "author": "J.K. Rowling",
  "published_year": 1997,
  "genre": "Fantasy"
}
```

This is structured as [JSON](/apidocumentation/JSON101.html), one of the most common data formats used in modern APIs.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>


## HTTP Methods: The Different Types of API Requests {#http-methods}

APIs aren't just about **getting** data; they can also **send, update, and delete** information. That's where HTTP methods come in.

<div class="http-methods-interactive">
  <h3>Explore HTTP Methods</h3>
  <div class="method-buttons">
    <button class="method-btn get active" data-method="get">GET</button>
    <button class="method-btn post" data-method="post">POST</button>
    <button class="method-btn put" data-method="put">PUT</button>
    <button class="method-btn patch" data-method="patch">PATCH</button>
    <button class="method-btn delete" data-method="delete">DELETE</button>
  </div>
  <div class="method-info-container">
    <div class="method-info" id="get-info">
      <h4>GET Method</h4>
      <p>Used to retrieve data from a specified resource. GET requests should be idempotent and have no side effects.</p>
      <div class="method-example">
        <h5>Example:</h5>
        <code>GET https://api.example.com/users/123</code>
        <p>Retrieves details about user with ID 123</p>
      </div>
      <div class="method-attributes">
        <span class="attribute">Has Body: <span class="no">No</span></span>
        <span class="attribute">Safe: <span class="yes">Yes</span></span>
        <span class="attribute">Idempotent: <span class="yes">Yes</span></span>
        <span class="attribute">Cacheable: <span class="yes">Yes</span></span>
      </div>
</div>

    <div class="method-info hidden" id="post-info">
      <h4>POST Method</h4>
      <p>Used to submit data to be processed to a specified resource. Often used for creating new resources.</p>
      <div class="method-example">
        <h5>Example:</h5>
        <code>POST https://api.example.com/users</code>
        <p>Creates a new user with the data provided in the request body</p>
      </div>
      <div class="method-attributes">
        <span class="attribute">Has Body: <span class="yes">Yes</span></span>
        <span class="attribute">Safe: <span class="no">No</span></span>
        <span class="attribute">Idempotent: <span class="no">No</span></span>
        <span class="attribute">Cacheable: <span class="no">Rarely</span></span>
      </div>
    </div>
    
    <div class="method-info hidden" id="put-info">
      <h4>PUT Method</h4>
      <p>Used to update a resource or create it if it doesn't exist at a specified URL. Replaces the entire resource.</p>
      <div class="method-example">
        <h5>Example:</h5>
        <code>PUT https://api.example.com/users/123</code>
        <p>Updates user 123 with entirely new data, replacing all existing fields</p>
      </div>
      <div class="method-attributes">
        <span class="attribute">Has Body: <span class="yes">Yes</span></span>
        <span class="attribute">Safe: <span class="no">No</span></span>
        <span class="attribute">Idempotent: <span class="yes">Yes</span></span>
        <span class="attribute">Cacheable: <span class="no">No</span></span>
      </div>
    </div>
    
    <div class="method-info hidden" id="patch-info">
      <h4>PATCH Method</h4>
      <p>Used to apply partial modifications to a resource. Only updates the fields provided in the request.</p>
      <div class="method-example">
        <h5>Example:</h5>
        <code>PATCH https://api.example.com/users/123</code>
        <p>Updates only specific fields of user 123, leaving other fields unchanged</p>
      </div>
      <div class="method-attributes">
        <span class="attribute">Has Body: <span class="yes">Yes</span></span>
        <span class="attribute">Safe: <span class="no">No</span></span>
        <span class="attribute">Idempotent: <span class="maybe">Sometimes</span></span>
        <span class="attribute">Cacheable: <span class="no">No</span></span>
      </div>
    </div>
    
    <div class="method-info hidden" id="delete-info">
      <h4>DELETE Method</h4>
      <p>Used to remove a specified resource. Deletes the target resource.</p>
      <div class="method-example">
        <h5>Example:</h5>
        <code>DELETE https://api.example.com/users/123</code>
        <p>Removes the user with ID 123 from the system</p>
      </div>
      <div class="method-attributes">
        <span class="attribute">Has Body: <span class="maybe">Sometimes</span></span>
        <span class="attribute">Safe: <span class="no">No</span></span>
        <span class="attribute">Idempotent: <span class="yes">Yes</span></span>
        <span class="attribute">Cacheable: <span class="no">No</span></span>
      </div>
    </div>
  </div>
</div>

<style>
.http-methods-interactive {
  background: #f8f9fb;
  border-radius: 8px;
  padding: 20px;
  margin: 30px 0;
  border: 1px solid #e2e8f0;
}
.method-buttons {
  display: flex;
  gap: 8px;
  margin-bottom: 15px;
  flex-wrap: wrap;
}
.method-btn {
  padding: 10px 20px;
  border-radius: 4px;
  border: none;
  font-weight: 600;
  color: white;
  cursor: pointer;
  transition: all 0.2s ease;
  opacity: 0.8;
}
.method-btn:hover {
  opacity: 1;
  transform: translateY(-2px);
}
.method-btn.active {
  opacity: 1;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
.get { background-color: #4CAF50; }
.post { background-color: #2196F3; }
.put { background-color: #FF9800; }
.patch { background-color: #9C27B0; }
.delete { background-color: #F44336; }

.method-info-container {
  background: white;
  border-radius: 6px;
  padding: 20px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
}
.method-info {
  display: block;
}
.method-info.hidden {
  display: none;
}
.method-info h4 {
  margin-top: 0;
  margin-bottom: 10px;
  color: #2d3748;
  border-bottom: 2px solid #f1f1f1;
  padding-bottom: 10px;
}
.method-example {
  background: #f1f5f9;
  padding: 15px;
  border-radius: 6px;
  margin: 15px 0;
}
.method-example h5 {
  margin-top: 0;
  margin-bottom: 10px;
  color: #4a5568;
}
.method-example code {
  display: block;
  background: #1e293b;
  color: #e2e8f0;
  padding: 8px 12px;
  border-radius: 4px;
  font-family: monospace;
  margin-bottom: 10px;
}
.method-attributes {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 15px;
}
.attribute {
  background: #f8f9fa;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.9rem;
}
.yes { color: #38a169; font-weight: bold; }
.no { color: #e53e3e; font-weight: bold; }
.maybe { color: #dd6b20; font-weight: bold; }
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const methodButtons = document.querySelectorAll('.method-btn');
  
  methodButtons.forEach(button => {
    button.addEventListener('click', function() {
      // Remove active class from all buttons
      methodButtons.forEach(btn => btn.classList.remove('active'));
      
      // Add active class to clicked button
      this.classList.add('active');
      
      // Get the method name
      const method = this.getAttribute('data-method');
      
      // Hide all method info divs
      document.querySelectorAll('.method-info').forEach(div => {
        div.classList.add('hidden');
      });
      
      // Show the selected method info
      document.getElementById(`${method}-info`).classList.remove('hidden');
    });
  });
});
</script>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>


## Understanding API Status Codes

When you get a text reply, sometimes it's all good, sometimes it's confusing, and sometimes it's just... an error. APIs use **status codes** to tell you how things went.

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>Code Range</th>
        <th>Category</th>
        <th>Common Codes</th>
        <th>Description</th>
        <th>Developer Action</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="code-range success-range">2xx</td>
        <td><span class="category-badge success">Success</span></td>
        <td class="code success-code">200 OK</td>
        <td>The request was successful</td>
        <td>Process the returned data</td>
      </tr>
      <tr class="highlight-row">
        <td></td>
        <td></td>
        <td class="code success-code">201 Created</td>
        <td>New resource created successfully</td>
        <td>Use the returned resource ID for further operations</td>
      </tr>
      <tr>
        <td></td>
        <td></td>
        <td class="code success-code">204 No Content</td>
        <td>Request succeeded, but no content returned</td>
        <td>Continue with application flow, no data to process</td>
      </tr>
      <tr class="highlight-row">
        <td class="code-range redirect-range">3xx</td>
        <td><span class="category-badge redirect">Redirection</span></td>
        <td class="code redirect-code">301 Moved Permanently</td>
        <td>Resource has moved to a new URL</td>
        <td>Update bookmarks and links</td>
      </tr>
      <tr>
        <td></td>
        <td></td>
        <td class="code redirect-code">304 Not Modified</td>
        <td>Resource hasn't changed since last request</td>
        <td>Use cached version of the resource</td>
      </tr>
      <tr class="highlight-row">
        <td class="code-range client-range">4xx</td>
        <td><span class="category-badge client">Client Error</span></td>
        <td class="code client-code">400 Bad Request</td>
        <td>Invalid syntax in the request</td>
        <td>Check request format and parameters</td>
      </tr>
      <tr>
        <td></td>
        <td></td>
        <td class="code client-code">401 Unauthorized</td>
        <td>Authentication required</td>
        <td>Provide authentication credentials</td>
      </tr>
      <tr class="highlight-row">
        <td></td>
        <td></td>
        <td class="code client-code">403 Forbidden</td>
        <td>Client doesn't have access rights</td>
        <td>Request access or use different credentials</td>
      </tr>
      <tr>
        <td></td>
        <td></td>
        <td class="code client-code">404 Not Found</td>
        <td>Resource doesn't exist</td>
        <td>Check the URL and resource ID</td>
      </tr>
      <tr class="highlight-row">
        <td></td>
        <td></td>
        <td class="code client-code">429 Too Many Requests</td>
        <td>Client sent too many requests</td>
        <td>Implement rate limiting or backoff strategy</td>
      </tr>
      <tr>
        <td class="code-range server-range">5xx</td>
        <td><span class="category-badge server">Server Error</span></td>
        <td class="code server-code">500 Internal Server Error</td>
        <td>Server encountered an error</td>
        <td>Report the issue to API provider</td>
      </tr>
      <tr class="highlight-row">
        <td></td>
        <td></td>
        <td class="code server-code">503 Service Unavailable</td>
        <td>Server temporarily unavailable</td>
        <td>Retry the request after a delay</td>
      </tr>
    </tbody>
  </table>
</div>

<style>
.code-range {
  font-weight: bold;
  font-size: 1.1rem;
  text-align: center;
}

.success-range {
  color: #4CAF50;
}

.redirect-range {
  color: #FF9800; 
}

.client-range {
  color: #F44336;
}

.server-range {
  color: #9C27B0;
}

.category-badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 4px;
  color: white;
  font-weight: 600;
  text-align: center;
  min-width: 100px;
}

.success {
  background-color: #4CAF50;
}

.redirect {
  background-color: #FF9800;
}

.client {
  background-color: #F44336;
}

.server {
  background-color: #9C27B0;
}

.code {
  font-family: monospace;
  font-weight: bold;
}

.success-code {
  color: #4CAF50;
}

.redirect-code {
  color: #FF9800;
}

.client-code {
  color: #F44336;
}

.server-code {
  color: #9C27B0;
}
</style>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>


## API Authentication and Security

Many APIs require authentication to ensure only authorized users can access certain resources. Common authentication methods include:

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>Authentication Method</th>
        <th>Implementation</th>
        <th>Ideal For</th>
        <th>Security Level</th>
        <th>Advantages</th>
        <th>Disadvantages</th>
      </tr>
    </thead>
    <tbody>
      <tr class="highlight-row">
        <td><div class="auth-badge api-key">API Keys</div></td>
        <td>Single token sent in header, query param, or cookie</td>
        <td>Public APIs with moderate security needs</td>
        <td>
          <div class="security-meter">
            <div class="security-level" style="width: 40%"></div>
          </div>
          <div class="security-text">Basic</div>
        </td>
        <td>
          <ul class="tight-list">
            <li>Simple to implement</li>
            <li>Low developer friction</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>Can be intercepted</li>
            <li>Limited granular control</li>
          </ul>
        </td>
      </tr>
      <tr>
        <td><div class="auth-badge basic-auth">Basic Auth</div></td>
        <td>Base64 encoded username:password in header</td>
        <td>Internal APIs, development environments</td>
        <td>
          <div class="security-meter">
            <div class="security-level" style="width: 30%"></div>
          </div>
          <div class="security-text">Low</div>
        </td>
        <td>
          <ul class="tight-list">
            <li>Universal support</li>
            <li>Simple to implement</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>Credentials sent with every request</li>
            <li>Base64 is easily decoded</li>
          </ul>
        </td>
      </tr>
      <tr class="highlight-row">
        <td><div class="auth-badge oauth">OAuth 2.0</div></td>
        <td>Token-based authorization framework</td>
        <td>APIs accessing user data across services</td>
        <td>
          <div class="security-meter">
            <div class="security-level" style="width: 90%"></div>
          </div>
          <div class="security-text">High</div>
        </td>
        <td>
          <ul class="tight-list">
            <li>Granular access control</li>
            <li>Token expiration</li>
            <li>Doesn't expose credentials</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>Complex implementation</li>
            <li>Higher developer overhead</li>
          </ul>
        </td>
      </tr>
      <tr>
        <td><div class="auth-badge jwt">JWT</div></td>
        <td>Signed tokens with encoded claims</td>
        <td>Modern web applications, microservices</td>
        <td>
          <div class="security-meter">
            <div class="security-level" style="width: 80%"></div>
          </div>
          <div class="security-text">High</div>
        </td>
        <td>
          <ul class="tight-list">
            <li>Self-contained</li>
            <li>Stateless authentication</li>
            <li>Can contain user data</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>Can't be revoked before expiration</li>
            <li>Size increases with claims</li>
          </ul>
        </td>
      </tr>
    </tbody>
  </table>
</div>

<style>
.auth-badge {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 4px;
  color: white;
  font-weight: bold;
  text-align: center;
  min-width: 100px;
}

.api-key {
  background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
}

.basic-auth {
  background: linear-gradient(135deg, #ff9966 0%, #ff5e62 100%);
}

.oauth {
  background: linear-gradient(135deg, #184e68 0%, #57ca85 100%);
}

.jwt {
  background: linear-gradient(135deg, #662d8c 0%, #ed1e79 100%);
}

.security-meter {
  height: 10px;
  background-color: #e9ecef;
  border-radius: 5px;
  overflow: hidden;
  margin-bottom: 5px;
}

.security-level {
  height: 100%;
  background-color: #4CAF50;
}

.security-text {
  font-weight: bold;
  font-size: 0.8rem;
  text-align: center;
}
</style>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## How to Test API Requests

Now that you understand API requests and responses, how do you test them? These tools will help you explore and validate APIs:

<div class="table-container">
  <table class="custom-table tools-table">
    <thead>
      <tr>
        <th>Tool</th>
        <th>Category</th>
        <th>Key Features</th>
        <th>Best For</th>
        <th>Complexity</th>
        <th>Price</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>
          <a href="https://www.postman.com/" target="_blank" class="tool-link">
            <img src="https://www.vectorlogo.zone/logos/getpostman/getpostman-icon.svg" alt="Postman logo" class="tool-logo">
            <span>Postman</span>
          </a>
        </td>
        <td>GUI Application</td>
        <td>
          <ul class="tight-list">
            <li>Request collections</li>
            <li>Environment variables</li>
            <li>Automated testing</li>
            <li>Team collaboration</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>API development</li>
            <li>Comprehensive testing</li>
            <li>Team workflows</li>
          </ul>
        </td>
        <td>
          <div class="complexity-meter">
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot"></span>
            <span class="dot"></span>
          </div>
          <div>Medium</div>
        </td>
        <td>
          <div class="price">
            <span class="price-tag">Free tier</span>
            <span>+ Paid plans</span>
          </div>
        </td>
      </tr>
      <tr class="highlight-row">
        <td>
          <a class="tool-link terminal-tool">
            <span class="terminal-icon">></span>
            <span>cURL</span>
          </a>
        </td>
        <td>Command Line</td>
        <td>
          <ul class="tight-list">
            <li>Universal availability</li>
            <li>Scriptable</li>
            <li>Detailed control</li>
            <li>SSL support</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>CI/CD integration</li>
            <li>Shell scripting</li>
            <li>Quick testing</li>
          </ul>
        </td>
        <td>
          <div class="complexity-meter">
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot"></span>
          </div>
          <div>Medium-High</div>
        </td>
        <td>
          <div class="price">
            <span class="price-tag free">Free</span>
            <span>Open Source</span>
          </div>
        </td>
      </tr>
      <tr>
        <td>
          <a href="https://swagger.io/" target="_blank" class="tool-link">
            <img src="https://static1.smartbear.co/swagger/media/assets/swagger_fav.png" alt="Swagger logo" class="tool-logo">
            <span>Swagger UI</span>
          </a>
        </td>
        <td>Interactive Docs</td>
        <td>
          <ul class="tight-list">
            <li>OpenAPI integration</li>
            <li>Try-it-out feature</li>
            <li>Self-documenting</li>
            <li>Code generation</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>API exploration</li>
            <li>Developer portals</li>
            <li>Documentation</li>
          </ul>
        </td>
        <td>
          <div class="complexity-meter">
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot"></span>
            <span class="dot"></span>
            <span class="dot"></span>
          </div>
          <div>Low</div>
        </td>
        <td>
          <div class="price">
            <span class="price-tag free">Free</span>
            <span>Open Source</span>
          </div>
        </td>
      </tr>
      <tr class="highlight-row">
        <td>
          <a href="https://insomnia.rest/" target="_blank" class="tool-link">
            <img src="https://insomnia.rest/images/insomnia-logo.svg" alt="Insomnia logo" class="tool-logo">
            <span>Insomnia</span>
          </a>
        </td>
        <td>GUI Application</td>
        <td>
          <ul class="tight-list">
            <li>Clean interface</li>
            <li>GraphQL support</li>
            <li>Plugin system</li>
            <li>Design spec</li>
          </ul>
        </td>
        <td>
          <ul class="tight-list">
            <li>GraphQL APIs</li>
            <li>Individual developers</li>
            <li>Design-first approach</li>
          </ul>
        </td>
        <td>
          <div class="complexity-meter">
            <span class="dot filled"></span>
            <span class="dot filled"></span>
            <span class="dot"></span>
            <span class="dot"></span>
            <span class="dot"></span>
          </div>
          <div>Low</div>
        </td>
        <td>
          <div class="price">
            <span class="price-tag">Free tier</span>
            <span>+ Paid plans</span>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</div>

<style>
.tools-table td {
  vertical-align: top;
}

.tool-link {
  display: flex;
  align-items: center;
  color: #2c3e50;
  text-decoration: none;
  font-weight: bold;
}

.tool-logo {
  width: 24px;
  height: 24px;
  margin-right: 8px;
}

.terminal-icon {
  display: inline-block;
  width: 24px;
  height: 24px;
  background-color: #000;
  color: #fff;
  border-radius: 4px;
  text-align: center;
  line-height: 24px;
  margin-right: 8px;
  font-family: monospace;
  font-weight: bold;
}

.terminal-tool {
  cursor: default;
}

.complexity-meter {
  display: flex;
  align-items: center;
  margin-bottom: 5px;
}

.dot {
  height: 10px;
  width: 10px;
  background-color: #e9ecef;
  border-radius: 50%;
  display: inline-block;
  margin-right: 3px;
}

.dot.filled {
  background-color: #2c3e50;
}

.price {
  display: flex;
  flex-direction: column;
}

.price-tag {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 4px;
  background-color: #f8f9fa;
  font-size: 0.8rem;
  font-weight: 600;
  margin-bottom: 4px;
}

.price-tag.free {
  background-color: #4CAF50;
  color: white;
}
</style>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## Common API Documentation Components

When [documenting APIs](/apidocumentation/whatwritersdo.html), it's important to include:

1. **Endpoint details**: Base URL and path
2. **HTTP method**: GET, POST, PUT, DELETE, etc.
3. **Request parameters**: Query parameters, path parameters
4. **Request headers**: Content-Type, Authorization, etc.
5. **Request body schema**: For POST/PUT requests
6. **Response structure**: What the API returns
7. **Status codes**: Possible response codes and their meanings
8. **Examples**: Sample requests and responses

{% include enhanced_note.html 
  title="Important" 
  type="info" 
  collapsible=true 
  expanded=true 
  content="REST API Best Practice" content="In RESTful APIs, HTTP methods are mapped to CRUD operations: GET (Read), POST (Create), PUT/PATCH (Update), and DELETE (Delete). The 'Safe' property means the method doesn't change server state, while 'Idempotent' means multiple identical requests have the same effect as a single request." 
%}

### Example: A POST Request for Creating Resources {#post-request-example}

```sh
POST https://api.example.com/books Content-Type: application/json
```

```json
{
  "title": "The Hobbit",
  "author": "J.R.R. Tolkien",
  "published_year": 1937
}
```

This tells the API: "Hey, add this book to the database!" The Content-Type header specifies we're sending [JSON](/JSON101.html) data.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>


## Understanding API Status Codes {#status-codes}

When you get a text reply, sometimes it's all good, sometimes it's confusing, and sometimes it's just... an error. APIs use **status codes** to tell you how things went.


<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## API Authentication and Security {#api-authentication}

Many APIs require authentication to ensure only authorized users can access certain resources. Common authentication methods include:

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## How to Test API Requests {#testing-api-requests}

Now that you understand API requests and responses, how do you test them? These tools will help you explore and validate APIs:

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## Common API Request and Response Patterns {#common-patterns}

When working with APIs, you'll encounter several standard patterns in the requests and responses. Understanding these patterns helps you navigate API documentation more effectively and implement API integrations correctly.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

### Pagination in API Responses {#pagination}

When dealing with large sets of data, APIs often use pagination to return results in manageable chunks:

```json
{
  "books": [
    { "id": 1, "title": "Book One" },
    { "id": 2, "title": "Book Two" }
  ],
  "pagination": {
    "total_items": 42,
    "items_per_page": 2,
    "current_page": 1,
    "total_pages": 21,
    "next_page": "/books?page=2",
    "prev_page": null
  }
}
```

### Filtering and Sorting {#filtering-sorting}

Most APIs allow you to refine your results through query parameters:

- **Filtering**: `GET /books?genre=fantasy&year=2023`
- **Sorting**: `GET /books?sort=title&order=asc`
- **Field selection**: `GET /books?fields=title,author,year`

### Error Handling {#error-handling}

A well-designed API provides detailed error information:

```json
{
  "error": {
    "code": "validation_error",
    "message": "The request contains invalid parameters",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ]
  }
}
```

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7149683584202371"
      crossorigin="anonymous"></script>
  <!-- AddTitleOne -->
  <ins class="adsbygoogle"
      style="display:block"
      data-ad-client="ca-pub-7149683584202371"
      data-ad-slot="7422872052"
      data-ad-format="auto"
      data-full-width-responsive="true"></ins>
  <script>
      (adsbygoogle = window.adsbygoogle || []).push({});
  </script>

## Best Practices for Working with API Requests and Responses {#best-practices}

To ensure smooth interaction with APIs, follow these best practices:

1. **Always check status codes** to understand if your request succeeded or failed
2. **Validate request data** before sending it to prevent validation errors
3. **Implement proper error handling** to gracefully manage API failures
4. **Use request IDs** for tracking and debugging purposes
5. **Cache responses** when appropriate to reduce unnecessary API calls
6. **Handle rate limiting** by tracking API usage and implementing backoff strategies
7. **Stay up-to-date** with API changes through changelogs and documentation

{% include faq-section.html 
  title="Frequently Asked Questions About API Requests and Responses"
  description="Get answers to common questions about working with API requests and responses."
  data_file="api_requests_responses_faqs"
%}

{% include key_takeaways.html content="
<ul>
  <li>API requests are messages sent to servers asking them to perform specific actions</li>
  <li>API responses contain the data or confirmation sent back from the server</li>
  <li>HTTP methods (GET, POST, PUT, DELETE) define the action to be performed</li>
  <li>Status codes indicate the outcome of a request (200s for success, 400s for client errors, 500s for server errors)</li>
  <li>Authentication methods secure APIs against unauthorized access</li>
  <li>Tools like Postman, cURL, and Swagger help test and explore APIs</li>
  <li>Comprehensive API documentation includes endpoints, parameters, examples, and response schemas</li>
</ul>
" %}

Congrats! You now understand how API requests and responses work. Whether it's fetching data, sending new information, or handling errors, you've got the basics down.

Now that you know how APIs send and receive data, it's time to document them effectively! In the next chapter, we'll dive into how to write clear and structured API documentation that developers will love. Get ready to level up your documentation skills!

{% include quiz.html
  title="Test Your Knowledge"
  description="Check your understanding of request and response with this quiz."
  quizDataFile="requestandresponse_questions"
  theme="blue"
  animate=true
%}

<div class="author-cta">
  <img src="{{ site.baseurl }}/assets/images/gaurav.svg" alt="Technical Writing Expert" class="author-image">
  <div class="author-message">
    <h4>Did this API requests and responses guide help you?</h4>
    <p>If you found this guide on API requests, responses, status codes, and authentication valuable, please share it with developers in your network. Your feedback helps improve our API documentation resources!</p>
    <div class="social-share">
      <a href="https://twitter.com/intent/tweet?url={{ site.url }}{{ page.url }}&text=Excellent guide on mastering API requests and responses" class="share-button twitter">Share on Twitter</a>
      <a href="https://www.linkedin.com/shareArticle?mini=true&url={{ site.url }}{{ page.url }}&title=Understanding API Requests and Responses" class="share-button linkedin">Share on LinkedIn</a>
    </div>
  </div>
</div>

<style>
.author-cta {
  display: flex;
  background: #f8f9fb;
  border-radius: 8px;
  padding: 20px;
  margin: 30px 0;
  border: 1px solid #e2e8f0;
  gap: 20px;
  align-items: center;
}
.author-image {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  object-fit: cover;
}
.author-message {
  flex: 1;
}
.author-message h4 {
  margin-top: 0;
  margin-bottom: 8px;
}
.author-message p {
  margin-bottom: 12px;
}
.social-share {
  display: flex;
  gap: 10px;
}
.share-button {
  display: inline-block;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 500;
  text-decoration: none;
  color: white;
}
.twitter {
  background: #1DA1F2;
}
.linkedin {
  background: #0077B5;
}
@media (max-width: 600px) {
  .author-cta {
    flex-direction: column;
    text-align: center;
  }
  .social-share {
    justify-content: center;
  }
}
</style>

{% include related_resources.html 
  title="Essential API Request & Response Resources"
  description="Deepen your understanding of API communication with these carefully selected resources."
  external_links="HTTP Status Codes Reference,https://httpstatuses.com/|Mozilla HTTP Documentation,https://developer.mozilla.org/en-US/docs/Web/HTTP|API Security Best Practices,https://owasp.org/www-project-api-security/|JSON Schema Documentation,https://json-schema.org/"
  tools="Postman API Platform,https://www.postman.com/|Insomnia REST Client,https://insomnia.rest/|HTTPie for API Testing,https://httpie.io/|JSON Formatter & Validator,https://jsonformatter.curiousconcept.com/"
%}