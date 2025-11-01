# 🚀 Quick Start: Your First Microservice

## Step-by-Step Guide to Extract Notification Service

This guide will help you extract the **Notification Service** as your first microservice. We chose this because it's:
- ✅ Low risk (non-critical path)
- ✅ Clear boundaries (email sending only)
- ✅ Easy to test
- ✅ Good learning experience

---

## 📋 Prerequisites

### **1. Install Required Tools**

```bash
# Docker
https://docs.docker.com/get-docker/

# Kubernetes (Docker Desktop includes it)
# OR Minikube
https://minikube.sigs.k8s.io/docs/start/

# kubectl
https://kubernetes.io/docs/tasks/tools/

# Node.js 18+
https://nodejs.org/

# RabbitMQ (via Docker)
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

### **2. Verify Installations**

```bash
docker --version
kubectl version --client
node --version
npm --version
```

---

## 🏗️ Phase 1: Create Notification Service (Week 1)

### **Step 1: Create Service Directory**

```bash
cd c:\Users\krish\Desktop\techthrive-hrms
mkdir hrms-notification-service
cd hrms-notification-service
npm init -y
```

### **Step 2: Install Dependencies**

```bash
npm install express cors dotenv nodemailer amqplib winston
npm install --save-dev nodemon jest supertest
```

### **Step 3: Create Project Structure**

```bash
mkdir -p src/{config,controllers,services,routes,middlewares,utils}
mkdir -p tests/{unit,integration}
mkdir k8s
```

### **Step 4: Copy Email Service**

Copy your existing `emailService.js` to the new service:

```bash
# From your monolith
copy ..\hrms-backend\src\services\emailService.js src\services\
```

### **Step 5: Create Configuration Files**

**File: `src/config/database.js`** (if needed for logging)
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ MongoDB connection error:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

**File: `src/config/rabbitmq.js`**
```javascript
const amqp = require('amqplib');
const logger = require('../utils/logger');

let connection = null;
let channel = null;

const connectRabbitMQ = async () => {
  try {
    connection = await amqp.connect(process.env.RABBITMQ_URL || 'amqp://localhost');
    channel = await connection.createChannel();
    
    // Declare queues
    await channel.assertQueue('email_queue', { durable: true });
    await channel.assertQueue('email_dlq', { durable: true });
    
    logger.info('✅ RabbitMQ connected');
    return channel;
  } catch (error) {
    logger.error('❌ RabbitMQ connection error:', error);
    setTimeout(connectRabbitMQ, 5000); // Retry after 5 seconds
  }
};

const getChannel = () => {
  if (!channel) {
    throw new Error('RabbitMQ channel not initialized');
  }
  return channel;
};

module.exports = { connectRabbitMQ, getChannel };
```

**File: `src/utils/logger.js`**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'notification-service' },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    ),
  }));
}

module.exports = logger;
```

**File: `src/services/messageConsumer.js`**
```javascript
const { getChannel } = require('../config/rabbitmq');
const emailService = require('./emailService');
const logger = require('../utils/logger');

const consumeEmailQueue = async () => {
  try {
    const channel = getChannel();
    
    logger.info('📥 Starting email queue consumer...');
    
    await channel.consume('email_queue', async (msg) => {
      if (msg !== null) {
        try {
          const emailData = JSON.parse(msg.content.toString());
          logger.info(`Processing email: ${emailData.type}`);
          
          // Route to appropriate email function
          switch (emailData.type) {
            case 'onboarding':
              await emailService.sendOnboardingEmail(emailData.data);
              break;
            case 'interview':
              await emailService.sendInterviewNotification(emailData.data);
              break;
            case 'application_received':
              await emailService.sendApplicationReceivedEmail(emailData.data);
              break;
            case 'shortlisted':
              await emailService.sendShortlistedEmail(emailData.data);
              break;
            case 'interview_completed':
              await emailService.sendInterviewCompletedEmail(emailData.data);
              break;
            case 'offer_extended':
              await emailService.sendOfferExtendedEmail(emailData.data);
              break;
            case 'rejection':
              await emailService.sendRejectionEmail(emailData.data);
              break;
            case 'hr_notification':
              await emailService.sendHRNotification(emailData.data);
              break;
            default:
              logger.warn(`Unknown email type: ${emailData.type}`);
          }
          
          // Acknowledge message
          channel.ack(msg);
          logger.info(`✅ Email sent successfully: ${emailData.type}`);
          
        } catch (error) {
          logger.error('❌ Error processing email:', error);
          
          // Send to dead letter queue
          channel.nack(msg, false, false);
          
          // Publish to DLQ
          channel.sendToQueue('email_dlq', msg.content, {
            persistent: true,
            headers: {
              'x-error': error.message,
              'x-original-queue': 'email_queue'
            }
          });
        }
      }
    }, { noAck: false });
    
    logger.info('✅ Email queue consumer started');
    
  } catch (error) {
    logger.error('❌ Error starting consumer:', error);
    throw error;
  }
};

module.exports = { consumeEmailQueue };
```

**File: `src/routes/healthRoutes.js`**
```javascript
const express = require('express');
const router = express.Router();
const emailService = require('../services/emailService');

// Health check
router.get('/health', (req, res) => {
  res.status(200).json({
    success: true,
    service: 'notification-service',
    status: 'healthy',
    timestamp: new Date().toISOString()
  });
});

// Readiness check
router.get('/ready', async (req, res) => {
  try {
    // Check email configuration
    const emailReady = await emailService.verifyEmailConfig();
    
    if (emailReady) {
      res.status(200).json({
        success: true,
        service: 'notification-service',
        status: 'ready',
        checks: {
          email: 'ok',
          rabbitmq: 'ok'
        }
      });
    } else {
      res.status(503).json({
        success: false,
        service: 'notification-service',
        status: 'not ready',
        checks: {
          email: 'failed'
        }
      });
    }
  } catch (error) {
    res.status(503).json({
      success: false,
      service: 'notification-service',
      status: 'not ready',
      error: error.message
    });
  }
});

module.exports = router;
```

**File: `src/routes/emailRoutes.js`** (REST API for testing)
```javascript
const express = require('express');
const router = express.Router();
const { getChannel } = require('../config/rabbitmq');
const logger = require('../utils/logger');

// Publish email to queue (REST endpoint for testing)
router.post('/send', async (req, res) => {
  try {
    const { type, data } = req.body;
    
    if (!type || !data) {
      return res.status(400).json({
        success: false,
        message: 'Missing required fields: type, data'
      });
    }
    
    const channel = getChannel();
    const message = JSON.stringify({ type, data });
    
    channel.sendToQueue('email_queue', Buffer.from(message), {
      persistent: true
    });
    
    logger.info(`📤 Email queued: ${type}`);
    
    res.status(202).json({
      success: true,
      message: 'Email queued for processing',
      type
    });
    
  } catch (error) {
    logger.error('❌ Error queuing email:', error);
    res.status(500).json({
      success: false,
      message: 'Failed to queue email',
      error: error.message
    });
  }
});

module.exports = router;
```

**File: `src/app.js`**
```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const { connectRabbitMQ } = require('./config/rabbitmq');
const { consumeEmailQueue } = require('./services/messageConsumer');
const healthRoutes = require('./routes/healthRoutes');
const emailRoutes = require('./routes/emailRoutes');
const logger = require('./utils/logger');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/', healthRoutes);
app.use('/api/email', emailRoutes);

// Error handler
app.use((err, req, res, next) => {
  logger.error('Unhandled error:', err);
  res.status(500).json({
    success: false,
    message: 'Internal server error',
    error: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
});

// Initialize
const startServer = async () => {
  try {
    // Connect to RabbitMQ
    await connectRabbitMQ();
    
    // Start consuming messages
    await consumeEmailQueue();
    
    // Start HTTP server
    const PORT = process.env.PORT || 3004;
    app.listen(PORT, () => {
      logger.info(`🚀 Notification Service running on port ${PORT}`);
      logger.info(`📧 Email queue consumer active`);
    });
    
  } catch (error) {
    logger.error('❌ Failed to start service:', error);
    process.exit(1);
  }
};

startServer();

module.exports = app;
```

**File: `.env.example`**
```env
# Server
PORT=3004
NODE_ENV=development

# Email Configuration (Gmail)
EMAIL_USER=your-email@gmail.com
EMAIL_APP_PASSWORD=your-app-password

# OR Custom SMTP
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your-email@example.com
SMTP_PASS=your-password

# RabbitMQ
RABBITMQ_URL=amqp://localhost:5672

# Logging
LOG_LEVEL=info

# Frontend URL (for email links)
FRONTEND_URL=http://localhost:5173
```

**File: `package.json`** (update scripts)
```json
{
  "name": "hrms-notification-service",
  "version": "1.0.0",
  "description": "HRMS Notification Microservice",
  "main": "src/app.js",
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/"
  },
  "keywords": ["hrms", "notification", "microservice"],
  "author": "",
  "license": "MIT"
}
```

**File: `Dockerfile`**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Create logs directory
RUN mkdir -p logs

# Expose port
EXPOSE 3004

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3004/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start service
CMD ["node", "src/app.js"]
```

**File: `.dockerignore`**
```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
tests
logs
*.md
```

**File: `k8s/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notification-service
  labels:
    app: notification-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: notification-service
  template:
    metadata:
      labels:
        app: notification-service
    spec:
      containers:
      - name: notification-service
        image: your-registry/hrms-notification-service:latest
        ports:
        - containerPort: 3004
        env:
        - name: PORT
          value: "3004"
        - name: NODE_ENV
          value: "production"
        - name: RABBITMQ_URL
          valueFrom:
            secretKeyRef:
              name: hrms-secrets
              key: rabbitmq-url
        - name: EMAIL_USER
          valueFrom:
            secretKeyRef:
              name: hrms-secrets
              key: email-user
        - name: EMAIL_APP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: hrms-secrets
              key: email-password
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3004
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3004
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: notification-service
spec:
  selector:
    app: notification-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3004
  type: ClusterIP
```

---

## 🧪 Phase 2: Test the Service (Week 1)

### **Step 1: Start RabbitMQ**

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

Access RabbitMQ Management UI: http://localhost:15672 (guest/guest)

### **Step 2: Configure Environment**

```bash
cp .env.example .env
# Edit .env with your email credentials
```

### **Step 3: Start the Service**

```bash
npm run dev
```

### **Step 4: Test Health Endpoints**

```bash
# Health check
curl http://localhost:3004/health

# Readiness check
curl http://localhost:3004/ready
```

### **Step 5: Test Email Sending**

```bash
# Send test email via REST API
curl -X POST http://localhost:3004/api/email/send \
  -H "Content-Type: application/json" \
  -d '{
    "type": "onboarding",
    "data": {
      "employeeName": "John Doe",
      "employeeEmail": "john@example.com",
      "employeeId": "EMP001",
      "tempPassword": "TempPass123!",
      "companyName": "TechThrive"
    }
  }'
```

### **Step 6: Monitor RabbitMQ**

Check the RabbitMQ UI to see messages being processed.

---

## 🔗 Phase 3: Integrate with Monolith (Week 2)

### **Step 1: Install RabbitMQ Client in Monolith**

```bash
cd ../hrms-backend
npm install amqplib
```

### **Step 2: Create Message Publisher**

**File: `hrms-backend/src/utils/messagePublisher.js`**
```javascript
const amqp = require('amqplib');

let connection = null;
let channel = null;

const connectQueue = async () => {
  if (channel) return channel;
  
  try {
    connection = await amqp.connect(process.env.RABBITMQ_URL || 'amqp://localhost');
    channel = await connection.createChannel();
    await channel.assertQueue('email_queue', { durable: true });
    console.log('✅ Connected to RabbitMQ');
    return channel;
  } catch (error) {
    console.error('❌ RabbitMQ connection error:', error);
    throw error;
  }
};

const publishEmail = async (type, data) => {
  try {
    const ch = await connectQueue();
    const message = JSON.stringify({ type, data });
    
    ch.sendToQueue('email_queue', Buffer.from(message), {
      persistent: true
    });
    
    console.log(`📤 Email queued: ${type}`);
    return { success: true };
  } catch (error) {
    console.error('❌ Failed to queue email:', error);
    // Fallback to direct email sending
    return { success: false, error: error.message };
  }
};

module.exports = { publishEmail };
```

### **Step 3: Update Onboarding Controller**

**File: `hrms-backend/src/controllers/onboardingController.js`**

Find the onboarding email call and replace:

```javascript
// OLD CODE (Direct call)
const emailService = require('../services/emailService');
await emailService.sendOnboardingEmail({...});

// NEW CODE (Queue-based)
const { publishEmail } = require('../utils/messagePublisher');
await publishEmail('onboarding', {
  employeeName,
  employeeEmail,
  employeeId,
  tempPassword,
  companyName: 'TechThrive'
});
```

### **Step 4: Test Integration**

1. Start RabbitMQ
2. Start Notification Service
3. Start Monolith
4. Create a new employee
5. Verify email is sent

---

## 📦 Phase 4: Containerize & Deploy (Week 2)

### **Step 1: Build Docker Image**

```bash
cd hrms-notification-service
docker build -t hrms-notification-service:v1.0 .
```

### **Step 2: Test Docker Container**

```bash
docker run -d \
  --name notification-service \
  -p 3004:3004 \
  --env-file .env \
  hrms-notification-service:v1.0
```

### **Step 3: Deploy to Kubernetes**

```bash
# Create secrets
kubectl create secret generic hrms-secrets \
  --from-literal=rabbitmq-url=amqp://rabbitmq:5672 \
  --from-literal=email-user=your-email@gmail.com \
  --from-literal=email-password=your-app-password

# Deploy service
kubectl apply -f k8s/deployment.yaml

# Check status
kubectl get pods
kubectl logs -f deployment/notification-service
```

---

## ✅ Success Checklist

- [ ] Notification service runs independently
- [ ] RabbitMQ processes messages
- [ ] Emails are sent successfully
- [ ] Monolith publishes to queue
- [ ] Health checks pass
- [ ] Docker image builds
- [ ] Kubernetes deployment works
- [ ] Monitoring is active
- [ ] Logs are structured

---

## 🎯 Next Steps

After successfully extracting the Notification Service:

1. **Extract Auth Service** (Week 3-4)
2. **Extract Employee Service** (Week 5-6)
3. **Continue with remaining services**

---

## 📚 Resources

- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [Microservices Patterns](https://microservices.io/patterns/)

---

**Need Help?** Review the main migration plan or reach out to your team!
