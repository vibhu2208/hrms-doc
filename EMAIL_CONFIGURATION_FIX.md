# Email Configuration Fix

## Issue
```
TypeError: nodemailer.createTransporter is not a function
```

## Root Cause
The email transporter was being created at module load time (top-level), which caused the application to crash if:
1. Email configuration was missing
2. There was any issue with nodemailer initialization

## Solution
Changed from **module-level transporter** to **function-level transporter creation**:

### Before (❌ Problematic):
```javascript
// This runs when module loads - crashes if config missing
const transporter = nodemailer.createTransporter({
  host: process.env.SMTP_HOST,
  port: process.env.SMTP_PORT,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
});
```

### After (✅ Fixed):
```javascript
// This runs only when needed - gracefully handles missing config
const createTransporter = () => {
  if (!process.env.EMAIL_USER || !process.env.EMAIL_APP_PASSWORD) {
    console.warn('Email configuration missing. Using SMTP fallback.');
    if (process.env.SMTP_USER && process.env.SMTP_PASS) {
      return nodemailer.createTransport({
        host: process.env.SMTP_HOST || 'smtp.gmail.com',
        port: process.env.SMTP_PORT || 587,
        secure: false,
        auth: {
          user: process.env.SMTP_USER,
          pass: process.env.SMTP_PASS
        }
      });
    }
    return null;
  }

  return nodemailer.createTransport({
    service: 'gmail',
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_APP_PASSWORD
    }
  });
};
```

## Files Updated

### 1. `candidateDocumentController.js`
- Changed from module-level `transporter` to `createTransporter()` function
- Added null checks in all email sending functions:
  - `sendCandidateConfirmationEmail()`
  - `sendHRNotificationEmail()`
  - `sendVerificationCompleteEmail()`

### 2. `onboardingController.js`
- Changed from module-level `transporter` to `createTransporter()` function
- Added null check in `sendOnboardingDocumentEmail()`

## Benefits

1. **Graceful Degradation**: App starts even if email config is missing
2. **Better Error Handling**: Logs warnings instead of crashing
3. **Dual Configuration Support**: 
   - Primary: Gmail (`EMAIL_USER`, `EMAIL_APP_PASSWORD`)
   - Fallback: SMTP (`SMTP_USER`, `SMTP_PASS`, `SMTP_HOST`, `SMTP_PORT`)
4. **Consistent Pattern**: Matches existing `emailService.js` implementation

## Environment Variables

### Option 1: Gmail (Recommended)
```env
EMAIL_USER=your-email@gmail.com
EMAIL_APP_PASSWORD=your-app-specific-password
```

### Option 2: SMTP Fallback
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-password
SMTP_FROM=noreply@hrms.com
```

### Additional
```env
HR_EMAIL=hr@company.com
FRONTEND_URL=http://localhost:3000
```

## Testing

### 1. Start Server
```bash
cd hrms-backend
npm run dev
```

Server should start successfully even without email configuration.

### 2. Test Email Sending
With proper email configuration, test:
- Move candidate to onboarding → Email should send
- Submit documents → Confirmation emails should send
- Verify documents → Verification email should send

### 3. Without Email Config
Without email configuration:
- Server starts normally ✅
- Email functions log warnings ⚠️
- Other functionality works normally ✅

## Error Messages

### With Fix:
```
Email configuration missing. Using SMTP fallback.
Email transporter not configured, skipping onboarding document email
```

### Before Fix:
```
TypeError: nodemailer.createTransporter is not a function
[Application crashes]
```

## Related Files
- `src/controllers/candidateDocumentController.js`
- `src/controllers/onboardingController.js`
- `src/services/emailService.js` (reference implementation)

---

**Status**: ✅ Fixed
**Impact**: Critical - Prevents application startup failure
**Priority**: High
