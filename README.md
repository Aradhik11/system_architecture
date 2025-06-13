# Republic Webstory Experience - Complete System Architecture

## Table of Contents
1. [UML Diagrams](#1-uml-diagrams)
2. [Technical Architecture Documentation](#2-technical-architecture-documentation)
3. [Database Schema](#3-database-schema)
4. [Caching Strategy](#4-caching-strategy)
5. [NBA Uniforms Case Study Redesign](#5-nba-uniforms-case-study-redesign)
6. [Testing Strategy (Bonus)](#6-testing-strategy-bonus)

---

## 1. UML Diagrams

### 1.1 Use Case Diagram

```
                    Republic Webstory CMS - Use Cases

    Content Creator          Story Editor           Reader              Admin
         |                      |                    |                  |
         |                      |                    |                  |
    ┌────────────┐         ┌────────────┐      ┌───────────┐     ┌──────────┐
    │Create Story│         │Edit Story  │      │View Story │     │Manage    │
    │Draft       │         │Content     │      │           │     │Users     │
    └────────────┘         └────────────┘      └───────────┘     └──────────┘
         |                      |                    |                  |
    ┌────────────┐         ┌────────────┐      ┌───────────┐     ┌──────────┐
    │Add         │         │Preview     │      │Interact   │     │System    │
    │Components  │         │Story       │      │with Story │     │Analytics │
    └────────────┘         └────────────┘      └───────────┘     └──────────┘
         |                      |                    |                  |
    ┌────────────┐         ┌────────────┐      ┌───────────┐     ┌──────────┐
    │Upload      │         │Publish     │      │Share Story│     │Content   │
    │Media Assets│         │Story       │      │           │     │Moderation│
    └────────────┘         └────────────┘      └───────────┘     └──────────┘
         |                      |                    |                  |
    ┌────────────┐         ┌────────────┐      ┌───────────┐     ┌──────────┐
    │Collaborate │         │Version     │      │Bookmark   │     │Backup &  │
    │on Story    │         │Control     │      │Story      │     │Recovery  │
    └────────────┘         └────────────┘      └───────────┘     └──────────┘

                                    |
                            ┌───────────────┐
                            │    CMS System │
                            │               │
                            │ - Story Mgmt  │
                            │ - Media Mgmt  │
                            │ - User Mgmt   │
                            │ - Analytics   │
                            │ - Caching     │
                            └───────────────┘
```

### 1.2 Class Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│      User       │    │     Story       │    │   Component     │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│- id: string     │    │- id: string     │    │- id: string     │
│- email: string  │    │- title: string  │    │- type: enum     │
│- name: string   │    │- slug: string   │    │- content: JSON  │
│- role: enum     │    │- description    │    │- position: int  │
│- createdAt      │    │- publishedAt    │    │- settings: JSON │
│- updatedAt      │    │- status: enum   │    │- storyId: string│
├─────────────────┤    │- authorId       │    │- createdAt      │
│+ createStory()  │    │- thumbnail      │    │- updatedAt      │
│+ editProfile()  │    │- tags: string[] │    ├─────────────────┤
│+ authenticate() │    │- viewCount: int │    │+ render()       │
└─────────────────┘    │- createdAt      │    │+ validate()     │
         │              │- updatedAt      │    │+ duplicate()    │
         │              ├─────────────────┤    └─────────────────┘
         │              │+ publish()      │              │
         │              │+ unpublish()    │              │
         │              │+ addComponent() │              │
         │              │+ getAnalytics() │              │
         │              └─────────────────┘              │
         │                       │                       │
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                                 │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   MediaAsset    │    │   StoryVersion  │    │  Interaction    │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│- id: string     │    │- id: string     │    │- id: string     │
│- filename       │    │- storyId        │    │- userId: string │
│- originalName   │    │- versionNumber  │    │- storyId        │
│- mimeType       │    │- content: JSON  │    │- type: enum     │
│- size: bigint   │    │- createdBy      │    │- data: JSON     │
│- url: string    │    │- createdAt      │    │- timestamp      │
│- thumbnailUrl   │    │- comment        │    │- sessionId      │
│- storyId        │    ├─────────────────┤    ├─────────────────┤
│- createdAt      │    │+ restore()      │    │+ track()        │
│- updatedAt      │    │+ compare()      │    │+ aggregate()    │
├─────────────────┤    └─────────────────┘    └─────────────────┘
│+ optimize()     │
│+ generateThumbs()│
│+ getMetadata()  │
└─────────────────┘

Component Types (Inheritance):
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   TextBlock     │    │  ImageSequence  │    │ DataVisualization│
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│- text: string   │    │- images: Array  │    │- chartType: enum│
│- formatting     │    │- transition     │    │- data: JSON     │
│- animations     │    │- autoplay: bool │    │- config: JSON   │
├─────────────────┤    │- duration: int  │    ├─────────────────┤
│+ renderText()   │    ├─────────────────┤    │+ renderChart()  │
└─────────────────┘    │+ playSequence() │    │+ updateData()   │
                       └─────────────────┘    └─────────────────┘

┌─────────────────┐    ┌─────────────────┐
│ ParallaxSection │    │ InteractiveMap  │
├─────────────────┤    ├─────────────────┤
│- backgroundImg  │    │- mapData: JSON  │
│- speed: float   │    │- markers: Array │
│- direction      │    │- interactions   │
├─────────────────┤    ├─────────────────┤
│+ calculatePos() │    │+ renderMap()    │
│+ updateScroll() │    │+ addMarker()    │
└─────────────────┘    └─────────────────┘
```

---

## 2. Technical Architecture Documentation

### 2.1 EC2 Deployment Workflow

#### Dockerfile
```dockerfile
# Backend Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

#### EC2 Setup Guide
```bash
#!/bin/bash
# EC2 Instance Setup Script

# Update system
sudo yum update -y

# Install Docker
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install Node.js and PM2
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 18
npm install -g pm2

# Setup application directory
sudo mkdir -p /var/www/republic-cms
sudo chown ec2-user:ec2-user /var/www/republic-cms

# Install Nginx for reverse proxy
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Configure Nginx
sudo tee /etc/nginx/conf.d/republic-cms.conf > /dev/null <<EOF
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

sudo systemctl reload nginx
```

#### GitHub Actions CI/CD Configuration
```yaml
# .github/workflows/deploy.yml
name: Deploy Republic CMS

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_DATABASE: republic_cms_test
          MYSQL_ROOT_PASSWORD: testpassword
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run database migrations
      run: npm run db:migrate
      env:
        DATABASE_URL: mysql://root:testpassword@localhost:3306/republic_cms_test
    
    - name: Run tests
      run: npm run test:cov
      env:
        DATABASE_URL: mysql://root:testpassword@localhost:3306/republic_cms_test
    
    - name: Build application
      run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Build and push Docker image
      run: |
        docker build -t republic-cms .
        docker tag republic-cms:latest ${{ secrets.ECR_REPOSITORY_URL }}:latest
        aws ecr get-login-password | docker login --username AWS --password-stdin ${{ secrets.ECR_REPOSITORY_URL }}
        docker push ${{ secrets.ECR_REPOSITORY_URL }}:latest
    
    - name: Deploy to EC2
      uses: appleboy/ssh-action@v0.1.5
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ec2-user
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          cd /var/www/republic-cms
          docker pull ${{ secrets.ECR_REPOSITORY_URL }}:latest
          docker-compose down
          docker-compose up -d
          docker system prune -f
```

### 2.2 NestJS Setup

#### API Endpoint Structure
```typescript
// src/app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    DatabaseModule,
    AuthModule,
    StoriesModule,
    ComponentsModule,
    MediaModule,
    AnalyticsModule,
    CacheModule.register({ isGlobal: true }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

// API Endpoints Structure:
/*
Authentication:
POST   /auth/login
POST   /auth/register
POST   /auth/refresh
DELETE /auth/logout

Stories:
GET    /stories              - List all published stories
POST   /stories              - Create new story
GET    /stories/:id          - Get story by ID
PUT    /stories/:id          - Update story
DELETE /stories/:id          - Delete story
POST   /stories/:id/publish  - Publish story
POST   /stories/:id/unpublish - Unpublish story
GET    /stories/:id/versions - Get story versions
POST   /stories/:id/duplicate - Duplicate story

Components:
GET    /stories/:storyId/components     - Get story components
POST   /stories/:storyId/components     - Add component to story
PUT    /components/:id                  - Update component
DELETE /components/:id                  - Delete component
POST   /components/:id/duplicate        - Duplicate component

Media:
POST   /media/upload         - Upload media file
GET    /media/:id           - Get media file
DELETE /media/:id           - Delete media file
POST   /media/optimize      - Optimize media files

Analytics:
GET    /analytics/stories/:id    - Get story analytics
GET    /analytics/dashboard      - Get dashboard data
POST   /analytics/interactions   - Track user interactions

Users (Admin):
GET    /users               - List users
POST   /users               - Create user
PUT    /users/:id          - Update user
DELETE /users/:id          - Delete user
*/
```

#### Folder Structure and Architectural Approach
```
src/
├── main.ts                    # Application entry point
├── app.module.ts             # Root module
├── config/                   # Configuration files
│   ├── database.config.ts
│   ├── aws.config.ts
│   └── cache.config.ts
├── common/                   # Shared utilities
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── dto/
├── modules/
│   ├── auth/                 # Authentication module
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.module.ts
│   │   ├── strategies/
│   │   └── dto/
│   ├── stories/              # Stories management
│   │   ├── stories.controller.ts
│   │   ├── stories.service.ts
│   │   ├── stories.module.ts
│   │   ├── entities/
│   │   └── dto/
│   ├── components/           # Story components
│   │   ├── components.controller.ts
│   │   ├── components.service.ts
│   │   ├── components.module.ts
│   │   ├── types/
│   │   └── dto/
│   ├── media/                # Media management
│   │   ├── media.controller.ts
│   │   ├── media.service.ts
│   │   ├── media.module.ts
│   │   └── dto/
│   ├── analytics/            # Analytics tracking
│   │   ├── analytics.controller.ts
│   │   ├── analytics.service.ts
│   │   ├── analytics.module.ts
│   │   └── dto/
│   └── users/                # User management
│       ├── users.controller.ts
│       ├── users.service.ts
│       ├── users.module.ts
│       └── dto/
├── database/                 # Database related files
│   ├── database.module.ts
│   ├── prisma.service.ts
│   ├── migrations/
│   └── seeds/
└── utils/                    # Utility functions
    ├── image-processing.ts
    ├── validation.ts
    └── helpers.ts
```

### 2.3 Frontend Stack Documentation

#### Tool Selection and Rationale

**Next.js 14 (App Router)**
- **Why**: Server-side rendering for better SEO and performance
- **Benefits**: Built-in optimization, API routes, image optimization
- **Use Case**: Dynamic story rendering with SSR/SSG hybrid approach

**TypeScript**
- **Why**: Type safety for complex component relationships
- **Benefits**: Better developer experience, fewer runtime errors
- **Use Case**: Strict typing for story components and data structures

**TailwindCSS**
- **Why**: Utility-first CSS for rapid UI development
- **Benefits**: Consistent design system, responsive utilities
- **Use Case**: Styling interactive components and animations

**Framer Motion**
- **Why**: Advanced animations for parallax and scroll effects
- **Benefits**: Declarative animations, gesture support
- **Use Case**: Scroll-triggered animations and parallax effects

**React Query (TanStack Query)**
- **Why**: Data fetching with caching and synchronization
- **Benefits**: Optimistic updates, background refetching
- **Use Case**: Story content loading and CMS data management

**Zustand**
- **Why**: Lightweight state management
- **Benefits**: Simple API, TypeScript support
- **Use Case**: Story editor state and component management

**React Hook Form**
- **Why**: Performant forms with minimal re-renders
- **Benefits**: Built-in validation, easy integration
- **Use Case**: CMS forms and story editing interfaces

#### Frontend Architecture
```
frontend/
├── app/                      # Next.js 14 App Router
│   ├── (cms)/               # CMS routes group
│   │   ├── dashboard/
│   │   ├── stories/
│   │   └── media/
│   ├── (public)/            # Public story routes
│   │   └── story/[slug]/
│   ├── api/                 # API routes
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── components/              # Reusable components
│   ├── ui/                  # Base UI components
│   ├── story/               # Story-specific components
│   │   ├── TextBlock.tsx
│   │   ├── ImageSequence.tsx
│   │   ├── ParallaxSection.tsx
│   │   ├── DataVisualization.tsx
│   │   └── InteractiveMap.tsx
│   ├── cms/                 # CMS interface components
│   └── layout/              # Layout components
├── lib/                     # Utilities and configurations
│   ├── api.ts              # API client
│   ├── auth.ts             # Authentication utilities
│   ├── utils.ts            # General utilities
│   └── validations.ts      # Form validations
├── hooks/                   # Custom React hooks
│   ├── useStory.ts
│   ├── useAnimations.ts
│   └── useIntersection.ts
├── store/                   # State management
│   ├── story-store.ts
│   └── cms-store.ts
├── styles/                  # Additional styles
└── types/                   # TypeScript type definitions
    ├── story.types.ts
    ├── component.types.ts
    └── api.types.ts
```

#### Frontend Deployment Strategy (Vercel)
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1", "lhr1", "hnd1"],
  "env": {
    "NEXT_PUBLIC_API_URL": "@api-url",
    "NEXT_PUBLIC_CDN_URL": "@cdn-url"
  },
  "build": {
    "env": {
      "DATABASE_URL": "@database-url"
    }
  },
  "functions": {
    "app/api/**": {
      "maxDuration": 30
    }
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api.republic.com.ng/:path*"
    }
  ]
}
```

---

## 3. Database Schema

### 3.1 Complete Prisma Schema (MySQL)

```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  ADMIN
  EDITOR
  CREATOR
  VIEWER
}

enum StoryStatus {
  DRAFT
  REVIEW
  PUBLISHED
  ARCHIVED
}

enum ComponentType {
  TEXT_BLOCK
  IMAGE_SEQUENCE
  PARALLAX_SECTION
  DATA_VISUALIZATION
  INTERACTIVE_MAP
  VIDEO_PLAYER
  AUDIO_PLAYER
  QUOTE_BLOCK
  DIVIDER
  SPACER
}

enum InteractionType {
  VIEW
  SCROLL
  CLICK
  HOVER
  SHARE
  BOOKMARK
  TIME_SPENT
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  avatar    String?
  role      UserRole @default(CREATOR)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  stories          Story[]
  storyVersions    StoryVersion[]
  interactions     Interaction[]
  collaborations   StoryCollaborator[]

  @@map("users")
}

model Story {
  id          String      @id @default(cuid())
  title       String
  slug        String      @unique
  description String?     @db.Text
  thumbnail   String?
  tags        String?     // JSON array stored as string
  status      StoryStatus @default(DRAFT)
  publishedAt DateTime?
  viewCount   Int         @default(0)
  
  // SEO fields
  metaTitle       String?
  metaDescription String?
  ogImage         String?
  
  // Story settings
  settings    Json?       // Theme, colors, fonts, etc.
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  author         User                @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId       String
  components     Component[]
  versions       StoryVersion[]
  mediaAssets    MediaAsset[]
  interactions   Interaction[]
  collaborators  StoryCollaborator[]

  @@map("stories")
}

model Component {
  id       String        @id @default(cuid())
  type     ComponentType
  content  Json          // Component-specific data
  position Int           // Order in story
  settings Json?         // Animation, styling settings
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  story   Story  @relation(fields: [storyId], references: [id], onDelete: Cascade)
  storyId String

  @@map("components")
}

model StoryVersion {
  id            String   @id @default(cuid())
  versionNumber Int
  content       Json     // Complete story snapshot
  comment       String?
  createdAt     DateTime @default(now())

  // Relations
  story     Story  @relation(fields: [storyId], references: [id], onDelete: Cascade)
  storyId   String
  createdBy User   @relation(fields: [userId], references: [id])
  userId    String

  @@unique([storyId, versionNumber])
  @@map("story_versions")
}

model MediaAsset {
  id           String  @id @default(cuid())
  filename     String
  originalName String
  mimeType     String
  size         BigInt
  url          String
  thumbnailUrl String?
  alt          String?
  caption      String?
  
  // Image-specific metadata
  width  Int?
  height Int?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  story   Story?  @relation(fields: [storyId], references: [id], onDelete: SetNull)
  storyId String?

  @@map("media_assets")
}

model Interaction {
  id        String          @id @default(cuid())
  type      InteractionType
  data      Json?           // Interaction-specific data
  timestamp DateTime        @default(now())
  sessionId String?
  
  // Analytics data
  userAgent   String?
  ipAddress   String?
  referrer    String?
  
  // Relations
  user    User?   @relation(fields: [userId], references: [id], onDelete: SetNull)
  userId  String?
  story   Story   @relation(fields: [storyId], references: [id], onDelete: Cascade)
  storyId String

  @@map("interactions")
}

model StoryCollaborator {
  id         String   @id @default(cuid())
  permission String   // READ, WRITE, ADMIN
  createdAt  DateTime @default(now())

  // Relations
  story   Story  @relation(fields: [storyId], references: [id], onDelete: Cascade)
  storyId String
  user    User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId  String

  @@unique([storyId, userId])
  @@map("story_collaborators")
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  color String?
  
  createdAt DateTime @default(now())

  @@map("tags")
}

model Analytics {
  id        String   @id @default(cuid())
  storyId   String
  date      DateTime @db.Date
  views     Int      @default(0)
  uniqueViews Int    @default(0)
  avgTimeSpent Float @default(0)
  bounceRate   Float @default(0)
  
  // Engagement metrics
  scrollDepth      Float @default(0)
  interactionCount Int   @default(0)
  shareCount       Int   @default(0)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([storyId, date])
  @@map("analytics")
}

// Configuration table for system settings
model SystemConfig {
  id    String @id @default(cuid())
  key   String @unique
  value Json
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("system_config")
}
```

---

## 4. Caching Strategy

### 4.1 Multi-Layer Caching Architecture

```typescript
// Caching Configuration
interface CachingStrategy {
  // Layer 1: Browser Cache
  browserCache: {
    staticAssets: '1 year',
    apiResponses: '5 minutes',
    storyContent: '1 hour'
  };
  
  // Layer 2: CDN Cache (CloudFront)
  cdnCache: {
    images: '1 year',
    videos: '1 year',
    staticFiles: '1 year',
    storyPages: '1 hour'
  };
  
  // Layer 3: Application Cache (Redis/ElastiCache)
  applicationCache: {
    storyData: '30 minutes',
    userSessions: '24 hours',
    analytics: '1 hour',
    mediaMetadata: '4 hours'
  };
  
  // Layer 4: Database Query Cache
  databaseCache: {
    publishedStories: '15 minutes',
    userProfiles: '1 hour',
    systemConfig: '24 hours'
  };
}
```

### 4.2 AWS-Based Caching Implementation

#### CloudFront Configuration
```typescript
// CloudFront Distribution Setup
const cloudFrontConfig = {
  origins: [
    {
      domainName: 'api.republic.com.ng',
      originPath: '/api',
      customOriginConfig: {
        httpPort: 443,
        originProtocolPolicy: 'https-only'
      }
    },
    {
      domainName: 'republic-media.s3.amazonaws.com',
      originPath: '/media',
      s3OriginConfig: {
        originAccessIdentity: 'origin-access-identity/cloudfront/ABCDEFGHIJKLMNOP'
      }
    }
  ],
  behaviors: [
    {
      pathPattern: '/api/stories/*',
      targetOriginId: 'api-origin',
      cachePolicyId: 'managed-caching-optimized',
      ttl: {
        defaultTTL: 3600, // 1 hour
        maxTTL: 86400     // 24 hours
      }
    },
    {
      pathPattern: '/media/*',
      targetOriginId: 's3-origin',
      cachePolicyId: 'managed-caching-optimized-for-uncompressed-objects',
      ttl: {
        defaultTTL: 31536000, // 1 year
        maxTTL: 31536000      // 1 year
      }
    }
  ]
};
```

#### ElastiCache (Redis) Setup
```typescript
// Redis Caching Service
@Injectable()
export class CacheService {
  constructor(
    @Inject('REDIS_CLIENT') private readonly redis: Redis,
  ) {}

  // Story caching
  async cacheStory(storyId: string, storyData: any): Promise<void> {
    const cacheKey = `story:${storyId}`;
    await this.redis.setex(cacheKey, 1800, JSON.stringify(storyData)); // 30 minutes
  }

  async getCachedStory(storyId: string): Promise<any | null> {
    const cacheKey = `story:${storyId}`;
    const cached = await this.redis.get(cacheKey);
    return cached ? JSON.parse(cached) : null;
  }

  // Image caching with metadata
  async cacheImageMetadata(imageId: string, metadata: any): Promise<void> {
    const cacheKey = `image:${imageId}:metadata`;
    await this.redis.setex(cacheKey, 14400, JSON.stringify(metadata)); // 4 hours
  }

  // Analytics caching
  async cacheAnalytics(storyId: string, period: string, data: any): Promise<void> {
    const cacheKey = `analytics:${storyId}:${period}`;
    await this.redis.setex(cacheKey, 3600, JSON.stringify(data)); // 1 hour
  }

  // Cache invalidation
  async invalidateStoryCache(storyId: string): Promise<void> {
    const patterns = [
      `story:${storyId}`,
      `story:${storyId}:*`,
      `analytics:${storyId}:*`
    ];
    
    for (const pattern of patterns) {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.redis.del(...keys);
      }
    }
  }
}
```

#### S3 + CloudFront Image Optimization
```typescript
// Image Processing and Caching Service
@Injectable()
export class ImageService {
  constructor(
    private readonly s3: AWS.S3,
    private readonly cacheService: CacheService,
  ) {}

  async uploadAndOptimizeImage(file: Express.Multer.File): Promise<MediaAsset> {
    const originalKey = `originals/${uuidv4()}-${file.originalname}`;
    const optimizedKey = `optimized/${uuidv4()}-${file.originalname}`;
    
    // Upload original to S3
    await this.s3.upload({
      Bucket: process.env.S3_BUCKET,
      Key: originalKey,
      Body: file.buffer,
      ContentType: file.mimetype,
    }).promise();

    // Create optimized versions
    const optimizedBuffer = await sharp(file.buffer)
      .resize(1920, 1080, { fit: 'inside', withoutEnlargement: true })
      .jpeg({ quality: 85 })
      .toBuffer();

    await this.s3.upload({
      Bucket: process.env.S3_BUCKET,
      Key: optimizedKey,
      Body: optimizedBuffer,
      ContentType: 'image/jpeg',
      CacheControl: 'max-age=31536000', // 1 year
    }).promise();

    // Generate thumbnails
    const thumbnailBuffer = await sharp(file.buffer)
      .resize(400, 300, { fit: 'cover' })
      .jpeg({ quality: 80 })
      .toBuffer();

    const thumbnailKey = `thumbnails/${uuidv4()}-thumb.jpg`;
    await this.s3.upload({
      Bucket: process.env.S3_BUCKET,
      Key: thumbnailKey,
      Body: thumbnailBuffer,
      ContentType: 'image/jpeg',
      CacheControl: 'max-age=31536000',
    }).promise();

    // Cache metadata
    const metadata = {
      originalUrl: `${process.env.CDN_URL}/${originalKey}`,
      optimizedUrl: `${process.env.CDN_URL}/${optimizedKey}`,
      thumbnailUrl: `${process.env.CDN_URL}/${thumbnailKey}`,
      size: file.size,
      dimensions: await this.getImageDimensions(file.buffer),
    };

    await this.cacheService.cacheImageMetadata(optimizedKey, metadata);
    
    return metadata;
  }
}
```

### 4.3 Cache Headers and Strategies
```typescript
// HTTP Cache Headers Configuration
export const cacheHeaders = {
  // Static assets (images, videos, fonts)
  staticAssets: {
    'Cache-Control': 'public, max-age=31536000, immutable',
    'ETag': true,
    'Last-Modified': true,
  },
  
  // Story content
  storyContent: {
    'Cache-Control': 'public, max-age=3600, stale-while-revalidate=86400',
    'ETag': true,
    'Vary': 'Accept-Encoding',
  },
  
  // API responses
  apiResponses: {
    'Cache-Control': 'private, max-age=300, stale-while-revalidate=600',
    'ETag': true,
  },
  
  // User-specific content
  userContent: {
    'Cache-Control': 'private, no-cache',
  },
};
```

---

## 5. NBA Uniforms Case Study Redesign

### 5.1 Component Analysis and Breakdown

Based on the NBA Uniforms story from Pudding.cool, here's how I would restructure it within our CMS:

#### Story Structure Components:
```typescript
interface NBAUniformsStory {
  components: [
    {
      type: 'HERO_SECTION',
      content: {
        title: 'NBA Uniforms: A Visual Evolution',
        background: 'gradient-animation',
        scrollIndicator: true
      }
    },
    {
      type: 'TEXT_BLOCK',
      content: {
        text: 'Introduction to NBA uniform evolution...',
        typography: 'large-intro',
        animations: ['fade-in-up']
      }
    },
    {
      type: 'PARALLAX_SECTION',
      content: {
        backgroundImage: 'nba-court-wide.jpg',
        speed: 0.5,
        content: [
          {
            type: 'floating-text',
            text: '30 Teams',
            position: { x: '20%', y: '30%' }
          },
          {
            type: 'floating-text', 
            text: '75+ Years',
            position: { x: '70%', y: '60%' }
          }
        ]
      }
    },
    {
      type: 'IMAGE_SEQUENCE',
      content: {
        title: 'Uniform Evolution Timeline',
        images: [
          { url: 'uniforms-1950s.jpg', year: '1950s', description: '...' },
          { url: 'uniforms-1960s.jpg', year: '1960s', description: '...' },
          // ... more decades
        ],
        transition: 'slide-fade',
        trigger: 'scroll',
        duration: 800
      }
    },
    {
      type: 'DATA_VISUALIZATION',
      content: {
        chartType: 'interactive-timeline',
        data: {
          teams: [...],
          years: [...],
          changes: [...]
        },
        interactions: ['hover', 'click', 'filter'],
        animations: ['stagger-in']
      }
    },
    {
      type: 'PARALLAX_SECTION',
      content: {
        backgroundImage: 'team-logos-collage.jpg',
        speed: 0.3,
        overlayContent: {
          type: 'scrolling-text',
          text: 'Every team has a story...'
        }
      }
    },
    {
      type: 'INTERACTIVE_COMPARISON',
      content: {
        title: 'Then vs Now',
        pairs: [
          {
            before: { image: 'lakers-1980.jpg', year: '1980' },
            after: { image: 'lakers-2024.jpg', year: '2024' }
          }
          // ... more comparisons
        ],
        interaction: 'slider-reveal'
      }
    },
    {
      type: 'DATA_VISUALIZATION',
      content: {
        chartType: 'color-analysis',
        title: 'Most Popular Colors Over Time',
        data: colorAnalysisData,
        animations: ['morphing-bars', 'color-transitions']
      }
    }
  ]
}
```

### 5.2 Parallax Implementation Strategy

```typescript
// Parallax Component Implementation
interface ParallaxConfig {
  trigger: 'scroll' | 'viewport' | 'manual';
  speed: number; // 0.1 to 2.0
  direction: 'vertical' | 'horizontal' | 'both';
  easing: 'linear' | 'ease-out' | 'bounce';
  breakpoints: {
    mobile: ParallaxSettings;
    tablet: ParallaxSettings;
    desktop: ParallaxSettings;
  };
}

// CMS Integration for Parallax
const parallaxComponentSchema = {
  id: 'parallax-section',
  name: 'Parallax Section',
  category: 'Visual Effects',
  settings: {
    backgroundMedia: {
      type: 'media-picker',
      label: 'Background Image/Video',
      required: true
    },
    parallaxSpeed: {
      type: 'slider',
      label: 'Parallax Speed',
      min: 0.1,
      max: 2.0,
      step: 0.1,
      default: 0.5
    },
    height: {
      type: 'select',
      label: 'Section Height',
      options: ['50vh', '75vh', '100vh', 'auto'],
      default: '100vh'
    },
    overlayContent: {
      type: 'rich-text-editor',
      label: 'Overlay Content'
    },
    overlayOpacity: {
      type: 'slider',
      label: 'Overlay Opacity',
      min: 0,
      max: 1,
      step: 0.1,
      default: 0.7
    }
  }
};

// Frontend Parallax Implementation
const ParallaxSection: React.FC<ParallaxProps> = ({ 
  backgroundImage, 
  speed, 
  children,
  height = '100vh' 
}) => {
  const { scrollY } = useScroll();
  const y = useTransform(scrollY, [0, 1000], [0, 1000 * speed]);
  
  return (
    <motion.div 
      className="relative overflow-hidden"
      style={{ height }}
    >
      <motion.div
        className="absolute inset-0 w-full h-[120%]"
        style={{ 
          backgroundImage: `url(${backgroundImage})`,
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          y 
        }}
      />
      <div className="relative z-10 flex items-center justify-center h-full">
        {children}
      </div>
    </motion.div>
  );
};
```

### 5.3 Component Library for NBA Story

```typescript
// Specialized Components for NBA Uniforms Story
const nbaStoryComponents = {
  UniformComparison: {
    type: 'UNIFORM_COMPARISON',
    template: {
      layout: 'side-by-side',
      interaction: 'slider-reveal',
      metadata: ['team', 'year', 'designer', 'significance']
    }
  },
  
  TeamEvolutionTimeline: {
    type: 'TEAM_TIMELINE',
    template: {
      visualization: 'horizontal-scroll',
      dataPoints: ['logo', 'colors', 'uniform-style'],
      interactions: ['zoom', 'filter-by-decade']
    }
  },
  
  ColorPaletteAnalysis: {
    type: 'COLOR_ANALYSIS',
    template: {
      chartType: 'stacked-area',
      animations: ['color-morphing', 'data-transitions'],
      filters: ['by-team', 'by-era', 'by-color-family']
    }
  },
  
  UniformGallery: {
    type: 'UNIFORM_GALLERY',
    template: {
      layout: 'masonry-grid',
      hover: 'overlay-details',
      zoom: 'modal-lightbox',
      sorting: ['chronological', 'team', 'popularity']
    }
  },
  
  StatisticalBreakdown: {
    type: 'STATS_VISUALIZATION',
    template: {
      charts: ['pie', 'bar', 'scatter'],
      data: 'uniform-changes-frequency',
      interactions: ['drill-down', 'comparison-mode']
    }
  }
};
```

### 5.4 Scroll-Based Animation Framework

```typescript
// Scroll Animation Configuration for CMS
interface ScrollAnimationConfig {
  trigger: {
    start: string; // 'top bottom', 'center center', etc.
    end: string;
    scrub?: boolean; // Smooth scrubbing
    markers?: boolean; // Debug markers
  };
  animations: Array<{
    target: string; // CSS selector
    from: Record<string, any>;
    to: Record<string, any>;
    duration?: number;
    ease?: string;
    stagger?: number;
  }>;
}

// Implementation in CMS Editor
const scrollAnimationEditor = {
  components: [
    {
      type: 'animation-timeline',
      settings: {
        keyframes: [
          { 
            progress: 0, 
            properties: { opacity: 0, y: 100 } 
          },
          { 
            progress: 50, 
            properties: { opacity: 1, y: 0 } 
          },
          { 
            progress: 100, 
            properties: { opacity: 1, scale: 1.1 } 
          }
        ]
      }
    }
  ]
};
```

---

## 6. Testing Strategy (Bonus)

### 6.1 Backend Testing (NestJS)

#### Story Service Unit Tests
```typescript
// src/modules/stories/stories.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { StoriesService } from './stories.service';
import { PrismaService } from '../../database/prisma.service';
import { CacheService } from '../../common/cache/cache.service';
import { NotFoundException } from '@nestjs/common';

describe('StoriesService', () => {
  let service: StoriesService;
  let prismaService: PrismaService;
  let cacheService: CacheService;

  const mockPrismaService = {
    story: {
      create: jest.fn(),
      findMany: jest.fn(),
      findUnique: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    },
  };

  const mockCacheService = {
    cacheStory: jest.fn(),
    getCachedStory: jest.fn(),
    invalidateStoryCache: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        StoriesService,
        {
          provide: PrismaService,
          useValue: mockPrismaService,
        },
        {
          provide: CacheService,
          useValue: mockCacheService,
        },
      ],
    }).compile();

    service = module.get<StoriesService>(StoriesService);
    prismaService = module.get<PrismaService>(PrismaService);
    cacheService = module.get<CacheService>(CacheService);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('createStory', () => {
    it('should create a new story successfully', async () => {
      const createStoryDto = {
        title: 'Test Story',
        slug: 'test-story',
        description: 'A test story',
        authorId: 'user-123',
      };

      const expectedStory = {
        id: 'story-123',
        ...createStoryDto,
        status: 'DRAFT',
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      mockPrismaService.story.create.mockResolvedValue(expectedStory);

      const result = await service.createStory(createStoryDto);

      expect(mockPrismaService.story.create).toHaveBeenCalledWith({
        data: createStoryDto,
        include: {
          author: true,
          components: true,
        },
      });
      expect(result).toEqual(expectedStory);
    });

    it('should handle database errors', async () => {
      const createStoryDto = {
        title: 'Test Story',
        slug: 'duplicate-slug',
        authorId: 'user-123',
      };

      mockPrismaService.story.create.mockRejectedValue(
        new Error('Unique constraint violation')
      );

      await expect(service.createStory(createStoryDto)).rejects.toThrow();
    });
  });

  describe('findStoryById', () => {
    it('should return cached story if available', async () => {
      const storyId = 'story-123';
      const cachedStory = {
        id: storyId,
        title: 'Cached Story',
        status: 'PUBLISHED',
      };

      mockCacheService.getCachedStory.mockResolvedValue(cachedStory);

      const result = await service.findStoryById(storyId);

      expect(mockCacheService.getCachedStory).toHaveBeenCalledWith(storyId);
      expect(mockPrismaService.story.findUnique).not.toHaveBeenCalled();
      expect(result).toEqual(cachedStory);
    });

    it('should fetch from database and cache if not in cache', async () => {
      const storyId = 'story-123';
      const dbStory = {
        id: storyId,
        title: 'Database Story',
        status: 'PUBLISHED',
        components: [],
      };

      mockCacheService.getCachedStory.mockResolvedValue(null);
      mockPrismaService.story.findUnique.mockResolvedValue(dbStory);

      const result = await service.findStoryById(storyId);

      expect(mockCacheService.getCachedStory).toHaveBeenCalledWith(storyId);
      expect(mockPrismaService.story.findUnique).toHaveBeenCalledWith({
        where: { id: storyId },
        include: {
          author: true,
          components: {
            orderBy: { position: 'asc' },
          },
        },
      });
      expect(mockCacheService.cacheStory).toHaveBeenCalledWith(storyId, dbStory);
      expect(result).toEqual(dbStory);
    });

    it('should throw NotFoundException if story not found', async () => {
      const storyId = 'non-existent-story';

      mockCacheService.getCachedStory.mockResolvedValue(null);
      mockPrismaService.story.findUnique.mockResolvedValue(null);

      await expect(service.findStoryById(storyId)).rejects.toThrow(
        NotFoundException
      );
    });
  });

  describe('publishStory', () => {
    it('should publish a draft story successfully', async () => {
      const storyId = 'story-123';
      const draftStory = {
        id: storyId,
        status: 'DRAFT',
        title: 'Draft Story',
      };
      const publishedStory = {
        ...draftStory,
        status: 'PUBLISHED',
        publishedAt: new Date(),
      };

      mockPrismaService.story.findUnique.mockResolvedValue(draftStory);
      mockPrismaService.story.update.mockResolvedValue(publishedStory);

      const result = await service.publishStory(storyId);

      expect(mockPrismaService.story.update).toHaveBeenCalledWith({
        where: { id: storyId },
        data: {
          status: 'PUBLISHED',
          publishedAt: expect.any(Date),
        },
        include: {
          author: true,
          components: {
            orderBy: { position: 'asc' },
          },
        },
      });
      expect(mockCacheService.invalidateStoryCache).toHaveBeenCalledWith(storyId);
      expect(result).toEqual(publishedStory);
    });
  });
});
```

#### Integration Tests
```typescript
// test/stories.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';
import { PrismaService } from '../src/database/prisma.service';

describe('Stories (e2e)', () => {
  let app: INestApplication;
  let prismaService: PrismaService;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    prismaService = moduleFixture.get<PrismaService>(PrismaService);
    
    await app.init();

    // Setup test user and get auth token
    const testUser = await prismaService.user.create({
      data: {
        email: 'test@republic.com.ng',
        name: 'Test User',
        role: 'CREATOR',
      },
    });

    const authResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({
        email: 'test@republic.com.ng',
        password: 'testpassword',
      });

    authToken = authResponse.body.accessToken;
  });

  afterAll(async () => {
    await prismaService.$disconnect();
    await app.close();
  });

  describe('/stories (POST)', () => {
    it('should create a new story', () => {
      return request(app.getHttpServer())
        .post('/stories')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Test Story',
          slug: 'test-story-e2e',
          description: 'Integration test story',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.title).toBe('Test Story');
          expect(res.body.status).toBe('DRAFT');
        });
    });

    it('should validate required fields', () => {
      return request(app.getHttpServer())
        .post('/stories')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          description: 'Missing title and slug',
        })
        .expect(400);
    });
  });

  describe('/stories (GET)', () => {
    it('should return published stories', async () => {
      // Create and publish a test story
      const story = await prismaService.story.create({
        data: {
          title: 'Published Story',
          slug: 'published-story',
          status: 'PUBLISHED',
          authorId: 'test-user-id',
          publishedAt: new Date(),
        },
      });

      return request(app.getHttpServer())
        .get('/stories')
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
          expect(res.body.length).toBeGreaterThan(0);
          expect(res.body[0]).toHaveProperty('id');
          expect(res.body[0]).toHaveProperty('title');
        });
    });
  });
});
```

### 6.2 Frontend Testing (Next.js + React)

#### Component Unit Tests
```typescript
// components/story/TextBlock.test.tsx
import { render, screen } from '@testing-library/react';
import { TextBlock } from './TextBlock';
import { AnimationProvider } from '../../contexts/AnimationContext';

const mockComponent = {
  id: 'component-123',
  type: 'TEXT_BLOCK' as const,
  content: {
    text: 'This is a test text block',
    typography: 'body',
    animations: ['fade-in-up'],
  },
  position: 1,
  settings: {
    backgroundColor: '#ffffff',
    textColor: '#000000',
  },
};

const renderWithProviders = (component: React.ReactElement) => {
  return render(
    <AnimationProvider>
      {component}
    </AnimationProvider>
  );
};

describe('TextBlock Component', () => {
  it('renders text content correctly', () => {
    renderWithProviders(
      <TextBlock component={mockComponent} />
    );

    expect(screen.getByText('This is a test text block')).toBeInTheDocument();
  });

  it('applies correct typography class', () => {
    renderWithProviders(
      <TextBlock component={mockComponent} />
    );

    const textElement = screen.getByText('This is a test text block');
    expect(textElement).toHaveClass('typography-body');
  });

  it('applies custom styling from settings', () => {
    renderWithProviders(
      <TextBlock component={mockComponent} />
    );

    const container = screen.getByTestId('text-block-container');
    expect(container).toHaveStyle({
      backgroundColor: '#ffffff',
      color: '#000000',
    });
  });

  it('handles animation configuration', () => {
    renderWithProviders(
      <TextBlock component={mockComponent} />
    );

    const animatedElement = screen.getByTestId('animated-text');
    expect(animatedElement).toHaveAttribute('data-animation', 'fade-in-up');
  });

  it('renders with markdown support', () => {
    const markdownComponent = {
      ...mockComponent,
      content: {
        ...mockComponent.content,
        text: '**Bold text** and *italic text*',
      },
    };

    renderWithProviders(
      <TextBlock component={markdownComponent} />
    );

    expect(screen.getByText('Bold text')).toHaveStyle({ fontWeight: 'bold' });
    expect(screen.getByText('italic text')).toHaveStyle({ fontStyle: 'italic' });
  });
});
```

#### Integration Tests
```typescript
// __tests__/story-viewer.integration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { StoryViewer } from '../components/story/StoryViewer';

const mockStoryData = {
  id: 'story-123',
  title: 'Test Story',
  slug: 'test-story',
  status: 'PUBLISHED',
  components: [
    {
      id: 'comp-1',
      type: 'TEXT_BLOCK',
      content: { text: 'Introduction text' },
      position: 1,
    },
    {
      id: 'comp-2',
      type: 'IMAGE_SEQUENCE',
      content: {
        images: [
          { url: '/test-image-1.jpg', alt: 'Test Image 1' },
          { url: '/test-image-2.jpg', alt: 'Test Image 2' },
        ],
      },
      position: 2,
    },
  ],
};

const server = setupServer(
  rest.get('/api/stories/:slug', (req, res, ctx) => {
    return res(ctx.json(mockStoryData));
  }),
  rest.post('/api/analytics/interactions', (req, res, ctx) => {
    return res(ctx.json({ success: true }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  });

describe('StoryViewer Integration', () => {
  it('loads and displays story content', async () => {
    const queryClient = createTestQueryClient();

    render(
      <QueryClientProvider client={queryClient}>
        <StoryViewer slug="test-story" />
      </QueryClientProvider>
    );

    // Check loading state
    expect(screen.getByTestId('story-loading')).toBeInTheDocument();

    // Wait for content to load
    await waitFor(() => {
      expect(screen.getByText('Test Story')).toBeInTheDocument();
    });

    expect(screen.getByText('Introduction text')).toBeInTheDocument();
    expect(screen.getByAltText('Test Image 1')).toBeInTheDocument();
  });

  it('handles story not found', async () => {
    server.use(
      rest.get('/api/stories/:slug', (req, res, ctx) => {
        return res(ctx.status(404), ctx.json({ message: 'Story not found' }));
      })
    );

    const queryClient = createTestQueryClient();

    render(
      <QueryClientProvider client={queryClient}>
        <StoryViewer slug="non-existent-story" />
      </QueryClientProvider>
    );

    await waitFor(() => {
      expect(screen.getByText('Story not found')).toBeInTheDocument();
    });
  });

  it('tracks user interactions', async () => {
    const queryClient = createTestQueryClient();
    let interactionTracked = false;

    server.use(
      rest.post('/api/analytics/interactions', (req, res, ctx) => {
        interactionTracked = true;
        return res(ctx.json({ success: true }));
      })
    );

    render(
      <QueryClientProvider client={queryClient}>
        <StoryViewer slug="test-story" />
      </QueryClientProvider>
    );

    await waitFor(() => {
      expect(screen.getByText('Test Story')).toBeInTheDocument();
    });

    // Simulate scroll interaction
    window.dispatchEvent(new Event('scroll'));

    await waitFor(() => {
      expect(interactionTracked).toBe(true);
    });
  });
});
```

#### Performance Tests
```typescript
// __tests__/performance.test.tsx
import { render } from '@testing-library/react';
import { performance } from 'perf_hooks';
import { StoryViewer } from '../components/story/StoryViewer';

describe('Performance Tests', () => {
      it('renders large story within performance budget', async () => {
    const largeStoryData = {
      id: 'large-story',
      title: 'Performance Test Story',
      components: Array.from({ length: 50 }, (_, index) => ({
        id: `comp-${index}`,
        type: 'TEXT_BLOCK',
        content: { text: `Component ${index} content` },
        position: index + 1,
      })),
    };

    const startTime = performance.now();
    
    render(<StoryViewer storyData={largeStoryData} />);
    
    const endTime = performance.now();
    const renderTime = endTime - startTime;

    // Should render within 100ms
    expect(renderTime).toBeLessThan(100);
  });

  it('handles image loading efficiently', async () => {
    const imageHeavyStory = {
      id: 'image-story',
      title: 'Image Heavy Story',
      components: [
        {
          id: 'img-comp',
          type: 'IMAGE_SEQUENCE',
          content: {
            images: Array.from({ length: 20 }, (_, i) => ({
              url: `/test-image-${i}.jpg`,
              alt: `Test Image ${i}`,
            })),
          },
          position: 1,
        },
      ],
    };

    const { container } = render(<StoryViewer storyData={imageHeavyStory} />);
    
    // Check that images are lazy loaded
    const images = container.querySelectorAll('img');
    images.forEach((img) => {
      expect(img).toHaveAttribute('loading', 'lazy');
    });
  });
});
```

---

## Conclusion

This comprehensive system architecture for Republic's visual storytelling CMS provides:

### Key Features Delivered:

1. **Complete UML Documentation**
   - Use Case Diagram mapping all user interactions
   - Detailed Class Diagram showing system relationships

2. **Full-Stack Technical Architecture**
   - NestJS backend with modular structure
   - Next.js frontend optimized for visual storytelling
   - Docker containerization and EC2 deployment
   - Automated CI/CD pipeline with GitHub Actions

3. **Robust Database Design**
   - Comprehensive Prisma schema for MySQL
   - Support for stories, components, media, analytics, and versioning
   - Optimized relationships and data integrity

4. **Advanced Caching Strategy**
   - Multi-layer caching with CloudFront, ElastiCache, and browser caching
   - Image optimization and CDN integration
   - Smart cache invalidation strategies

5. **NBA Uniforms Case Study Redesign**
   - Component-based breakdown of the story
   - Parallax implementation strategy
   - Interactive elements and data visualizations
   - CMS-friendly component structure

6. **Comprehensive Testing**
   - Backend unit and integration tests
   - Frontend component and performance tests
   - E2E testing strategy

### System Strengths:

- **Scalability**: AWS-based infrastructure with auto-scaling capabilities
- **Performance**: Multi-layer caching and optimized media delivery
- **Flexibility**: Modular component system for diverse story types
- **User Experience**: Smooth parallax scrolling and interactive elements
- **Developer Experience**: Type-safe development with comprehensive testing

### Technical Innovation:

- **Visual Story Builder**: Drag-and-drop interface for complex narratives
- **Real-time Collaboration**: Multi-user editing with version control
- **Analytics Integration**: Detailed user interaction tracking
- **Mobile-First Design**: Responsive components optimized for all devices
- **SEO Optimization**: Server-side rendering for better discoverability

This architecture supports Republic's mission of creating highly visual, data-rich storytelling platforms that elevate African narratives through cutting-edge technology and user-centered design.

### Next Steps:

1. **Phase 1**: Backend API development and database setup
2. **Phase 2**: Frontend component library and story viewer
3. **Phase 3**: CMS interface and editorial workflow
4. **Phase 4**: Analytics dashboard and performance optimization
5. **Phase 5**: Advanced features and integrations

The system is designed to be production-ready, scalable, and maintainable, providing a solid foundation for Republic's visual storytelling platform.
