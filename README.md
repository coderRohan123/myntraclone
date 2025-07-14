# 🛒 E-Commerce Platform

A comprehensive e-commerce platform built with **NestJS microservices** and **Next.js frontend**.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [NestJS Concepts](#nestjs-concepts)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Development Guide](#development-guide)

## 🏗️ Architecture Overview

### Backend Architecture (Microservices)
```
API Gateway (Port 3001) ←→ Redis ←→ Microservices
```

**Key Services:**
- **API Gateway**: Routes all requests to appropriate microservices
- **Auth Service**: Handles authentication, JWT tokens, user management
- **Catalog Service**: Products, categories, variants
- **Cart Service**: Shopping cart functionality
- **Order Service**: Order processing
- **Payment Service**: Payment processing
- **Profile Service**: User profiles and addresses
- **Review Service**: Product reviews
- **Wishlist Service**: User wishlists
- **Discount Service**: Coupons and discounts
- **Image Service**: Image upload/management
- **Notification Service**: Notifications

### Frontend Architecture (Next.js)
```
App Layout → AuthProvider → CartProvider → Header → Pages
```

**Key Components:**
- **AuthContext**: Manages user authentication state
- **CartContext**: Manages shopping cart state
- **Protected Routes**: Role-based access control
- **Admin Dashboard**: Admin-only features
- **User Interface**: Customer-facing features

## 🛠️ Tech Stack

### Backend
- **NestJS**: Progressive Node.js framework
- **TypeORM**: Database ORM
- **MySQL**: Primary database
- **Redis**: Message broker for microservices
- **JWT**: Authentication tokens
- **bcrypt**: Password hashing

### Frontend
- **Next.js 14**: React framework
- **TypeScript**: Type safety
- **Tailwind CSS**: Styling
- **React Context**: State management
- **JWT Decode**: Token handling

## 📁 Project Structure

```
myproject/
├── backend/
│   └── apps/
│       ├── api-gateway/     # Routes all requests
│       ├── auth/            # Authentication & users
│       ├── catalog/         # Products & categories
│       ├── cart/            # Shopping cart
│       ├── order/           # Order management
│       ├── payment/         # Payment processing
│       ├── profile/         # User profiles
│       ├── review/          # Product reviews
│       ├── wishlist/        # User wishlists
│       ├── discount/        # Coupons & discounts
│       ├── image/           # Image management
│       └── notification/    # Notifications
└── frontend/
    ├── app/                 # Next.js pages
    ├── components/          # Reusable components
    ├── context/             # State management
    ├── hooks/              # Custom hooks
    └── lib/                # API utilities
```

## 🎓 NestJS Concepts

### 1. Modules (@Module)
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])], // Import other modules
  controllers: [AuthController],                 // Route handlers
  providers: [AuthService],                     // Services/utilities
  exports: [AuthService],                       // Share with other modules
})
export class AuthModule {}
```

## 🎨 Next.js Concepts (Simple Guide)

### 1. What is Next.js?
Next.js is a React framework that makes building websites easier. It handles routing, server-side rendering, and optimization automatically.

### 2. File Structure (App Router)
```
app/
├── page.tsx              # Homepage (/)
├── login/page.tsx        # Login page (/login)
├── layout.tsx            # Wraps all pages
└── user/
    ├── cart/page.tsx     # Cart page (/user/cart)
    ├── profile/page.tsx  # Profile page (/user/profile)
    └── product/
        └── [id]/page.tsx # Product page (/user/product/123)
```

### 3. Pages (How they work)
```typescript
// app/page.tsx - Homepage
"use client"  // This means it runs in browser
import { useState, useEffect } from 'react';
import { getProductsWithFilters } from '../lib/api';

export default function HomePage() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    // Load products when page opens
    getProductsWithFilters({ page: 1, limit: 10 })
      .then(response => setProducts(response.data.items));
  }, []);

  return (
    <div>
      <h1>Welcome to our store</h1>
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </div>
  );
}
```

### 4. Dynamic Routes
```typescript
// app/user/product/[id]/page.tsx
export default function ProductPage({ params }) {
  return <div>Product ID: {params.id}</div>;
}
// This creates: /user/product/123, /user/product/456, etc.
```

### 5. Layout (Wraps all pages)
```typescript
// app/layout.tsx
import { AuthProvider } from '../context/AuthContext';
import Header from '../components/Header';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <AuthProvider>
          <Header />
          {children}  {/* This is where your pages go */}
        </AuthProvider>
      </body>
    </html>
  );
}
```

### 6. Components (Reusable pieces)
```typescript
// components/Header.tsx
"use client"
import { useAuth } from '../context/AuthContext';

export default function Header() {
  const { user, logout } = useAuth();

  return (
    <header>
      <h1>My Store</h1>
      {user ? (
        <div>
          <span>Hello, {user.email}</span>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <a href="/login">Login</a>
      )}
    </header>
  );
}
```

### 7. Context (Global state)
```typescript
// context/AuthContext.tsx
"use client"
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const login = (token) => {
    localStorage.setItem('token', token);
    setUser({ email: 'user@example.com' });
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  return context;
};
```

### 8. API Calls (lib/api.ts)
```typescript
// lib/api.ts - ALL API functions
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3001',
});

// Add token to every request
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// API functions
export const login = (data) => api.post('/auth/login', data);
export const getProducts = () => api.get('/catalog/products');
export const addToCart = (data) => api.post('/cart/add', data);
export const getCart = () => api.get('/cart');
// ... many more functions
```

### 9. Using API Functions
```typescript
// In any component
import { login, getProducts, addToCart } from '../lib/api';

// Login
const handleLogin = async () => {
  try {
    const response = await login({ email: 'user@example.com', password: 'password' });
    console.log('Success:', response.data);
  } catch (error) {
    console.error('Failed:', error);
  }
};

// Get products
const loadProducts = async () => {
  try {
    const response = await getProducts();
    setProducts(response.data);
  } catch (error) {
    console.error('Failed:', error);
  }
};

// Add to cart
const handleAddToCart = async (productId) => {
  try {
    await addToCart({ productId, quantity: 1 });
    console.log('Added to cart');
  } catch (error) {
    console.error('Failed:', error);
  }
};
```

### 10. Forms
```typescript
// components/LoginForm.tsx
"use client"
import { useState } from 'react';
import { login } from '../lib/api';
import { useAuth } from '../context/AuthContext';

export default function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { login: authLogin } = useAuth();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await login({ email, password });
      authLogin(response.data.accessToken);
    } catch (error) {
      setError('Login failed');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      {error && <div className="text-red-600">{error}</div>}
      <button type="submit">Login</button>
    </form>
  );
}
```

### 11. State Management
```typescript
// useState - Local state
const [products, setProducts] = useState([]);
const [loading, setLoading] = useState(false);

// useEffect - Run code when component mounts/changes
useEffect(() => {
  loadProducts();
}, []); // Empty array = run once when component mounts

// Custom hooks - Reusable logic
const useProducts = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    getProducts()
      .then(response => setProducts(response.data))
      .finally(() => setLoading(false));
  }, []);

  return { products, loading };
};
```

### 12. Styling (Tailwind CSS)
```typescript
// Simple styling with Tailwind
<div className="bg-white rounded-lg shadow p-4">
  <h1 className="text-2xl font-bold text-gray-900">Title</h1>
  <p className="text-gray-600">Description</p>
  <button className="bg-blue-600 text-white px-4 py-2 rounded">
    Click me
  </button>
</div>

// Responsive design
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* 1 column on mobile, 2 on tablet, 3 on desktop */}
</div>
```

### 13. Loading States
```typescript
// Simple loading
{loading ? (
  <div>Loading...</div>
) : (
  <div>{products.map(p => <div key={p.id}>{p.name}</div>)}</div>
)}

// Loading spinner
const LoadingSpinner = () => (
  <div className="flex items-center justify-center">
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
  </div>
);
```

### 14. Error Handling
```typescript
// Try-catch in async functions
const loadData = async () => {
  try {
    const response = await getData();
    setData(response.data);
  } catch (error) {
    console.error('Failed to load:', error);
    setError('Something went wrong');
  }
};

// Error state
const [error, setError] = useState('');
{error && <div className="text-red-600">{error}</div>}
```

### 15. Navigation
```typescript
// Link component (client-side navigation)
import Link from 'next/link';

<Link href="/user/cart">Go to Cart</Link>

// Programmatic navigation
import { useRouter } from 'next/navigation';

const router = useRouter();
router.push('/user/profile'); // Navigate to profile page
```

### 16. Environment Variables
```typescript
// .env.local file
NEXT_PUBLIC_API_URL=http://localhost:3001

// Use in code
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

### 17. Key Concepts Summary
- **"use client"** - Makes component run in browser (for state, effects, events)
- **useState** - Local state management
- **useEffect** - Run code when component mounts/changes
- **Context** - Global state (auth, cart, etc.)
- **lib/api.ts** - All API calls in one place
- **Components** - Reusable UI pieces
- **Pages** - Routes in app/ folder
- **Layout** - Wraps all pages
- **Tailwind** - Utility-first CSS framework

### 2. Controllers (@Controller)
```typescript
@Controller('auth')  // Route prefix
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')     // HTTP method + route
  async login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }
}
```

### 3. Services (@Injectable)
```typescript
@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async login(dto: LoginDto) {
    // Business logic here
  }
}
```

### 4. Microservices Communication
```typescript
// API Gateway sends messages to microservices
@Post('auth/login')
async login(@Body() loginData: any) {
  return firstValueFrom(
    this.authClient.send({ cmd: 'login' }, loginData)
  );
}

// Microservice receives messages
@MessagePattern({ cmd: 'login' })
async login(@Payload() dto: LoginUserDto) {
  return this.authService.login(dto);
}
```

### 5. Database Integration (TypeORM)
```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('increment', { type: 'bigint' })
  id: string;

  @Column({ type: 'varchar', length: 255, unique: true })
  email: string;

  @Column({ type: 'text', name: 'password_hash' })
  passwordHash: string;

  @Column({ type: 'json' })
  roles: string[];

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

### 6. Data Transfer Objects (DTOs)
```typescript
import { IsEmail, IsString, MinLength, Matches } from 'class-validator';

export class RegisterUserDto {
  @IsEmail({}, { message: 'Please provide a valid email address' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters long' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character'
  })
  password: string;
}
```

### 7. Authentication & Authorization
```typescript
// JWT Token Generation
const payload = { sub: user.id, email: user.email, roles: user.roles };
const accessToken = jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });

// Password Hashing
const passwordHash = await bcrypt.hash(dto.password, 12);

// Password Verification
const valid = await bcrypt.compare(dto.password, user.passwordHash);
```

### 8. Guards & Interceptors
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      throw new UnauthorizedException();
    }
    
    try {
      const payload = jwt.verify(token, JWT_SECRET);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }
}

// Using Guards
@Post('protected-route')
@UseGuards(AuthGuard)
async protectedMethod(@Req() req) {
  // req.user contains decoded JWT data
  return this.service.method(req.user);
}
```

### 9. Error Handling
```typescript
// RPC Exceptions
import { RpcException } from '@nestjs/microservices';

async register(dto: RegisterUserDto) {
  const existing = await this.userRepository.findOne({ where: { email: dto.email } });
  
  if (existing) {
    throw new RpcException('Email already in use');
  }
}

// Global Exception Filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const message = exception?.message || 'Internal server error';
    
    response.status(status).json({ statusCode: status, message });
  }
}
```

### 10. Dependency Injection
```typescript
@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    @InjectRepository(RefreshToken)
    private readonly refreshTokenRepository: Repository<RefreshToken>,
  ) {}
}
```

## 🚀 Getting Started

### Prerequisites
- Node.js (v18+)
- MySQL
- Redis
- npm or yarn

### Installation

1. **Clone the repository**
```bash
git clone <repository-url>
cd myproject
```

2. **Install backend dependencies**
```bash
cd backend
npm install
```

3. **Install frontend dependencies**
```bash
cd ../frontend
npm install
```

4. **Set up environment variables**
```bash
# Backend (.env)
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASS=your_password
JWT_SECRET=your_jwt_secret
REDIS_URL=redis://localhost:6379

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=http://localhost:3001
```

5. **Start the services**
```bash
# Start Redis
redis-server

# Start MySQL
mysql.server start

# Start backend (from backend directory)
npm run start:dev

# Start frontend (from frontend directory)
npm run dev
```

## 📚 API Documentation

### Authentication Endpoints
- `POST /auth/register` - Register new user
- `POST /auth/login` - User login
- `POST /auth/refresh-token` - Refresh access token
- `POST /auth/logout` - User logout

### Product Endpoints
- `GET /products` - List all products
- `GET /products/:id` - Get product details
- `POST /products` - Create new product (admin)
- `PUT /products/:id` - Update product (admin)
- `DELETE /products/:id` - Delete product (admin)

### Cart Endpoints
- `GET /cart` - Get user cart
- `POST /cart/add` - Add item to cart
- `PUT /cart/update/:itemId` - Update cart item
- `DELETE /cart/remove/:itemId` - Remove item from cart

### Order Endpoints
- `POST /orders` - Create new order
- `GET /orders` - List user orders
- `GET /orders/:id` - Get order details
- `PUT /orders/:id/status` - Update order status

## 🛠️ Development Guide

### Next.js Development Examples

#### 1. **Simple Component**
```typescript
// components/ProductCard.tsx
export default function ProductCard({ product, onAddToCart }) {
  return (
    <div className="bg-white rounded-lg shadow p-4">
      <img src={product.imageUrl} alt={product.name} className="w-full h-48 object-cover" />
      <h3 className="font-semibold">{product.name}</h3>
      <p className="text-lg font-bold text-pink-600">₹{product.price}</p>
      <button 
        onClick={() => onAddToCart(product.id)}
        className="w-full bg-pink-600 text-white py-2 rounded mt-2"
      >
        Add to Cart
      </button>
    </div>
  );
}
```

#### 2. **Simple Page**
```typescript
// app/user/product/[id]/page.tsx
"use client"
import { useState, useEffect } from 'react';
import { getProductDetails } from '../../../lib/api';

export default function ProductPage({ params }) {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    getProductDetails(params.id)
      .then(response => setProduct(response.data))
      .finally(() => setLoading(false));
  }, [params.id]);

  if (loading) return <div>Loading...</div>;
  if (!product) return <div>Product not found</div>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>₹{product.price}</p>
      <p>{product.description}</p>
    </div>
  );
}
```

#### 3. **Simple Form**
```typescript
// components/LoginForm.tsx
"use client"
import { useState } from 'react';
import { login } from '../lib/api';

export default function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await login({ email, password });
      console.log('Login successful:', response.data);
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

#### 4. **Simple API Call**
```typescript
// In any component
import { getProducts, addToCart } from '../lib/api';

// Get products
const loadProducts = async () => {
  try {
    const response = await getProducts();
    setProducts(response.data);
  } catch (error) {
    console.error('Failed:', error);
  }
};

// Add to cart
const handleAddToCart = async (productId) => {
  try {
    await addToCart({ productId, quantity: 1 });
    console.log('Added to cart');
  } catch (error) {
    console.error('Failed:', error);
  }
};
```

### Adding a New Feature

1. **Create Service in Backend**
```bash
cd backend/apps
nest generate service new-feature
```

2. **Create Entity**
```typescript
@Entity('new_feature')
export class NewFeature {
  @PrimaryGeneratedColumn('increment')
  id: string;

  @Column({ type: 'varchar', length: 255 })
  name: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

3. **Create DTOs**
```typescript
export class CreateNewFeatureDto {
  @IsString()
  @MinLength(1)
  name: string;
}
```

4. **Add Service Methods**
```typescript
@Injectable()
export class NewFeatureService {
  constructor(
    @InjectRepository(NewFeature)
    private newFeatureRepository: Repository<NewFeature>
  ) {}

  async create(dto: CreateNewFeatureDto): Promise<NewFeature> {
    const feature = this.newFeatureRepository.create(dto);
    return this.newFeatureRepository.save(feature);
  }
}
```

5. **Add Controller Endpoints**
```typescript
@Controller()
export class NewFeatureController {
  constructor(private newFeatureService: NewFeatureService) {}

  @MessagePattern({ cmd: 'create-new-feature' })
  async create(@Payload() dto: CreateNewFeatureDto) {
    return this.newFeatureService.create(dto);
  }
}
```

6. **Add to API Gateway**
```typescript
@Post('new-feature')
async createNewFeature(@Body() body: any) {
  return firstValueFrom(
    this.newFeatureClient.send({ cmd: 'create-new-feature' }, body)
  );
}
```

### Common Patterns

#### Service Pattern
```typescript
// 1. Define entity
@Entity('users')
export class User { /* ... */ }

// 2. Create DTOs
export class RegisterUserDto { /* ... */ }

// 3. Create service
@Injectable()
export class AuthService {
  async register(dto: RegisterUserDto) { /* ... */ }
}

// 4. Create controller
@Controller()
export class AuthController {
  @MessagePattern({ cmd: 'register' })
  async register(@Payload() dto: RegisterUserDto) {
    return this.authService.register(dto);
  }
}

// 5. Create module
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

#### API Gateway Pattern
```typescript
// API Gateway receives HTTP requests
@Post('auth/login')
async login(@Body() loginData: any) {
  return firstValueFrom(
    this.authClient.send({ cmd: 'login' }, loginData)
  );
}

// Microservice handles the command
@MessagePattern({ cmd: 'login' })
async login(@Payload() dto: LoginUserDto) {
  return this.authService.login(dto);
}
```

## 🔑 Key Concepts to Remember

### Lifecycle
1. **Module** imports dependencies
2. **Controller** receives requests
3. **Service** handles business logic
4. **Repository** manages database operations
5. **Entity** defines data structure

### Data Flow
```
HTTP Request → API Gateway → Redis → Microservice → Service → Repository → Database
```

### Error Handling
```
Exception → RpcException → Global Filter → HTTP Response
```

### Authentication Flow
```
Login Request → Validate Credentials → Generate JWT → Return Tokens
```

## 🎯 Best Practices

### Next.js Best Practices

#### 1. **Always use "use client" for interactive components**
```typescript
// Good - for components with state, events, effects
"use client"
import { useState } from 'react';

export default function InteractiveComponent() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// Bad - for static content
export default function StaticComponent() {
  return <div>Static content</div>; // No "use client" needed
}
```

#### 2. **Keep API calls in lib/api.ts**
```typescript
// Good - All API functions in one place
// lib/api.ts
export const login = (data) => api.post('/auth/login', data);
export const getProducts = () => api.get('/catalog/products');

// Bad - API calls scattered in components
const handleLogin = async () => {
  const response = await fetch('/auth/login', { ... }); // Don't do this
};
```

#### 3. **Use try-catch for error handling**
```typescript
// Good
const loadData = async () => {
  try {
    const response = await getData();
    setData(response.data);
  } catch (error) {
    console.error('Failed:', error);
    setError('Something went wrong');
  }
};

// Bad
const loadData = async () => {
  const response = await getData(); // No error handling
  setData(response.data);
};
```

#### 4. **Use loading states**
```typescript
// Good
const [loading, setLoading] = useState(false);

const handleSubmit = async () => {
  setLoading(true);
  try {
    await submitData();
  } finally {
    setLoading(false);
  }
};

return (
  <button disabled={loading}>
    {loading ? 'Submitting...' : 'Submit'}
  </button>
);
```

#### 5. **Use Tailwind classes for styling**
```typescript
// Good - Utility classes
<div className="bg-white rounded-lg shadow p-4">
  <h1 className="text-2xl font-bold text-gray-900">Title</h1>
</div>

// Bad - Inline styles
<div style={{ backgroundColor: 'white', borderRadius: '8px' }}>
  <h1 style={{ fontSize: '24px', fontWeight: 'bold' }}>Title</h1>
</div>
```

#### 6. **Use Link for navigation**
```typescript
// Good - Client-side navigation
import Link from 'next/link';

<Link href="/user/cart">Go to Cart</Link>

// Bad - Full page reload
<a href="/user/cart">Go to Cart</a>
```

#### 7. **Use environment variables**
```typescript
// Good - Environment variables
// .env.local
NEXT_PUBLIC_API_URL=http://localhost:3001

// In code
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Bad - Hardcoded values
const apiUrl = 'http://localhost:3001';
```

### Always Use DTOs for Validation
```typescript
@MessagePattern({ cmd: 'register' })
async register(@Payload() dto: RegisterUserDto) {
  // DTO automatically validates input
  return this.authService.register(dto);
}
```

### Use Repository Pattern
```typescript
// Good
const user = await this.userRepository.findOne({ where: { email } });

// Bad
const user = await this.userRepository.query('SELECT * FROM users WHERE email = ?', [email]);
```

### Handle Errors Properly
```typescript
try {
  const result = await this.service.method();
  return result;
} catch (err) {
  if (err instanceof RpcException) throw err;
  throw new RpcException('Operation failed: ' + err.message);
}
```

### Use Dependency Injection
```typescript
// Good
constructor(
  @InjectRepository(User)
  private userRepository: Repository<User>
) {}

// Bad
constructor() {
  this.userRepository = new Repository();
}
```

## 📊 Architecture Benefits

- **Scalability**: Each service can scale independently
- **Maintainability**: Clear separation of concerns
- **Security**: Centralized authentication and authorization
- **Flexibility**: Easy to add new features
- **Reliability**: Microservices can fail independently

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

---

**Happy Coding! 🚀** 
