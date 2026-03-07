---
name: security-review
description: 在添加身份验证、处理用户输入、处理机密信息、创建API端点或实现支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
origin: ECC
---

# 安全审查技能

此技能确保所有代码遵循安全最佳实践，并识别潜在漏洞。

## 何时激活

* 实现身份验证或授权时
* 处理用户输入或文件上传时
* 创建新的 API 端点时
* 处理密钥或凭据时
* 实现支付功能时
* 存储或传输敏感数据时
* 集成第三方 API 时

## 安全检查清单

### 1. 密钥管理

#### ❌ 绝对不要这样做

```typescript
const apiKey = "sk-proj-xxxxx"  // Hardcoded secret
const dbPassword = "password123" // In source code
```

#### ✅ 始终这样做

```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// Verify secrets exist
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 验证步骤

* \[ ] 没有硬编码的 API 密钥、令牌或密码
* \[ ] 所有密钥都存储在环境变量中
* \[ ] `.env` 文件在 .gitignore 中
* \[ ] git 历史记录中没有密钥
* \[ ] 生产环境密钥存储在托管平台中（Vercel, Railway）

### 2. 输入验证

#### 始终验证用户输入

```typescript
import { z } from 'zod'

// Define validation schema
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// Validate before processing
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 文件上传验证

```typescript
function validateFileUpload(file: File) {
  // Size check (5MB max)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // Type check
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // Extension check
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 验证步骤

* \[ ] 所有用户输入都使用模式进行了验证
* \[ ] 文件上传受到限制（大小、类型、扩展名）
* \[ ] 查询中没有直接使用用户输入
* \[ ] 使用白名单验证（而非黑名单）
* \[ ] 错误消息不会泄露敏感信息

### 3. SQL 注入防护

#### ❌ 绝对不要拼接 SQL

```typescript
// DANGEROUS - SQL Injection vulnerability
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 始终使用参数化查询

```typescript
// Safe - parameterized query
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// Or with raw SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 验证步骤

* \[ ] 所有数据库查询都使用参数化查询
* \[ ] SQL 中没有字符串拼接
* \[ ] 正确使用 ORM/查询构建器

#### 验证步骤

* \[ ] 实现了基于角色的访问控制
* \[ ] 会话管理安全

### 5. XSS 防护

#### 清理 HTML

```typescript
import DOMPurify from 'isomorphic-dompurify'

// ALWAYS sanitize user-provided HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### 验证步骤

* \[ ] 用户提供的 HTML 已被清理
* \[ ] 没有渲染未经验证的动态内容


### 6. 敏感数据泄露

#### 日志记录

```typescript
// ❌ WRONG: Logging sensitive data
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// ✅ CORRECT: Redact sensitive data
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### 错误消息

```typescript
// ❌ WRONG: Exposing internal details
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ CORRECT: Generic error messages
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 验证步骤

* \[ ] 日志中没有密码、令牌或密钥
* \[ ] 对用户显示通用错误消息
* \[ ] 详细错误信息仅在服务器日志中
* \[ ] 没有向用户暴露堆栈跟踪

**请记住**：安全不是可选项。一个漏洞就可能危及整个平台。如有疑问，请谨慎行事。
