---
date: '2025-01-15T00:00:00+09:00'
title: 'Code Highlight Test'
draft: false
description: "코드 하이라이팅 스타일 테스트용 포스트"
tags: [test, code]
categories: [test]
---

# 코드 하이라이팅 스타일 테스트

이 포스트는 다양한 프로그래밍 언어의 코드 하이라이팅을 테스트하기 위한 것입니다.

## JavaScript/TypeScript

```javascript
// React Hook 예시
import { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    let interval = null;
    if (isActive) {
      interval = setInterval(() => {
        setCount(count => count + 1);
      }, 1000);
    } else if (!isActive && count !== 0) {
      clearInterval(interval);
    }
    return () => clearInterval(interval);
  }, [isActive, count]);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setIsActive(!isActive)}>
        {isActive ? 'Pause' : 'Start'}
      </button>
    </div>
  );
}
```

## Python

```python
# 데이터 처리 예시
from typing import List, Dict
import json

def process_data(data: List[Dict]) -> Dict:
    """데이터를 처리하고 통계를 반환합니다."""
    if not data:
        return {"error": "No data provided"}
    
    total = len(data)
    processed = [item for item in data if item.get("status") == "active"]
    
    return {
        "total": total,
        "active": len(processed),
        "percentage": (len(processed) / total * 100) if total > 0 else 0
    }

# 사용 예시
data = [
    {"id": 1, "name": "Item 1", "status": "active"},
    {"id": 2, "name": "Item 2", "status": "inactive"},
    {"id": 3, "name": "Item 3", "status": "active"}
]

result = process_data(data)
print(json.dumps(result, indent=2))
```

## Java

```java
// Spring Boot Controller 예시
package com.example.controller;

import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.findAll();
        return ResponseEntity.ok(users);
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.save(user);
        return ResponseEntity.status(201).body(created);
    }
}
```

## Go

```go
// HTTP 서버 예시
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

type Response struct {
    Message string    `json:"message"`
    Time    time.Time `json:"time"`
}

func handler(w http.ResponseWriter, r *http.Request) {
    response := Response{
        Message: "Hello, World!",
        Time:    time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## CSS

```css
/* 미니멀 디자인 스타일 */
:root {
    --primary: rgb(30, 30, 30);
    --secondary: rgb(108, 108, 108);
    --background: rgb(255, 255, 255);
    --border: rgb(238, 238, 238);
}

.post-entry {
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 1.5rem;
    transition: all 0.3s ease;
}

.post-entry:hover {
    border-color: var(--primary);
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

@media (max-width: 768px) {
    .post-entry {
        padding: 1rem;
    }
}
```

## HTML

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>코드 하이라이팅 테스트</title>
</head>
<body>
    <header>
        <h1>Code Highlight Test</h1>
    </header>
    <main>
        <article>
            <h2>JavaScript 예시</h2>
            <pre><code class="language-javascript">
const greeting = "Hello, World!";
console.log(greeting);
            </code></pre>
        </article>
    </main>
    <footer>
        <p>&copy; 2025 AMT9 Devlog</p>
    </footer>
</body>
</html>
```

## SQL

```sql
-- 사용자 통계 쿼리
SELECT 
    u.id,
    u.name,
    u.email,
    COUNT(p.id) AS post_count,
    MAX(p.created_at) AS last_post_date
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.status = 'active'
GROUP BY u.id, u.name, u.email
HAVING COUNT(p.id) > 0
ORDER BY post_count DESC
LIMIT 10;
```

## Shell/Bash

```bash
#!/bin/bash

# 배포 스크립트 예시
set -e

echo "Starting deployment..."

# 환경 변수 확인
if [ -z "$DEPLOY_ENV" ]; then
    echo "Error: DEPLOY_ENV is not set"
    exit 1
fi

# 빌드
echo "Building application..."
npm run build

# 테스트
echo "Running tests..."
npm test

# 배포
echo "Deploying to $DEPLOY_ENV..."
hugo deploy --target=$DEPLOY_ENV

echo "Deployment completed successfully!"
```

## JSON

```json
{
  "name": "AMT9 Devlog",
  "version": "1.0.0",
  "description": "개발 블로그",
  "author": {
    "name": "amt9us",
    "email": "tmdgus877@gmail.com"
  },
  "keywords": [
    "backend",
    "devlog",
    "blog"
  ],
  "config": {
    "theme": "PaperMod",
    "language": "ko"
  }
}
```

## YAML

```yaml
# Hugo 설정 예시
baseURL: "https://amt9us.github.io/"
languageCode: ko
title: AMT9 Devlog

params:
  title: AMT9 Devlog
  description: "백엔드 개발 블로그"
  author: amt9us
  
  socialIcons:
    - name: github
      url: "https://github.com/amt9us"
    - name: email
      url: "mailto:tmdgus877@gmail.com"
```

## Docker

```dockerfile
# Dockerfile 예시
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Markdown

```markdown
# 제목
## 부제목

**굵게** *기울임* `코드`

- 리스트 1
- 리스트 2

[링크](https://example.com)
```

---

이 포스트를 사용하여 다양한 코드 하이라이팅 스타일을 테스트할 수 있습니다.
```

이 포스트는 다음을 포함합니다:
- JavaScript/TypeScript
- Python
- Java
- Go
- CSS
- HTML
- SQL
- Shell/Bash
- JSON
- YAML
- Docker
- Markdown

이 포스트로 여러 하이라이팅 스타일(monokai, github, dracula 등)을 비교할 수 있습니다.

파일을 생성한 후 `hugo server`로 확인하세요.
