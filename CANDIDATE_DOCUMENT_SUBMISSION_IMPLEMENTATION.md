# Candidate Document Submission Flow - Implementation Summary

## ✅ What Was Implemented

### **1. Fixed Salary Structure Error** ✓
**Issue**: `Cannot read properties of undefined (reading 'designation')` and salary cast error
**Solution**: 
- Fixed `offer.salary` to be an object with `basic`, `hra`, `allowances`, and `total` properties
- Added salary breakdown calculation (60% basic, 40% HRA)
- Added optional chaining and fallback values for all offer details

### **2. Complete Document Submission System** ✓

## 📋 Backend Implementation

### **A. New Models Created**

#### **1. CandidateDocument Model** (`src/models/CandidateDocument.js`)
Stores all candidate-submitted documents with verification tracking:

**Fields:**
- `candidateId` - Unique candidate code reference
- `aadhar` - Aadhar card document with verification status
- `pan` - PAN card document with verification status
- `bankDetails` - Bank information with proof document
- `allDocumentsSubmitted` - Boolean flag
- `allDocumentsVerified` - Boolean flag
- `submissionEmailSent` - Email tracking
- `hrNotificationSent` - HR notification tracking

**Methods:**
- `checkAllDocumentsSubmitted()` - Validates all required documents
- `checkAllDocumentsVerified()` - Checks verification status

### **B. Controllers Created**

#### **1. candidateDocumentController.js** (`src/controllers/candidateDocumentController.js`)

**Functions:**
1. **`validateCandidate`** (Public)
   - Route: `POST /api/candidate-documents/public/validate`
   - Validates candidate ID
   - Returns candidate info and submission status
   - No authentication required

2. **`submitDocuments`** (Public)
   - Route: `POST /api/candidate-documents/public/submit`
   - Accepts file uploads (Aadhar, PAN, Bank Proof)
   - Validates bank details
   - Sends confirmation emails
   - No authentication required

3. **`getCandidateDocuments`** (Protected - HR/Admin)
   - Route: `GET /api/candidate-documents/:candidateCode`
   - Returns all documents for a candidate
   - Includes verification status

4. **`verifyDocument`** (Protected - HR/Admin)
   - Route: `PUT /api/candidate-documents/:candidateCode/verify`
   - Verify or reject individual documents
   - Sends final confirmation when all verified

**Email Functions:**
- `sendCandidateConfirmationEmail()` - Confirms document receipt
- `sendHRNotificationEmail()` - Notifies HR of new submission
- `sendVerificationCompleteEmail()` - Final confirmation to candidate

### **C. Routes Created**

#### **candidateDocumentRoutes.js** (`src/routes/candidateDocumentRoutes.js`)

**Public Routes** (No Auth):
- `POST /api/candidate-documents/public/validate`
- `POST /api/candidate-documents/public/submit`

**Protected Routes** (HR/Admin):
- `GET /api/candidate-documents/:candidateCode`
- `PUT /api/candidate-documents/:candidateCode/verify`

**File Upload Configuration:**
- Multer storage in `uploads/candidate-documents/`
- Accepts: JPEG, JPG, PNG, PDF
- Max file size: 5MB per file
- File naming: `{fieldname}-{timestamp}-{random}.{ext}`

### **D. Updated Controllers**

#### **onboardingController.js** - Enhanced `sendToOnboarding` function

**New Features:**
1. Automatically sends email when candidate moves to onboarding
2. Email includes:
   - Candidate ID (auto-generated if not exists)
   - Document submission link
   - List of required documents
   - Step-by-step instructions

**Email Template:**
- Professional HTML design
- Highlighted Candidate ID section
- Clear call-to-action button
- Mobile-responsive
- Instructions and warnings

## 🔄 Complete Workflow

### **Step 1: Candidate Moves to Onboarding**
```
HR Action: Click "Send to Onboarding" on candidate
↓
System Actions:
1. Creates onboarding record
2. Generates/retrieves Candidate ID (e.g., CAN00001)
3. Sends email to candidate with:
   - Candidate ID
   - Document submission link
   - Instructions
```

### **Step 2: Candidate Receives Email**
```
Email Contains:
- Subject: "Welcome to Onboarding - Submit Your Documents"
- Candidate ID prominently displayed
- Link to document submission portal
- List of required documents:
  * Aadhar Card
  * PAN Card
  * Bank Details
  * Bank Proof
```

### **Step 3: Candidate Submits Documents**
```
1. Candidate clicks link → Opens /candidate-documents page
2. Enters Candidate ID
3. System validates ID
4. Shows document upload form
5. Candidate fills:
   - Uploads Aadhar (PDF/Image)
   - Uploads PAN (PDF/Image)
   - Enters bank details (Name, Account, IFSC)
   - Uploads bank proof (PDF/Image)
6. Submits form
```

### **Step 4: System Processing**
```
Backend Actions:
1. Validates all files and data
2. Stores files in uploads/candidate-documents/
3. Creates/updates CandidateDocument record
4. Sends confirmation email to candidate
5. Sends notification email to HR
```

### **Step 5: HR Reviews Documents**
```
HR Actions in HRMS:
1. Opens onboarding panel
2. Views candidate's submitted documents
3. For each document:
   - Preview/Download
   - Verify or Reject
   - Add rejection reason if needed
4. System tracks verification status
```

### **Step 6: Verification Complete**
```
When all documents verified:
1. System sets allDocumentsVerified = true
2. Sends final confirmation email to candidate
3. Candidate ready for next onboarding steps
```

## 📧 Email Templates

### **1. Onboarding Welcome Email**
**Sent**: When candidate moves to onboarding
**To**: Candidate
**Contains**:
- Welcome message
- Candidate ID (large, highlighted)
- Document submission link (CTA button)
- Required documents list
- Step-by-step instructions
- Important warnings

### **2. Document Submission Confirmation**
**Sent**: After candidate submits documents
**To**: Candidate
**Contains**:
- Confirmation message
- Candidate ID
- Submission timestamp
- Documents submitted checklist
- Next steps information

### **3. HR Notification Email**
**Sent**: After candidate submits documents
**To**: HR team
**Contains**:
- Candidate information
- Candidate ID
- Submission timestamp
- Documents submitted list
- Link to HRMS onboarding panel

### **4. Verification Complete Email**
**Sent**: When all documents verified
**To**: Candidate
**Contains**:
- Success message
- Verification status
- Next steps information

## 🗂️ Database Schema

### **CandidateDocument Collection**
```javascript
{
  candidateId: "CAN00001",
  aadhar: {
    documentUrl: "/uploads/candidate-documents/aadhar-123456.pdf",
    documentName: "aadhar-123456.pdf",
    uploadedAt: Date,
    verified: false,
    verifiedBy: ObjectId,
    verifiedAt: Date,
    rejectionReason: String
  },
  pan: {
    // Same structure as aadhar
  },
  bankDetails: {
    bankName: "HDFC Bank",
    accountNumber: "1234567890",
    ifscCode: "HDFC0001234",
    accountHolderName: "John Doe",
    proofDocumentUrl: "/uploads/candidate-documents/bank-123456.pdf",
    proofDocumentName: "bank-123456.pdf",
    uploadedAt: Date,
    verified: false,
    verifiedBy: ObjectId,
    verifiedAt: Date,
    rejectionReason: String
  },
  allDocumentsSubmitted: true,
  allDocumentsVerified: false,
  submittedAt: Date,
  submissionEmailSent: true,
  hrNotificationSent: true,
  createdAt: Date,
  updatedAt: Date
}
```

## 🔐 Security Features

1. **Public Routes** - No authentication for candidate submission
2. **Protected Routes** - HR/Admin only for viewing and verification
3. **File Validation** - Only PDF and images allowed
4. **File Size Limit** - 5MB per file
5. **Candidate ID Validation** - Must exist and be in onboarding stage
6. **Duplicate Prevention** - Updates existing record if resubmitting
7. **Error Handling** - Graceful failures with user-friendly messages

## 📁 Files Created/Modified

### **Created Files:**
1. `hrms-backend/src/models/CandidateDocument.js`
2. `hrms-backend/src/controllers/candidateDocumentController.js`
3. `hrms-backend/src/routes/candidateDocumentRoutes.js`
4. `hrms-backend/uploads/candidate-documents/` (directory)

### **Modified Files:**
1. `hrms-backend/src/app.js` - Added route registration
2. `hrms-backend/src/controllers/onboardingController.js` - Added email sending logic
3. `hrms-backend/src/controllers/onboardingController.js` - Fixed salary structure

## 🎯 API Endpoints Summary

### **Public Endpoints:**
```
POST /api/candidate-documents/public/validate
Body: { candidateCode: "CAN00001" }
Response: { success, data: { candidateCode, name, email, documentsSubmitted } }

POST /api/candidate-documents/public/submit
Body: FormData with files and bankDetails
Response: { success, message, data: { candidateCode, submittedAt } }
```

### **Protected Endpoints:**
```
GET /api/candidate-documents/:candidateCode
Headers: Authorization: Bearer {token}
Response: { success, data: CandidateDocument }

PUT /api/candidate-documents/:candidateCode/verify
Headers: Authorization: Bearer {token}
Body: { documentType: "aadhar", verified: true, rejectionReason: "" }
Response: { success, message, data: CandidateDocument }
```

## ✅ Testing Checklist

### **Backend Testing:**
- [ ] Candidate ID generation works
- [ ] Email sends when moving to onboarding
- [ ] Public validation endpoint works
- [ ] File upload works (Aadhar, PAN, Bank Proof)
- [ ] Bank details validation works
- [ ] Confirmation emails send
- [ ] HR notification emails send
- [ ] Document retrieval works for HR
- [ ] Verification/rejection works
- [ ] Final confirmation email sends

### **Integration Testing:**
- [ ] Complete flow from onboarding to submission
- [ ] Email delivery confirmed
- [ ] Files stored correctly
- [ ] Database records created
- [ ] HR can view documents
- [ ] Verification updates status

## 🚀 Next Steps (Frontend)

### **To Complete:**
1. Create `/candidate-documents` page (public)
   - Candidate ID input
   - Validation
   - Document upload form
   - Bank details form
   - Submit button
   - Success/error messages

2. Integrate document display in onboarding panel
   - Add "Documents" section
   - Show submitted documents
   - Preview/download buttons
   - Verify/reject buttons
   - Status badges

## 📝 Environment Variables Required

```env
# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=noreply@hrms.com
HR_EMAIL=hr@company.com

# Frontend URL
FRONTEND_URL=http://localhost:3000
```

## 🎉 Summary

### **Completed:**
✅ Fixed salary structure error in sendOffer
✅ Created CandidateDocument model
✅ Candidate ID auto-generation (already existed)
✅ Public document submission API
✅ File upload with multer
✅ Email notifications (4 types)
✅ Document verification system
✅ Complete backend workflow

### **Pending:**
⏳ Frontend candidate document submission page
⏳ Frontend document display in onboarding panel

### **Key Benefits:**
- ✅ Automated candidate ID generation
- ✅ Email-based document submission link
- ✅ No authentication needed for candidates
- ✅ Secure file storage
- ✅ Complete verification workflow
- ✅ Email notifications at every step
- ✅ HR can review and verify documents
- ✅ Audit trail for all actions

---

## 🔧 Quick Start

### **1. Start Backend:**
```bash
cd hrms-backend
npm run dev
```

### **2. Test Email Sending:**
- Move a candidate to onboarding
- Check candidate's email for welcome message
- Verify Candidate ID is included

### **3. Test Document Submission (via API):**
```bash
# Validate candidate
curl -X POST http://localhost:5000/api/candidate-documents/public/validate \
  -H "Content-Type: application/json" \
  -d '{"candidateCode":"CAN00001"}'

# Submit documents (use Postman for file uploads)
```

### **4. Test HR Verification:**
```bash
# Get documents (requires auth token)
curl -X GET http://localhost:5000/api/candidate-documents/CAN00001 \
  -H "Authorization: Bearer {token}"

# Verify document
curl -X PUT http://localhost:5000/api/candidate-documents/CAN00001/verify \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{"documentType":"aadhar","verified":true}'
```

---

**Implementation Status**: Backend Complete ✅ | Frontend Pending ⏳
