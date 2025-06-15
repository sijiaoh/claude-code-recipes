# MVC 架构最佳实践

本文件定义了 MVC (Model-View-Controller) 架构模式的最佳实践和设计原则。

**重要**: 使用 MVC 架构时，必须同时遵守：
- `base.md` - 通用编程最佳实践
- `oop.md` - 面向对象编程原则（Model 层特别重要）
- 本文件中的 MVC 架构规范

## MVC 核心原则

### 架构纯粹性
- **严格三层**: 只使用 Model、View、Controller 三层，避免添加额外层级
- **明确职责**: 每层只负责自己的职责，不得跨界
- **单向依赖**: Controller → Model，Controller → View，避免反向依赖

### 数据访问处理
- **框架集成**: 根据使用的 MVC 框架（Rails、Django、Express 等）采用相应的数据访问模式
- **Active Record 模式**: 在 Active Record 框架中，Model 直接包含数据访问方法
- **Repository 模式**: 在其他架构中，Model 可以委托给 Data Access 层
- **保持概念完整性**: 无论采用何种模式，Model 都应保持业务概念的完整性

## Model 层设计

### 严格遵循 OOP 原则
- **概念完整性**: 每个 Model 代表一个完整的业务概念
- **富领域模型**: Model 包含数据和相关的业务逻辑
- **遵循 oop.md**: 严格按照 `oop.md` 中的面向对象设计原则

```javascript
// 好的 Model 设计 - 完整的用户概念
class User {
    constructor(attributes = {}) {
        this.name = attributes.name
        this.email = attributes.email
        this.age = attributes.age
        this.createdAt = attributes.createdAt || new Date()
        this.id = attributes.id
    }
    
    static create(userData) {
        // 用户创建 - 用户概念的核心行为
        const user = new User(userData)
        
        if (!user.isValid()) {
            throw new Error("用户数据无效")
        }
        
        // 保存用户 - 具体实现取决于框架
        user.save()
        return user
    }
    
    changeEmail(newEmail) {
        // 用户修改邮箱 - 用户概念的内在行为
        if (!this.isValidEmail(newEmail)) {
            throw new Error("无效的邮箱格式")
        }
        this.email = newEmail
        this.save() // 保存变更
    }
    
    save() {
        // 用户保存 - 实现方式取决于框架
        // Active Record: 直接调用框架的 save 方法
        // Repository: 委托给数据访问层
        // 具体实现由框架决定
        return this.persist()
    }
    
    isAdult() {
        // 用户年龄判断 - 用户概念的属性计算
        return this.age >= 18
    }
    
    isValid() {
        // 用户数据验证 - 用户概念的内在职责
        return this.name && this.isValidEmail(this.email) && this.age >= 0
    }
    
    isValidEmail(email) {
        // 邮箱格式验证
        return email && email.includes('@') && email.includes('.')
    }
    
    // 框架特定的持久化方法
    persist() {
        // 在 Active Record 框架中：直接操作数据库
        // 在 Repository 模式中：委托给数据访问层
        // 具体实现留给框架
    }
}
```

### 跨 Model 操作的处理

#### 谨慎考虑归属
当需要跨多个 Model 的操作时，仔细思考：
1. 这个操作属于哪个 Model 的概念？
2. 是否需要创建新的概念（新的 Model）？
3. 是否需要在 Controller 中协调？

#### 创建新概念的时机
- **复杂业务逻辑**: 跨多个 Model 的复杂业务逻辑
- **独立的业务概念**: 操作本身代表一个业务概念
- **重复出现**: 相同的跨 Model 操作在多处出现

```javascript
// 跨 Model 操作示例：订单处理
class Order {
    constructor(user, items) {
        this.user = user
        this.items = items
        this.status = OrderStatus.PENDING
        this.totalAmount = this.calculateTotal()
    }
    
    calculateTotal() {
        // 订单总额计算 - 订单概念的内在职责
        return this.items.reduce((sum, item) => sum + item.subtotal(), 0)
    }
    
    applyDiscount(discount) {
        // 应用折扣 - 订单概念的行为
        if (discount.isApplicableTo(this)) {
            this.totalAmount = discount.applyTo(this.totalAmount)
        }
    }
    
    processPayment(paymentMethod, paymentGateway) {
        // 支付处理 - 订单概念的行为，委托给支付网关
        this.validatePaymentEligibility(paymentMethod)
        
        const result = paymentGateway.processPayment(this.totalAmount, paymentMethod)
        
        if (result.isSuccessful()) {
            this.markAsPaid()
        }
        
        return result
    }
    
    validatePaymentEligibility(paymentMethod) {
        if (!this.user.canMakePayment()) {
            throw new PaymentError("用户无法进行支付")
        }
        
        if (!paymentMethod.isValid()) {
            throw new PaymentError("支付方式无效")
        }
    }
}

// 支付网关：真正的独立概念，委托技术实现
class PaymentGateway {
    constructor(apiKey, endpoint) {
        this.apiKey = apiKey
        this.endpoint = endpoint
    }
    
    processPayment(amount, paymentMethod) {
        // 支付网关处理 - 独立的技术概念
        // 实际的支付API调用逻辑
        return this.executePayment(amount, paymentMethod)
    }
}
```

## View 层设计

### View 层职责
- **数据展示**: 负责数据的显示和用户界面
- **用户交互**: 处理用户输入和界面事件
- **格式化**: 将 Model 数据格式化为用户可读形式
- **无业务逻辑**: 不包含任何业务逻辑

### View 层原则
- **被动视图**: View 不主动获取数据，由 Controller 提供
- **模板化**: 使用模板引擎分离显示逻辑
- **响应式**: 根据数据变化更新显示

```javascript
// View 层示例
class UserView {
    renderProfile(user) {
        // 渲染用户资料页面
        return renderTemplate('user_profile.html', {
            name: user.name,
            email: user.email,
            isAdult: user.isAdult(),
            memberSince: user.createdAt.toLocaleDateString()
        })
    }
    
    renderError(errorMessage) {
        // 渲染错误页面
        return renderTemplate('error.html', {
            message: errorMessage
        })
    }
}
```

## Controller 层设计

### Controller 职责
- **协调者**: 协调 Model 和 View 之间的交互
- **流程控制**: 控制应用程序的执行流程
- **输入处理**: 处理用户输入和请求
- **调用业务逻辑**: 调用 Model 的业务方法

### Controller 原则
- **薄控制器**: Controller 应该尽可能薄，主要负责协调
- **不包含业务逻辑**: 业务逻辑应该在 Model 中
- **异常处理**: 处理和转换异常为用户友好的错误

```javascript
class UserController {
    constructor(userView) {
        this.userView = userView
    }
    
    create(userData) {
        // 用户创建
        try {
            // 调用 Model 的业务方法
            const user = User.create(userData)
            return this.userView.renderProfile(user)
        
        } catch (validationError) {
            return this.userView.renderError(validationError.message)
        } catch (error) {
            return this.userView.renderError("创建失败")
        }
    }
    
    show(userId) {
        // 显示用户资料
        try {
            // 通过 Model 查找用户 - 具体实现取决于框架
            const user = User.findById(userId)
            if (!user) {
                return this.userView.renderError("用户不存在")
            }
            
            return this.userView.renderProfile(user)
        
        } catch (error) {
            return this.userView.renderError("系统错误")
        }
    }
    
    updateEmail(userId, newEmail) {
        // 更新用户邮箱
        try {
            const user = User.findById(userId)
            if (!user) {
                return this.userView.renderError("用户不存在")
            }
            
            // 业务逻辑在 Model 中
            user.changeEmail(newEmail)
            
            return this.userView.renderProfile(user)
        
        } catch (validationError) {
            return this.userView.renderError(validationError.message)
        } catch (error) {
            return this.userView.renderError("系统错误")
        }
    }
}
```

## 数据持久化模式

### 不同框架的数据访问方式

#### Active Record 模式（Rails、Django）
- **Model 直接包含数据访问**: Model 类直接提供 save、find、delete 等方法
- **框架内置 ORM**: 框架提供数据库抽象层
- **约定优于配置**: 遵循框架的命名和结构约定

```javascript
// Active Record 风格示例
class User extends ActiveRecord {
    // 框架提供的方法: save(), find(), where(), etc.
    
    static create(attributes) {
        const user = new User(attributes)
        if (user.isValid()) {
            user.save() // 框架提供的持久化方法
        }
        return user
    }
    
    changeEmail(newEmail) {
        this.email = newEmail
        this.save() // 直接调用框架方法
    }
}
```

#### Repository 模式（自定义架构）
- **分离关注点**: Model 专注业务逻辑，Repository 负责数据访问
- **接口抽象**: 定义抽象的数据访问接口
- **依赖注入**: Model 通过依赖注入获取数据访问能力

```javascript
// Repository 风格示例
class User {
    constructor(attributes, repository = null) {
        this.name = attributes.name
        this.email = attributes.email
        this._repository = repository
    }
    
    save() {
        return this._repository.save(this)
    }
    
    static findById(id, repository) {
        return repository.findById(id)
    }
}

class UserRepository {
    save(user) {
        // 数据库操作
    }
    
    findById(id) {
        // 数据库查询
    }
}
```

## 层间通信规则

### 依赖方向
```
Controller → Model
Controller → View

# Active Record 模式
Model → Framework ORM

# Repository 模式  
Model → Repository (通过委托)
Repository → Model (数据映射)
```

### 禁止的依赖
```
Model → Controller  ❌
Model → View        ❌
View → Model        ❌
View → Controller   ❌
```

### 数据流向

#### Active Record 模式流向
1. **用户请求** → Controller
2. **Controller** → Model 调用业务方法
3. **Model** → Framework ORM 进行数据操作
4. **Controller** → View 渲染结果
5. **View** → 用户响应

#### Repository 模式流向
1. **用户请求** → Controller
2. **Controller** → Model 调用业务方法
3. **Model** → Repository 获取/保存数据（委托）
4. **Repository** → Model 转换数据
5. **Controller** → View 渲染结果
6. **View** → 用户响应

## 常见问题和解决方案

### 跨 Model 操作的归属问题

#### 问题：订单状态变更需要通知用户和更新库存
```javascript
// ❌ 错误：在 Controller 中写业务逻辑
class OrderController {
    confirmOrder(orderId) {
        const order = this.orderDataAccess.findById(orderId)
        order.status = OrderStatus.CONFIRMED
        
        // 业务逻辑泄露到 Controller
        for (const item of order.items) {
            const inventory = this.inventoryDataAccess.findByProductId(item.productId)
            inventory.reduceStock(item.quantity)
            this.inventoryDataAccess.save(inventory)
        }
        
        const user = this.userDataAccess.findById(order.userId)
        user.sendOrderConfirmation(order)
    }
}

// ✅ 更好：逻辑归位到订单概念，使用委托模式
class Order {
    confirm(inventoryManager, notificationHandler) {
        // 订单确认 - 订单概念的核心行为
        this.validateCanConfirm()
        this.status = OrderStatus.CONFIRMED
        
        // 委托给相关概念和工具
        this.updateInventory(inventoryManager)
        this.notifyStakeholders(notificationHandler)
    }
    
    updateInventory(inventoryManager) {
        // 委托库存更新
        for (const item of this.items) {
            inventoryManager.reduceStock(item.productId, item.quantity)
        }
    }
    
    notifyStakeholders(notificationHandler) {
        // 委托通知处理
        notificationHandler.sendOrderConfirmation(this.userId, this)
    }
}

// 🔄 仅当确实需要独立概念时才创建专门类
// 注意：避免仅为单个方法创建类
class InventoryManager {
    // 库存管理 - 如果库存管理确实是独立概念
    constructor(inventoryDataAccess) {
        this.inventoryDataAccess = inventoryDataAccess
    }
    
    reduceStock(productId, quantity) {
        const inventory = this.inventoryDataAccess.findByProductId(productId)
        inventory.reduceStock(quantity)  // 业务逻辑在 Inventory 模型中
        this.inventoryDataAccess.save(inventory)
    }
}
```

### Model 过度分割问题

#### 问题：将用户信息分割成多个小 Model
```javascript
// ❌ 错误：过度分割破坏概念完整性
class UserBasicInfo {
    constructor(name, email) {
        this.name = name
        this.email = email
    }
}

class UserProfile {
    constructor(age, phone) {
        this.age = age
        this.phone = phone
    }
}

// ✅ 正确：保持用户概念的完整性
class User {
    constructor(name, email, age, phone) {
        this.name = name
        this.email = email
        this.age = age
        this.phone = phone
    }
    
    updateContactInfo(email, phone) {
        // 用户联系信息更新 - 用户概念的完整行为
        this.email = email
        this.phone = phone
    }
}
```

## 总结

MVC 架构的成功关键在于：

1. **严格的层次边界**: 保持 Model、View、Controller 三层的清晰职责
2. **Model 层的正确设计**: 遵循 `oop.md` 原则，保持概念完整性
3. **合理的跨 Model 操作**: 谨慎考虑归属，必要时创建新概念
4. **薄 Controller**: Controller 只负责协调，不包含业务逻辑
5. **框架适应性**: 根据使用的 MVC 框架采用相应的数据访问模式

**核心原则**: MVC 不仅是技术架构，更是业务概念的清晰表达。每个 Model 都应该是一个完整、独立的业务概念，Controller 负责这些概念之间的协调，View 负责概念的展示。

**框架兼容性**: 无论使用 Active Record 模式（Rails、Django）还是 Repository 模式，都应保持业务逻辑在 Model 中的原则，数据访问的具体实现可以根据框架特点进行调整。

---

*在遵循 MVC 架构原则的基础上，根据具体的技术栈和业务需求进行适当调整。*