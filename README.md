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
