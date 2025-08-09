---
title: "API Documentation for DocsGPT"
weight: 3
subtitle: ""
excerpt: "API endpoint documentation for the open-source genAI tool DocsGPT."
draft: false
---

DocsGPT is a generative AI tool that helps users find, retrieve, and summarize information from uploaded knowledge sources. [Learn more about DocsGPT here.](https://github.com/arc53/DocsGPT)

------

The base URL for all API paths is `http://localhost.7901/`.

Currently, the application provides the following endpoints:


## 1. POST /api/answer 

Responds with answers to user-provided questions.

**Headers**:

```curl
curl -H "Content-Type: application/json; charset=utf-8"
```

**Sample JSON Request Body**:

```json
{
 "question": "Summarize current context",
 "api_key": "OPENAI_API_KEY",
 "embeddings_key":  "OPENAI_API_KEY",
 "active_docs": "67088ee3d14d6f75727f218c"
}
```

| Key | Data Type | Description | Use                                   |
|-----|-----------|-------------|---------------------------------------|
| question | string | The user's question. | Required                   |
| history | string | The conversation history. | Optional               |
| api_key | string | The API key for the selected LLM model. | Optional |
| embeddings_key | string | The API key for embeddings. If using OpenaAI, use the api_key value. | Optional |
| active_docs | string | The ID of the active documentation. | Optional |

**Sample Request**:

```curl
curl --location 'http://localhost:7091/api/answer' \
--header 'Content-Type: application/json; charset=utf-8' \
--data '{
    "question": "Hi",
    "active_docs": "67088ee3d14d6f75727f218c"
}'
```

**Sample JSON Response**:

```json
{
    "answer": "Hello! How can I assist you today? If you have any questions or need help with something specific, feel free to ask!",
    "conversation_id": "67196c4fb25c8573993bab33",
    "sources": [
        {
            "source": "https://docs.docsgpt.cloud/API/API-docs",
            "text": "API Endpoints Documentation - DocsGPT Documentation Welcome to the new DocsGPT",
            "title": "API Endpoints Documentation - DocsGPT Documentation"
        }
    ]
}
```

## 2. POST /api/docs_check

Checks if documentation is loaded on the server. Should be run every time the user is switching between documentation sources.

**Headers**:

```curl
curl -H "Content-Type: application/json; charset=utf-8"
```

**Sample JSON Request Body**:

```json
{
 "docs": "default"
}
```

| Key | Data Type | Description | Use                           |
|-----|-----------|-------------|-------------------------------|
| docs | string | The location of the documentation. | Required |

**Sample Request**:

```curl
curl --location 'http://localhost:7091/api/docs_check' \
--header 'Content-Type: application/json' \
--data '{
"docs": "default"
}'
```

**Sample JSON Response**:

```json
{
  "status": "exists"
}
```

## 3. GET /api/combine

Retrieves information about available vector databases.

**Sample Request**

```curl
curl --location 'http://localhost:7091/api/combine'
```

**Sample JSON Response:**

```json
[
  {
    "date": "default",
    "location": "remote",
    "model": "huggingface_sentence-transformers/all-mpnet-base-v2",
    "name": "default",
    "retriever": "classic",
    "tokens": ""
  },
  {
    "date": "Wed, 31 Jan 2024 12:00:00 GMT",
    "id": "67088ee3d14d6f75727f218c",
    "location": "local",
    "model": "huggingface_sentence-transformers/all-mpnet-base-v2",
    "name": "API-docs.md",
    "retriever": "classic",
    "syncFrequency": null,
    "tokens": "2030"
  },
  {
    "date": "duckduck_search",
    "location": "custom",
    "model": "huggingface_sentence-transformers/all-mpnet-base-v2",
    "name": "DuckDuckGo Search",
    "retriever": "duckduck_search",
    "tokens": ""
  }
]
```

| Key | Data Type | Description | Use                                        |
|-----|-----------|-------------|--------------------------------------------|
| date | string | When the document was uploaded. | Required                 |
| id | string | The ID of the uploaded document. | Optional         |
| location | string | Where the document is stored. | Required               |
| model | string | The LLM model used for training the document. |  Required |
| name | string | The name of the document. | Required                       |
| retriever | string | The retriever used to return the document. | Required |
| syncFrequency | string | How often the document is synced with the origin. Can be never, daily, weekly, monthly, or null. | Optional |
| tokens | string | Number of tokens used by the document. | Required        |

The JSON response values can also be viewed on [DocsHUB](https://github.com/arc53/DocsHUB), if the documentation is stored there.

<img width="295" alt="image" src="https://user-images.githubusercontent.com/15183589/224714085-f09f51a4-7a9a-4efb-bd39-798029bb4273.png">

On DocsHub, additional metadata fields include:

| Key | Data Type | Description | Use                                         |
|-----|-----------|-------------|---------------------------------------------|
| description | string | A description of the document's contents. | Required |
| docLink | string | The path to the document. | Optional                     |
| fullName | string | The document's full name. | Required                    |
| language | string | The coding language used in the document. |  Required   |
| version | string | The document version. | Optional                         |

## 4. POST /api/upload

Uploads a file to be vectorized and indexed.

**Request Body**: A multipart/form-data object with the following fields:

* `file`: The file to be uploaded from local device
* `user`: The source of the upload
* `name`: The user-designated file name

**Sample Request**:

```curl
curl --location 'http://localhost:7091/api/upload' \
--form 'file=@"YOUR-FILE-PATH"' \
--form 'user="local"' \
--form 'name="Name"'
```

**Sample JSON Response:**

```json
{
  "success": true,
  "task_id": "cef0a8f8-22a3-452f-bf7b-29d181c83ad9"
}
```

| Key | Data Type | Description | Use                                            |
|-----|-----------|-------------|------------------------------------------------|
| success | boolean | Indicates whether the upload was successful. | Required    |
| task_id | string | The ID of the uploaded document. | Required                 |

## 5. GET /api/task_status

Returns the status of a file upload with designated `task_id` from `/api/upload`.

**Query Parameter**:

| Key | Data Type | Description | Use |
|-----|-----------|-------------|-----|
| task_id | string | The ID of the task. | Required |

**Sample Request:**
```curl
curl --location 'http://localhost:7091/api/task_status?task_id=YOUR_TASK_ID'
```

**Response:**

There are two types of responses:

1. When the task is running, returns a `PROGRESS` value for `status` and a `current` value showing progress from 0 to 100:

   ```json
   {
     "result": {
       "current": 1
     },
     "status": "PROGRESS"
   }
   ```

2. When the task is complete, returns `SUCCESS` for `status`:

   ```json
   {
     "result": {
       "directory": "inputs",
       "filename": "API-docs.md",
       "formats": [
         ".rst",
         ".md",
         ".pdf"
         ".txt"
         ".docx"
         ".csv"
         ".epub"
         ".html"
         ".mdx"
       ],
       "limited": false,
       "name_job": "API-docs",
       "user": "local"
     },
     "status": "SUCCESS"
   }
   ```

**Error Messages**

If `task_id` is not provided, returns an error.

```json
{
    "message": "Task ID is required",
    "success": false
}
```

## 6. GET /api/delete_old

Deletes old indexes.

**Query Parameter**:

| Key | Data Type | Description | Use                                          |
|-----|-----------|-------------|----------------------------------------------|
| source_id | string | The source ID of the document to be deleted. | Required |

**Sample Request:**
```curl
curl --location 'http://localhost:7091/api/delete_old?source_id=0c8382a8-e544-4f75-b28c-037500417a7a'
```

**Sample JSON Response:**

```json
{
    "success": true
}
```

## 7. GET /api/get_api_keys

Retrieves a list of API keys for the user.

**Sample Request:**
```curl
curl --location 'http://localhost:7091/api/get_api_keys'
```

**Sample JSON Response:**

```json
[
  {
    "chunks": "2",
    "id": "670f41429e396d5266b1fea8",
    "key": "API-KEY-1",
    "name": "API-docs",
    "prompt_id": "default",
    "source": "API-docs"
  },
  {
    "chunks": "2",
    "id": "670f424d2ad68e881af5258b",
    "key": "API-KEY-1",
    "name": "DuckDuckGo Search",
    "prompt_id": "default",
    "source": "default"
  }
]
```

| Key | Data Type | Description | Use                                            |
|-----|-----------|-------------|------------------------------------------------|
| chunks | string | The number of chunks used to process an answer. | Required   |
| id | string | The string ID of the API key. | Required                         |
| key | string | The API key. | Required                                         |
| name | string | The name of the API key. | Required                            |
| prompt_id | string | The prompt used to generate the API key. Values can be default, strict, or creative. | Required |
| source | string | The source document used to generate the API key. | Required |

## 8. POST /api/create_api_key

Creates a new API key for the user.

**Headers**:

```curl
curl -H "Content-Type: application/json; charset=utf-8"
```

**Sample JSON Request Body**:

```json
{
  "name":"New Key",
  "prompt_id":"default",
  "chunks":"2",
  "source":"670f41429e396d5266b1fea8"
}
```

| Key | Data Type | Description | Use                                            |
|-----|-----------|-------------|------------------------------------------------|
| name | string | The user-designated name for the API key. | Required           |
| prompt_id | string | The prompt used to generate the API key. Values can be default, strict, ore creative. | Required |
| chunks | string | The number of chunks used to process an answer. | Required   |
| source | string | The source document used to generate the API key. | Required |

**Sample Request**:

```curl
curl --location 'http://localhost:7091/api/create_api_key' \
--header 'Content-Type: application/json; charset=utf-8' \
--data '{
    "name":"test",
    "prompt_id":"default",
    "chunks": "2",
    "source": "670f41429e396d5266b1fea8"
}'
```

**Sample JSON Response**:

```json
{
  "id": "670f46162ad68e881af5258e",
  "key": "YOUR-API-KEY"
}
```

## 9. POST /api/delete_api_key

Deletes a user's API key.

**Headers**:

```curl
curl -H "Content-Type: application/json; charset=utf-8"
```

**Request Body**:

```json
{
 "id": "YOUR-API-KEY"
}
```

| Key | Data Type | Description | Use                          |
|-----|-----------|-------------|------------------------------|
| id | string | The ID of the API key to be deleted | Required |

**Sample Request**:

```curl
curl --location --request GET 'http://localhost:7091/api/delete_old?source_id=6721a21540fd2cb5b73c95ae' \
--header 'Content-Type: application/json; charset=utf-8' \
--data '{
    "id" : "670f41429e396d5266b1fea8"
}'
```

**Sample JSON Response**:

```json
{
  "success": true
}
```

## 10. GET /api/get_conversations

Retrieves a list of the last 30 conversations with their IDs.

**Sample Request**

```curl
curl --location 'http://localhost:7091/api/get_conversations'
```

**Sample JSON Response**

```json
[
    {
        "id": "67200f1e6126e888c4d2fe84",
        "name": "\"Beginning a conversation.\""
    },
    {
        "id": "67196c4fb25c8573993bab33",
        "name": "Assistance offered."
    },
    {
        "id": "67196bfcb25c8573993bab2f",
        "name": "Prompt user engagement"
    }
]`
```

## 11. POST /api/delete_conversation

Deletes a conversation with the specified ID.

**Query Parameter**:

| Key | Data Type | Description | Use                            |
|-----|-----------|-------------|--------------------------------|
| id | string | ID of the conversation to be deleted. | Required |

**Sample Request**

```curl
curl --location --request POST 'http://localhost:7091/api/delete_conversation?id=670893ea22bbc7f1e1431960'
```

**Sample JSON Response**

```json
{
  "success": true
}
```

## 12. POST /stream

Streams the LLM's response to a user's question based on the specified retriever.

**Sample Request Body**

```json
{
"active_docs": "6721a200281f4c1a8b9a3a84",
"chunks": "2",
"conversation_id": "67196c4fb25c8573993bab33",
"history": "[{\"prompt\":\"Hi\",\"response\":\"Hello! How can I assist you today?\"},{\"prompt\":\"summarize current context\"}]",
"isNoneDoc": false,
"prompt_id": "creative",
"question": "summarize current context",
"token_limit": 2000
}
```

| Key | Data Type | Description | Use |
|-----|-----------|-------------|-----------|
| active_docs | string | The ID of the documentation used to generate the answer. | Optional |
| chunks | string | The number of chunks used to process an answer. | Required |
| conversation_id | string | The ID of the conversation. | Optional |
| history | string | The conversation history. | Optional |
| isNoneDoc | Boolean | Flag indicating if no document is selected. | Optional |
| prompt_id | string | The prompt type used to generate responses. Values can be default, strict, or creative. | Optional |
| question | string | The user's question. | Required |
| token_limit | integer | The maximum number of token uses allowed for generating the answer. | Optional |

**Sample Request**

```curl
curl --location 'http://localhost:7091/stream' \
--header 'Content-Type: application/json' \
--data '{
"active_docs": "6721a200281f4c1a8b9a3a84",
"chunks": "2",
"conversation_id": "67196c4fb25c8573993bab33",
"history": "[{\"prompt\":\"Hi\",\"response\":\"Hello! How can I assist you today?\"},{\"prompt\":\"summarize current context\"}]",
"isNoneDoc": false,
"prompt_id": "creative",
"question": "summarize current context",
"token_limit": 2000
}'
```
