# OpenAPI

## OpenAPI 简介

OpenAPI 是一个用于描述 RESTful API 的规范，前身是 Swagger。通过 OpenAPI 文档，开发者可以清晰地描述、生成和消费 API。它被广泛用于自动生成代码、文档和测试工具。

一个典型的 OpenAPI 文档以 JSON 或 YAML 格式编写，结构清晰，主要包括以下几个部分：

- **`info`**: 描述 API 的基本信息，比如标题、版本和描述。
- **`paths`**: 定义 API 的每个端点（URI），以及支持的 HTTP 方法（如 GET、POST）。
- **`components`**: 提供可复用的对象，比如 schema、参数和安全定义。
- **`servers`**: 列出 API 可用的服务器地址。
- **`security`**: 描述 API 的认证方式。

示例：

```go
openapi: 3.0.3
info:
  title: Sample API
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get all users
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

## OpenAPI 必须要写的字段

在 OpenAPI 规范中，文档的基础结构必须包含一些关键字段，确保 API 文档的有效性和可读性。以下是 OpenAPI 必须要写的字段及其详细说明：

------

### 必须字段列表

1. **`openapi`**
   - 描述 OpenAPI 规范的版本。
   - 必须是 `3.x.x` 格式（如 `3.0.0` 或 `3.1.0`）。
2. **`info`**
   - 提供 API 的元信息。
   - 必须包含以下子字段：
     - **`title`**：API 的标题。
     - **`version`**：API 的版本。
3. **`paths`**
   - 描述 API 的所有路径及其对应的操作（如 GET、POST）。
   - 必须至少包含一个路径。

------

### 示例及解释

以下是一个最小化的有效 OpenAPI 文档示例及详细解释：

```yaml
openapi: 3.0.3
info:
  title: Example API  # API 的标题，便于理解和描述用途
  version: 1.0.0     # API 的版本号，通常与软件版本一致
paths:
  /users:            # 定义一个路径
    get:             # 路径支持的 HTTP 方法（GET 请求）
      summary: Get a list of users
      description: Returns a list of all users.
      responses:
        '200':       # 状态码为 200 的响应
          description: Successful response
```

------

### 字段详细解释

#### 1. `openapi`

- **作用**：指定文档的 OpenAPI 版本。

- **重要性**：确保工具能够解析并使用文档。

- 示例

  ：

  ```yaml
  openapi: 3.0.3
  ```

#### 2. `info`

- **作用**：描述 API 的元信息，帮助开发者快速了解文档。

- 子字段

  ：

  - **`title`**：API 的名称。
  - **`version`**：API 的当前版本，通常与应用程序版本保持一致。

- 示例

  ：

  ```yaml
  info:
    title: Example API
    version: 1.0.0
  ```

#### 3. `paths`

- **作用**：定义 API 的所有端点及支持的操作。

- **重要性**：路径是 API 的核心，至少需要定义一个路径。

- 子字段

  ：

  - 路径：具体的 API 端点，例如 `/users`。
  - HTTP 方法：如 `get`、`post`、`put`。
  - **`responses`**：描述可能的响应情况，是必须的。

- 示例

  ：

  ```yaml
  paths:
    /users:
      get:
        summary: Get a list of users
        description: Returns a list of all users.
        responses:
          '200':
            description: Successful response
  ```

------

### 可选但常用字段

虽然不是必须字段，但以下内容在实际开发中非常重要：

1. **`servers`**

   - 描述 API 可用的服务器地址。

   ```yaml
   servers:
     - url: https://api.example.com
       description: Production server
   ```

2. **`components`**

   - 定义可复用的对象，如 schemas 和 parameters。

   ```yaml
   components:
     schemas:
       User:
         type: object
         properties:
           id:
             type: integer
           name:
             type: string
   ```

------

### 完整示例

以下是一个稍微复杂一些的 OpenAPI 文档，包含了更多字段：

```yaml
openapi: 3.0.3
info:
  title: Example API
  version: 1.0.0
  description: This is a simple example of OpenAPI documentation.
servers:
  - url: https://api.example.com
    description: Production server
paths:
  /users:
    get:
      summary: Get all users
      description: Fetch a list of users from the system.
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

------

## paths详细解析

### OpenAPI 中 `paths` 的详细写法

`paths` 是 OpenAPI 文档中描述 API 各个端点的核心部分，它定义了路径、支持的 HTTP 方法以及相关的请求与响应结构。以下详细介绍 `get`、`post`、`responses`、`parameters`、`in`、`schema`、`required` 和 `requestBody` 的写法和用途。

------

### 1. **`paths` 基本结构**

```yaml
paths:
  /example:
    get:
      summary: "Retrieve example data"
      responses:
        '200':
          description: "Successful response"
```

- 每个路径（如 `/example`）表示一个 API 端点。
- HTTP 方法（如 `get`、`post`）描述了路径的具体操作。

------

### 2. **`get` 和 `post`**

- **`get`**
  - 用于从服务器获取资源。
  - 通常无请求体，但可以有查询参数。
- **`post`**
  - 用于向服务器发送数据。
  - 通常需要请求体。

#### 示例

```yaml
paths:
  /users:
    get:
      summary: "Get all users"
      responses:
        '200':
          description: "A list of users"
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
    post:
      summary: "Create a new user"
      requestBody:
        description: "User object that needs to be created"
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
      responses:
        '201':
          description: "User created successfully"
```

------

### 3. **`responses`**

`responses` 描述 API 的响应状态及返回数据。

#### 基本结构

```yaml
responses:
  '200':
    description: "OK"
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/ExampleSchema'
```

- **状态码**：如 `200`、`404`。
- **`description`**：对响应的简短描述。
- **`content`**：描述响应的数据类型（如 JSON）及其结构。

#### 示例

```yaml
responses:
  '200':
    description: "Successful response"
    content:
      application/json:
        schema:
          type: object
          properties:
            id:
              type: integer
            name:
              type: string
  '400':
    description: "Bad request"
  '404':
    description: "Resource not found"
```

------

### 4. **`parameters`**

`parameters` 描述 API 中的请求参数（路径参数、查询参数、请求头等）。

#### 基本结构

```yaml
parameters:
  - name: "id"
    in: "path"
    description: "ID of the resource"
    required: true
    schema:
      type: "integer"
```

- **`name`**：参数名称。

- `in`

  ：参数位置，可选值包括：

  - `path`：路径参数，必须在 URL 中指定。
  - `query`：查询参数，通过 `?key=value` 格式传递。
  - `header`：请求头。
  - `cookie`：Cookie 参数。

- **`required`**：是否为必需参数。

- **`schema`**：参数的数据类型。

#### 示例

```yaml
paths:
  /users/{id}:
    get:
      summary: "Get user by ID"
      parameters:
        - name: "id"
          in: "path"
          description: "User ID"
          required: true
          schema:
            type: "integer"
      responses:
        '200':
          description: "User details"
```

------

### 5. **`in`**

`in` 用于指定参数的位置，取值包括：

- `path`：路径参数，必须在 URL 中定义，例如 `/users/{id}`。
- `query`：查询参数，例如 `/users?name=john`。
- `header`：请求头，例如 `Authorization: Bearer <token>`。
- `cookie`：Cookie 参数，例如 `session_id=abcd1234`。

------

### 6. **`schema`**

`schema` 定义数据的结构和类型。

#### 基本类型

- `string`
- `integer`
- `boolean`
- `array`
- `object`

#### 示例

```yaml
schema:
  type: object
  properties:
    id:
      type: integer
    name:
      type: string
  required:
    - id
    - name
```

------

### 7. **`required`**

`required` 用于指定参数或属性是否为必填。

#### 示例：参数必填

```yaml
parameters:
  - name: "id"
    in: "path"
    required: true
    schema:
      type: "integer"
```

#### 示例：属性必填

```yaml
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
      required:
        - id
        - name
```

------

### 8. **`requestBody`**

`requestBody` 描述请求体的结构和内容。

#### 基本结构

```yaml
requestBody:
  description: "Request body description"
  required: true
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/ExampleSchema'
```

#### 示例

```yaml
paths:
  /users:
    post:
      summary: "Create a new user"
      requestBody:
        description: "User object that needs to be created"
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                id:
                  type: integer
                name:
                  type: string
                email:
                  type: string
              required:
                - name
                - email
      responses:
        '201':
          description: "User created successfully"
```

------

### 综合完整示例

```yaml
openapi: 3.0.3
info:
  title: Example API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      summary: "Get user by ID"
      parameters:
        - name: "id"
          in: "path"
          description: "User ID"
          required: true
          schema:
            type: "integer"
      responses:
        '200':
          description: "Successful response"
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
  /users:
    post:
      summary: "Create a new user"
      requestBody:
        description: "User object that needs to be created"
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
      responses:
        '201':
          description: "User created successfully"
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
      required:
        - name
        - email
```

以上示例综合展示了 `get`、`post`、`responses`、`parameters`、`in`、`schema`、`required` 和 `requestBody` 的写法。