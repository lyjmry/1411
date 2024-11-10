# Mini Apps API 参考文档

## 核心API

### WorldApp
主要的应用程序类，用于初始化和管理Mini App。

```javascript
class WorldApp {
  constructor(options: {
    appId: string;
    debug?: boolean;
    version?: string;
  });

  // 生命周期方法
  onReady(callback: () => void): void;
  onError(callback: (error: Error) => void): void;
  onPause(callback: () => void): void;
  onResume(callback: () => void): void;

  // 状态管理
  getState(): AppState;
  setState(state: Partial<AppState>): void;
}
```

### WorldAuth
处理用户认证和授权的API。

```javascript
class WorldAuth {
  constructor(options?: AuthOptions);

  // 认证方法
  async login(options?: LoginOptions): Promise<User>;
  async logout(): Promise<void>;
  async verifyIdentity(): Promise<VerificationResult>;

  // 用户信息
  async getCurrentUser(): Promise<User>;
  async updateProfile(data: ProfileData): Promise<User>;

  // 权限管理
  async requestPermission(permission: string): Promise<boolean>;
  async checkPermission(permission: string): Promise<boolean>;
}

interface LoginOptions {
  scope?: string[];
  prompt?: 'none' | 'consent' | 'select_account';
}

interface User {
  id: string;
  username: string;
  email?: string;
  profile?: UserProfile;
}
```

### WorldPayment
支付相关的API接口。

```javascript
class WorldPayment {
  constructor(options?: PaymentOptions);

  // 支付方法
  async createPayment(params: PaymentParams): Promise<Payment>;
  async processPayment(paymentId: string): Promise<PaymentResult>;
  async cancelPayment(paymentId: string): Promise<void>;

  // 退款处理
  async createRefund(params: RefundParams): Promise<Refund>;
  async getRefundStatus(refundId: string): Promise<RefundStatus>;
}

interface PaymentParams {
  amount: number;
  currency: string;
  description?: string;
  metadata?: Record<string, any>;
}

interface Payment {
  id: string;
  status: PaymentStatus;
  amount: number;
  currency: string;
  created: number;
}
```

### WorldStorage
数据存储API。

```javascript
class WorldStorage {
  constructor(options?: StorageOptions);

  // 基础存储方法
  async get(key: string): Promise<any>;
  async set(key: string, value: any): Promise<void>;
  async remove(key: string): Promise<void>;
  async clear(): Promise<void>;

  // 高级存储方法
  async getMany(keys: string[]): Promise<Record<string, any>>;
  async setMany(items: Record<string, any>): Promise<void>;
}

interface StorageOptions {
  namespace?: string;
  encryption?: boolean;
  persistence?: 'local' | 'session' | 'memory';
}
```

## UI组件

### WorldUI
提供标准UI组件的集合。

```javascript
namespace WorldUI {
  // 基础组件
  export class Button extends Component<ButtonProps> {}
  export class Input extends Component<InputProps> {}
  export class Card extends Component<CardProps> {}
  
  // 导航组件
  export class Navigation extends Component<NavigationProps> {}
  export class TabBar extends Component<TabBarProps> {}
  export class Menu extends Component<MenuProps> {}
  
  // 表单组件
  export class Form extends Component<FormProps> {}
  export class Select extends Component<SelectProps> {}
  export class Checkbox extends Component<CheckboxProps> {}
  
  // 反馈组件
  export class Toast extends Component<ToastProps> {}
  export class Modal extends Component<ModalProps> {}
  export class Alert extends Component<AlertProps> {}
}

interface ButtonProps {
  type?: 'primary' | 'secondary' | 'text';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
}
```

## 网络请求

### WorldHttp
HTTP请求客户端。

```javascript
class WorldHttp {
  constructor(options?: HttpOptions);

  // 请求方法
  async get<T>(url: string, config?: RequestConfig): Promise<T>;
  async post<T>(url: string, data?: any, config?: RequestConfig): Promise<T>;
  async put<T>(url: string, data?: any, config?: RequestConfig): Promise<T>;
  async delete<T>(url: string, config?: RequestConfig): Promise<T>;

  // 请求拦截器
  interceptors: {
    request: Interceptor;
    response: Interceptor;
  };
}

interface HttpOptions {
  baseURL?: string;
  timeout?: number;
  headers?: Record<string, string>;
}
```

## 事件系统

### WorldEvents
事件管理系统。

```javascript
class WorldEvents {
  // 事件监听
  on(event: string, callback: EventCallback): void;
  once(event: string, callback: EventCallback): void;
  off(event: string, callback?: EventCallback): void;

  // 事件触发
  emit(event: string, data?: any): void;
  
  // 事件管理
  listenerCount(event: string): number;
  removeAllListeners(event?: string): void;
}

type EventCallback = (data: any) => void;
```

## 工具类

### WorldUtils
通用工具函数集合。

```javascript
namespace WorldUtils {
  // 字符串处理
  export function formatString(template: string, params: object): string;
  export function truncate(str: string, length: number): string;

  // 数字处理
  export function formatNumber(num: number, options?: NumberFormatOptions): string;
  export function formatCurrency(amount: number, currency?: string): string;

  // 日期处理
  export function formatDate(date: Date | number, format?: string): string;
  export function parseDate(dateString: string): Date;

  // 验证工具
  export function validateEmail(email: string): boolean;
  export function validatePhone(phone: string): boolean;
}
```

## 类型定义

### 基础类型
```typescript
// 用户相关
interface User {
  id: string;
  username: string;
  email?: string;
  profile?: UserProfile;
  createdAt: number;
}

interface UserProfile {
  displayName?: string;
  avatar?: string;
  bio?: string;
}

// 支付相关
type PaymentStatus = 'pending' | 'processing' | 'completed' | 'failed' | 'cancelled';

interface PaymentResult {
  success: boolean;
  transactionId?: string;
  error?: PaymentError;
}

// 存储相关
interface StorageItem {
  key: string;
  value: any;
  timestamp: number;
  expiry?: number;
}
```

### 错误类型
```typescript
class WorldError extends Error {
  constructor(
    message: string,
    code: string,
    details?: Record<string, any>
  );

  readonly code: string;
  readonly details?: Record<string, any>;
}

interface PaymentError {
  code: string;
  message: string;
  transaction?: string;
}
```

## 常量定义

```javascript
// 事件类型
export const EventTypes = {
  APP_READY: 'app:ready',
  APP_ERROR: 'app:error',
  AUTH_LOGIN: 'auth:login',
  AUTH_LOGOUT: 'auth:logout',
  PAYMENT_SUCCESS: 'payment:success',
  PAYMENT_FAILURE: 'payment:failure'
};

// 错误代码
export const ErrorCodes = {
  NETWORK_ERROR: 'NETWORK_ERROR',
  AUTH_FAILED: 'AUTH_FAILED',
  PAYMENT_FAILED: 'PAYMENT_FAILED',
  INVALID_PARAM: 'INVALID_PARAM',
  PERMISSION_DENIED: 'PERMISSION_DENIED'
};

// 权限类型
export const Permissions = {
  CAMERA: 'camera',
  LOCATION: 'location',
  STORAGE: 'storage',
  NOTIFICATIONS: 'notifications'
};
```

## 配置选项

```typescript
interface AppConfig {
  // 基础配置
  appId: string;
  version: string;
  debug?: boolean;

  // 功能配置
  features?: {
    auth?: boolean;
    payment?: boolean;
    storage?: boolean;
  };

  // 网络配置
  api?: {
    baseURL?: string;
    timeout?: number;
    headers?: Record<string, string>;
  };

  // 存储配置
  storage?: {
    namespace?: string;
    encryption?: boolean;
    persistence?: 'local' | 'session' | 'memory';
  };

  // UI配置
  ui?: {
    theme?: 'light' | 'dark' | 'auto';
    primaryColor?: string;
    fontFamily?: string;
  };
}
```

## 使用示例

### 基础应用初始化
```javascript
import { WorldApp } from '@world/minikit';

const app = new WorldApp({
  appId: 'your-app-id',
  debug: true,
  version: '1.0.0'
});

app.onReady(() => {
  console.log('应用已准备就绪');
});

app.onError((error) => {
  console.error('应用错误:', error);
});
```

### 用户认证流程
```javascript
import { WorldAuth } from '@world/minikit';

const auth = new WorldAuth();

async function handleLogin() {
  try {
    const user = await auth.login({
      scope: ['profile', 'payment']
    });
    console.log('登录成功:', user);
  } catch (error) {
    console.error('登录失败:', error);
  }
}

async function checkCurrentUser() {
  const user = await auth.getCurrentUser();
  if (user) {
    console.log('当前用户:', user);
  }
}
```

### 支付处理
```javascript
import { WorldPayment } from '@world/minikit';

const payment = new WorldPayment();

async function processPayment(amount: number) {
  try {
    const paymentResult = await payment.createPayment({
      amount,
      currency: 'USDC',
      description: '商品购买'
    });
    
    console.log('支付创建成功:', paymentResult);
    
    const result = await payment.processPayment(paymentResult.id);
    console.log('支付处理结果:', result);
  } catch (error) {
    console.error('支付失败:', error);
  }
}
```

### 数据存储
```javascript
import { WorldStorage } from '@world/minikit';

const storage = new WorldStorage({
  namespace: 'myapp',
  encryption: true
});

async function handleData() {
  // 存储数据
  await storage.set('user-preferences', {
    theme: 'dark',
    notifications: true
  });
  
  // 获取数据
  const preferences = await storage.get('user-preferences');
  console.log('用户偏好:', preferences);
}
```

## 注意事项

1. 错误处理
   - 始终使用try-catch处理异步操作
   - 适当处理和显示错误信息
   - 实现错误重试机制

2. 性能优化
   - 合理使用缓存
   - 避免频繁的API调用
   - 实现请求防抖和节流

3. 安全考虑
   - 保护敏感数据
   - 实施适当的加密
   - 验证所有输入数据

4. 最佳实践
   - 遵循API使用规范
   - 实现适当的错误处理
   - 保持代码简洁可维护
