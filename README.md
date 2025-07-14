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

## 🎨 Next.js Concepts

### 1. App Router (Next.js 13+)
```typescript
// app/page.tsx - Home page
"use client"
import { useState, useEffect } from 'react';

export default function HomePage() {
  const [products, setProducts] = useState([]);
  
  useEffect(() => {
    // Fetch data on component mount
    fetchProducts();
  }, []);

  return (
    <main className="min-h-screen">
      {/* Your JSX here */}
    </main>
  );
}
```

### 2. Client vs Server Components
```typescript
// Server Component (default) - No "use client"
export default function ServerComponent() {
  // Runs on server, no browser APIs
  return <div>Server rendered content</div>;
}

// Client Component - Has "use client"
"use client"
import { useState } from 'react';

export default function ClientComponent() {
  const [count, setCount] = useState(0);
  // Can use browser APIs, state, effects
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### 3. Routing with App Router
```typescript
// app/user/product/[id]/page.tsx - Dynamic route
export default function ProductPage({ params }: { params: { id: string } }) {
  return <div>Product ID: {params.id}</div>;
}

// app/user/category/[id]/page.tsx - Category page
export default function CategoryPage({ params }: { params: { id: string } }) {
  return <div>Category ID: {params.id}</div>;
}
```

### 4. Context API for State Management
```typescript
// context/AuthContext.tsx
"use client"
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  
  const login = (token) => {
    // Handle login logic
    setUser(decodedUser);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};
```

### 5. API Integration with Axios
```typescript
// lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// Add auth token to requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export const login = (data) => api.post('/auth/login', data);
export const getProducts = () => api.get('/catalog/products');
```

### 6. Custom Hooks
```typescript
// hooks/useProductDetails.ts
import { useState, useEffect } from 'react';
import { getProductDetails } from '../lib/api';

export const useProductDetails = (productId: string) => {
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchProduct = async () => {
      try {
        setLoading(true);
        const response = await getProductDetails(productId);
        setProduct(response.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    if (productId) {
      fetchProduct();
    }
  }, [productId]);

  return { product, loading, error };
};
```

### 7. Form Handling
```typescript
// components/LoginForm.tsx
"use client"
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';

export default function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { login } = useAuth();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const response = await apiLogin({ email, password });
      login(response.data.accessToken);
    } catch (err) {
      setError(err.response?.data?.message || 'Login failed');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        className="w-full p-2 border rounded"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        className="w-full p-2 border rounded"
        required
      />
      {error && <div className="text-red-600">{error}</div>}
      <button type="submit" className="w-full bg-blue-600 text-white py-2 rounded">
        Login
      </button>
    </form>
  );
}
```

### 8. Protected Routes
```typescript
// components/ProtectedRoute.tsx
"use client"
import { useAuth } from '../context/AuthContext';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';

export default function ProtectedRoute({ children, requiredRoles = [] }) {
  const { user, isLoading } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!isLoading && !user) {
      router.push('/login');
    }
    
    if (user && requiredRoles.length > 0) {
      const hasRequiredRole = requiredRoles.some(role => 
        user.roles.includes(role)
      );
      if (!hasRequiredRole) {
        router.push('/unauthorized');
      }
    }
  }, [user, isLoading, requiredRoles, router]);

  if (isLoading) return <div>Loading...</div>;
  if (!user) return null;

  return <>{children}</>;
}
```

### 9. Infinite Scroll
```typescript
// Infinite scroll implementation
useEffect(() => {
  const handleScroll = () => {
    if (
      window.innerHeight + document.documentElement.scrollTop >=
      document.documentElement.offsetHeight - 300
    ) {
      if (hasMore && !loadingProducts) {
        loadMoreProducts();
      }
    }
  };
  
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, [hasMore, loadingProducts]);
```

### 10. Responsive Design with Tailwind
```typescript
// Responsive component example
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {products.map(product => (
    <div key={product.id} className="bg-white rounded-lg shadow p-4">
      <img src={product.image} alt={product.name} className="w-full h-48 object-cover" />
      <h3 className="text-lg font-semibold mt-2">{product.name}</h3>
      <p className="text-gray-600">₹{product.price}</p>
    </div>
  ))}
</div>
```

### 11. Error Boundaries
```typescript
// Error handling in components
try {
  const response = await apiCall();
  setData(response.data);
} catch (error) {
  if (error.response?.status === 401) {
    // Handle unauthorized
    logout();
    router.push('/login');
  } else {
    setError(error.message);
  }
}
```

### 12. Image Optimization
```typescript
import Image from 'next/image';

// Optimized image component
<Image
  src={product.imageUrl}
  alt={product.name}
  width={300}
  height={200}
  className="object-cover"
  priority={true} // For above-the-fold images
/>
```

### 13. Loading States
```typescript
// Loading component
export default function LoadingSpinner() {
  return (
    <div className="flex items-center justify-center p-4">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
    </div>
  );
}

// Usage in components
{loading ? <LoadingSpinner /> : <ProductList products={products} />}
```

### 14. Environment Variables
```typescript
// .env.local
NEXT_PUBLIC_API_URL=http://localhost:3001

// Usage in code
const API_URL = process.env.NEXT_PUBLIC_API_URL;
```

### 15. SEO and Metadata
```typescript
// app/layout.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'E-Commerce Platform',
  description: 'A full-featured e-commerce platform.',
};

// In page components
export const metadata: Metadata = {
  title: 'Product Details',
  description: 'View product details and specifications',
};
```

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

### Next.js Development Patterns

#### 1. **Component Structure**
```typescript
// components/ProductCard.tsx
interface ProductCardProps {
  product: {
    id: string;
    name: string;
    price: number;
    imageUrl?: string;
  };
  onAddToCart?: (productId: string) => void;
}

export default function ProductCard({ product, onAddToCart }: ProductCardProps) {
  return (
    <div className="bg-white rounded-lg shadow hover:shadow-lg transition-shadow">
      <img src={product.imageUrl} alt={product.name} className="w-full h-48 object-cover" />
      <div className="p-4">
        <h3 className="font-semibold">{product.name}</h3>
        <p className="text-lg font-bold text-pink-600">₹{product.price}</p>
        <button 
          onClick={() => onAddToCart?.(product.id)}
          className="w-full bg-pink-600 text-white py-2 rounded mt-2"
        >
          Add to Cart
        </button>
      </div>
    </div>
  );
}
```

#### 2. **Page Structure**
```typescript
// app/user/product/[id]/page.tsx
"use client"
import { useProductDetails } from '../../../hooks/useProductDetails';
import { useAuth } from '../../../context/AuthContext';

export default function ProductPage({ params }: { params: { id: string } }) {
  const { product, loading, error } = useProductDetails(params.id);
  const { user } = useAuth();

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!product) return <div>Product not found</div>;

  return (
    <div className="container mx-auto px-4 py-8">
      <div className="grid md:grid-cols-2 gap-8">
        <div>
          <img src={product.imageUrl} alt={product.name} className="w-full" />
        </div>
        <div>
          <h1 className="text-3xl font-bold">{product.name}</h1>
          <p className="text-2xl font-bold text-pink-600">₹{product.price}</p>
          <p className="text-gray-600 mt-4">{product.description}</p>
          {/* Add to cart button */}
        </div>
      </div>
    </div>
  );
}
```

#### 3. **State Management Patterns**
```typescript
// Global state with Context
const CartContext = createContext();

export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState([]);
  const [loading, setLoading] = useState(false);

  const addToCart = async (productId, quantity = 1) => {
    setLoading(true);
    try {
      const response = await addToCartAPI({ productId, quantity });
      setCart(response.data);
    } catch (error) {
      console.error('Failed to add to cart:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <CartContext.Provider value={{ cart, addToCart, loading }}>
      {children}
    </CartContext.Provider>
  );
};
```

#### 4. **API Integration Patterns**
```typescript
// lib/api.ts - Centralized API calls
export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// Request interceptor for auth
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

#### 5. **Form Handling Patterns**
```typescript
// components/RegisterForm.tsx
"use client"
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';

export default function RegisterForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    confirmPassword: ''
  });
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};
    if (!formData.email) newErrors.email = 'Email is required';
    if (!formData.password) newErrors.password = 'Password is required';
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!validateForm()) return;

    setLoading(true);
    try {
      const response = await register(formData);
      login(response.data.accessToken);
    } catch (error) {
      setErrors({ submit: error.response?.data?.message || 'Registration failed' });
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className={`w-full p-2 border rounded ${errors.email ? 'border-red-500' : ''}`}
          placeholder="Email"
        />
        {errors.email && <p className="text-red-500 text-sm">{errors.email}</p>}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          className={`w-full p-2 border rounded ${errors.password ? 'border-red-500' : ''}`}
          placeholder="Password"
        />
        {errors.password && <p className="text-red-500 text-sm">{errors.password}</p>}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
          className={`w-full p-2 border rounded ${errors.confirmPassword ? 'border-red-500' : ''}`}
          placeholder="Confirm Password"
        />
        {errors.confirmPassword && <p className="text-red-500 text-sm">{errors.confirmPassword}</p>}
      </div>

      {errors.submit && <p className="text-red-500 text-sm">{errors.submit}</p>}

      <button 
        type="submit" 
        disabled={loading}
        className="w-full bg-blue-600 text-white py-2 rounded disabled:opacity-50"
      >
        {loading ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
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

#### 1. **Component Organization**
```typescript
// Good: Separate concerns
components/
├── ui/           # Reusable UI components
│   ├── Button.tsx
│   ├── Input.tsx
│   └── Modal.tsx
├── forms/        # Form components
│   ├── LoginForm.tsx
│   └── RegisterForm.tsx
├── layout/       # Layout components
│   ├── Header.tsx
│   └── Footer.tsx
└── features/     # Feature-specific components
    ├── ProductCard.tsx
    └── CartItem.tsx
```

#### 2. **Custom Hooks for Logic**
```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}
```

#### 3. **Error Boundaries**
```typescript
// components/ErrorBoundary.tsx
"use client"
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="text-center py-8">
          <h2 className="text-xl font-bold text-red-600">Something went wrong</h2>
          <button 
            onClick={() => this.setState({ hasError: false })}
            className="mt-4 px-4 py-2 bg-blue-600 text-white rounded"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

#### 4. **Loading States**
```typescript
// components/LoadingSpinner.tsx
export default function LoadingSpinner({ size = 'md' }: { size?: 'sm' | 'md' | 'lg' }) {
  const sizeClasses = {
    sm: 'h-4 w-4',
    md: 'h-8 w-8',
    lg: 'h-12 w-12'
  };

  return (
    <div className="flex items-center justify-center">
      <div className={`animate-spin rounded-full border-b-2 border-blue-600 ${sizeClasses[size]}`}></div>
    </div>
  );
}

// Usage
{loading ? <LoadingSpinner /> : <ProductList products={products} />}
```

#### 5. **Optimistic Updates**
```typescript
// Optimistic cart update
const addToCartOptimistic = async (productId: string) => {
  // Optimistically update UI
  setCart(prev => [...prev, { id: Date.now(), productId, quantity: 1 }]);
  
  try {
    // Make API call
    const response = await addToCartAPI({ productId, quantity: 1 });
    // Update with real data
    setCart(response.data);
  } catch (error) {
    // Revert on error
    setCart(prev => prev.filter(item => item.productId !== productId));
    console.error('Failed to add to cart:', error);
  }
};
```

#### 6. **Debounced Search**
```typescript
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage in search
const [searchTerm, setSearchTerm] = useState('');
const debouncedSearchTerm = useDebounce(searchTerm, 500);

useEffect(() => {
  if (debouncedSearchTerm) {
    searchProducts(debouncedSearchTerm);
  }
}, [debouncedSearchTerm]);
```

#### 7. **Responsive Design Patterns**
```typescript
// Responsive grid
<div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
  {products.map(product => (
    <ProductCard key={product.id} product={product} />
  ))}
</div>

// Responsive navigation
<nav className="hidden md:flex items-center space-x-4">
  {/* Desktop navigation */}
</nav>
<button className="md:hidden" onClick={() => setMobileMenuOpen(true)}>
  {/* Mobile menu button */}
</button>
```

#### 8. **Performance Optimization**
```typescript
// Memoized components
import { memo } from 'react';

const ProductCard = memo(({ product, onAddToCart }) => {
  return (
    <div className="product-card">
      {/* Component content */}
    </div>
  );
});

// Lazy loading
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <LoadingSpinner />,
  ssr: false // Disable server-side rendering for this component
});
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
