# CATALOGO-AWS-DETERMINISTICO

CATALOGO-AWS-DETERMINISTICO per Next.js 14 Deployment
§ AWS AMPLIFY HOSTING
amplify.yml configurazione completa
yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - corepack enable pnpm
        - pnpm install
        - npx prisma generate
    build:
      commands:
        - pnpm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
      - .next/cache/**/*
      - $HOME/.cache/ppx/**/*
backend:
  phases:
    build:
      commands:
        - echo "No backend build required"
appRoot: .
test:
  phases:
    preTest:
      commands:
        - pnpm run lint
        - pnpm run type-check
    test:
      commands:
        - pnpm run test:ci
  artifacts:
    baseDirectory: coverage
    files:
      - '**/*'
customHeaders:
  - pattern: '**/*'
    headers:
      - key: 'Strict-Transport-Security'
        value: 'max-age=31536000; includeSubDomains'
      - key: 'X-Frame-Options'
        value: 'DENY'
      - key: 'X-Content-Type-Options'
        value: 'nosniff'
      - key: 'Referrer-Policy'
        value: 'strict-origin-when-cross-origin'
Build settings per Next.js 14 App Router
yaml
# Amplify Console Build Settings - ambiente-specifico
env:
  variables:
    NEXT_PUBLIC_APP_ENV: "production"
    NEXT_TELEMETRY_DISABLED: "1"
    PRISMA_SCHEMA_DISABLE_ADVISORY_LOCK: "1"
  build:
    NODE_OPTIONS: "--max-old-space-size=8192"
    NODE_ENV: "production"

build:
  commands:
    - pnpm next build
    - pnpm next-sitemap --config sitemap.config.js
    # ISR path generation
    - node scripts/generate-isr-paths.js

# Ottimizzazioni per App Router
nextConfig:
  trailingSlash: false
  skipTrailingSlashRedirect: true
  output: "standalone"
  experimental:
    optimizeCss: true
    scrollRestoration: true
Environment variables management
typescript
// amplify/environment-variables.ts
export const amplifyEnv = {
  production: {
    NEXT_PUBLIC_API_URL: 'https://api.example.com',
    DATABASE_URL: '${env:DATABASE_URL}',
    REDIS_URL: '${env:REDIS_URL}',
    NEXTAUTH_SECRET: '${env:NEXTAUTH_SECRET}',
    NEXTAUTH_URL: 'https://app.example.com',
  },
  staging: {
    NEXT_PUBLIC_API_URL: 'https://staging-api.example.com',
    DATABASE_URL: '${env:STAGING_DB_URL}',
    REDIS_URL: '${env:STAGING_REDIS_URL}',
    NEXTAUTH_SECRET: '${env:STAGING_NEXTAUTH_SECRET}',
    NEXTAUTH_URL: 'https://staging.example.com',
  },
  development: {
    NEXT_PUBLIC_API_URL: 'http://localhost:3000',
    DATABASE_URL: '${env:DEV_DB_URL}',
    REDIS_URL: '${env:DEV_REDIS_URL}',
    NEXTAUTH_SECRET: '${env:DEV_NEXTAUTH_SECRET}',
    NEXTAUTH_URL: 'http://localhost:3000',
  },
};

// Sistema di gestione variabili sicure
const secretPatterns = [
  /_SECRET$/,
  /_KEY$/,
  /_TOKEN$/,
  /_PASSWORD$/,
  /DATABASE_URL/,
  /REDIS_URL/,
];
Preview deployments per branch
yaml
# Amplify branch configurations
branchConfig:
  main:
    stage: PRODUCTION
    environmentVariables:
      AMPLIFY_MONOREPO_APP_ROOT: "apps/web"
      AMPLIFY_DIFF_DEPLOY: "false"
    buildSpec: "amplify-build-main.yml"
    enablePullRequestPreview: true
    pullRequestPreviewType: "CUSTOM"
    
  develop:
    stage: BETA
    environmentVariables:
      AMPLIFY_MONOREPO_APP_ROOT: "apps/web"
      AMPLIFY_DIFF_DEPLOY: "true"
    buildSpec: "amplify-build-develop.yml"
    enableAutoBuild: true
    enablePullRequestPreview: true
    
  feature/*:
    stage: DEVELOPMENT
    environmentVariables:
      AMPLIFY_MONOREPO_APP_ROOT: "apps/web"
      AMPLIFY_DIFF_DEPLOY: "true"
    buildSpec: "amplify-build-feature.yml"
    enablePullRequestPreview: true
    pullRequestPreviewSettings:
      previewMode: "CUSTOM"
      branches:
        - "main"
        - "develop"
      stackName: "amplify-preview-${PR_NUMBER}"

# Configurazione per preview automatiche
previewSettings:
  autoBuildPrBranch: true
  previewTimeToLive:
    unit: "DAYS"
    value: 7
  environmentVariables:
    NEXT_PUBLIC_APP_ENV: "preview"
    NEXT_PUBLIC_PREVIEW_ID: "${PULL_REQUEST_ID}"
Custom domain + SSL
yaml
# Amplify Domain Management
domainConfig:
  domainName: "app.example.com"
  subDomain: "www"
  enableAutoSubdomain: false
  autoSubDomainCreationPatterns:
    - "feature/*"
    - "develop"
  autoSubDomainIAMRole: "arn:aws:iam::${AWS_ACCOUNT_ID}:role/AmplifyAutoSubDomainRole"

sslConfig:
  certificateType: "AMPLIFY_MANAGED"
  customCertificateArn: null
  enableSslVerification: true

# Route53 Integration
route53Config:
  hostedZoneId: "${HOSTED_ZONE_ID}"
  manageDnsRecords: true
  ttl: 300
  failover:
    type: "PRIMARY"
    healthCheckId: "${HEALTH_CHECK_ID}"

# CloudFront Distribution per custom domain
cloudFrontConfig:
  defaultCacheBehavior:
    viewerProtocolPolicy: "redirect-to-https"
    allowedMethods:
      - "GET"
      - "HEAD"
      - "OPTIONS"
      - "PUT"
      - "POST"
      - "PATCH"
      - "DELETE"
    cachedMethods:
      - "GET"
      - "HEAD"
      - "OPTIONS"
    compress: true
    minTTL: 0
    defaultTTL: 86400
    maxTTL: 31536000
§ RDS + PRISMA
Setup RDS PostgreSQL
yaml
# CloudFormation Template per RDS PostgreSQL
Resources:
  DBCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      Engine: aurora-postgresql
      EngineVersion: "15.3"
      EngineMode: provisioned
      DatabaseName: "app_production"
      MasterUsername: "admin"
      MasterUserPassword: "{{resolve:secretsmanager:db-credentials:SecretString:password}}"
      BackupRetentionPeriod: 35
      PreferredBackupWindow: "03:00-04:00"
      PreferredMaintenanceWindow: "sun:04:00-sun:05:00"
      StorageEncrypted: true
      KmsKeyId: "arn:aws:kms:${AWS::Region}:${AWS::AccountId}:key/${KMSKeyId}"
      DeletionProtection: true
      EnableHttpEndpoint: true
      ServerlessV2ScalingConfiguration:
        MinCapacity: 0.5
        MaxCapacity: 16
      DBClusterParameterGroupName: "aurora-postgresql15"
      EnableCloudwatchLogsExports:
        - "postgresql"
      Port: 5432
      VpcSecurityGroupIds:
        - !Ref DBSecurityGroup

  DBInstance1:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: aurora-postgresql
      DBInstanceClass: db.serverless
      DBClusterIdentifier: !Ref DBCluster
      PubliclyAccessible: false
      EnablePerformanceInsights: true
      PerformanceInsightsRetentionPeriod: 7

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Security group for RDS"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          SourceSecurityGroupId: !Ref AppSecurityGroup
Connection pooling con PgBouncer
yaml
# PgBouncer su EC2 o RDS Proxy
PgBouncerConfig:
  type: "AWS::RDS::DBProxy"
  properties:
    DBProxyName: "app-db-proxy"
    EngineFamily: "POSTGRESQL"
    IdleClientTimeout: 1800
    RequireTLS: true
    RoleArn: "arn:aws:iam::${AWS::AccountId}:role/rds-proxy-role"
    VpcSecurityGroupIds:
      - !Ref DBSecurityGroup
    VpcSubnetIds:
      - !Ref Subnet1
      - !Ref Subnet2
    Auth:
      - AuthScheme: "SECRETS"
        IAMAuth: "DISABLED"
        SecretArn: "{{resolve:secretsmanager:db-credentials}}"
    DebugLogging: false

# Configurazione PgBouncer (se self-hosted)
pgbouncer.ini:
  [databases]
  appdb = host=${DB_HOST} port=5432 dbname=app_production
  
  [pgbouncer]
  listen_addr = 0.0.0.0
  listen_port = 6432
  auth_type = scram-sha-256
  auth_file = /etc/pgbouncer/userlist.txt
  pool_mode = transaction
  default_pool_size = 20
  max_client_conn = 1000
  server_idle_timeout = 600
  server_lifetime = 3600
  log_connections = 1
  log_disconnections = 1
  stats_period = 60
  admin_users = admin
Prisma connection string per RDS
typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? createPrismaClient();

function createPrismaClient() {
  const connectionString = process.env.DATABASE_URL;
  
  if (!connectionString) {
    throw new Error('DATABASE_URL is not defined');
  }

  // Configurazione per RDS Proxy/PgBouncer
  const url = new URL(connectionString);
  
  // Per connection pooling
  if (process.env.NODE_ENV === 'production') {
    url.port = '6432'; // Porta PgBouncer
    url.searchParams.set('pgbouncer', 'true');
    url.searchParams.set('connection_limit', '10');
    url.searchParams.set('pool_timeout', '10');
  }

  const client = new PrismaClient({
    datasources: {
      db: {
        url: url.toString(),
      },
    },
    log: process.env.NODE_ENV === 'development' 
      ? ['query', 'error', 'warn']
      : ['error'],
    // Ottimizzazioni per produzione
    ...(process.env.NODE_ENV === 'production' && {
      transactionOptions: {
        maxWait: 5000,
        timeout: 10000,
      },
    }),
  });

  // Middleware per logging e metriche
  client.$use(async (params, next) => {
    const start = Date.now();
    const result = await next(params);
    const duration = Date.now() - start;
    
    // Log lente query
    if (duration > 1000) {
      console.warn(`Slow query detected: ${params.model}.${params.action} took ${duration}ms`);
    }
    
    // Metriche CloudWatch
    if (process.env.NODE_ENV === 'production') {
      // Invia metriche a CloudWatch
    }
    
    return result;
  });

  return client;
}

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
Migrations in CI/CD
yaml
# GitHub Actions per migrazioni Prisma
name: Prisma Migrations
on:
  push:
    branches: [main, develop]
    paths:
      - 'prisma/**'
  pull_request:
    branches: [main]

jobs:
  test-migrations:
    runs-on: ubuntu-latest
    env:
      DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Generate Prisma Client
        run: pnpm prisma generate
      
      - name: Check for schema drift
        run: pnpm prisma migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code
      
      - name: Validate migrations
        run: pnpm prisma migrate dev --create-only --name ci-check --skip-seed

  deploy-migrations:
    runs-on: ubuntu-latest
    needs: test-migrations
    if: github.ref == 'refs/heads/main'
    env:
      DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Generate Prisma Client
        run: pnpm prisma generate
      
      - name: Deploy migrations
        run: |
          pnpm prisma migrate deploy
          pnpm prisma db seed
      
      - name: Run data consistency checks
        run: node scripts/validate-migration.js
Backup automatici e point-in-time recovery
yaml
# RDS Automated Backups Configuration
BackupPolicy:
  Type: AWS::RDS::DBCluster
  Properties:
    BackupRetentionPeriod: 35
    PreferredBackupWindow: "01:00-02:00"
    CopyTagsToSnapshot: true
    DeletionProtection: true
    
    # Point-in-Time Recovery
    EnableCloudwatchLogsExports:
      - "postgresql"
    
    # Automated Snapshots
    SnapshotIdentifier: !If 
      - ShouldCreateSnapshot
      - "app-snapshot-${timestamp}"
      - !Ref "AWS::NoValue"

# Lambda per backup personalizzati
BackupLambda:
  Type: AWS::Lambda::Function
  Properties:
    Runtime: nodejs18.x
    Handler: index.handler
    Timeout: 900
    Environment:
      Variables:
        DB_CLUSTER_ARN: !Ref DBCluster
        S3_BACKUP_BUCKET: !Ref BackupBucket
    Policies:
      - Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action:
              - "rds:DescribeDBClusters"
              - "rds:StartExportTask"
            Resource: !Ref DBCluster
          - Effect: Allow
            Action:
              - "s3:PutObject"
              - "s3:GetObject"
            Resource: !Sub "${BackupBucket.Arn}/*"

# EventBridge per backup schedule
BackupSchedule:
  Type: AWS::Events::Rule
  Properties:
    ScheduleExpression: "cron(0 2 ? * SUN *)"
    State: ENABLED
    Targets:
      - Arn: !GetAtt BackupLambda.Arn
        Id: "weekly-backup"

# S3 Lifecycle per retention backup
BackupBucket:
  Type: AWS::S3::Bucket
  Properties:
    LifecycleConfiguration:
      Rules:
        - Id: "TransitionToGlacier"
          Status: "Enabled"
          Transitions:
            - TransitionInDays: 90
              StorageClass: "GLACIER"
        - Id: "ExpireOldBackups"
          Status: "Enabled"
          ExpirationInDays: 365
§ ELASTICACHE REDIS
Session storage
typescript
// lib/redis/session-store.ts
import Redis from 'ioredis';
import { SessionStore } from 'next-auth';

const redisConfig = {
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  tls: process.env.NODE_ENV === 'production' ? {} : undefined,
  retryStrategy: (times: number) => {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
  maxRetriesPerRequest: 3,
};

class RedisSessionStore implements SessionStore {
  private client: Redis;
  
  constructor() {
    this.client = new Redis({
      ...redisConfig,
      keyPrefix: 'session:',
    });
  }
  
  async get(sessionId: string) {
    const data = await this.client.get(sessionId);
    return data ? JSON.parse(data) : null;
  }
  
  async set(sessionId: string, session: any, maxAge?: number) {
    const ttl = maxAge || 30 * 24 * 60 * 60; // 30 giorni default
    await this.client.setex(sessionId, ttl, JSON.stringify(session));
  }
  
  async destroy(sessionId: string) {
    await this.client.del(sessionId);
  }
  
  async touch(sessionId: string, maxAge?: number) {
    const ttl = maxAge || 30 * 24 * 60 * 60;
    await this.client.expire(sessionId, ttl);
    return true;
  }
}

// Configurazione per NextAuth.js
export const authOptions = {
  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60,
  },
  jwt: {
    maxAge: 30 * 24 * 60 * 60,
  },
  adapter: RedisAdapter(redisConfig),
};
Rate limiting distribuito
typescript
// lib/redis/rate-limiter.ts
import Redis from 'ioredis';

export class RateLimiter {
  private redis: Redis;
  private windowMs: number;
  private maxRequests: number;
  
  constructor(
    windowMs: number = 60 * 1000, // 1 minuto
    maxRequests: number = 100
  ) {
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT || '6379'),
      keyPrefix: 'ratelimit:',
    });
    this.windowMs = windowMs;
    this.maxRequests = maxRequests;
  }
  
  async limit(key: string): Promise<{
    success: boolean;
    remaining: number;
    reset: number;
  }> {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // Usa pipeline per performance
    const pipeline = this.redis.pipeline();
    
    // Aggiungi timestamp corrente
    pipeline.zadd(key, now, now.toString());
    
    // Rimuovi timestamp vecchi
    pipeline.zremrangebyscore(key, 0, windowStart);
    
    // Conta richieste nella finestra
    pipeline.zcard(key);
    
    // Imposta expiration sulla key
    pipeline.expire(key, Math.ceil(this.windowMs / 1000));
    
    const results = await pipeline.exec();
    const requestCount = results[2][1] as number;
    
    return {
      success: requestCount <= this.maxRequests,
      remaining: Math.max(0, this.maxRequests - requestCount),
      reset: Math.ceil((now - windowStart) / 1000),
    };
  }
  
  // Rate limiting per IP
  async limitByIp(ip: string, endpoint: string) {
    const key = `ip:${ip}:${endpoint}`;
    return this.limit(key);
  }
  
  // Rate limiting per user ID
  async limitByUserId(userId: string, endpoint: string) {
    const key = `user:${userId}:${endpoint}`;
    return this.limit(key);
  }
}

// Middleware Next.js per rate limiting
export async function rateLimitMiddleware(
  request: Request,
  identifier: string
) {
  const limiter = new RateLimiter();
  const endpoint = new URL(request.url).pathname;
  const result = await limiter.limitByIp(identifier, endpoint);
  
  if (!result.success) {
    throw new Error('Rate limit exceeded');
  }
  
  return {
    'X-RateLimit-Limit': '100',
    'X-RateLimit-Remaining': result.remaining.toString(),
    'X-RateLimit-Reset': result.reset.toString(),
  };
}
Cache invalidation patterns
typescript
// lib/redis/cache-manager.ts
import Redis from 'ioredis';

export class CacheManager {
  private redis: Redis;
  private prefix: string;
  
  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT || '6379'),
      keyPrefix: 'cache:',
    });
    this.prefix = 'app';
  }
  
  private buildKey(...parts: string[]): string {
    return `${this.prefix}:${parts.join(':')}`;
  }
  
  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(this.buildKey(key));
    return data ? JSON.parse(data) : null;
  }
  
  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    if (ttl) {
      await this.redis.setex(this.buildKey(key), ttl, serialized);
    } else {
      await this.redis.set(this.buildKey(key), serialized);
    }
  }
  
  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(this.buildKey(pattern));
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Pattern per invalidazione basata su tags
  async tagInvalidation(tags: string[]): Promise<void> {
    const pipeline = this.redis.pipeline();
    
    tags.forEach(tag => {
      const tagKey = `tag:${tag}`;
      pipeline.smembers(tagKey);
    });
    
    const results = await pipeline.exec();
    const keysToDelete = new Set<string>();
    
    results.forEach(([err, keys]) => {
      if (!err && keys) {
        (keys as string[]).forEach(key => keysToDelete.add(key));
      }
    });
    
    if (keysToDelete.size > 0) {
      await this.redis.del(...Array.from(keysToDelete));
    }
  }
  
  // Invalidation pattern per Next.js ISR
  async invalidateISRPaths(paths: string[]): Promise<void> {
    const pipeline = this.redis.pipeline();
    
    paths.forEach(path => {
      const cacheKey = `next:isr:${path}`;
      pipeline.del(cacheKey);
      
      // Invalida anche le dipendenze
      const dependencyKey = `deps:${path}`;
      pipeline.smembers(dependencyKey);
    });
    
    const results = await pipeline.exec();
    const dependentKeys = new Set<string>();
    
    // Raccoglie tutte le keys dipendenti
    results.slice(paths.length).forEach(([err, keys]) => {
      if (!err && keys) {
        (keys as string[]).forEach(key => dependentKeys.add(key));
      }
    });
    
    // Elimina le dipendenze
    if (dependentKeys.size > 0) {
      await this.redis.del(...Array.from(dependentKeys));
    }
  }
}
Cluster mode vs single node
yaml
# Elasticache Redis Cluster Configuration
ElastiCacheCluster:
  Type: AWS::ElastiCache::ReplicationGroup
  Properties:
    CacheNodeType: cache.r6g.large
    Engine: redis
    EngineVersion: "7.0"
    AutomaticFailoverEnabled: true
    MultiAZEnabled: true
    NumNodeGroups: 3
    ReplicasPerNodeGroup: 1
    SnapshotRetentionLimit: 7
    SnapshotWindow: "05:00-09:00"
    AtRestEncryptionEnabled: true
    TransitEncryptionEnabled: true
    AuthToken: "{{resolve:secretsmanager:redis-auth-token}}"
    CacheParameterGroupName: default.redis7.cluster.on
    
    # Cluster mode specific settings
    ClusterModeEnabled: true
    DataTieringEnabled: false
    
    # Network configuration
    SecurityGroupIds:
      - !Ref CacheSecurityGroup
    CacheSubnetGroupName: !Ref CacheSubnetGroup
    
    # Maintenance
    PreferredMaintenanceWindow: "sun:02:00-sun:04:00"

# Single Node Configuration (per dev/test)
ElastiCacheSingleNode:
  Type: AWS::ElastiCache::CacheCluster
  Properties:
    CacheNodeType: cache.t3.micro
    Engine: redis
    EngineVersion: "7.0"
    NumCacheNodes: 1
    SnapshotRetentionLimit: 1
    PreferredAvailabilityZone: !Select [0, !GetAZs ""]
    
    # No clustering
    AZMode: single-az
    
    # Security
    SecurityGroupIds:
      - !Ref CacheSecurityGroup
    CacheSubnetGroupName: !Ref CacheSubnetGroup

# Decision Matrix per cluster vs single node
DecisionMatrix:
  ClusterMode:
    QuandoUsare:
      - "Produzione con alta disponibilità richiesta"
      - "Carichi di lavoro > 1000 richieste/secondo"
      - "Dataset > 100GB"
      - "Requisiti di auto-scaling"
    Vantaggi:
      - "Auto-failover"
      - "Scalabilità orizzontale"
      - "Sharding automatico"
      - "Read replicas"
    Svantaggi:
      - "Costo 3x"
      - "Setup complesso"
      - "Overhead di gestione"
  
  SingleNode:
    QuandoUsare:
      - "Ambienti di sviluppo/test"
      - "Carichi leggeri (< 100 richieste/secondo)"
      - "Dataset < 10GB"
      - "Budget limitato"
    Vantaggi:
      - "Costo ridotto"
      - "Setup semplice"
      - "Gestione facile"
    Svantaggi:
      - "Single point of failure"
      - "Scalabilità verticale limitata"
      - "No read replicas"
§ SES EMAIL
Domain verification
yaml
# CloudFormation per SES Domain Verification
SESDomain:
  Type: AWS::SES::EmailIdentity
  Properties:
    EmailIdentity: "example.com"
    ConfigurationSetAttributes:
      ConfigurationSetName: !Ref SESConfigurationSet
    DkimSigningAttributes:
      DomainSigningSelector: "ses"
      DomainSigningPrivateKey: "{{resolve:secretsmanager:dkim-private-key}}"
    FeedbackAttributes:
      EmailForwardingEnabled: true

# DNS Records per verification
DNSRecords:
  TXTRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: !Ref HostedZone
      Name: "_amazonses.example.com."
      Type: TXT
      TTL: "1800"
      ResourceRecords:
        - !GetAtt SESDomain.VerificationToken
  
  DKIMRecords:
    Type: AWS::Route53::RecordSetGroup
    Properties:
      HostedZoneId: !Ref HostedZone
      RecordSets:
        - Name: "ses._domainkey.example.com."
          Type: CNAME
          TTL: "1800"
          ResourceRecords:
            - !GetAtt SESDomain.DkimDNSToken1
        - Name: "ses._domainkey.example.com."
          Type: CNAME
          TTL: "1800"
          ResourceRecords:
            - !GetAtt SESDomain.DkimDNSToken2
        - Name: "ses._domainkey.example.com."
          Type: CNAME
          TTL: "1800"
          ResourceRecords:
            - !GetAtt SESDomain.DkimDNSToken3
  
  DMARCRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: !Ref HostedZone
      Name: "_dmarc.example.com."
      Type: TXT
      TTL: "1800"
      ResourceRecords:
        - "v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc-reports@example.com; ruf=mailto:dmarc-forensics@example.com"
  
  SPFRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: !Ref HostedZone
      Name: "example.com."
      Type: TXT
      TTL: "1800"
      ResourceRecords:
        - "v=spf1 include:amazonses.com ~all"
Template email con SES
typescript
// lib/email/templates.ts
import { SESv2Client, SendEmailCommand } from '@aws-sdk/client-sesv2';

const sesClient = new SESv2Client({ region: 'us-east-1' });

export interface EmailTemplate {
  templateName: string;
  subject: string;
  html: string;
  text: string;
}

export const emailTemplates: Record<string, EmailTemplate> = {
  WELCOME: {
    templateName: 'WelcomeTemplate',
    subject: 'Benvenuto in {{appName}}',
    html: `
      <!DOCTYPE html>
      <html>
        <head>
          <meta charset="utf-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
        </head>
        <body>
          <h1>Benvenuto in {{appName}}!</h1>
          <p>Ciao {{name}},</p>
          <p>Grazie per esserti registrato.</p>
          <a href="{{verificationLink}}">Verifica il tuo account</a>
        </body>
      </html>
    `,
    text: `Benvenuto in {{appName}}!\n\nCiao {{name}},\n\nGrazie per esserti registrato.\n\nVerifica il tuo account: {{verificationLink}}`
  },
  
  PASSWORD_RESET: {
    templateName: 'PasswordResetTemplate',
    subject: 'Reset della password per {{appName}}',
    html: `
      <!DOCTYPE html>
      <html>
        <body>
          <h1>Reset Password</h1>
          <p>Clicca il link per resettare la tua password:</p>
          <a href="{{resetLink}}">Reset Password</a>
          <p>Il link scadrà tra {{expiryHours}} ore.</p>
        </body>
      </html>
    `,
    text: `Reset Password\n\nClicca il link per resettare la tua password:\n{{resetLink}}\n\nIl link scadrà tra {{expiryHours}} ore.`
  },
  
  NOTIFICATION: {
    templateName: 'NotificationTemplate',
    subject: '{{notificationTitle}}',
    html: `
      <!DOCTYPE html>
      <html>
        <body>
          <h2>{{notificationTitle}}</h2>
          <p>{{notificationBody}}</p>
          <p><small>Non rispondere a questa email.</small></p>
        </body>
      </html>
    `,
    text: `{{notificationTitle}}\n\n{{notificationBody}}\n\nNon rispondere a questa email.`
  }
};

export async function sendTemplatedEmail(
  templateName: string,
  to: string[],
  templateData: Record<string, string>
) {
  const command = new SendEmailCommand({
    FromEmailAddress: 'noreply@example.com',
    Destination: {
      ToAddresses: to,
    },
    Content: {
      Template: {
        TemplateName: templateName,
        TemplateData: JSON.stringify(templateData),
      },
    },
    ConfigurationSetName: 'email-tracking',
    ListManagementOptions: {
      ContactListName: 'subscribers',
      TopicName: 'general',
    },
  });
  
  return sesClient.send(command);
}

// Lambda per creazione template SES
export async function createSESTemplate(template: EmailTemplate) {
  const createTemplateCommand = new CreateEmailTemplateCommand({
    TemplateName: template.templateName,
    TemplateContent: {
      Subject: template.subject,
      Html: template.html,
      Text: template.text,
    },
  });
  
  await sesClient.send(createTemplateCommand);
}
Bounce/complaint handling
typescript
// lib/email/bounce-handler.ts
import { SNSClient, SubscribeCommand } from '@aws-sdk/client-sns';
import { LambdaClient } from '@aws-sdk/client-lambda';

export class BounceHandler {
  private snsClient: SNSClient;
  private lambdaClient: LambdaClient;
  
  constructor() {
    this.snsClient = new SNSClient({ region: 'us-east-1' });
    this.lambdaClient = new LambdaClient({ region: 'us-east-1' });
  }
  
  async setupNotifications() {
    // Configura SNS per bounce/complaint
    const topicArn = await this.createSNSTopic('ses-bounce-topic');
    
    // Sottoscrivi SES agli eventi
    await this.snsClient.send(new SubscribeCommand({
      TopicArn: topicArn,
      Protocol: 'lambda',
      Endpoint: process.env.BOUNCE_HANDLER_LAMBDA_ARN,
    }));
    
    // Configura SES per inviare notifiche a SNS
    await this.configureSESFeedback(topicArn);
  }
  
  // Lambda handler per processare bounce/complaint
  async handleSESEvent(event: any) {
    for (const record of event.Records) {
      const message = JSON.parse(record.Sns.Message);
      
      if (message.notificationType === 'Bounce') {
        await this.handleBounce(message);
      } else if (message.notificationType === 'Complaint') {
        await this.handleComplaint(message);
      } else if (message.notificationType === 'Delivery') {
        await this.handleDelivery(message);
      }
    }
  }
  
  private async handleBounce(bounce: any) {
    const bounceType = bounce.bounce.bounceType;
    const bounceSubType = bounce.bounce.bounceSubType;
    const emailAddress = bounce.mail.destination[0];
    
    console.log(`Bounce detected: ${bounceType} - ${bounceSubType} for ${emailAddress}`);
    
    // Aggiorna database per bloccare invii futuri
    if (bounceType === 'Permanent') {
      await this.blockEmailAddress(emailAddress, 'permanent_bounce');
    } else if (bounceType === 'Transient') {
      await this.throttleEmailAddress(emailAddress, bounceSubType);
    }
    
    // Invia alert se bounce rate troppo alto
    const bounceRate = await this.calculateBounceRate();
    if (bounceRate > 0.05) { // 5% threshold
      await this.sendBounceAlert(bounceRate);
    }
  }
  
  private async handleComplaint(complaint: any) {
    const emailAddress = complaint.mail.destination[0];
    const feedbackType = complaint.complaint.complaintFeedbackType;
    
    console.log(`Complaint: ${feedbackType} from ${emailAddress}`);
    
    // Blocca immediatamente per complaint
    await this.blockEmailAddress(emailAddress, '

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-02-AWS-DETERMINISTICO
Prompt ID: 2 / 48
Parte: 2
Exported: 2026-02-06T11:36:38.017Z
Characters: 1559
════════════════════════════════════════════════════════════

ement:
      OrStatement:
        Statements:
          - ByteMatchStatement:
              SearchString: ".onion"
              FieldToMatch:
                Headers:
                  MatchPattern:
                    IncludedHeaders: ["Host"]
                  MatchScope: "ALL"
              TextTransformations:
                - Priority: 0
                  Type: "LOWERCASE"
              PositionalConstraint: "CONTAINS"
          - IPSetReferenceStatement:
              Arn: !Ref TORExitNodesIPSet

TORExitNodesIPSet:
  Type: AWS::WAFv2::IPSet
  Properties:
    Name: "tor-exit-nodes"
    Scope: "REGIONAL"
    Description: "Known TOR exit nodes"
    IPAddressVersion: "IPV4"
    Addresses:
      - "1.1.1.1/32"  # Esempio, aggiornare con lista reale
      - "2.2.2.2/32"
Bot control
yaml
WAFBotControl:
  - Name: "AWSManagedBotControl"
    Priority: 50
    Statement:
      ManagedRuleGroupStatement:
        VendorName: "AWS"
        Name: "AWSManagedRulesBotControlRuleSet"
        ManagedRuleGroupConfigs:
          - AWSManagedRulesBotControlRuleSet:
              InspectionLevel: "COMMON"
        ExcludedRules: []
    OverrideAction:
      None: {}
    VisibilityConfig:
      SampledRequestsEnabled: true
      CloudWatchMetricsEnabled: true
      MetricName: "AWSManagedRulesBotControlRuleSet"
  
  - Name: "BotAllowSearchEngine"
    Priority: 51
    Action:
      Allow: {}
    VisibilityConfig:
      SampledRequestsEnabled: true
      CloudWatchMetricsEnabled: true
      MetricName: "BotAllowSearchEngine"
    Statement:

## § ADVANCED PATTERNS: AWS DETERMINISTICO

### Server Actions con Validazione

```typescript
// app/actions/aws-deterministico.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### AWS DETERMINISTICO - Utility Helper #151

```typescript
// lib/utils/aws-deterministico-helper-151.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #466

```typescript
// lib/utils/aws-deterministico-helper-466.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #836

```typescript
// lib/utils/aws-deterministico-helper-836.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #827

```typescript
// lib/utils/aws-deterministico-helper-827.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #727

```typescript
// lib/utils/aws-deterministico-helper-727.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #383

```typescript
// lib/utils/aws-deterministico-helper-383.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #35

```typescript
// lib/utils/aws-deterministico-helper-35.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #730

```typescript
// lib/utils/aws-deterministico-helper-730.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #560

```typescript
// lib/utils/aws-deterministico-helper-560.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #27

```typescript
// lib/utils/aws-deterministico-helper-27.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #432

```typescript
// lib/utils/aws-deterministico-helper-432.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #189

```typescript
// lib/utils/aws-deterministico-helper-189.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #314

```typescript
// lib/utils/aws-deterministico-helper-314.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #196

```typescript
// lib/utils/aws-deterministico-helper-196.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #298

```typescript
// lib/utils/aws-deterministico-helper-298.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #639

```typescript
// lib/utils/aws-deterministico-helper-639.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #173

```typescript
// lib/utils/aws-deterministico-helper-173.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<AWSDETERMINISTICOConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<AWSDETERMINISTICOConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### AWS DETERMINISTICO - Utility Helper #894

```typescript
// lib/utils/aws-deterministico-helper-894.ts
import { z } from "zod";

interface AWSDETERMINISTICOConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class AWSDETERMINISTICOProcessor<TInput, TOutput> {
  private config: AWSDETERMINISTICOConfig;
  private processors: Array<(input: TInput) => Promi