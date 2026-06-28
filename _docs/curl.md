---
module: 2
lesson: 7
slug: curl
title: cURL Best Practices for Technical Writers
description: "A cURL tutorial for technical writers: run requests, set auth headers, post JSON, and read responses, with best practices that keep API docs accurate."
keywords: REST API documentation, API writing exercises, API reference, API documentation practice, API documentation test, API writing test, Technical Writing API exercises, API documentation challenges, API documentation best practices, hands-on API documentation, advanced API writing, API error handling, API authentication, cURL API testing, API debugging, command-line API testing, cURL commands, API request headers, API payloads, HTTP methods cURL
permalink: /curl.html
next_page:
  url: /JSON101.html
  title: "JSON like you mean it"
previous_page:
  url: /pagination-idempotency-rate-limits.html
  title: "Pagination, idempotency, rate limits"
image: /assets/images/api-docs-curl.svg
last_modified_at: 2023-10-18T09:00:00+00:00
author_name: Technical Writing Expert
author_description: Senior technical writer specializing in API documentation and command line tools
author_expertise: 
  - "API Documentation"
  - "Technical Writing"
  - "cURL"
  - "Command Line Tools"
reading_time: 15
level: Intermediate
speakable: true
speakable_selectors:
  - ".doc-content h1" 
  - ".doc-content h2"
  - ".doc-content p:first-of-type"
---

You've already tackled **API fundamentals**—rate limits, authentication, pagination, and error handling. Now, it's time to **level up** and **master cURL**, the command-line Swiss Army knife for API testing. 

{% include trivia.html 
  type="tech" 
  icon="fa-terminal" 
  title="cURL History" 
  content="Daniel Stenberg created cURL in 1997, and it's now installed on billions of devices worldwide. The name originally stood for 'Client for URLs' but is now simply known as cURL, a versatile tool that supports dozens of protocols beyond HTTP."
%}

{% include enhanced_note.html 
  title="Why Every API Professional Needs cURL" 
  type="important" 
  collapsible=true 
  expanded=true 
  content="Whether you're documenting, developing, or testing APIs, cURL allows you to test endpoints without writing code, authenticate requests, and debug responses - all from your command line. It's a universal tool that works on nearly all operating systems." 
%}

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

## What You'll Learn in This Chapter

After completing this chapter, you'll be able to:

- Execute **API requests** using various HTTP methods (GET, POST, PUT, DELETE)
- Include **headers and authentication tokens** in your requests
- Send **data payloads** in JSON and form-encoded formats
- **Upload and download files** through API endpoints
- **Debug API responses** using cURL's powerful features
- **Troubleshoot** common API issues directly from the command line

Let's get started with the essentials!

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

## What is cURL?

cURL (short for *Client URL*) is a **command-line tool** used for sending HTTP requests to APIs. It's like **Postman**, but without the fancy UI.

A basic cURL command looks like this:

```sh
curl https://api.example.com
```

This sends a **GET request** to `https://api.example.com`. Simple, right? Now, let's explore more powerful features.

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

## Sending API Requests with cURL

### HTTP Methods and cURL Commands

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>HTTP Method</th>
        <th>cURL Command</th>
        <th>Purpose</th>
        <th>Example Use Case</th>
      </tr>
    </thead>
    <tbody>
      <tr class="highlight-row">
        <td><span class="http-method get">GET</span></td>
        <td><code>curl -X GET "https://api.example.com/users"</code></td>
        <td>Retrieve data</td>
        <td>Fetch user profiles, product listings</td>
      </tr>
      <tr>
        <td><span class="http-method post">POST</span></td>
        <td><code>curl -X POST "https://api.example.com/users" -d '{"name":"John"}' -H "Content-Type: application/json"</code></td>
        <td>Create new data</td>
        <td>Create new user, submit form data</td>
      </tr>
      <tr class="highlight-row">
        <td><span class="http-method put">PUT</span></td>
        <td><code>curl -X PUT "https://api.example.com/users/1" -d '{"name":"John Updated"}' -H "Content-Type: application/json"</code></td>
        <td>Update existing data</td>
        <td>Update user profile, replace a resource</td>
      </tr>
      <tr>
        <td><span class="http-method delete">DELETE</span></td>
        <td><code>curl -X DELETE "https://api.example.com/users/1"</code></td>
        <td>Remove data</td>
        <td>Delete a user account, remove a post</td>
      </tr>
      <tr class="highlight-row">
        <td><span class="http-method patch">PATCH</span></td>
        <td><code>curl -X PATCH "https://api.example.com/users/1" -d '{"status":"active"}' -H "Content-Type: application/json"</code></td>
        <td>Partially update data</td>
        <td>Update specific fields only</td>
      </tr>
    </tbody>
  </table>
</div>

### Adding Request Headers

Headers provide additional information to the API server. Use the `-H` flag to include headers:

```sh
curl -X GET "https://api.example.com" -H "Authorization: Bearer your_token" -H "Accept: application/json"
```

### Common API Request Headers

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>Header</th>
        <th>Purpose</th>
        <th>Example</th>
      </tr>
    </thead>
    <tbody>
      <tr class="highlight-row">
        <td><strong>Authorization</strong></td>
        <td>Authenticate requests</td>
        <td><code>-H "Authorization: Bearer token123"</code></td>
      </tr>
      <tr>
        <td><strong>Content-Type</strong></td>
        <td>Specify request body format</td>
        <td><code>-H "Content-Type: application/json"</code></td>
      </tr>
      <tr class="highlight-row">
        <td><strong>Accept</strong></td>
        <td>Specify desired response format</td>
        <td><code>-H "Accept: application/json"</code></td>
      </tr>
      <tr>
        <td><strong>User-Agent</strong></td>
        <td>Identify client application</td>
        <td><code>-H "User-Agent: MyApiClient/1.0"</code></td>
      </tr>
    </tbody>
  </table>
</div>

### Sending Query Parameters

To filter or sort data, include query parameters in your URL:

```sh
curl -X GET "https://api.example.com/users?limit=10&sort=desc&status=active"
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


## Working with Request Data

### Sending JSON Data

Use the `-d` flag to send JSON data in **POST**, **PUT**, or **PATCH** requests:

```sh
curl -X POST "https://api.example.com/users" \
     -d '{"name":"John Doe","email":"john@example.com","role":"admin"}' \
     -H "Content-Type: application/json"
```

### Sending Form Data

For form submissions, use `--data-urlencode` to properly handle special characters:

```sh
curl -X POST "https://api.example.com/login" \
     --data-urlencode "username=johndoe" \
     --data-urlencode "password=s3cure!p@ss"
```

{% include enhanced_note.html 
  title="Tip" 
  type="tip" 
  collapsible=false 
  expanded=true 
  content="Use the backslash character \\ at the end of a line to break long cURL commands into multiple lines for better readability." 
%}

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

## Authentication Methods with cURL

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>Authentication Type</th>
        <th>cURL Command</th>
        <th>When to Use</th>
      </tr>
    </thead>
    <tbody>
      <tr class="highlight-row">
        <td><strong>API Key</strong></td>
        <td><code>curl -H "Authorization: ApiKey your_api_key" "https://api.example.com"</code></td>
        <td>Simple API authentication</td>
      </tr>
      <tr>
        <td><strong>Bearer Token</strong></td>
        <td><code>curl -H "Authorization: Bearer your_token" "https://api.example.com"</code></td>
        <td>OAuth or JWT authentication</td>
      </tr>
      <tr class="highlight-row">
        <td><strong>Basic Auth</strong></td>
        <td><code>curl -u username:password "https://api.example.com"</code></td>
        <td>Username/password authentication</td>
      </tr>
      <tr>
        <td><strong>OAuth 2.0</strong></td>
        <td><code>curl -H "Authorization: Bearer oauth_token" "https://api.example.com"</code></td>
        <td>Third-party API access</td>
      </tr>
    </tbody>
  </table>
</div>

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

## File Operations with cURL

### Uploading Files

The `-F` flag allows you to upload files using multipart form data:

```sh
curl -X POST "https://api.example.com/upload" \
     -F "file=@/path/to/myfile.jpg" \
     -F "description=My vacation photo" \
     -H "Authorization: Bearer your_token"
```

### Downloading Files

Save API responses directly to files:

```sh
# Save with original filename
curl -O "https://api.example.com/files/document.pdf"

# Save with custom filename
curl -o custom_name.pdf "https://api.example.com/files/document.pdf"
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

## Debugging API Calls

### Viewing Response Headers

Use the `-i` flag to see both the headers and the response body:

```sh
curl -i "https://api.example.com/users/1"
```

### Using Verbose Mode

The `-v` flag shows the complete request and response flow, perfect for debugging:

```sh
curl -v "https://api.example.com/users/1"
```

Example output:
```
> GET /users/1 HTTP/1.1
> Host: api.example.com
> User-Agent: curl/7.68.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json
< Content-Length: 234
<
{"id":1,"name":"John Doe","email":"john@example.com"}
```

### Silent Mode

For scripting, use `-s` to suppress progress meters and error messages:

```sh
curl -s "https://api.example.com/users/1" | jq
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

## Advanced cURL Features

<div class="table-container">
  <table class="custom-table">
    <thead>
      <tr>
        <th>Feature</th>
        <th>Flag</th>
        <th>Example</th>
        <th>Purpose</th>
      </tr>
    </thead>
    <tbody>
      <tr class="highlight-row">
        <td>Follow Redirects</td>
        <td><code>-L</code></td>
        <td><code>curl -L "https://api.example.com/redirect"</code></td>
        <td>Automatically follow HTTP redirects</td>
      </tr>
      <tr>
        <td>Request Timeout</td>
        <td><code>--connect-timeout</code></td>
        <td><code>curl --connect-timeout 10 "https://api.example.com"</code></td>
        <td>Set connection timeout in seconds</td>
      </tr>
      <tr class="highlight-row">
        <td>Max Time</td>
        <td><code>--max-time</code></td>
        <td><code>curl --max-time 30 "https://api.example.com"</code></td>
        <td>Set maximum time allowed for operation</td>
      </tr>
      <tr>
        <td>Retry Requests</td>
        <td><code>--retry</code></td>
        <td><code>curl --retry 3 "https://api.example.com"</code></td>
        <td>Retry failed requests</td>
      </tr>
      <tr class="highlight-row">
        <td>Custom Request Method</td>
        <td><code>-X</code></td>
        <td><code>curl -X PATCH "https://api.example.com"</code></td>
        <td>Use specific HTTP method</td>
      </tr>
    </tbody>
  </table>
</div>

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

## Practical cURL Examples for API Documentation

### 1. Fetching resources with authentication

```sh
curl -X GET "https://api.example.com/users" \
     -H "Authorization: Bearer your_token" \
     -H "Accept: application/json"
```

### 2. Creating a new resource

```sh
curl -X POST "https://api.example.com/products" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer your_token" \
     -d '{
       "name": "New Product",
       "price": 29.99,
       "description": "This is a new product",
       "category": "electronics"
     }'
```

### 3. Updating a resource

```sh
curl -X PUT "https://api.example.com/products/123" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer your_token" \
     -d '{
       "name": "Updated Product Name",
       "price": 39.99,
       "description": "This product has been updated",
       "category": "electronics"
     }'
```

### 4. Deleting a resource

```sh
curl -X DELETE "https://api.example.com/products/123" \
     -H "Authorization: Bearer your_token"
```

<style>
.table-container {
  overflow-x: auto;
  margin: 25px 0;
  border-radius: 8px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.custom-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  background: white;
  font-size: 0.95rem;
}

.custom-table th {
  background: #2c3e50;
  color: white;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  padding: 16px;
  text-align: left;
  border: none;
}

.custom-table td {
  padding: 12px 16px;
  border-bottom: 1px solid #e9ecef;
  vertical-align: middle;
}

.highlight-row {
  background-color: #f8f9fa;
}

.http-method {
  display: inline-block;
  padding: 5px 8px;
  border-radius: 4px;
  font-family: monospace;
  font-weight: bold;
  color: white;
  text-align: center;
  min-width: 60px;
}

.get {
  background-color: #4CAF50;
}

.post {
  background-color: #2196F3;
}

.put {
  background-color: #FF9800;
}

.delete {
  background-color: #F44336;
}

.patch {
  background-color: #9C27B0;
}

code {
  background-color: #f5f5f5;
  padding: 3px 5px;
  border-radius: 4px;
  font-family: monospace;
  font-size: 0.9em;
  color: #e83e8c;
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

## Putting It All Together

cURL is an incredibly powerful tool for:

- **Testing APIs** before implementing them in code
- **Documenting API endpoints** with clear examples
- **Debugging API issues** with detailed request/response information
- **Automating API workflows** with shell scripts
- **Validating API responses** against expected results

{% include key_takeaways.html content="
<ul>
  <li>cURL allows you to interact with APIs directly from the command line</li>
  <li>Use different HTTP methods with the -X flag (GET, POST, PUT, DELETE, PATCH)</li>
  <li>Add request headers with -H for authentication and content type specification</li>
  <li>Send data payloads with -d for JSON or --data-urlencode for form data</li>
  <li>Upload files using -F and download using -O or -o</li>
  <li>Debug API calls using the -v (verbose) and -i (include headers) flags</li>
</ul>
" %}

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

<div class="interactive-quiz">
  <h3>Test Your cURL Knowledge</h3>
  <div class="quiz-question">
    <p>What cURL command would you use to send a POST request with JSON data and include the response headers in the output?</p>
    <button class="quiz-button" onclick="toggleAnswer()">Show Answer</button>
    <div class="quiz-answer" id="quiz-answer" style="display: none;">
      <p><strong>Command:</strong></p>
      <pre><code>curl -X POST "https://api.example.com/endpoint" \
     -H "Content-Type: application/json" \
     -d '{"key":"value"}' \
     -i</code></pre>
      <p><strong>Explanation:</strong></p>
      <ul>
        <li><code>-X POST</code> specifies the POST method</li>
        <li><code>-H "Content-Type: application/json"</code> sets the content type header</li>
        <li><code>-d '{"key":"value"}'</code> provides the JSON data payload</li>
        <li><code>-i</code> includes the response headers in the output</li>
      </ul>
    </div>
  </div>
</div>

<script>
  function toggleAnswer() {
    var answerDiv = document.getElementById('quiz-answer');
    var button = document.querySelector('.quiz-button');
    
    if (answerDiv.style.display === 'none') {
      answerDiv.style.display = 'block';
      button.textContent = 'Hide Answer';
    } else {
      answerDiv.style.display = 'none';
      button.textContent = 'Show Answer';
    }
  }
</script>

<style>
  .interactive-quiz {
    background-color: #f0f7fb;
    border-left: 5px solid #3a87ad;
    padding: 20px;
    margin: 30px 0;
    border-radius: 0 8px 8px 0;
  }
  
  .interactive-quiz h3 {
    color: #3a87ad;
    margin-top: 0;
    margin-bottom: 15px;
  }
  
  .quiz-question p {
    margin-bottom: 10px;
  }
  
  .quiz-button {
    background-color: #3a87ad;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
    font-weight: 500;
    margin-top: 10px;
    transition: background-color 0.2s ease;
  }
  
  .quiz-button:hover {
    background-color: #2a6d8e;
  }
  
  .quiz-answer {
    margin-top: 15px;
    background-color: white;
    padding: 15px;
    border-radius: 4px;
    border-left: 3px solid #28a745;
  }
  
  .quiz-answer pre {
    background-color: #f6f8fa;
    padding: 12px;
    border-radius: 4px;
    overflow-x: auto;
  }
  
  .quiz-answer code {
    font-family: monospace;
  }
  
  .quiz-answer ul {
    margin-bottom: 0;
  }
</style>

{% include faq-section.html 
  title="Frequently Asked Questions About cURL for API Testing"
  description="Get answers to common questions about using cURL for API testing and documentation."
  data_file="curl_faqs"
%}

Congratulations! You're now **fluent in cURL**, ready to test APIs like a pro. You're fully prepared for **intermediate and advanced-level API documentation challenges**.

In the next chapter, we'll put your skills to the test with [expert REST API exercises](/apidocumentation/expertrestexercise.html) that combine everything you've learned so far.

{% include quiz.html
  title="Test Your Knowledge"
  description="Check your understanding of cURL for API testing with this quiz."
  quizDataFile="curl_questions"
  theme="blue"
  animate=true
%}


<div class="author-cta">
  <img src="{{ site.baseurl }}/assets/images/gaurav.svg" alt="Technical Writing Expert" class="author-image">
  <div class="author-message">
    <h4>Level Up Your API Testing Skills</h4>
    <p>Has this cURL guide helped you master API testing from the command line? Bookmark it for reference or share it with your team to improve your API documentation workflow!</p>
    <div class="social-share">
      <a href="https://twitter.com/intent/tweet?url={{ site.url }}{{ page.url }}&text=Check out this comprehensive guide to mastering cURL for API testing and documentation" class="share-button twitter">Share on Twitter</a>
      <a href="https://www.linkedin.com/shareArticle?mini=true&url={{ site.url }}{{ page.url }}&title=Mastering cURL for API Testing and Documentation" class="share-button linkedin">Share on LinkedIn</a>
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
  title="API Testing Resources"
  description="Explore more tools and resources for API testing and documentation."
  external_links="Official cURL Documentation,https://curl.se/docs/|HTTP Status Code Reference,https://developer.mozilla.org/en-US/docs/Web/HTTP/Status|JSON Formatter for API Responses,https://jsonformatter.org/"
  tools="Postman API Platform,https://www.postman.com/|jq - JSON processor,https://stedolan.github.io/jq/|HTTPie - Modern HTTP Client,https://httpie.io/"
%}

