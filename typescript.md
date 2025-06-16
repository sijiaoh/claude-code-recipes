# TypeScript 编程最佳实践

本文件定义了 TypeScript 编程的特定规范和最佳实践，遵循 TypeScript 社区广泛认同的标准。

**重要**: 使用 TypeScript 编程时，必须同时遵守：
- `base.md` - 通用编程最佳实践
- `oop.md` - 面向对象编程原则（当使用 OOP 时）
- 本文件中的 TypeScript 特定规范

## 核心原则

### 1. 严格类型设置
- **严格模式**: 始终使用 `strict: true` 配置
- **noImplicitAny**: 禁止隐式 any 类型
- **strictNullChecks**: 启用严格空值检查
- **noImplicitReturns**: 函数必须有明确的返回值

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### 2. 命名约定
- **变量和函数**: 使用 `camelCase` 命名
- **常量**: 使用 `UPPER_CASE` 命名
- **类名**: 使用 `PascalCase` 命名
- **接口**: 使用 `PascalCase` 命名，避免 `I` 前缀
- **类型别名**: 使用 `PascalCase` 命名
- **枚举**: 使用 `PascalCase` 命名，成员使用 `PascalCase`

```typescript
// 好的命名
const MAX_RETRY_COUNT = 3;
const userName = "alice";

interface User {
  name: string;
  email: string;
}

type UserStatus = "active" | "inactive";

enum UserRole {
  Admin = "admin",
  Member = "member",
  Guest = "guest"
}

class UserManager {
  private userRepository: UserRepository;
}
```

## 类型定义和使用

### 基础类型使用
- **优先使用具体类型**: 避免使用 `any`，优先使用具体类型
- **联合类型**: 使用联合类型表示多种可能的值
- **字面量类型**: 使用字面量类型增强类型安全性

```typescript
// 推荐
type Theme = "light" | "dark";
type Status = "pending" | "success" | "error";

function setTheme(theme: Theme): void {
  // 类型安全的实现
}

// 避免
function setTheme(theme: string): void {
  // 缺乏类型约束
}
```

### 接口设计
- **单一职责**: 每个接口应该有明确的职责
- **组合优于继承**: 使用接口组合构建复杂类型
- **可选属性**: 合理使用可选属性标记

```typescript
interface BaseUser {
  readonly id: string;
  name: string;
  email: string;
}

interface UserPreferences {
  theme: Theme;
  language: string;
  notifications: boolean;
}

interface User extends BaseUser {
  preferences?: UserPreferences;
  createdAt: Date;
  updatedAt: Date;
}
```

### 泛型使用
- **有意义的泛型约束**: 使用泛型约束增强类型安全
- **默认泛型参数**: 为常用泛型提供默认值
- **泛型命名**: 使用有意义的泛型名称

```typescript
// 好的泛型设计
interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

interface ApiResponse<TData = unknown> {
  data: TData;
  status: number;
  message: string;
}

// 泛型约束
function processItems<T extends { id: string; name: string }>(
  items: T[]
): T[] {
  return items.filter(item => item.name.length > 0);
}
```

## 函数和方法设计

### 函数签名
- **明确的参数类型**: 所有参数都应有明确类型
- **返回类型声明**: 明确声明函数返回类型
- **可选参数**: 可选参数应放在必需参数之后

```typescript
// 推荐的函数签名
function createUser(
  userData: {
    name: string;
    email: string;
    age?: number;
  }
): Promise<User> {
  // 实现
}

// 使用类型别名简化
type CreateUserRequest = {
  name: string;
  email: string;
  age?: number;
};

function createUser(userData: CreateUserRequest): Promise<User> {
  // 实现
}
```

### 箭头函数和普通函数
- **优先使用箭头函数**: 对于简短的函数表达式
- **普通函数**: 用于需要 `this` 绑定或函数提升的场景
- **async/await**: 优先使用 async/await 而非 Promise 链

```typescript
// 箭头函数 - 适用于简短逻辑
const validateEmail = (email: string): boolean => {
  return email.includes("@") && email.includes(".");
};

// 普通函数 - 适用于复杂逻辑或需要提升
function processUserData(userData: CreateUserRequest): User {
  // 复杂的处理逻辑
  return user;
}

// async/await
async function fetchUser(id: string): Promise<User | null> {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    console.error(`Failed to fetch user ${id}:`, error);
    return null;
  }
}
```

## 类和面向对象编程

### TypeScript 特定 OOP 实践
- **遵循 oop.md 原则**: 严格遵循面向对象编程最佳实践
- **访问修饰符**: 明确使用 `public`、`private`、`protected`
- **readonly 属性**: 对不可变属性使用 `readonly`
- **抽象类**: 合理使用抽象类定义基础行为

```typescript
abstract class BaseEntity {
  protected readonly id: string;
  protected readonly createdAt: Date;

  constructor(id: string) {
    this.id = id;
    this.createdAt = new Date();
  }

  public getId(): string {
    return this.id;
  }

  public abstract validate(): boolean;
}

class User extends BaseEntity {
  private email: string;
  private name: string;

  constructor(id: string, email: string, name: string) {
    super(id);
    this.email = email;
    this.name = name;
  }

  public validate(): boolean {
    return this.email.includes("@") && this.name.length > 0;
  }

  public getEmail(): string {
    return this.email;
  }

  public changeName(newName: string): void {
    if (newName.length === 0) {
      throw new Error("Name cannot be empty");
    }
    this.name = newName;
  }
}
```

## 错误处理

### TypeScript 错误处理最佳实践
- **自定义错误类**: 创建特定的错误类型
- **结果类型**: 使用 Result 模式处理可能失败的操作
- **类型守卫**: 使用类型守卫进行运行时类型检查

```typescript
// 自定义错误类
class ValidationError extends Error {
  constructor(
    message: string,
    public readonly field: string
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

// Result 模式
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

function validateUser(userData: unknown): Result<User, ValidationError> {
  if (!isValidUserData(userData)) {
    return {
      success: false,
      error: new ValidationError("Invalid user data", "userData")
    };
  }

  return {
    success: true,
    data: new User(userData.id, userData.email, userData.name)
  };
}

// 类型守卫
function isValidUserData(data: unknown): data is CreateUserRequest {
  return (
    typeof data === "object" &&
    data !== null &&
    typeof (data as any).name === "string" &&
    typeof (data as any).email === "string"
  );
}
```

## 模块和导入导出

### 模块组织
- **命名导出优于默认导出**: 提高代码可搜索性
- **明确的导入**: 使用明确的导入语句
- **路径别名**: 使用路径别名简化导入

```typescript
// 推荐 - 命名导出
export class User {
  // 实现
}

export interface UserRepository {
  // 接口定义
}

export type CreateUserRequest = {
  // 类型定义
};

// 导入
import { User, UserRepository, CreateUserRequest } from "./user";

// 路径别名配置 (tsconfig.json)
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/types/*": ["src/types/*"],
      "@/services/*": ["src/services/*"],
      "@/models/*": ["src/models/*"]
    }
  }
}
```

### 类型导出
- **类型专用导出**: 使用 `type` 关键字导出类型
- **重新导出**: 合理使用重新导出组织 API

```typescript
// types/user.ts
export type User = {
  id: string;
  name: string;
  email: string;
};

export type CreateUserRequest = Omit<User, "id">;

// index.ts - 重新导出
export type { User, CreateUserRequest } from "./types/user";
export { UserService } from "./services/user-service";
export { UserRepository } from "./repositories/user-repository";
```

## 工具类型和实用类型

### 内置工具类型
- **合理使用工具类型**: 充分利用 TypeScript 内置工具类型
- **自定义工具类型**: 创建项目特定的工具类型

```typescript
// 内置工具类型使用
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

type PublicUser = Omit<User, "password">;
type CreateUserRequest = Pick<User, "name" | "email" | "password">;
type UpdateUserRequest = Partial<Pick<User, "name" | "email">>;

// 自定义工具类型
type NonEmptyArray<T> = [T, ...T[]];

type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// 条件类型
type ApiResponse<T> = T extends string
  ? { message: T }
  : { data: T };
```

## 枚举和常量

### 枚举使用
- **字符串枚举**: 优先使用字符串枚举
- **const 断言**: 对于简单常量使用 const 断言

```typescript
// 推荐 - 字符串枚举
enum UserStatus {
  Active = "active",
  Inactive = "inactive",
  Suspended = "suspended"
}

// 或使用 const 断言
const UserStatus = {
  Active: "active",
  Inactive: "inactive",
  Suspended: "suspended"
} as const;

type UserStatusType = typeof UserStatus[keyof typeof UserStatus];
```

## 性能和编译优化

### 编译优化
- **增量编译**: 使用增量编译提高构建速度
- **项目引用**: 对于大型项目使用项目引用
- **声明文件**: 为库项目生成声明文件

```json
// tsconfig.json - 性能配置
{
  "compilerOptions": {
    "incremental": true,
    "declaration": true,
    "declarationMap": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 测试

### TypeScript 测试最佳实践
- **类型测试**: 为复杂类型编写类型测试
- **测试类型安全**: 确保测试代码也有良好的类型安全

```typescript
// 类型测试
type AssertEqual<T, U> = T extends U ? (U extends T ? true : false) : false;

// 测试类型是否正确
const test1: AssertEqual<CreateUserRequest, Omit<User, "id" | "createdAt">> = true;

// 测试代码类型安全
describe("User", () => {
  it("should create user with valid data", () => {
    const userData: CreateUserRequest = {
      name: "Alice",
      email: "alice@example.com",
      password: "securePassword"
    };
    
    const user = new User("1", userData.email, userData.name);
    expect(user.getEmail()).toBe(userData.email);
  });
});
```

## 工具和环境

### 开发工具
- **Biome**: 推荐使用 Biome 作为现代化的 linter 和 formatter，替代 ESLint + Prettier 组合
- **TypeScript ESLint**: 如果项目仍使用 ESLint，使用 `@typescript-eslint` 进行代码检查
- **类型检查**: 使用 TypeScript 编译器进行类型检查

```json
// biome.json - 推荐的现代工具
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noExplicitAny": "error"
      },
      "style": {
        "useConst": "error",
        "noUnusedTemplateLiteral": "error"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2
  },
  "organizeImports": {
    "enabled": true
  }
}

// 或者传统的 ESLint 配置
// .eslintrc.json
{
  "extends": [
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```

---

*本规范应该与项目的具体需求相结合，在遵循社区标准的基础上保持适当的灵活性。*