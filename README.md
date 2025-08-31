# n8n: The Complete Practical Guide
## From Basics to Advanced Automation

### Table of Contents

**Part I: Foundations**
1. Introduction to n8n
2. Getting Started with n8n
3. Core Concepts and Architecture
4. Building Your First Workflows

**Part II: Practical Workflows**
5. Data Integration Workflows
6. API Automation Examples
7. Database Operations
8. File Processing Automation
9. Communication Workflows (Email, Slack, Discord)
10. Web Scraping and Data Extraction

**Part III: Advanced Techniques**
11. Error Handling and Debugging
12. Advanced Node Operations
13. Custom Functions and Code
14. Webhooks and Triggers
15. Authentication and Security

**Part IV: Real-World Projects**
16. Building a Customer Support Automation System
17. Creating a Social Media Management Pipeline
18. E-commerce Order Processing Workflow
19. Data Analytics and Reporting Dashboard
20. CI/CD Integration Workflows

**Part V: Deployment and Best Practices**
21. Self-Hosting n8n
22. Cloud Deployment Options
23. Performance Optimization
24. Monitoring and Maintenance
25. Best Practices and Design Patterns

---

# Introduction

Welcome to the complete practical guide to n8n - the powerful workflow automation tool that's revolutionizing how businesses and developers approach task automation. This book is designed to take you from a complete beginner to an n8n expert through hands-on examples and real-world projects.

## Who This Book Is For

This book is perfect for:
- Developers looking to automate repetitive tasks
- Business analysts wanting to create no-code/low-code solutions
- System administrators managing complex integrations
- Entrepreneurs building automated business processes
- Anyone interested in workflow automation

## What You'll Learn

By the end of this book, you'll be able to:
- Build complex multi-step workflows
- Integrate hundreds of different services
- Process and transform data efficiently
- Create custom nodes and functions
- Deploy and maintain n8n in production
- Design scalable automation architectures

## How to Use This Book

Each chapter includes:
- **Conceptual explanations** to understand the "why"
- **Step-by-step tutorials** with screenshots
- **Practical examples** you can implement immediately
- **Pro tips** from real-world experience
- **Troubleshooting guides** for common issues
- **Exercises** to reinforce your learning

---

# Chapter 1: Getting Started with n8n

## What is n8n?

n8n (pronounced "nodemation") is a free, open-source workflow automation tool that allows you to connect different services and automate tasks without extensive coding. Think of it as IFTTT or Zapier on steroids, but with the added benefit of being self-hostable and highly customizable.

### Key Features

- **Visual Workflow Editor**: Drag-and-drop interface for building workflows
- **350+ Integrations**: Pre-built nodes for popular services
- **Self-Hostable**: Full control over your data and infrastructure
- **Code When Needed**: JavaScript support for custom logic
- **Fair-Code License**: Source available with sustainable business model

## Installation Methods

### Method 1: Docker (Recommended for Production)

```bash
# Pull the n8n image
docker pull n8nio/n8n

# Run n8n with persistent data
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Method 2: npm (Quick Start)

```bash
# Install globally
npm install n8n -g

# Start n8n
n8n start
```

### Method 3: Docker Compose (Production Setup)

Create a `docker-compose.yml` file:

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=password
      - N8N_HOST=n8n.yourdomain.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://n8n.yourdomain.com/
    volumes:
      - ./n8n_data:/home/node/.n8n
      - ./local_files:/files
```

Then run:
```bash
docker-compose up -d
```

## Your First Workflow

Let's create a simple workflow that demonstrates n8n's power:

### Example 1: Daily Weather Report via Email

**Goal**: Automatically send yourself a weather report every morning at 8 AM.

**Nodes needed**:
1. Cron (Trigger)
2. OpenWeatherMap (Data)
3. Email (Action)

**Step-by-step**:

1. **Add Cron Node**:
   - Click "+" to add a new node
   - Search for "Cron"
   - Set to trigger daily at 8:00 AM

2. **Add OpenWeatherMap Node**:
   - Add new node after Cron
   - Configure with your API key
   - Set your city
   - Choose "Current Weather" operation

3. **Add Email Node**:
   - Connect to OpenWeatherMap output
   - Configure SMTP settings
   - Use expressions to format the email:
   
```javascript
Subject: Weather Report for {{$json["name"]}}
Body:
Current Temperature: {{$json["main"]["temp"]}}°C
Feels Like: {{$json["main"]["feels_like"]}}°C
Conditions: {{$json["weather"][0]["description"]}}
Humidity: {{$json["main"]["humidity"]}}%
Wind Speed: {{$json["wind"]["speed"]}} m/s
```

4. **Save and Activate**:
   - Click "Save"
   - Toggle workflow to "Active"

---

# Chapter 2: Core Concepts

## Understanding Nodes

Nodes are the building blocks of n8n workflows. Each node represents an action or trigger.

### Node Types

1. **Trigger Nodes** (Green circle icon):
   - Start workflows
   - Examples: Webhook, Cron, Email Trigger
   - Can only be placed at the beginning

2. **Action Nodes** (Regular icons):
   - Perform operations
   - Examples: HTTP Request, Database, Transform

3. **Core Nodes** (Special operations):
   - IF: Conditional branching
   - Switch: Multiple condition routing
   - Merge: Combine data streams
   - SplitInBatches: Process large datasets

## Data Flow

### JSON Structure
n8n passes data between nodes as JSON. Understanding this is crucial:

```javascript
// Single item
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

// Multiple items (array)
[
  {"name": "John", "age": 30},
  {"name": "Jane", "age": 25}
]
```

### Accessing Data with Expressions

n8n uses expressions to reference data from previous nodes:

- `{{$json["field"]}}` - Current node's input
- `{{$node["NodeName"].json["field"]}}` - Specific node's output
- `{{$items()[0].json}}` - First item in array
- `{{$now}}` - Current timestamp
- `{{$today}}` - Today's date

### Example: Data Transformation

```javascript
// Input from previous node
{
  "user": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "JOHN@EXAMPLE.COM"
  }
}

// Function node to transform
const items = $input.all();
return items.map(item => {
  return {
    json: {
      fullName: `${item.json.user.firstName} ${item.json.user.lastName}`,
      email: item.json.user.email.toLowerCase(),
      processed: new Date().toISOString()
    }
  };
});
```

## Error Handling

### Error Workflow
Create a dedicated error handling workflow:

1. **Settings** → **Error Workflow**
2. Create new workflow with "Error Trigger" node
3. Add notifications (Email, Slack, etc.)

### Node Error Handling
Configure per-node error behavior:
- **Stop Workflow**: Default, stops on error
- **Continue**: Ignores error, continues execution
- **Continue Using Error Output**: Routes errors to specific branch

---

# Chapter 3: Basic Workflows

## Workflow 1: RSS Feed to Slack

**Use Case**: Monitor industry news and post to team Slack channel

**Nodes**:
1. Cron (every hour)
2. RSS Feed Read
3. IF (check if new items)
4. Slack

**Implementation**:

```javascript
// RSS Feed Read configuration
URL: https://techcrunch.com/feed/
// Outputs array of articles

// IF node condition
{{$json["pubDate"]}} > {{DateTime.now().minus({hours: 1}).toISO()}}

// Slack message formatting
New Article: *{{$json["title"]}}*
{{$json["description"]}}
Read more: {{$json["link"]}}
```

## Workflow 2: CSV Processing Pipeline

**Use Case**: Daily CSV processing from FTP to database

**Nodes**:
1. Cron (daily trigger)
2. FTP (download file)
3. Spreadsheet File (read CSV)
4. Item Lists (transform data)
5. MySQL (insert records)

**Code for transformation**:

```javascript
// Clean and validate data
return items.map(item => {
  // Data validation
  if (!item.json.email || !item.json.email.includes('@')) {
    throw new Error(`Invalid email: ${item.json.email}`);
  }
  
  return {
    json: {
      email: item.json.email.trim().toLowerCase(),
      name: item.json.name.trim(),
      phone: item.json.phone.replace(/[^\d]/g, ''),
      created_at: new Date().toISOString(),
      status: 'pending'
    }
  };
});
```

## Workflow 3: API Data Aggregation

**Use Case**: Combine data from multiple APIs

**Nodes**:
1. HTTP Request (API 1)
2. HTTP Request (API 2)
3. Merge (combine results)
4. Code (process combined data)
5. Google Sheets (save results)

**Merge strategies**:
- **Append**: Stack datasets
- **Merge By Index**: Join by position
- **Merge By Key**: Join by field value
- **Multiplex**: Create combinations

---

# Chapter 4: Building Your First Workflows

## Project: Customer Onboarding Automation

Let's build a complete customer onboarding workflow that:
1. Receives new signup via webhook
2. Creates user in database
3. Sends welcome email
4. Adds to CRM
5. Schedules follow-up
6. Notifies sales team

### Step 1: Webhook Setup

```javascript
// Webhook node configuration
HTTP Method: POST
Path: /new-customer
Response Mode: Last Node

// Test payload
{
  "email": "customer@example.com",
  "name": "Jane Smith",
  "company": "Tech Corp",
  "plan": "professional"
}
```

### Step 2: Database Operations

```sql
-- MySQL node: Check if user exists
SELECT * FROM users WHERE email = :email;

-- If not exists, insert
INSERT INTO users (email, name, company, plan, created_at)
VALUES (:email, :name, :company, :plan, NOW());
```

### Step 3: Email Templates

```javascript
// Function node: Generate personalized content
const templates = {
  'starter': {
    subject: 'Welcome to n8n Starter!',
    benefits: ['5 workflows', 'Community support', 'Basic integrations']
  },
  'professional': {
    subject: 'Welcome to n8n Professional!',
    benefits: ['Unlimited workflows', 'Priority support', 'All integrations']
  }
};

const plan = $input.first().json.plan;
const template = templates[plan];

return [{
  json: {
    to: $input.first().json.email,
    subject: template.subject,
    html: `
      <h1>Welcome ${$input.first().json.name}!</h1>
      <p>Your ${plan} plan includes:</p>
      <ul>
        ${template.benefits.map(b => `<li>${b}</li>`).join('')}
      </ul>
    `
  }
}];
```

### Step 4: CRM Integration

```javascript
// HubSpot node configuration
Operation: Create Contact
Properties:
  Email: {{$json["email"]}}
  First Name: {{$json["name"].split(' ')[0]}}
  Last Name: {{$json["name"].split(' ')[1]}}
  Company: {{$json["company"]}}
  Lifecycle Stage: Customer
```

### Step 5: Schedule Follow-up

```javascript
// Wait node
Resume: At Specific Time
Resume At: {{DateTime.now().plus({days: 3}).toISO()}}

// Follow-up email
Subject: How's your n8n experience so far?
Body: Personalized check-in message
```

---

# Chapter 5: Data Integration Workflows

## ETL Pipeline Example

**Extract, Transform, Load** workflow for data warehouse:

### Source: Multiple Databases

```javascript
// Parallel data extraction
// Branch 1: PostgreSQL
SELECT * FROM orders WHERE created_at > :last_sync;

// Branch 2: MongoDB
db.customers.find({"updated": {"$gte": ISODate(last_sync)}});

// Branch 3: REST API
GET https://api.system.com/products?modified_since={{last_sync}}
```

### Transform: Data Normalization

```javascript
// Function node: Standardize data structure
const postgres_data = $node["PostgreSQL"].json;
const mongo_data = $node["MongoDB"].json;
const api_data = $node["HTTP Request"].json;

// Unified schema
const normalized = [];

// Process PostgreSQL orders
postgres_data.forEach(order => {
  normalized.push({
    source: 'postgres',
    type: 'order',
    id: order.order_id,
    customer_id: order.customer_id,
    amount: parseFloat(order.total),
    date: order.created_at,
    status: order.status.toLowerCase()
  });
});

// Process MongoDB customers
mongo_data.forEach(customer => {
  normalized.push({
    source: 'mongodb',
    type: 'customer',
    id: customer._id,
    customer_id: customer.customer_id,
    amount: customer.lifetime_value,
    date: customer.updated,
    status: customer.active ? 'active' : 'inactive'
  });
});

// Process API products
api_data.products.forEach(product => {
  normalized.push({
    source: 'api',
    type: 'product',
    id: product.sku,
    amount: product.price,
    date: product.modified,
    status: product.in_stock ? 'available' : 'out_of_stock'
  });
});

return normalized.map(item => ({json: item}));
```

### Load: Data Warehouse

```javascript
// BigQuery node
Operation: Insert
Dataset: analytics
Table: unified_data
Options:
  Create table if doesn't exist: true
  Write disposition: WRITE_APPEND
```

## Real-time Data Sync

### Bi-directional Sync: Salesforce ↔ PostgreSQL

```javascript
// Webhook trigger for Salesforce changes
// Outbound Message or Platform Events

// Process Salesforce update
const sf_data = $input.first().json;

// Transform Salesforce to PostgreSQL schema
const pg_record = {
  sf_id: sf_data.Id,
  account_name: sf_data.Name,
  revenue: sf_data.AnnualRevenue,
  employees: sf_data.NumberOfEmployees,
  industry: sf_data.Industry,
  updated_at: new Date().toISOString()
};

// Upsert to PostgreSQL
`INSERT INTO accounts (sf_id, account_name, revenue, employees, industry, updated_at)
 VALUES (:sf_id, :account_name, :revenue, :employees, :industry, :updated_at)
 ON CONFLICT (sf_id) 
 DO UPDATE SET 
   account_name = EXCLUDED.account_name,
   revenue = EXCLUDED.revenue,
   employees = EXCLUDED.employees,
   industry = EXCLUDED.industry,
   updated_at = EXCLUDED.updated_at;`

// Reverse sync: PostgreSQL → Salesforce
// Trigger: PostgreSQL changes via LISTEN/NOTIFY
// Update Salesforce via API
```

---

# Chapter 6: API Automation Examples

## REST API Integration

### Example: Building a Meta-API

Create a unified API that combines multiple services:

```javascript
// Webhook: /api/user/:id
// Combines data from multiple sources

// Node 1: Get user from database
const user = await $node["MySQL"].json;

// Node 2: Get social profiles
const twitter = await fetch(`https://api.twitter.com/users/${user.twitter_handle}`);
const linkedin = await fetch(`https://api.linkedin.com/users/${user.linkedin_id}`);

// Node 3: Get analytics
const analytics = await $node["Google Analytics"].json;

// Node 4: Combine and return
return [{
  json: {
    user: {
      id: user.id,
      name: user.name,
      email: user.email
    },
    social: {
      twitter: {
        followers: twitter.followers_count,
        tweets: twitter.statuses_count
      },
      linkedin: {
        connections: linkedin.numConnections,
        headline: linkedin.headline
      }
    },
    analytics: {
      pageviews: analytics.pageviews,
      sessions: analytics.sessions,
      avgDuration: analytics.avgSessionDuration
    }
  }
}];
```

## GraphQL Integration

```javascript
// HTTP Request node for GraphQL
Method: POST
URL: https://api.example.com/graphql
Headers:
  Content-Type: application/json
  Authorization: Bearer {{$credentials.api_token}}

Body:
{
  "query": `
    query GetUserData($userId: ID!) {
      user(id: $userId) {
        id
        name
        posts {
          title
          content
          likes
        }
      }
    }
  `,
  "variables": {
    "userId": "{{$json["userId"]}}"
  }
}
```

## OAuth 2.0 Flow Implementation

```javascript
// Custom OAuth implementation
// Node 1: Initialize OAuth
const client_id = $credentials.client_id;
const redirect_uri = 'https://your-n8n.com/webhook/oauth-callback';
const auth_url = `https://provider.com/oauth/authorize?client_id=${client_id}&redirect_uri=${redirect_uri}&response_type=code`;

// Node 2: Exchange code for token
const code = $json["code"];
const token_response = await fetch('https://provider.com/oauth/token', {
  method: 'POST',
  headers: {'Content-Type': 'application/x-www-form-urlencoded'},
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: code,
    client_id: client_id,
    client_secret: $credentials.client_secret,
    redirect_uri: redirect_uri
  })
});

// Node 3: Store token
const token = await token_response.json();
// Save to database or cache for future use
```

---

# Chapter 7: Database Operations

## Advanced SQL Operations

### Batch Processing with Transactions

```javascript
// Function node: Prepare batch data
const records = $input.all();
const batchSize = 100;
const batches = [];

for (let i = 0; i < records.length; i += batchSize) {
  batches.push(records.slice(i, i + batchSize));
}

// Process each batch with transaction
for (const batch of batches) {
  // Begin transaction
  await $node["MySQL"].execute('START TRANSACTION');
  
  try {
    for (const record of batch) {
      await $node["MySQL"].execute(
        'INSERT INTO table (col1, col2) VALUES (?, ?)',
        [record.json.value1, record.json.value2]
      );
    }
    await $node["MySQL"].execute('COMMIT');
  } catch (error) {
    await $node["MySQL"].execute('ROLLBACK');
    throw error;
  }
}
```

### Database Migration Workflow

```javascript
// Migration workflow with rollback capability
// Node 1: Check current version
const current_version = await $node["MySQL"].execute(
  'SELECT version FROM migrations ORDER BY id DESC LIMIT 1'
);

// Node 2: Get pending migrations
const migrations = [
  {
    version: '1.0.1',
    up: 'ALTER TABLE users ADD COLUMN last_login TIMESTAMP',
    down: 'ALTER TABLE users DROP COLUMN last_login'
  },
  {
    version: '1.0.2',
    up: 'CREATE INDEX idx_users_email ON users(email)',
    down: 'DROP INDEX idx_users_email ON users'
  }
];

// Node 3: Apply migrations
for (const migration of migrations) {
  if (migration.version > current_version) {
    try {
      await $node["MySQL"].execute(migration.up);
      await $node["MySQL"].execute(
        'INSERT INTO migrations (version, applied_at) VALUES (?, NOW())',
        [migration.version]
      );
    } catch (error) {
      // Rollback
      await $node["MySQL"].execute(migration.down);
      throw error;
    }
  }
}
```

## NoSQL Operations

### MongoDB Aggregation Pipeline

```javascript
// MongoDB node: Complex aggregation
{
  "aggregate": [
    {
      "$match": {
        "created_at": {
          "$gte": "{{DateTime.now().minus({days: 30}).toISO()}}"
        }
      }
    },
    {
      "$group": {
        "_id": "$category",
        "total": {"$sum": "$amount"},
        "count": {"$sum": 1},
        "avg": {"$avg": "$amount"}
      }
    },
    {
      "$sort": {"total": -1}
    },
    {
      "$limit": 10
    }
  ]
}
```

---

# Chapter 8: File Processing Automation

## File System Operations

### Watch Folder and Process Files

```javascript
// Workflow: Monitor folder for new files
// Node 1: Local File Trigger
// Watches /input folder for new files

// Node 2: File Router (Switch node)
// Route based on file extension
const extension = $json["fileName"].split('.').pop().toLowerCase();

switch(extension) {
  case 'csv':
    $output = 'csv_processing';
    break;
  case 'pdf':
    $output = 'pdf_processing';
    break;
  case 'json':
    $output = 'json_processing';
    break;
  default:
    $output = 'unknown';
}

// Node 3a: CSV Processing
// Read CSV → Transform → Database
const csv = $node["Read CSV"].json;
const processed = csv.map(row => ({
  ...row,
  processed_date: new Date().toISOString(),
  status: 'processed'
}));

// Node 3b: PDF Processing
// Extract text → Parse → Store
const pdfText = $node["Extract PDF"].json.text;
const parsed = parseInvoice(pdfText); // Custom parsing logic

// Node 4: Archive processed files
// Move to /processed folder with timestamp
const timestamp = DateTime.now().toFormat('yyyyMMdd_HHmmss');
const newPath = `/processed/${timestamp}_${$json["fileName"]}`;
```

## Image Processing

### Automated Image Optimization

```javascript
// HTTP Request to image optimization API
// Or use Execute Command node with ImageMagick

// Node 1: Receive image via webhook
// Node 2: Download image
// Node 3: Process image
const optimizeImage = async (imagePath) => {
  // Resize
  await $node["Execute Command"].execute(
    `convert ${imagePath} -resize 1920x1080 ${imagePath}_resized.jpg`
  );
  
  // Compress
  await $node["Execute Command"].execute(
    `jpegoptim --max=85 ${imagePath}_resized.jpg`
  );
  
  // Generate thumbnail
  await $node["Execute Command"].execute(
    `convert ${imagePath} -thumbnail 200x200 ${imagePath}_thumb.jpg`
  );
  
  return {
    original: imagePath,
    optimized: `${imagePath}_resized.jpg`,
    thumbnail: `${imagePath}_thumb.jpg`
  };
};

// Node 4: Upload to CDN
// Node 5: Update database with URLs
```

---

# Chapter 9: Communication Workflows

## Email Automation

### Smart Email Routing

```javascript
// Workflow: Email support ticket system
// Node 1: Email Trigger (IMAP)

// Node 2: Analyze email content
const email = $input.first().json;
const subject = email.subject.toLowerCase();
const body = email.text.toLowerCase();

// Determine priority and department
let priority = 'normal';
let department = 'general';

// Priority detection
if (subject.includes('urgent') || subject.includes('critical')) {
  priority = 'high';
} else if (subject.includes('bug') || subject.includes('error')) {
  priority = 'high';
  department = 'technical';
}

// Department routing
if (body.includes('billing') || body.includes('payment')) {
  department = 'billing';
} else if (body.includes('technical') || body.includes('api')) {
  department = 'technical';
} else if (body.includes('sales') || body.includes('pricing')) {
  department = 'sales';
}

// Node 3: Create ticket in system
const ticket = {
  subject: email.subject,
  from: email.from,
  body: email.text,
  priority: priority,
  department: department,
  created_at: new Date().toISOString(),
  status: 'open'
};

// Node 4: Send auto-response
const response = `
Thank you for contacting support.
Ticket #${ticket.id} has been created.
Priority: ${priority}
Department: ${department}
Expected response time: ${priority === 'high' ? '2 hours' : '24 hours'}
`;
```

## Slack Integration

### Interactive Slack Bot

```javascript
// Slash command workflow
// Node 1: Webhook (Slack slash command)
// Command: /task [action] [parameters]

// Node 2: Parse command
const text = $json["text"];
const [action, ...params] = text.split(' ');

// Node 3: Switch based on action
switch(action) {
  case 'create':
    // Create task in project management tool
    const task = {
      title: params.join(' '),
      creator: $json["user_name"],
      channel: $json["channel_name"]
    };
    // Create in Asana/Jira/Trello
    break;
    
  case 'status':
    // Get project status
    // Query database for metrics
    break;
    
  case 'report':
    // Generate report
    // Aggregate data and format
    break;
}

// Node 4: Send response to Slack
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": `✅ Task created: *${task.title}*`
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {"type": "plain_text", "text": "View Task"},
          "url": `https://app.asana.com/task/${task.id}`
        }
      ]
    }
  ]
}
```

---

# Chapter 10: Web Scraping and Data Extraction

## Web Scraping Patterns

### Dynamic Website Scraping

```javascript
// Using Puppeteer via Execute Command node
// Node 1: Function node to prepare scraping script
const scrapingScript = `
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({headless: true});
  const page = await browser.newPage();
  
  // Navigate to page
  await page.goto('${$json["url"]}', {waitUntil: 'networkidle2'});
  
  // Wait for dynamic content
  await page.waitForSelector('.product-list');
  
  // Extract data
  const products = await page.evaluate(() => {
    return Array.from(document.querySelectorAll('.product-item')).map(item => ({
      name: item.querySelector('.product-name')?.textContent,
      price: item.querySelector('.price')?.textContent,
      image: item.querySelector('img')?.src,
      link: item.querySelector('a')?.href
    }));
  });
  
  console.log(JSON.stringify(products));
  await browser.close();
})();
`;

// Save script and execute
await $node["Write File"].write('/tmp/scraper.js', scrapingScript);
const result = await $node["Execute Command"].execute('node /tmp/scraper.js');
const products = JSON.parse(result);
```

### API-First Scraping

```javascript
// Check if website has hidden API
// Node 1: HTTP Request with browser headers
{
  headers: {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'application/json',
    'Referer': 'https://website.com'
  }
}

// Node 2: Parse JSON response
// Often websites load data via JSON APIs
```

## Data Extraction Patterns

### PDF Data Extraction

```javascript
// Extract structured data from invoices
// Node 1: Read PDF
// Node 2: Function node for parsing

function parseInvoice(text) {
  const invoice = {};
  
  // Extract invoice number
  const invoiceMatch = text.match(/Invoice #:\s*(\S+)/);
  invoice.number = invoiceMatch ? invoiceMatch[1] : null;
  
  // Extract date
  const dateMatch = text.match(/Date:\s*(\S+)/);
  invoice.date = dateMatch ? dateMatch[1] : null;
  
  // Extract total
  const totalMatch = text.match(/Total:\s*\$?([\d,]+\.?\d*)/);
  invoice.total = totalMatch ? parseFloat(totalMatch[1].replace(',', '')) : null;
  
  // Extract line items
  const lineItemRegex = /(\d+)\s+(.*?)\s+\$?([\d,]+\.?\d*)/g;
  invoice.items = [];
  let match;
  while ((match = lineItemRegex.exec(text)) !== null) {
    invoice.items.push({
      quantity: parseInt(match[1]),
      description: match[2].trim(),
      amount: parseFloat(match[3].replace(',', ''))
    });
  }
  
  return invoice;
}
```

---

# Chapter 11: Error Handling and Debugging

## Comprehensive Error Handling

### Error Recovery Workflow

```javascript
// Main workflow with error handling
// Node 1: Try-Catch pattern using IF nodes

// Node 2: Main process with error output
// Configure: Continue (using error output)

// Node 3: Error branch
const error = $json["error"];
const context = {
  workflow: $workflow.name,
  node: $node.name,
  timestamp: new Date().toISOString(),
  input: $input.all(),
  error: {
    message: error.message,
    stack: error.stack,
    code: error.code
  }
};

// Node 4: Classify error
let severity = 'low';
let retry = false;

if (error.message.includes('timeout') || error.message.includes('ECONNREFUSED')) {
  severity = 'medium';
  retry = true;
} else if (error.message.includes('auth') || error.message.includes('unauthorized')) {
  severity = 'high';
  retry = false;
} else if (error.message.includes('rate limit')) {
  severity = 'medium';
  retry = true;
  // Add delay before retry
}

// Node 5: Retry logic
if (retry && context.retryCount < 3) {
  // Wait node: exponential backoff
  const delay = Math.pow(2, context.retryCount) * 1000;
  // Re-execute main process
}

// Node 6: Alert based on severity
if (severity === 'high') {
  // Send immediate alert via Slack/PagerDuty
  // Create incident ticket
} else if (severity === 'medium') {
  // Send email notification
  // Log to monitoring system
}
```

## Debugging Techniques

### Debug Logger Workflow

```javascript
// Subworkflow for debugging
// Can be called from any workflow

// Node 1: Receive debug data
const debugData = $input.all();

// Node 2: Format debug output
const formatted = {
  timestamp: new Date().toISOString(),
  workflow: $execution.workflowId,
  execution: $execution.id,
  mode: $execution.mode,
  data: debugData,
  memory: process.memoryUsage(),
  performance: {
    executionTime: $execution.duration,
    nodeCount: $execution.nodeExecutionOrder.length
  }
};

// Node 3: Multiple output destinations
// Console log for development
console.log(JSON.stringify(formatted, null, 2));

// File log for production
const logFile = `/logs/n8n_${DateTime.now().toFormat('yyyy-MM-dd')}.log`;
fs.appendFileSync(logFile, JSON.stringify(formatted) + '\n');

// Database for analysis
await $node["PostgreSQL"].execute(
  'INSERT INTO debug_logs (data) VALUES ($1)',
  [JSON.stringify(formatted)]
);
```

---

# Chapter 12: Advanced Node Operations

## Custom Function Nodes

### Advanced Data Processing

```javascript
// Complex data transformation with caching
// Initialize cache (outside main function for persistence)
const cache = new Map();
const CACHE_TTL = 3600000; // 1 hour

// Main processing function
async function processData(items) {
  const results = [];
  
  for (const item of items) {
    const cacheKey = `${item.json.type}_${item.json.id}`;
    const cached = cache.get(cacheKey);
    
    // Check cache validity
    if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
      results.push(cached.data);
      continue;
    }
    
    // Process item
    const processed = await complexProcessing(item.json);
    
    // Update cache
    cache.set(cacheKey, {
      data: processed,
      timestamp: Date.now()
    });
    
    results.push(processed);
  }
  
  return results;
}

async function complexProcessing(data) {
  // Validation
  const schema = {
    type: 'required',
    id: 'required',
    value: 'number'
  };
  
  for (const [field, rule] of Object.entries(schema)) {
    if (rule === 'required' && !data[field]) {
      throw new Error(`Missing required field: ${field}`);
    }
    if (rule === 'number' && typeof data[field] !== 'number') {
      throw new Error(`Field ${field} must be a number`);
    }
  }
  
  // Enrichment
  data.processed_at = new Date().toISOString();
  data.hash = crypto.createHash('md5').update(JSON.stringify(data)).digest('hex');
  
  // External API call with retry
  let apiResult;
  for (let attempt = 0; attempt < 3; attempt++) {
    try {
      apiResult = await fetchWithTimeout(`https://api.example.com/enrich/${data.id}`, 5000);
      break;
    } catch (error) {
      if (attempt === 2) throw error;
      await sleep(1000 * Math.pow(2, attempt));
    }
  }
  
  return {
    ...data,
    enriched: apiResult
  };
}

// Utility functions
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function fetchWithTimeout(url, timeout) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(url, {signal: controller.signal});
    return await response.json();
  } finally {
    clearTimeout(timeoutId);
  }
}

// Execute
return await processData($input.all());
```

## Split and Merge Patterns

### Parallel Processing Pattern

```javascript
// Node 1: Split data into chunks
const items = $input.all();
const chunkSize = 10;
const chunks = [];

for (let i = 0; i < items.length; i += chunkSize) {
  chunks.push({
    json: {
      chunkId: Math.floor(i / chunkSize),
      items: items.slice(i, i + chunkSize)
    }
  });
}

return chunks;

// Nodes 2-5: Parallel processing branches
// Each processes a subset of data

// Node 6: Merge results
const allResults = [];
for (let i = 0; i < $input.length; i++) {
  const input = $input.item(i);
  allResults.push(...input.json.processedItems);
}

// Sort by original order
allResults.sort((a, b) => a.originalIndex - b.originalIndex);

return [{json: {results: allResults}}];
```

---

# Chapter 13: Custom Functions and Code

## JavaScript Mastery in n8n

### Working with External Libraries

```javascript
// Loading and using external libraries
// Note: Requires n8n configuration to allow external modules

// Node 1: Install required packages
// Execute Command: npm install lodash moment axios

// Node 2: Use in Function node
const _ = require('lodash');
const moment = require('moment');
const axios = require('axios');

// Data processing with lodash
const groupedData = _.groupBy($input.all(), item => item.json.category);
const summaries = _.mapValues(groupedData, group => ({
  count: group.length,
  total: _.sumBy(group, item => item.json.amount),
  average: _.meanBy(group, item => item.json.amount)
}));

// Date manipulation with moment
const dateRanges = {
  today: moment().startOf('day'),
  yesterday: moment().subtract(1, 'day').startOf('day'),
  lastWeek: moment().subtract(1, 'week').startOf('week'),
  lastMonth: moment().subtract(1, 'month').startOf('month')
};

// API calls with axios
const apiCalls = Object.entries(summaries).map(([category, data]) =>
  axios.post('https://analytics.api.com/events', {
    category,
    metrics: data,
    timestamp: moment().toISOString()
  })
);

const results = await Promise.all(apiCalls);
return results.map(r => ({json: r.data}));
```

### Creating Reusable Code Modules

```javascript
// Shared code module pattern
// Store in Code node and reference from others

// Module: DataValidator
class DataValidator {
  constructor(rules) {
    this.rules = rules;
  }
  
  validate(data) {
    const errors = [];
    
    for (const [field, rule] of Object.entries(this.rules)) {
      if (!this.validateField(data[field], rule)) {
        errors.push({
          field,
          value: data[field],
          rule,
          message: `Validation failed for field: ${field}`
        });
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
  
  validateField(value, rule) {
    if (typeof rule === 'function') {
      return rule(value);
    }
    
    switch(rule) {
      case 'required':
        return value !== null && value !== undefined && value !== '';
      case 'email':
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
      case 'phone':
        return /^\+?[\d\s-()]+$/.test(value);
      case 'url':
        return /^https?:\/\/.+/.test(value);
      default:
        return true;
    }
  }
}

// Module: APIClient
class APIClient {
  constructor(baseURL, apiKey) {
    this.baseURL = baseURL;
    this.apiKey = apiKey;
  }
  
  async request(endpoint, method = 'GET', data = null) {
    const options = {
      method,
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json'
      }
    };
    
    if (data) {
      options.body = JSON.stringify(data);
    }
    
    const response = await fetch(`${this.baseURL}${endpoint}`, options);
    
    if (!response.ok) {
      throw new Error(`API Error: ${response.status} - ${response.statusText}`);
    }
    
    return await response.json();
  }
}

// Export for use in other nodes
$node.DataValidator = DataValidator;
$node.APIClient = APIClient;
```

---

# Chapter 14: Webhooks and Triggers

## Advanced Webhook Patterns

### Webhook Security Implementation

```javascript
// Secure webhook with signature verification
// Node 1: Webhook node

// Node 2: Verify webhook signature
function verifyWebhookSignature(payload, signature, secret) {
  const crypto = require('crypto');
  
  // Different providers use different methods
  // GitHub/Stripe style
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Verify request
const signature = $input.first().headers['x-webhook-signature'];
const secret = $credentials.webhook_secret;
const payload = $input.first().json;

if (!verifyWebhookSignature(payload, signature, secret)) {
  throw new Error('Invalid webhook signature');
}

// Node 3: Rate limiting
const redis = require('redis');
const client = redis.createClient();

const ip = $input.first().headers['x-forwarded-for'] || $input.first().ip;
const key = `rate_limit:${ip}`;
const limit = 100; // requests per hour

const current = await client.incr(key);
if (current === 1) {
  await client.expire(key, 3600);
}

if (current > limit) {
  throw new Error('Rate limit exceeded');
}

// Node 4: Process webhook
// Your business logic here
```

### Event-Driven Architecture

```javascript
// Multi-trigger workflow orchestration
// Central event router workflow

// Node 1: Webhook receiver (multiple paths)
// /webhook/order, /webhook/customer, /webhook/inventory

// Node 2: Event classifier
const path = $input.first().path;
const event = $input.first().json;

const eventType = path.split('/').pop(); // order, customer, inventory
const eventAction = event.action; // created, updated, deleted

const eventKey = `${eventType}.${eventAction}`;

// Node 3: Route to appropriate workflow
const workflows = {
  'order.created': 'workflow_123',
  'order.updated': 'workflow_124',
  'order.deleted': 'workflow_125',
  'customer.created': 'workflow_126',
  'customer.updated': 'workflow_127',
  'inventory.low': 'workflow_128'
};

const targetWorkflow = workflows[eventKey];

if (targetWorkflow) {
  // Execute workflow
  await $execution.executeWorkflow(targetWorkflow, event);
}

// Node 4: Event logging
await $node["PostgreSQL"].execute(
  `INSERT INTO events (type, action, data, processed_at) 
   VALUES ($1, $2, $3, NOW())`,
  [eventType, eventAction, JSON.stringify(event)]
);
```

---

# Chapter 15: Authentication and Security

## OAuth Implementation

### Custom OAuth Provider Integration

```javascript
// Complete OAuth 2.0 flow implementation
// Workflow 1: Initiate OAuth

// Node 1: Generate state token
const crypto = require('crypto');
const state = crypto.randomBytes(32).toString('hex');

// Store state in database with expiry
await $node["Redis"].set(`oauth_state:${state}`, JSON.stringify({
  user_id: $json["user_id"],
  timestamp: Date.now(),
  redirect_uri: $json["redirect_uri"]
}), 'EX', 600); // 10 minute expiry

// Node 2: Build authorization URL
const authUrl = new URL('https://provider.com/oauth/authorize');
authUrl.searchParams.append('client_id', $credentials.client_id);
authUrl.searchParams.append('redirect_uri', 'https://your-n8n.com/webhook/oauth-callback');
authUrl.searchParams.append('response_type', 'code');
authUrl.searchParams.append('scope', 'read write');
authUrl.searchParams.append('state', state);

return [{json: {authUrl: authUrl.toString()}}];

// Workflow 2: OAuth Callback
// Node 1: Webhook at /webhook/oauth-callback

// Node 2: Verify state
const state = $json["state"];
const code = $json["code"];

const storedState = await $node["Redis"].get(`oauth_state:${state}`);
if (!storedState) {
  throw new Error('Invalid or expired state token');
}

const stateData = JSON.parse(storedState);
await $node["Redis"].del(`oauth_state:${state}`);

// Node 3: Exchange code for token
const tokenResponse = await fetch('https://provider.com/oauth/token', {
  method: 'POST',
  headers: {'Content-Type': 'application/x-www-form-urlencoded'},
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: code,
    client_id: $credentials.client_id,
    client_secret: $credentials.client_secret,
    redirect_uri: 'https://your-n8n.com/webhook/oauth-callback'
  })
});

const tokens = await tokenResponse.json();

// Node 4: Securely store tokens
const encrypted = encrypt(tokens.access_token, $credentials.encryption_key);
await $node["PostgreSQL"].execute(
  `INSERT INTO user_tokens (user_id, provider, access_token, refresh_token, expires_at)
   VALUES ($1, $2, $3, $4, $5)
   ON CONFLICT (user_id, provider) DO UPDATE
   SET access_token = $3, refresh_token = $4, expires_at = $5`,
  [
    stateData.user_id,
    'provider_name',
    encrypted,
    encrypt(tokens.refresh_token, $credentials.encryption_key),
    new Date(Date.now() + tokens.expires_in * 1000)
  ]
);

// Encryption functions
function encrypt(text, key) {
  const crypto = require('crypto');
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(key, 'hex'), iv);
  let encrypted = cipher.update(text);
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  return iv.toString('hex') + ':' + encrypted.toString('hex');
}

function decrypt(text, key) {
  const parts = text.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encryptedText = Buffer.from(parts[1], 'hex');
  const decipher = crypto.createDecipheriv('aes-256-cbc', Buffer.from(key, 'hex'), iv);
  let decrypted = decipher.update(encryptedText);
  decrypted = Buffer.concat([decrypted, decipher.final()]);
  return decrypted.toString();
}
```

## API Key Management

```javascript
// Secure API key rotation system
// Workflow: Automatic key rotation

// Node 1: Cron trigger (monthly)

// Node 2: Generate new API key
const crypto = require('crypto');
const newApiKey = crypto.randomBytes(32).toString('base64url');
const keyId = crypto.randomBytes(16).toString('hex');

// Node 3: Update provider with new key
await fetch('https://api.provider.com/keys', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${$credentials.current_api_key}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    key: newApiKey,
    name: `n8n-key-${keyId}`,
    permissions: ['read', 'write']
  })
});

// Node 4: Update n8n credentials
// Store new key with overlap period
await $node["PostgreSQL"].execute(
  `INSERT INTO api_keys (key_id, api_key, created_at, expires_at, status)
   VALUES ($1, $2, NOW(), NOW() + INTERVAL '35 days', 'active')`,
  [keyId, encrypt(newApiKey, $credentials.master_key)]
);

// Node 5: Schedule old key deactivation
// Wait 5 days for propagation
await $node["Wait"].wait(5 * 24 * 60 * 60 * 1000);

// Node 6: Deactivate old key
await fetch(`https://api.provider.com/keys/${$credentials.current_key_id}`, {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${newApiKey}`
  }
});

// Update status in database
await $node["PostgreSQL"].execute(
  `UPDATE api_keys SET status = 'revoked' WHERE key_id = $1`,
  [$credentials.current_key_id]
);
```

---

# Appendix A: n8n Best Practices

## Workflow Design Patterns

### 1. Modular Workflow Design
- Break complex workflows into smaller, reusable sub-workflows
- Use the Execute Workflow node for composition
- Maintain single responsibility principle

### 2. Error Handling Strategy
- Always implement error workflows
- Use try-catch patterns with IF nodes
- Log errors comprehensively
- Implement retry logic with exponential backoff

### 3. Performance Optimization
- Batch process large datasets
- Use SplitInBatches for memory management
- Implement caching where appropriate
- Optimize database queries

### 4. Security Best Practices
- Never hardcode credentials
- Use environment variables
- Implement webhook signature verification
- Encrypt sensitive data at rest
- Regular credential rotation

### 5. Monitoring and Observability
- Implement comprehensive logging
- Set up alerts for failures
- Monitor execution times
- Track resource usage

## Common Pitfalls to Avoid

1. **Memory Issues**
   - Processing entire datasets at once
   - Not using batching for large operations
   - Accumulating data in loops

2. **Rate Limiting**
   - Not implementing delays between API calls
   - Missing retry logic
   - No exponential backoff

3. **Data Loss**
   - Not implementing transaction patterns
   - Missing error recovery
   - No data validation

4. **Security Vulnerabilities**
   - Exposed credentials in logs
   - Unvalidated webhook inputs
   - SQL injection in dynamic queries

---

# Appendix B: Troubleshooting Guide

## Common Issues and Solutions

### Issue: Workflow Times Out

**Symptoms**: Workflow stops after 60 seconds

**Solutions**:
```javascript
// 1. Increase timeout in workflow settings
Settings → Timeout → 300 seconds

// 2. Break into smaller chunks
const chunks = _.chunk(largeArray, 100);
for (const chunk of chunks) {
  // Process chunk
  await processChunk(chunk);
}

// 3. Use queuing pattern
// Send to queue for async processing
```

### Issue: Memory Exceeded

**Symptoms**: "JavaScript heap out of memory"

**Solutions**:
```javascript
// 1. Use streaming/batching
// Instead of loading all data:
const allData = await getAllRecords(); // Bad

// Stream process:
const batchSize = 100;
let offset = 0;
while (true) {
  const batch = await getRecords(offset, batchSize);
  if (batch.length === 0) break;
  await processBatch(batch);
  offset += batchSize;
}

// 2. Clear variables
largeArray = null; // Free memory
```

### Issue: Webhook Not Receiving Data

**Checklist**:
1. Verify webhook URL is accessible
2. Check firewall/security groups
3. Validate SSL certificate
4. Test with curl/Postman
5. Check webhook node is in "Production" mode

---

# Appendix C: Resources and Community

## Official Resources

- **Documentation**: https://docs.n8n.io
- **Community Forum**: https://community.n8n.io
- **GitHub**: https://github.com/n8n-io/n8n
- **YouTube Channel**: n8n tutorials and demos

## Useful Node Packages

```json
{
  "recommended_packages": {
    "data_processing": ["lodash", "ramda"],
    "date_time": ["moment", "date-fns", "luxon"],
    "validation": ["joi", "yup", "ajv"],
    "http_clients": ["axios", "got", "node-fetch"],
    "databases": ["pg", "mysql2", "mongodb"],
    "crypto": ["bcrypt", "jsonwebtoken", "uuid"],
    "testing": ["jest", "mocha", "chai"]
  }
}
```

## Community Workflows

Top community-contributed workflows:
1. Social Media Scheduler
2. Invoice Processing System
3. Customer Feedback Analyzer
4. SEO Monitoring Dashboard
5. DevOps Automation Suite

---

# Index

A
- API Integration: Chapter 6
- Authentication: Chapter 15
- Automation Patterns: Throughout

B
- Batch Processing: Chapter 7
- Best Practices: Appendix A

C
- Communication Workflows: Chapter 9
- Core Concepts: Chapter 2
- Custom Functions: Chapter 13

D
- Data Integration: Chapter 5
- Database Operations: Chapter 7
- Debugging: Chapter 11
- Deployment: Part V

E
- Email Automation: Chapter 9
- Error Handling: Chapter 11
- ETL Pipelines: Chapter 5

F
- File Processing: Chapter 8
- Function Nodes: Chapter 13

G
- Getting Started: Chapter 1
- GraphQL: Chapter 6

H
- HTTP Requests: Chapter 6

I
- Installation: Chapter 1
- Integration Patterns: Chapter 5

J
- JavaScript: Chapter 13
- JSON Processing: Chapter 2

M
- Monitoring: Appendix A

N
- Nodes: Chapter 2
- NoSQL: Chapter 7

O
- OAuth: Chapter 15
- Optimization: Appendix A

P
- Performance: Appendix A
- Projects: Part IV

R
- REST APIs: Chapter 6
- Resources: Appendix C

S
- Security: Chapter 15
- Slack Integration: Chapter 9
- SQL Operations: Chapter 7

T
- Triggers: Chapter 14
- Troubleshooting: Appendix B

W
- Web Scraping: Chapter 10
- Webhooks: Chapter 14
- Workflow Design: Appendix A

---

# About This Book

This comprehensive guide was created to help you master n8n from the ground up. Whether you're automating simple tasks or building complex enterprise workflows, the patterns and examples in this book will accelerate your journey.

Remember: The best way to learn n8n is by doing. Start with simple workflows and gradually increase complexity as you become comfortable with the concepts.

Happy automating!

---

*Last Updated: 2024*
*Version: 1.0*
*For n8n version: 1.x*
