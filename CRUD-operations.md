# CRUD Operations: Architecture and Implementation Guide

This document explains how to build CRUD (Create, Read, Update, Delete) operations in a web application backend. We'll use a fun example to make these concepts clear.

## Our Example Scenario: SweetBot AI

Imagine you're building **SweetBot AI**, a tech support chatbot for **LolliPop Dreams**, a company that sells custom-made lollipops in hundreds of flavors. The AI chatbot helps customers:

- Create custom lollipop orders based on flavor preferences
- Track order status
- Update flavor combinations
- View their order history
- Cancel orders

This scenario has unique requirements:
- Customers can combine multiple flavors (strawberry + vanilla swirl)
- Orders have statuses (pending, mixing, cooling, packaging, shipped)
- The AI needs to learn from customer preferences
- Flavor combinations must be validated (some flavors don't mix well!)

## Table of Contents

1. [What is CRUD?](#what-is-crud)
2. [Understanding the Example Format](#understanding-the-example-format)
3. [Understanding the Basics](#understanding-the-basics)
4. [The Request-Response Cycle](#the-request-response-cycle)
5. [Input Parsing: How Data Comes In](#input-parsing-how-data-comes-in)
6. [CRUD Operations in Detail](#crud-operations-in-detail)
7. [Output Formatting: How Data Goes Out](#output-formatting-how-data-goes-out)
8. [Complete Example: Creating and Managing Orders](#complete-example-creating-and-managing-orders)
9. [Using AI to Build Endpoints](#using-ai-to-build-endpoints)
10. [Security Basics](#security-basics)

---

## What is CRUD?

CRUD stands for the four basic operations you can perform on data:

- **Create**: Add new records to the database (e.g., create a new lollipop order)
- **Read**: Retrieve existing records from the database (e.g., get order status)
- **Update**: Modify existing records in the database (e.g., change flavor combination)
- **Delete**: Remove records from the database (e.g., cancel an order)

---

## Understanding the Example Format

Throughout this document, you'll see examples labeled as **Input Example** and **Output Example**.

**What do these mean?**

- **Input Example**: Shows what data the **frontend sends** to the backend (the request)
- **Output Example**: Shows what data the **backend sends back** to the frontend (the response)

**Why this format?**

When you're building an endpoint, you need to know:
1. What format should the input be in? (Look at the Input Example)
2. What format should the output be in? (Look at the Output Example)

---

## Understanding the Basics

### What is an Endpoint?

An **endpoint** is like a specific address or phone number that your application can call to perform an action. Just like you might call a pizza place at a specific number to order pizza, your frontend calls specific endpoints to interact with your backend.

**Example:**
- `https://api.lollipopdreams.com/orders` - This is an endpoint for managing orders
- `https://api.lollipopdreams.com/customers` - This is an endpoint for managing customers

Each endpoint does a specific job. The `/orders` endpoint handles everything related to orders.

### What are HTTP Methods? (GET, POST, PUT, DELETE)

HTTP methods are like different types of requests you can make. Think of them as different actions:

- **GET**: "Hey, can I see that?" (Reading data)
- **POST**: "Hey, can you create this for me?" (Creating new data)
- **PUT**: "Hey, can you update this?" (Modifying existing data)
- **DELETE**: "Hey, can you remove this?" (Deleting data)

**Real-world analogy:**
- **GET** = Looking at a menu (you're just reading, not changing anything)
- **POST** = Placing an order (you're creating something new)
- **PUT** = Modifying your order (you're changing something that exists)
- **DELETE** = Canceling your order (you're removing something)

### How Endpoints and Methods Work Together

When you combine an endpoint with a method, you get a specific action:

```
GET    /orders     → "Show me all my orders"
GET    /orders/123 → "Show me order number 123"
POST   /orders     → "Create a new order"
PUT    /orders/123 → "Update order number 123"
DELETE /orders/123 → "Cancel order number 123"
```

The number `123` in `/orders/123` is called a **parameter** - it's a way to specify which specific order you're talking about.

---

## The Request-Response Cycle

Every time your frontend needs to do something with data, it follows this cycle:

```
1. Frontend sends a request → 2. Backend processes it → 3. Backend sends a response → 4. Frontend receives response
```

Let's break this down with our lollipop example:

**Step 1: Frontend sends a request**
- Alice wants to see her orders
- Frontend sends: "GET /orders" request to the backend
- Includes authentication (proves Alice is who she says she is)

**Step 2: Backend processes it**
- Backend receives the request
- Checks: Is Alice authenticated? Yes ✓
- Queries database: "Get all orders for Alice"
- Formats the data nicely

**Step 3: Backend sends a response**
- Backend sends back Alice's orders in a structured format
- Includes status code (200 = success, 404 = not found, etc.)

**Step 4: Frontend receives response**
- Frontend gets the data
- Displays Alice's orders on the screen

---

## Input Parsing: How Data Comes In

When the frontend sends data to your backend, you need to **parse** it - which means reading it, understanding it, and converting it into a format your code can work with.

### Understanding Request Bodies

When you create or update something, the frontend sends data in the **request body**. Think of it like filling out a form - you're sending information.

**Input Example: Creating a new order**

Alice wants to create a custom lollipop order. The frontend sends this data:

```json
{
    "customer_id": 456,
    "flavor_combination": ["strawberry", "vanilla_swirl"],
    "size": "large",
    "quantity": 12,
    "special_instructions": "Extra sweet, please!"
}
```

**What the backend needs to do:**
1. **Parse** the JSON (read and understand the structure)
2. **Extract** each field (customer_id, flavor_combination, etc.)
3. **Validate** that the data is correct:
   - Is customer_id a number? ✓
   - Are the flavors valid? ✓
   - Is size one of: "small", "medium", "large"? ✓
   - Is quantity a positive number? ✓
4. **Sanitize** special_instructions (remove any dangerous content)

### Parsing Query Parameters

For reading data (GET requests), information often comes in the **URL as query parameters**. These are the parts after the `?` in a URL.

**Input Example: Getting filtered orders**

Alice wants to see only her pending orders. The frontend sends:

```
GET /orders?customer_id=456&status=pending&page=1&limit=20
```

**Breaking this down:**
- `/orders` = the endpoint
- `?` = starts the query parameters
- `customer_id=456` = filter by customer ID 456
- `&` = separates parameters
- `status=pending` = only show pending orders
- `page=1` = show page 1
- `limit=20` = show 20 orders per page

**What the backend needs to do:**
1. **Parse** the query parameters
2. **Extract** each parameter value
3. **Use** these values to filter the database query

### Parsing URL Parameters

Sometimes the ID of what you want is in the URL path itself.

**Input Example: Getting a specific order**

Alice wants to see details of order #789:

```
GET /orders/789
```

**Breaking this down:**
- `/orders` = the endpoint
- `/789` = the order ID (this is a URL parameter)

**What the backend needs to do:**
1. **Extract** the number `789` from the URL
2. **Use** it to find that specific order in the database

### Input Validation

Before you save data, you need to **validate** it - make sure it's correct and safe.

**Input Example: Invalid flavor combination**

Someone tries to create an order with invalid flavors:

```json
{
    "flavor_combination": ["chocolate", "mint", "garlic"]
}
```

**Validation checks:**
1. Do all flavors exist? "chocolate" ✓, "mint" ✓, "garlic" ✗ (not a valid flavor)
2. Can these flavors be combined? Chocolate + mint ✓, but garlic doesn't go with anything!

**Output Example: Validation error response**

```json
{
    "error": "Invalid flavor combination: 'garlic' is not a valid flavor. Please choose from our available flavors.",
    "code": "VALIDATION_ERROR"
}
```

### Input Sanitization

**Sanitization** means cleaning the data to remove dangerous content.

**Input Example: Dangerous input**

Someone tries to include malicious code:

```json
{
    "special_instructions": "<script>alert('hack')</script>Extra sweet, please!"
}
```

**After sanitization:**
```json
{
    "special_instructions": "Extra sweet, please!"
}
```

The dangerous script tag is removed, but the legitimate text remains.

---

## Input Formatting and Response Parsing

When working with AI services, you need to:
1. **Format input** - Convert your data into the format the AI expects
2. **Send to AI** - Make the API call
3. **Parse response** - Extract and structure the data from the AI's response (which comes as function calls)
4. **Save to database** - Store the parsed data

### Formatting Input for AI

Before sending data to an AI service, you need to format it in a specific way. Here's how:

**Example: Formatting Input**

You have this data from the frontend:
```json
{
    "customer_id": 456,
    "flavor_combination": ["strawberry", "vanilla_swirl"],
    "size": "large",
    "quantity": 12,
    "special_instructions": "Extra sweet, please!"
}
```

**Format it for the AI service like this:**

```javascript
// Define a function to format order data for the AI service
function formatOrderForAI(orderData, customerInfo) {
    // Create an empty string to build the formatted prompt
    let formattedPrompt = "";
    
    // Add the customer ID to the prompt with a label
    formattedPrompt += `customer-id: ${orderData.customer_id}\n`;
    // Add the customer name to the prompt with a label
    formattedPrompt += `customer-name: ${customerInfo.name}\n`;
    
    // Join the flavor array into a comma-separated string and add to prompt
    formattedPrompt += `flavor-combination: ${orderData.flavor_combination.join(", ")}\n`;
    
    // Add the size to the prompt with a label
    formattedPrompt += `size: ${orderData.size}\n`;
    // Add the quantity to the prompt with a label
    formattedPrompt += `quantity: ${orderData.quantity}\n`;
    
    // Check if special instructions were provided
    if (orderData.special_instructions) {
        // Add the special instructions to the prompt with quotes around the text
        formattedPrompt += `special-instructions: "${orderData.special_instructions}"\n`;
    }
    
    // Return the complete formatted prompt string
    return formattedPrompt;
}
```

**The formatted input would be:**
```
customer-id: 456
customer-name: Alice Sweettooth
flavor-combination: strawberry, vanilla_swirl
size: large
quantity: 12
special-instructions: "Extra sweet, please!"
```

### Parsing AI Responses (Function Call Format)

The AI responds with **function calls**, not JSON. You need to parse these function calls to extract the data.

**Example: AI Response**

The AI service returns this:
```
flavors("strawberry", "vanilla_swirl", "caramel")
price(24.99)
status("pending")
production-time("2 days")
special-notes("Customer requested extra sweet - will add extra sugar")
```

**Example: Parsing the Response**

```javascript
// Define a function to parse the AI response that contains function calls
function parseAIResponse(aiResponse) {
    // Split the response text into an array of lines using newline character
    const lines = aiResponse.split('\n');
    
    // Create an object to store the parsed data with default values
    const parsed = {
        flavors: [],        // Array to hold flavor names
        price: 0,           // Number to hold the price
        status: "",         // String to hold the status
        production_time: "", // String to hold production time
        special_notes: ""   // String to hold special notes
    };
    
    // Loop through each line in the response
    for (let line of lines) {
        // Remove whitespace from the beginning and end of the line
        line = line.trim();
        
        // Skip to the next line if this line is empty
        if (!line) continue;
        
        // Check if this line starts with the flavors function call
        if (line.startsWith('flavors(')) {
            // Use regex to extract everything between the parentheses
            const match = line.match(/flavors\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Get the content inside the parentheses (the arguments)
                const flavorString = match[1];
                // Split by comma, trim each item, and remove quotes from each flavor name
                parsed.flavors = flavorString.split(',').map(f => f.trim().replace(/^"|"$/g, ''));
            }
        }
        
        // Check if this line starts with the price function call
        if (line.startsWith('price(')) {
            // Use regex to extract the number between the parentheses
            const match = line.match(/price\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Convert the string number to an actual number and store it
                parsed.price = parseFloat(match[1]);
            }
        }
        
        // Check if this line starts with the status function call
        if (line.startsWith('status(')) {
            // Use regex to extract the status value between the parentheses
            const match = line.match(/status\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Remove quotes from the status value and store it
                parsed.status = match[1].replace(/^"|"$/g, '');
            }
        }
        
        // Check if this line starts with the production-time function call
        if (line.startsWith('production-time(')) {
            // Use regex to extract the time value between the parentheses
            const match = line.match(/production-time\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Remove quotes from the time value and store it
                parsed.production_time = match[1].replace(/^"|"$/g, '');
            }
        }
        
        // Check if this line starts with the special-notes function call
        if (line.startsWith('special-notes(')) {
            // Use regex to extract the notes text between the parentheses
            const match = line.match(/special-notes\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Remove quotes from the notes text and store it
                parsed.special_notes = match[1].replace(/^"|"$/g, '');
            }
        }
    }
    
    // Return the parsed object with all extracted data
    return parsed;
}
```

**The parsed output would be:**
```json
{
    "flavors": ["strawberry", "vanilla_swirl", "caramel"],
    "price": 24.99,
    "status": "pending",
    "production_time": "2 days",
    "special_notes": "Customer requested extra sweet - will add extra sugar"
}
```

**Example: Multiple Function Calls**

The AI might return multiple function calls on separate lines:
```
flavors("strawberry", "vanilla_swirl")
price(24.99)
status("pending")
flavors("chocolate", "mint")
price(19.99)
status("pending")
```

**Example: Parsing Multiple Function Calls**

```javascript
// Define a function to parse multiple function calls from AI response
function parseMultipleFunctionCalls(aiResponse) {
    // Create an array to store all parsed results
    const results = [];
    // Create an object to hold the current result being built
    let currentResult = {
        flavors: [],  // Array for flavor names
        price: 0,     // Number for price
        status: ""    // String for status
    };
    
    // Split the response into an array of lines
    const lines = aiResponse.split('\n');
    
    // Loop through each line in the response
    for (let line of lines) {
        // Remove whitespace from the beginning and end
        line = line.trim();
        // Skip empty lines
        if (!line) continue;
        
        // Check if this line is a flavors function call
        if (line.startsWith('flavors(')) {
            // Check if we already have data in currentResult (means we're starting a new item)
            if (currentResult.flavors.length > 0 || currentResult.price > 0) {
                // Save the previous result to the results array (copy it, don't reference it)
                results.push({...currentResult});
                // Reset currentResult to empty for the new item
                currentResult = { flavors: [], price: 0, status: "" };
            }
            
            // Extract the flavors from between the parentheses
            const match = line.match(/flavors\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Get the string of flavors inside the parentheses
                const flavorString = match[1];
                // Split by comma, trim each, remove quotes, and store in currentResult
                currentResult.flavors = flavorString.split(',').map(f => f.trim().replace(/^"|"$/g, ''));
            }
        }
        
        // Check if this line is a price function call
        if (line.startsWith('price(')) {
            // Extract the price number from between the parentheses
            const match = line.match(/price\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Convert the string to a number and store in currentResult
                currentResult.price = parseFloat(match[1]);
            }
        }
        
        // Check if this line is a status function call
        if (line.startsWith('status(')) {
            // Extract the status value from between the parentheses
            const match = line.match(/status\(([^)]+)\)/);
            // Check if a match was found
            if (match) {
                // Remove quotes and store the status in currentResult
                currentResult.status = match[1].replace(/^"|"$/g, '');
                // Status is usually the last field, so we've completed this result
                // Save currentResult to results array (copy it)
                results.push({...currentResult});
                // Reset currentResult for the next item
                currentResult = { flavors: [], price: 0, status: "" };
            }
        }
    }
    
    // Check if there's an incomplete result left (didn't end with status)
    if (currentResult.flavors.length > 0 || currentResult.price > 0) {
        // Save the incomplete result to the results array
        results.push(currentResult);
    }
    
    // Return the array of all parsed results
    return results;
}
```

**The parsed output would be:**
```json
[
    {
        "flavors": ["strawberry", "vanilla_swirl"],
        "price": 24.99,
        "status": "pending"
    },
    {
        "flavors": ["chocolate", "mint"],
        "price": 19.99,
        "status": "pending"
    }
]
```

### Complete Flow: Format → AI → Parse → Save

Here's how it all works together:

**Example: Complete Flow**

```javascript
// Define an async function to create an order using AI (async means it can wait for operations)
async function createOrderWithAI(orderData, customerInfo) {
    // Step 1: Call the formatting function to convert JSON data into prompt format
    const formattedInput = formatOrderForAI(orderData, customerInfo);
    
    // Step 2: Send the formatted input to the AI service and wait for response (await pauses until done)
    const aiResponse = await sendToAIService(formattedInput);
    
    // Step 3: Parse the AI response to extract structured data from function calls
    const parsedResponse = parseAIResponse(aiResponse);
    
    // Step 4: Create an order object using the original data and parsed AI response
    const order = {
        customer_id: orderData.customer_id,              // Use customer ID from original request
        flavor_combination: parsedResponse.flavors,      // Use flavors from parsed AI response
        size: orderData.size,                            // Use size from original request
        quantity: orderData.quantity,                    // Use quantity from original request
        status: parsedResponse.status,                    // Use status from parsed AI response
        price: parsedResponse.price,                     // Use price from parsed AI response
        special_instructions: parsedResponse.special_notes, // Use notes from parsed AI response
        created_at: new Date()                           // Set current date/time as creation timestamp
    };
    
    // Step 5: Save the order object to the database and wait for it to complete
    const savedOrder = await database.save(order);
    
    // Step 6: Return a formatted response object to send back to the frontend
    return {
        message: "Order created successfully",  // Success message for the user
        order: savedOrder                       // The saved order data with database ID
    };
}
```

### Common Function Call Parsing Patterns

**Pattern 1: Single argument function**
```
flavors("strawberry", "vanilla")
price(24.99)
status("pending")
```

**Pattern 2: Multiple arguments**
```
flavors("strawberry", "vanilla", "caramel")
```

**Pattern 3: Quoted strings**
```
status("pending")
special-notes("Customer requested extra sweet")
```

**Pattern 4: Numbers (no quotes)**
```
price(24.99)
quantity(12)
```

When parsing function calls, always:
- Use regex to extract content between parentheses: `functionName\(([^)]+)\)`
- Split multiple arguments by comma
- Remove quotes from string arguments
- Convert number strings to actual numbers
- Handle empty lines
- Trim whitespace

---

## CRUD Operations in Detail

### CREATE: Adding New Data

**What it does:** Creates a new record in the database.

**Example: Creating a new lollipop order**

**Input Example: Create order request**

```json
POST /orders
{
    "customer_id": 456,
    "flavor_combination": ["strawberry", "vanilla_swirl"],
    "size": "large",
    "quantity": 12,
    "special_instructions": "Extra sweet, please!"
}
```

**What happens on the backend:**

1. **Authenticate**: Verify the customer is who they say they are
2. **Parse**: Read the JSON data
3. **Validate**: 
   - Check flavors exist and are compatible
   - Check size is valid ("small", "medium", or "large")
   - Check quantity is between 1 and 100
4. **Sanitize**: Clean special_instructions
5. **Calculate**: Determine price based on size and quantity
6. **Create**: Save the order to database with status "pending"
7. **Return**: Send back the created order

**Output Example: Create order response**

```json
{
    "message": "Order created successfully",
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "size": "large",
        "quantity": 12,
        "status": "pending",
        "price": 24.99,
        "special_instructions": "Extra sweet, please!",
        "created_at": "2024-01-15T10:30:00Z"
    }
}
```

**Key points:**
- The backend assigns an ID (789) automatically
- The backend sets default values (status: "pending")
- The backend calculates derived values (price: 24.99)

### READ: Retrieving Data

There are two types of read operations:

#### Reading a Single Item

**What it does:** Gets one specific record.

**Input Example: Get single order**

```
GET /orders/789
```

**What happens on the backend:**

1. **Extract** order ID: 789
2. **Authenticate**: Verify the customer
3. **Authorize**: Check if this order belongs to this customer
4. **Query**: Find order #789 in database
5. **Return**: Send back the order details

**Output Example: Get single order response**

```json
{
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "size": "large",
        "quantity": 12,
        "status": "mixing",
        "price": 24.99,
        "special_instructions": "Extra sweet, please!",
        "created_at": "2024-01-15T10:30:00Z",
        "updated_at": "2024-01-15T11:00:00Z"
    }
}
```

#### Reading Multiple Items (List)

**What it does:** Gets multiple records, often with filtering and pagination.

**Input Example: Get list of orders**

```
GET /orders?customer_id=456&status=pending&page=1&limit=20
```

**What happens on the backend:**

1. **Parse** query parameters:
   - customer_id: 456
   - status: "pending"
   - page: 1
   - limit: 20
2. **Authenticate**: Verify the customer
3. **Build query**: Find orders WHERE customer_id = 456 AND status = "pending"
4. **Count total**: How many orders match? (e.g., 3 orders)
5. **Apply pagination**: Skip 0, take 20 (for page 1)
6. **Order**: Sort by newest first
7. **Return**: Send back the list with pagination info

**Output Example: Get list of orders response**

```json
{
    "orders": [
        {
            "id": 789,
            "flavor_combination": ["strawberry", "vanilla_swirl"],
            "size": "large",
            "quantity": 12,
            "status": "pending",
            "price": 24.99,
            "created_at": "2024-01-15T10:30:00Z"
        },
        {
            "id": 788,
            "flavor_combination": ["chocolate", "mint"],
            "size": "medium",
            "quantity": 6,
            "status": "pending",
            "price": 12.99,
            "created_at": "2024-01-14T15:20:00Z"
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 20,
        "total": 3,
        "total_pages": 1
    }
}
```

**Key points:**
- Pagination helps when there are many results
- Filters let users see only what they want
- Total count helps the frontend show "Page 1 of 3" etc.

### UPDATE: Modifying Existing Data

**What it does:** Changes an existing record.

**Input Example: Update order**

```json
PUT /orders/789
{
    "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
    "quantity": 24
}
```

**What happens on the backend:**

1. **Extract** order ID: 789
2. **Authenticate**: Verify the customer
3. **Authorize**: Check if order #789 belongs to this customer
4. **Check rules**: Can this order be updated? (Maybe only "pending" orders can be changed)
5. **Parse** update data
6. **Validate**: Check new flavor combination is valid
7. **Update**: Change only the provided fields
8. **Recalculate**: Update price based on new quantity
9. **Return**: Send back the updated order

**Output Example: Update order response**

```json
{
    "message": "Order updated successfully",
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
        "size": "large",
        "quantity": 24,
        "status": "pending",
        "price": 49.98,
        "updated_at": "2024-01-15T11:30:00Z"
    }
}
```

**Key points:**
- Only the fields provided are updated (partial update)
- Other fields stay the same
- Derived values (like price) are recalculated

### DELETE: Removing Data

**What it does:** Removes a record (or marks it as deleted).

**Input Example: Cancel order**

```
DELETE /orders/789
```

**What happens on the backend:**

1. **Extract** order ID: 789
2. **Authenticate**: Verify the customer
3. **Authorize**: Check if order #789 belongs to this customer
4. **Check rules**: Can this order be canceled? (Maybe "shipped" orders can't be canceled)
5. **Delete**: Remove from database OR mark as "canceled" (soft delete)
6. **Handle side effects**: Process refund if needed
7. **Return**: Send confirmation

**Output Example: Delete order response**

```json
{
    "message": "Order canceled successfully",
    "order_id": 789,
    "refund_amount": 24.99,
    "refund_processed": true
}
```

**Key points:**
- Always verify ownership before deleting
- Check business rules (when can things be deleted?)
- Handle related actions (refunds, notifications, etc.)

---

## Output Formatting: How Data Goes Out

After processing a request, the backend needs to send a **response** back to the frontend. This response should be consistent and well-structured.

### Consistent Response Structure

All responses should follow the same format so the frontend knows what to expect.

**Success Response Format:**
```json
{
    "message": "Operation completed successfully",
    "data": { /* the actual data */ },
    "pagination": { /* if it's a list */ }
}
```

**Error Response Format:**
```json
{
    "error": "Human-readable error message",
    "code": "ERROR_CODE"
}
```

### HTTP Status Codes

Status codes tell the frontend if the request succeeded or failed:

- **200 OK**: Everything worked! (for GET, PUT, DELETE)
- **201 Created**: Successfully created something new (for POST)
- **400 Bad Request**: The data you sent is invalid
- **401 Unauthorized**: You're not logged in
- **403 Forbidden**: You're logged in, but you don't have permission
- **404 Not Found**: The thing you're looking for doesn't exist
- **500 Internal Server Error**: Something went wrong on the server

**Output Example: Success response**

```json
{
    "message": "Order created successfully",
    "order": {
        "id": 789,
        "status": "pending",
        "price": 24.99
    }
}
```
Status code: **201 Created**

**Output Example: Error response**

```json
{
    "error": "Order not found",
    "code": "NOT_FOUND"
}
```
Status code: **404 Not Found**

### Formatting Related Data

Sometimes you need to include related information in your response.

**Output Example: Order with customer details**

```json
{
    "order": {
        "id": 789,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "status": "mixing",
        "price": 24.99
    },
    "customer": {
        "id": 456,
        "name": "Alice Sweettooth",
        "loyalty_points": 1250
    },
    "flavor_details": [
        {
            "name": "strawberry",
            "description": "Sweet and fruity"
        },
        {
            "name": "vanilla_swirl",
            "description": "Creamy vanilla"
        }
    ]
}
```

This gives the frontend all the information it needs in one response, so it doesn't have to make multiple requests.

---

## Complete Example: Creating and Managing Orders

Let's walk through a complete scenario where Alice creates and manages her lollipop orders.

### Step 1: Alice Creates a New Order

**Input Example: Create order**

```json
POST /orders
Headers: {
    "Authorization": "Bearer abc123token"
}
Body: {
    "flavor_combination": ["strawberry", "vanilla_swirl"],
    "size": "large",
    "quantity": 12,
    "special_instructions": "Extra sweet, please!"
}
```

**Backend processing:**
1. Extract customer from token (Alice, customer_id: 456)
2. Parse the JSON body
3. Validate flavors, size, quantity
4. Sanitize special_instructions
5. Calculate price: $24.99
6. Create order in database
7. Return created order

**Output Example: Create order response**

```json
{
    "message": "Order created successfully",
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "size": "large",
        "quantity": 12,
        "status": "pending",
        "price": 24.99,
        "special_instructions": "Extra sweet, please!",
        "created_at": "2024-01-15T10:30:00Z"
    }
}
```

### Step 2: Alice Views Her Orders

**Input Example: List orders**

```
GET /orders?status=pending&page=1&limit=10
Headers: {
    "Authorization": "Bearer abc123token"
}
```

**Backend processing:**
1. Extract customer_id from token (456)
2. Parse query: status=pending, page=1, limit=10
3. Query database: WHERE customer_id = 456 AND status = "pending"
4. Count total: 3 orders
5. Apply pagination
6. Return results

**Output Example: List orders response**

```json
{
    "orders": [
        {
            "id": 789,
            "flavor_combination": ["strawberry", "vanilla_swirl"],
            "size": "large",
            "quantity": 12,
            "status": "pending",
            "price": 24.99,
            "created_at": "2024-01-15T10:30:00Z"
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 10,
        "total": 3,
        "total_pages": 1
    }
}
```

### Step 3: Alice Views a Specific Order

**Input Example: Get single order**

```
GET /orders/789
Headers: {
    "Authorization": "Bearer abc123token"
}
```

**Backend processing:**
1. Extract customer_id from token (456)
2. Extract order ID: 789
3. Query: WHERE id = 789 AND customer_id = 456
4. Return order details

**Output Example: Get single order response**

```json
{
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "size": "large",
        "quantity": 12,
        "status": "pending",
        "price": 24.99,
        "special_instructions": "Extra sweet, please!",
        "created_at": "2024-01-15T10:30:00Z"
    }
}
```

### Step 4: Alice Updates Her Order

**Input Example: Update order**

```json
PUT /orders/789
Headers: {
    "Authorization": "Bearer abc123token"
}
Body: {
    "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
    "quantity": 24
}
```

**Backend processing:**
1. Verify ownership (order 789 belongs to customer 456)
2. Check if order can be updated (status is "pending" ✓)
3. Validate new flavor combination
4. Recalculate price: $49.98
5. Update order
6. Return updated order

**Output Example: Update order response**

```json
{
    "message": "Order updated successfully",
    "order": {
        "id": 789,
        "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
        "size": "large",
        "quantity": 24,
        "status": "pending",
        "price": 49.98,
        "updated_at": "2024-01-15T11:30:00Z"
    }
}
```

### Step 5: Alice Cancels Her Order

**Input Example: Cancel order**

```
DELETE /orders/789
Headers: {
    "Authorization": "Bearer abc123token"
}
```

**Backend processing:**
1. Verify ownership
2. Check if order can be canceled (status is "pending" ✓)
3. Mark order as "canceled"
4. Process refund
5. Return confirmation

**Output Example: Cancel order response**

```json
{
    "message": "Order canceled successfully",
    "order_id": 789,
    "refund_amount": 49.98,
    "refund_processed": true
}
```

---

## Using AI to Build Endpoints

If you're using an AI-powered IDE (like Cursor, GitHub Copilot, or similar), you can use prompts to help build your endpoints. Here are example prompts you can use, based on the patterns in this document.

### Prompt Template for Creating an Endpoint

Use this template and fill in the details for your specific endpoint:

```
I need to build a [CREATE/READ/UPDATE/DELETE] endpoint for [resource name].

The endpoint should:
1. Accept input in this format: [paste Input Example from the document]
2. Format this input using the formatting pattern shown in the "Formatting Input" section (convert JSON to prompt format)
3. Send the formatted input to an AI service
4. Receive a response from the AI in function call format (like flavors("strawberry"), price(24.99), status("pending"))
5. Parse the AI response using the parsing pattern shown in the "Parsing the Response" section (extract data from function calls like flavors(), price(), status())
6. Save the parsed data to the database
7. Return this format to the frontend: [paste Output Example from the document]

Please include frequent comments throughout the code explaining what each line does so I can learn as I go. Also include:
- Input validation
- Authentication check
- Authorization check (verify ownership)
- Error handling with appropriate HTTP status codes
- Consistent response formatting
```

### Example Prompts

#### Creating a "Create Order" Endpoint

```
I need to build a CREATE endpoint for lollipop orders.

The endpoint should:
1. Accept input in this format:
{
    "customer_id": 456,
    "flavor_combination": ["strawberry", "vanilla_swirl"],
    "size": "large",
    "quantity": 12,
    "special_instructions": "Extra sweet, please!"
}

2. Format this input using the formatting pattern from the "Formatting Input" section (convert JSON to prompt format)
3. Send the formatted input to an AI service to generate order suggestions
4. Receive a response from the AI in function call format: flavors("strawberry", "vanilla_swirl"), price(24.99), status("pending")
5. Parse the AI response using the parsing pattern from the "Parsing the Response" section (extract data from function calls like flavors(), price(), status())
6. Save the parsed order data to the database
5. Return this format to the frontend:
{
    "message": "Order created successfully",
    "order": {
        "id": 789,
        "customer_id": 456,
        "flavor_combination": ["strawberry", "vanilla_swirl"],
        "size": "large",
        "quantity": 12,
        "status": "pending",
        "price": 24.99,
        "created_at": "2024-01-15T10:30:00Z"
    }
}

Please include frequent comments throughout the code explaining what each line does so I can learn as I go. Also include:
- Input validation (check flavors exist, size is valid, quantity is 1-100)
- Authentication check (verify user is logged in)
- Authorization check (verify customer_id matches authenticated user)
- Error handling with appropriate HTTP status codes (400 for validation errors, 401 for auth errors, 500 for server errors)
- Consistent response formatting
```

#### Creating a "Get Orders" Endpoint

```
I need to build a READ endpoint to get a list of orders.

The endpoint should:
1. Accept query parameters: ?customer_id=456&status=pending&page=1&limit=20
2. Query the database for orders matching these filters
3. Return this format to the frontend:
{
    "orders": [
        {
            "id": 789,
            "flavor_combination": ["strawberry", "vanilla_swirl"],
            "size": "large",
            "quantity": 12,
            "status": "pending",
            "price": 24.99,
            "created_at": "2024-01-15T10:30:00Z"
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 20,
        "total": 3,
        "total_pages": 1
    }
}

Please include frequent comments throughout the code explaining what each line does so I can learn as I go. Also include:
- Parse query parameters with defaults (page=1, limit=20, max limit=100)
- Authentication check (verify user is logged in)
- Authorization check (only return orders for the authenticated user)
- Pagination logic (calculate offset, total pages)
- Error handling with appropriate HTTP status codes
- Consistent response formatting
```

#### Creating an "Update Order" Endpoint

```
I need to build an UPDATE endpoint for orders.

The endpoint should:
1. Accept the order ID from the URL: /orders/789
2. Accept input in this format:
{
    "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
    "quantity": 24
}

3. Format this input using the formatting pattern from the "Formatting Input" section (convert JSON to prompt format)
4. Send the formatted input to an AI service to validate the update
5. Receive a response from the AI in function call format: flavors("strawberry", "vanilla_swirl", "caramel"), price(49.98), status("pending")
6. Parse the AI response using the parsing pattern from the "Parsing the Response" section (extract data from function calls)
7. Update the order in the database with the parsed data
6. Recalculate the price based on new quantity
7. Return this format to the frontend:
{
    "message": "Order updated successfully",
    "order": {
        "id": 789,
        "flavor_combination": ["strawberry", "vanilla_swirl", "caramel"],
        "size": "large",
        "quantity": 24,
        "status": "pending",
        "price": 49.98,
        "updated_at": "2024-01-15T11:30:00Z"
    }
}

Please include frequent comments throughout the code explaining what each line does so I can learn as I go. Also include:
- Extract order ID from URL parameter
- Parse and validate input
- Authentication check (verify user is logged in)
- Authorization check (verify order belongs to authenticated user)
- Business rule check (can this order be updated? maybe only "pending" orders)
- Partial update logic (only update provided fields)
- Price recalculation
- Error handling with appropriate HTTP status codes (404 if not found, 403 if not authorized)
- Consistent response formatting
```

#### Creating a "Delete Order" Endpoint

```
I need to build a DELETE endpoint for orders.

The endpoint should:
1. Accept the order ID from the URL: /orders/789
2. Verify the order exists and belongs to the authenticated user
3. Check if the order can be canceled (business rules - maybe only "pending" orders)
4. Mark the order as "canceled" in the database (soft delete)
5. Process a refund if needed
6. Return this format to the frontend:
{
    "message": "Order canceled successfully",
    "order_id": 789,
    "refund_amount": 24.99,
    "refund_processed": true
}

Please include frequent comments throughout the code explaining what each line does so I can learn as I go. Also include:
- Extract order ID from URL parameter
- Authentication check (verify user is logged in)
- Authorization check (verify order belongs to authenticated user)
- Business rule check (can this order be canceled?)
- Soft delete logic (mark as canceled, don't actually delete)
- Refund processing
- Error handling with appropriate HTTP status codes (404 if not found, 403 if not authorized, 400 if can't be canceled)
- Consistent response formatting
```

### Tips for Using AI Prompts

1. **Be specific about the format**: Always include the exact input and output formats you want
2. **Request comments**: Always ask for frequent comments so you can learn
3. **Include security**: Always mention authentication, authorization, and validation
4. **Specify error handling**: Tell the AI what status codes to use for different errors
5. **Mention consistency**: Ask for consistent response formatting

### The Complete Flow

When building an endpoint with AI, the flow should be:

```
1. Frontend sends request → 
2. Backend receives and parses input → 
3. Backend formats input for AI service → 
4. Backend sends to AI service → 
5. Backend receives AI response → 
6. Backend parses AI response → 
7. Backend saves to database → 
8. Backend formats response → 
9. Backend returns response to frontend
```

Make sure your AI prompt covers all these steps!

---

## Security Basics

Security is critical for any application. Here are the essential concepts.

### Authentication: Who Are You?

**Authentication** answers: "Who are you?" It verifies that a user is who they claim to be.

**How it works:**
1. User logs in with email and password
2. Backend verifies the credentials
3. Backend creates a **token** (like a temporary ID card)
4. Backend sends token to frontend
5. Frontend includes token in all future requests
6. Backend validates token to identify the user

**Input Example: Login request**

```json
POST /login
{
    "email": "alice@example.com",
    "password": "securepassword123"
}
```

**Output Example: Login response**

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "customer_id": 456
}
```

**Subsequent requests include the token:**

**Input Example: Authenticated request**

```
GET /orders
Headers: {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

The backend extracts the token, validates it, and knows this is Alice (customer_id: 456).

**Key principles:**
- Never store passwords in plain text (always hash them)
- Tokens should expire after a reasonable time
- Always use HTTPS to encrypt data in transit

### Authorization: What Are You Allowed to Do?

**Authorization** answers: "What are you allowed to do?" It determines what resources a user can access.

**Example: Order ownership check**

Alice tries to view order #789:

**Input Example: Authorized request**

```
GET /orders/789
Headers: {
    "Authorization": "Bearer alice_token"
}
```

**Backend checks:**
1. Is this a valid token? → Yes, it's Alice (customer_id: 456)
2. Does order 789 belong to customer 456? → Yes ✓
3. Return order data

**If Bob tries to access Alice's order:**

**Input Example: Unauthorized request**

```
GET /orders/789
Headers: {
    "Authorization": "Bearer bob_token"
}
```

**Backend checks:**
1. Is this a valid token? → Yes, it's Bob (customer_id: 789)
2. Does order 789 belong to customer 789? → No!
3. Return 403 Forbidden

**Output Example: Authorization error**

```json
{
    "error": "Access denied. This order does not belong to you.",
    "code": "FORBIDDEN"
}
```

### API Keys: Never Expose on Frontend

**Why this matters:**
- Frontend code is visible to anyone
- API keys in frontend can be stolen
- Stolen keys can be used to make unauthorized requests

**Bad approach (DON'T DO THIS):**
```javascript
// ❌ NEVER DO THIS - Anyone can see this!
const API_KEY = "sk_live_1234567890abcdef";
```

**Good approach:**
- Frontend calls YOUR backend
- YOUR backend calls external services with the API key
- API key stays on the server, never exposed

**Input Example: Frontend to backend**

```json
POST /api/generate-lollipop-suggestion
{
    "preferences": "sweet"
}
```

The frontend doesn't know about any external API keys. Your backend handles that securely.

**Key principle:**
- Store secrets (API keys, passwords) in environment variables on the server
- Never commit secrets to version control
- Never send secrets to the frontend

### Rate Limiting: Preventing Abuse

**What is rate limiting?**
Rate limiting prevents abuse by limiting how many requests a user can make in a given time period.

**Why it matters:**
- Prevents API abuse
- Protects against attacks
- Ensures fair resource usage
- Controls costs

**Example:**
A customer tries to create 100 orders in 1 minute.

**Without rate limiting:**
- All 100 orders are processed
- Server is overwhelmed
- Costs skyrocket

**With rate limiting:**
- First 10 requests succeed
- Remaining 90 requests return error
- Server stays healthy

**Output Example: Rate limit error**

```json
{
    "error": "Rate limit exceeded. Please wait before making another request.",
    "code": "RATE_LIMIT_EXCEEDED",
    "retry_after": 60
}
```

**Common limits:**
- Free tier: 10 requests per hour
- Premium tier: 1000 requests per hour

### Input Validation and Sanitization

**Why validate input?**
- Prevents malicious data from entering your system
- Ensures data integrity
- Protects against attacks

**Validation checks:**
1. **Type**: Is quantity actually a number?
2. **Range**: Is quantity between 1 and 100?
3. **Format**: Are flavors from the approved list?
4. **Sanitization**: Remove dangerous HTML/scripts

**Input Example: Invalid input**

```json
{
    "quantity": "not a number"
}
```

**Output Example: Validation error**

```json
{
    "error": "Invalid input: 'quantity' must be a number",
    "code": "VALIDATION_ERROR"
}
```

**Input Example: Dangerous input**

```json
{
    "special_instructions": "<script>alert('hack')</script>Extra sweet!"
}
```

**After sanitization:**
```json
{
    "special_instructions": "Extra sweet!"
}
```

The dangerous script is removed.

### HTTPS: Encrypting Data

**Always use HTTPS in production:**
- Encrypts data in transit
- Prevents man-in-the-middle attacks
- Required for sensitive data (passwords, payment info)

**What HTTPS does:**
- Encrypts data between client and server
- Verifies server identity
- Protects against eavesdropping

---

## Summary

### Key Principles

1. **Always authenticate and authorize** before processing requests
2. **Validate and sanitize** all input data
3. **Use consistent response formats** for success and error cases
4. **Include ownership checks** in all database queries
5. **Handle errors gracefully** with appropriate HTTP status codes
6. **Provide pagination** for list endpoints
7. **Structure responses** for easy frontend consumption

### The CRUD Flow

**Create:**
- Parse input → Validate → Create in database → Return created record

**Read:**
- Parse parameters → Authenticate → Query database → Return results

**Update:**
- Parse input → Verify ownership → Validate → Update database → Return updated record

**Delete:**
- Verify ownership → Check rules → Delete from database → Return confirmation

### Security Checklist

- ✅ Implement authentication (verify who the user is)
- ✅ Implement authorization (verify what they can access)
- ✅ Never expose API keys on the frontend
- ✅ Use environment variables for secrets
- ✅ Implement rate limiting
- ✅ Validate and sanitize all input
- ✅ Use HTTPS in production
- ✅ Don't expose sensitive information in error messages

---

This guide provides a foundation for building robust, secure CRUD operations. Remember: security is not optional—it's essential for protecting your users and your application.
