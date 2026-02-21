# NestJS Interview Questions & Answers

## 1. What is NestJS?

**Answer:** NestJS is a progressive Node.js framework for building efficient, scalable, and enterprise-grade server-side applications. It uses TypeScript by default and combines elements of Object-Oriented Programming (OOP), Functional Programming (FP), and Functional Reactive Programming (FRP).

**Key Features:**
- Built with TypeScript
- Modular architecture
- Dependency Injection
- Decorators-based syntax
- Built-in support for microservices
- Compatible with Express and Fastify
- Extensive documentation

---

## 2. What are the core building blocks of a NestJS application?

**Answer:**

1. **Modules:** Organize application structure
2. **Controllers:** Handle incoming requests and return responses
3. **Providers/Services:** Business logic and data access
4. **Middleware:** Functions executed before route handlers
5. **Guards:** Authorization and authentication logic
6. **Interceptors:** Transform request/response data
7. **Pipes:** Data transformation and validation
8. **Filters:** Exception handling

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

## 3. What is Dependency Injection in NestJS?

**Answer:** Dependency Injection (DI) is a design pattern where dependencies are injected into classes rather than created within them. NestJS has a built-in IoC (Inversion of Control) container that manages dependencies.

**Benefits:**
- Loose coupling
- Better testability
- Code reusability
- Easier maintenance

Example:
```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}
}

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}
}
```

---

## 4. What are Providers in NestJS?

**Answer:** Providers are classes that can be injected as dependencies. They are decorated with `@Injectable()` and can be services, repositories, factories, helpers, etc.

**Types of Providers:**
- Class providers
- Value providers
- Factory providers
- Async providers

Example:
```typescript
@Injectable()
export class UsersService {
  private readonly users: User[] = [];

  findAll(): User[] {
    return this.users;
  }

  create(user: User): User {
    this.users.push(user);
    return user;
  }
}
```

---

## 5. What is the difference between @Module and @Global decorator?

**Answer:**

| @Module | @Global |
|---------|---------|
| Encapsulates providers | Makes module globally available |
| Providers scoped to module | No need to import in other modules |
| Must be imported explicitly | Imported once, available everywhere |
| Default behavior | Used for shared modules |

Example:
```typescript
// Regular module
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}

// Global module
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

---

## 6. What are Controllers in NestJS?

**Answer:** Controllers handle incoming HTTP requests and return responses to the client. They are responsible for routing and request handling.

Example:
```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')
  update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<User> {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string): Promise<void> {
    return this.usersService.remove(id);
  }
}
```

---

## 7. What are Middleware in NestJS?

**Answer:** Middleware are functions that have access to request, response objects, and the next middleware function. They execute before route handlers.

**Use cases:**
- Logging
- Authentication
- Request validation
- CORS handling

Example:
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Request: ${req.method} ${req.url}`);
    next();
  }
}

// Apply middleware
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('users');
  }
}
```

---

## 8. What are Guards in NestJS?

**Answer:** Guards determine whether a request should be handled by the route handler. They implement the `CanActivate` interface and are used for authorization.

Example:
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return this.validateRequest(request);
  }

  private validateRequest(request: any): boolean {
    return !!request.headers.authorization;
  }
}

// Usage
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {}
```

---

## 9. What are Interceptors in NestJS?

**Answer:** Interceptors intercept incoming requests and outgoing responses. They can transform data, add extra logic, or handle errors.

**Use cases:**
- Response transformation
- Caching
- Logging
- Timeout handling
- Exception mapping

Example:
```typescript
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Usage
@UseInterceptors(TransformInterceptor)
@Controller('users')
export class UsersController {}
```

---

## 10. What are Pipes in NestJS?

**Answer:** Pipes transform or validate input data before it reaches the route handler.

**Built-in Pipes:**
- `ValidationPipe`
- `ParseIntPipe`
- `ParseBoolPipe`
- `ParseArrayPipe`
- `ParseUUIDPipe`
- `DefaultValuePipe`

Example:
```typescript
// Validation pipe
@Post()
create(@Body(ValidationPipe) createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}

// Parse pipe
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.usersService.findOne(id);
}

// Custom pipe
@Injectable()
export class CustomValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Custom validation logic
    return value;
  }
}
```

---

## 11. What are Exception Filters in NestJS?

**Answer:** Exception Filters handle exceptions thrown in the application. They catch exceptions and send appropriate HTTP responses.

Example:
```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}

// Usage
@UseFilters(HttpExceptionFilter)
@Controller('users')
export class UsersController {}
```

---

## 12. What is the difference between @Injectable and @Controller?

**Answer:**

| @Injectable | @Controller |
|-------------|-------------|
| Marks a class as a provider | Marks a class as a controller |
| Business logic | Request handling |
| Can be injected | Cannot be injected |
| No routing | Handles HTTP routes |
| Services, repositories | Route handlers |

Example:
```typescript
// Provider
@Injectable()
export class UserService {
  // Business logic
}

// Controller
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}
  // Route handlers
}
```

---

## 13. How do you handle validation in NestJS?

**Answer:** NestJS uses `class-validator` and `class-transformer` packages along with `ValidationPipe` for validation.

Example:
```typescript
// DTO with validation
import { IsString, IsEmail, IsNotEmpty, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsNotEmpty()
  @MinLength(6)
  password: string;
}

// Enable validation globally
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));
  await app.listen(3000);
}

// Controller
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}
```

---

## 14. What is the difference between @Param, @Query, and @Body decorators?

**Answer:**

| Decorator | Use | Example URL |
|-----------|-----|-------------|
| @Param | Route parameters | /users/123 |
| @Query | Query strings | /users?name=john |
| @Body | Request body | POST data |

Example:
```typescript
@Controller('users')
export class UsersController {
  // @Param
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  // @Query
  @Get()
  findAll(@Query('page') page: number, @Query('limit') limit: number) {
    return this.usersService.findAll(page, limit);
  }

  // @Body
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

---

## 15. How do you implement authentication in NestJS?

**Answer:** NestJS provides `@nestjs/passport` for implementing authentication strategies.

Example with JWT:
```typescript
// JWT Strategy
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}

// Auth Service
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async login(user: any) {
    const payload = { username: user.username, sub: user.userId };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}

// Auth Guard
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

// Usage
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile(@Request() req) {
  return req.user;
}
```

---

## 16. What are Dynamic Modules in NestJS?

**Answer:** Dynamic Modules allow you to create modules with dynamic configuration at runtime.

Example:
```typescript
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useValue: options,
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
    };
  }
}

// Usage
@Module({
  imports: [
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
    }),
  ],
})
export class AppModule {}
```

---

## 17. How do you implement file upload in NestJS?

**Answer:** NestJS uses `multer` for file uploads through `FileInterceptor` or `FilesInterceptor`.

Example:
```typescript
@Controller('upload')
export class UploadController {
  // Single file
  @Post('single')
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(@UploadedFile() file: Express.Multer.File) {
    return {
      filename: file.filename,
      size: file.size,
      mimetype: file.mimetype,
    };
  }

  // Multiple files
  @Post('multiple')
  @UseInterceptors(FilesInterceptor('files'))
  uploadFiles(@UploadedFiles() files: Express.Multer.File[]) {
    return files.map(file => ({
      filename: file.filename,
      size: file.size,
    }));
  }
}
```

---

## 18. What is the difference between synchronous and asynchronous providers?

**Answer:**

**Synchronous Provider:**
```typescript
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: { port: 3000 },
    },
  ],
})
export class AppModule {}
```

**Asynchronous Provider:**
```typescript
@Module({
  providers: [
    {
      provide: 'ASYNC_CONFIG',
      useFactory: async () => {
        const config = await fetchConfigFromAPI();
        return config;
      },
    },
  ],
})
export class AppModule {}
```

---

## 19. How do you implement request scoped providers?

**Answer:** Request scoped providers create a new instance for each request.

**Scopes:**
- `DEFAULT`: Single instance (singleton)
- `REQUEST`: New instance per request
- `TRANSIENT`: New instance each time it's injected

Example:
```typescript
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  constructor(@Inject(REQUEST) private request: Request) {}

  getRequestId() {
    return this.request.headers['x-request-id'];
  }
}
```

---

## 20. What are Custom Decorators in NestJS?

**Answer:** Custom decorators allow you to create reusable parameter decorators.

Example:
```typescript
// Custom decorator
export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('profile')
getProfile(@User() user: UserEntity) {
  return user;
}

@Get('email')
getEmail(@User('email') email: string) {
  return { email };
}
```

---

## 21. How do you implement database integration with TypeORM?

**Answer:**

```typescript
// Install: npm install @nestjs/typeorm typeorm mysql2

// Entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;
}

// Module
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'test',
      entities: [User],
      synchronize: true,
    }),
    TypeOrmModule.forFeature([User]),
  ],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}

// Service
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: number): Promise<User> {
    return this.usersRepository.findOne({ where: { id } });
  }

  create(user: Partial<User>): Promise<User> {
    const newUser = this.usersRepository.create(user);
    return this.usersRepository.save(newUser);
  }
}
```

---

## 22. What is the execution order of NestJS components?

**Answer:**

**Request Flow:**
1. Middleware
2. Guards
3. Interceptors (before)
4. Pipes
5. Controller/Route Handler
6. Service
7. Interceptors (after)
8. Exception Filters (if error occurs)
9. Response

---

## 23. How do you implement caching in NestJS?

**Answer:**

```typescript
// Install: npm install cache-manager

// Module
@Module({
  imports: [
    CacheModule.register({
      ttl: 5, // seconds
      max: 100, // maximum number of items
    }),
  ],
})
export class AppModule {}

// Controller
@Controller('users')
export class UsersController {
  constructor(
    private usersService: UsersService,
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
  ) {}

  @Get()
  @UseInterceptors(CacheInterceptor)
  async findAll() {
    return this.usersService.findAll();
  }

  // Manual caching
  @Get(':id')
  async findOne(@Param('id') id: string) {
    const cacheKey = `user_${id}`;
    const cached = await this.cacheManager.get(cacheKey);

    if (cached) {
      return cached;
    }

    const user = await this.usersService.findOne(id);
    await this.cacheManager.set(cacheKey, user, { ttl: 300 });
    return user;
  }
}
```

---

## 24. What are Lifecycle Events in NestJS?

**Answer:** Lifecycle hooks allow you to tap into key moments in the lifecycle of modules, providers, and controllers.

**Lifecycle Hooks:**
- `onModuleInit()`: Called once module is initialized
- `onApplicationBootstrap()`: Called after all modules initialized
- `onModuleDestroy()`: Called before module is destroyed
- `beforeApplicationShutdown()`: Called before app shutdown
- `onApplicationShutdown()`: Called during app shutdown

Example:
```typescript
@Injectable()
export class UsersService implements OnModuleInit, OnModuleDestroy {
  onModuleInit() {
    console.log('UsersService initialized');
  }

  onModuleDestroy() {
    console.log('UsersService destroyed');
  }
}
```

---

## 25. How do you implement microservices in NestJS?

**Answer:**

```typescript
// Microservice setup
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: {
        host: 'localhost',
        port: 8877,
      },
    },
  );
  await app.listen();
}

// Message pattern
@Controller()
export class MathController {
  @MessagePattern({ cmd: 'sum' })
  accumulate(data: number[]): number {
    return (data || []).reduce((a, b) => a + b);
  }
}

// Client
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'localhost',
          port: 8877,
        },
      },
    ]),
  ],
})
export class AppModule {}

// Usage
@Controller()
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private client: ClientProxy,
  ) {}

  @Get('sum')
  async accumulate() {
    const pattern = { cmd: 'sum' };
    const payload = [1, 2, 3];
    return this.client.send<number>(pattern, payload);
  }
}
```

---

## 26. What is the difference between @Req() and @Request()?

**Answer:** Both `@Req()` and `@Request()` are aliases and work identically. They inject the request object into the controller method.

```typescript
@Get()
findAll(@Req() request: Request) {
  return request.headers;
}

// Same as
@Get()
findAll(@Request() request: Request) {
  return request.headers;
}
```

---

## 27. How do you implement configuration management in NestJS?

**Answer:**

```typescript
// Install: npm install @nestjs/config

// .env file
DATABASE_HOST=localhost
DATABASE_PORT=5432
JWT_SECRET=mysecret

// Module
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      validationSchema: Joi.object({
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().default(5432),
      }),
    }),
  ],
})
export class AppModule {}

// Usage in service
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getDatabaseHost(): string {
    return this.configService.get<string>('DATABASE_HOST');
  }
}

// Custom configuration
export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
}));
```

---

## 28. What are the different HTTP status codes you can use in NestJS?

**Answer:**

```typescript
import { HttpStatus, HttpCode } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }

  // Throwing exceptions
  @Get(':id')
  async findOne(@Param('id') id: string) {
    const user = await this.usersService.findOne(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return user;
  }
}
```

---

## 29. How do you implement Swagger/OpenAPI documentation?

**Answer:**

```typescript
// Install: npm install @nestjs/swagger swagger-ui-express

// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('API Documentation')
    .setDescription('The API description')
    .setVersion('1.0')
    .addTag('users')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}

// DTO with Swagger decorators
export class CreateUserDto {
  @ApiProperty({ example: 'John Doe' })
  @IsString()
  name: string;

  @ApiProperty({ example: 'john@example.com' })
  @IsEmail()
  email: string;
}

// Controller
@ApiTags('users')
@Controller('users')
export class UsersController {
  @ApiOperation({ summary: 'Create user' })
  @ApiResponse({ status: 201, description: 'User created' })
  @ApiResponse({ status: 400, description: 'Bad request' })
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

---

## 30. What is the difference between Custom Provider patterns?

**Answer:**

```typescript
// 1. Value Provider
{
  provide: 'CONFIG',
  useValue: { port: 3000 },
}

// 2. Class Provider
{
  provide: UsersService,
  useClass: UsersService,
}

// 3. Factory Provider
{
  provide: 'CONNECTION',
  useFactory: (config: ConfigService) => {
    return createConnection(config);
  },
  inject: [ConfigService],
}

// 4. Existing Provider (Alias)
{
  provide: 'AliasService',
  useExisting: UsersService,
}
```

---

## 31. How do you implement logging in NestJS?

**Answer:**

```typescript
// Built-in logger
import { Logger } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  findAll() {
    this.logger.log('Finding all users');
    this.logger.debug('Debug message');
    this.logger.warn('Warning message');
    this.logger.error('Error message', 'Stack trace');
    return this.users;
  }
}

// Custom logger
@Injectable()
export class CustomLogger implements LoggerService {
  log(message: string) {
    console.log(`[LOG] ${message}`);
  }

  error(message: string, trace: string) {
    console.error(`[ERROR] ${message}`, trace);
  }

  warn(message: string) {
    console.warn(`[WARN] ${message}`);
  }

  debug(message: string) {
    console.debug(`[DEBUG] ${message}`);
  }

  verbose(message: string) {
    console.log(`[VERBOSE] ${message}`);
  }
}

// Use custom logger
const app = await NestFactory.create(AppModule, {
  logger: new CustomLogger(),
});
```

---

## 32. What are Serialization and how do you implement it?

**Answer:** Serialization transforms objects before sending them in the response.

```typescript
// Install: npm install class-transformer

// Entity
export class UserEntity {
  id: number;
  name: string;
  email: string;

  @Exclude()
  password: string;

  @Expose()
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  constructor(partial: Partial<UserEntity>) {
    Object.assign(this, partial);
  }
}

// Controller
@Controller('users')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string): UserEntity {
    const user = this.usersService.findOne(id);
    return new UserEntity(user);
  }
}

// Global serialization
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

---

## 33. How do you implement rate limiting in NestJS?

**Answer:**

```typescript
// Install: npm install @nestjs/throttler

// Module
@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60,
      limit: 10,
    }),
  ],
})
export class AppModule {}

// Controller
@Controller('users')
@UseGuards(ThrottlerGuard)
export class UsersController {
  @Throttle(3, 60) // 3 requests per 60 seconds
  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}

// Global rate limiting
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new ThrottlerGuard());
```

---

## 34. What is the difference between Module Re-exporting and Global Modules?

**Answer:**

**Module Re-exporting:**
```typescript
@Module({
  imports: [DatabaseModule],
  exports: [DatabaseModule], // Re-export for others
})
export class CoreModule {}
```

**Global Module:**
```typescript
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

Re-exporting makes modules available through the importing module, while global modules are available everywhere without explicit imports.

---

## 35. How do you implement CORS in NestJS?

**Answer:**

```typescript
// Method 1: Enable in main.ts
const app = await NestFactory.create(AppModule);
app.enableCors();

// Method 2: With options
app.enableCors({
  origin: 'http://localhost:3000',
  methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
  credentials: true,
});

// Method 3: Multiple origins
app.enableCors({
  origin: ['http://localhost:3000', 'http://example.com'],
  credentials: true,
});

// Method 4: Dynamic origin
app.enableCors({
  origin: (origin, callback) => {
    const whitelist = ['http://localhost:3000'];
    if (whitelist.indexOf(origin) !== -1 || !origin) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
});
```

---

## 36. How do you implement WebSockets in NestJS?

**Answer:**

```typescript
// Install: npm install @nestjs/websockets @nestjs/platform-socket.io

// Gateway
@WebSocketGateway({
  cors: {
    origin: '*',
  },
})
export class EventsGateway implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  afterInit(server: Server) {
    console.log('WebSocket initialized');
  }

  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
  }

  @SubscribeMessage('message')
  handleMessage(client: Socket, payload: any): string {
    this.server.emit('message', payload);
    return 'Message received';
  }
}

// Module
@Module({
  providers: [EventsGateway],
})
export class EventsModule {}
```

---

## 37. How do you implement testing in NestJS?

**Answer:**

```typescript
// Unit test
describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            find: jest.fn(),
            findOne: jest.fn(),
            save: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  it('should return all users', async () => {
    const users = [{ id: 1, name: 'John' }];
    jest.spyOn(repository, 'find').mockResolvedValue(users);
    expect(await service.findAll()).toBe(users);
  });
});

// E2E test
describe('AppController (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect('Content-Type', /json/);
  });
});
```

---

## 38. What are the best practices for structuring a NestJS application?

**Answer:**

**1. Feature-based Structure:**
```
src/
├── common/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── pipes/
├── config/
├── modules/
│   ├── users/
│   │   ├── dto/
│   │   ├── entities/
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   └── users.module.ts
│   └── auth/
├── app.module.ts
└── main.ts
```

**2. Best Practices:**
- Use DTOs for data validation
- Implement proper error handling
- Use dependency injection
- Write tests
- Document APIs with Swagger
- Use environment variables
- Implement logging
- Follow SOLID principles
- Use async/await
- Implement proper authentication/authorization

---

## 39. How do you handle circular dependencies in NestJS?

**Answer:**

**Problem:**
```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(private authService: AuthService) {}
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}
}
```

**Solutions:**

**1. Forward Reference:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    @Inject(forwardRef(() => AuthService))
    private authService: AuthService,
  ) {}
}

@Injectable()
export class AuthService {
  constructor(
    @Inject(forwardRef(() => UsersService))
    private usersService: UsersService,
  ) {}
}
```

**2. ModuleRef (Better approach):**
```typescript
@Injectable()
export class UsersService {
  private authService: AuthService;

  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.authService = this.moduleRef.get(AuthService);
  }
}
```

**3. Restructure (Best approach):**
- Extract common logic to a separate service
- Re-evaluate architecture

---

## 40. What are the differences between NestJS and Express?

**Answer:**

| Feature | NestJS | Express |
|---------|--------|---------|
| Architecture | Opinionated, modular | Unopinionated, flexible |
| TypeScript | Built-in | Requires setup |
| Dependency Injection | Yes | No |
| Decorators | Yes | No |
| Structure | Enforced | Flexible |
| Learning Curve | Steeper | Easier |
| Testing | Built-in support | Manual setup |
| Microservices | Built-in | Manual |
| GraphQL | Built-in | Requires packages |
| Documentation | Extensive | Community-driven |
| Enterprise Ready | Yes | Requires setup |
| Performance | Slightly slower | Faster |
| Best For | Large applications | Small to medium apps |

**When to use NestJS:**
- Enterprise applications
- Microservices
- Teams preferring structure
- TypeScript projects
- Complex applications

**When to use Express:**
- Simple APIs
- Prototypes
- Lightweight applications
- Maximum flexibility needed
- Performance critical apps

---

## Additional Resources:

- Official Documentation: https://docs.nestjs.com
- GitHub: https://github.com/nestjs/nest
- Discord Community
- Stack Overflow

## Practice Tips:

1. Build CRUD applications
2. Implement authentication/authorization
3. Practice with TypeORM or Mongoose
4. Create microservices
5. Write tests
6. Build real-world projects
7. Read official documentation
8. Contribute to open source

