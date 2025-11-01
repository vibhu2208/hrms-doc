# Bulk Employee Upload Feature Guide

## Overview
The HRMS system now supports bulk employee upload via CSV, Excel, and Google Sheets. This feature allows administrators and HR personnel to import multiple employee records at once, with comprehensive validation and error handling.

## Features

### 1. Multiple Upload Methods
- **CSV Upload**: Upload employee data from CSV files
- **Excel Upload**: Support for .xlsx and .xls files
- **Google Sheets**: Direct integration with Google Sheets (public or authenticated)

### 2. Data Validation
- Required field validation (firstName, lastName, email, phone, department, designation, joiningDate)
- Email format validation
- Phone number format validation (10-15 digits)
- Date format validation
- Enum validation (gender, blood group, marital status, employment type, status)
- Duplicate email detection
- Duplicate employee code detection
- Department existence validation

### 3. Preview & Confirmation
- Visual preview of parsed data
- Validation status for each record
- Detailed error messages for invalid records
- Summary statistics (total, valid, invalid)

### 4. Progress Tracking
- Real-time upload progress indicator
- Success/error toast notifications
- Detailed upload results

## How to Use

### Step 1: Download Template
1. Navigate to **Employees** → **Bulk Upload**
2. Click **Download Template** button
3. A sample CSV file will be downloaded with all required columns

### Step 2: Prepare Your Data
Fill in the template with your employee data. Here are the available fields:

#### Required Fields
- `firstName` - Employee's first name
- `lastName` - Employee's last name
- `email` - Employee's email address (must be unique)
- `phone` - Contact number (10-15 digits)
- `department` - Department name or ID
- `designation` - Job title/position
- `joiningDate` - Date of joining (YYYY-MM-DD format)

#### Optional Fields
- `employeeCode` - Custom employee code (auto-generated if not provided)
- `dateOfBirth` - Date of birth (YYYY-MM-DD format)
- `gender` - male, female, or other
- `bloodGroup` - A+, A-, B+, B-, AB+, AB-, O+, O-
- `maritalStatus` - single, married, divorced, widowed
- `alternatePhone` - Alternative contact number
- `employmentType` - full-time, part-time, contract, intern
- `status` - active, inactive, terminated, on-leave

#### Address Fields
- `street` - Street address
- `city` - City
- `state` - State/Province
- `zipCode` - ZIP/Postal code
- `country` - Country

#### Salary Fields
- `basicSalary` - Basic salary amount
- `hra` - House Rent Allowance
- `allowances` - Other allowances
- `deductions` - Deductions

#### Bank Details
- `accountNumber` - Bank account number
- `bankName` - Name of the bank
- `ifscCode` - IFSC code
- `accountHolderName` - Account holder name
- `branch` - Bank branch

#### Emergency Contact
- `emergencyContactName` - Emergency contact person name
- `emergencyContactRelationship` - Relationship
- `emergencyContactPhone` - Emergency contact phone

### Step 3: Upload File

#### Option A: CSV/Excel Upload
1. Click on the **Upload File** tab
2. Click the upload area or drag and drop your file
3. The system will automatically parse the file
4. Review the parsed data in the preview table

#### Option B: Google Sheets
1. Click on the **Google Sheets** tab
2. Paste your Google Sheets URL
3. Click **Fetch Data**
4. The system will retrieve data from the sheet

**Note**: For private Google Sheets, you need to:
- Make the sheet publicly accessible, OR
- Configure Google OAuth credentials (see Setup section below)

### Step 4: Validate Data
1. Click the **Validate** button
2. Review validation results:
   - Green checkmark: Valid record
   - Red X: Invalid record with error details
3. Fix any validation errors in your source file if needed

### Step 5: Upload
1. Once validation is successful, click **Upload Employees**
2. Monitor the progress bar
3. Wait for the success message
4. You'll be redirected to the employee list

## Google Sheets Integration Setup

### For Public Sheets (Simple)
1. Open your Google Sheet
2. Click **Share** → **Get link**
3. Set to "Anyone with the link can view"
4. Copy the URL and paste it in the bulk upload page

### For Private Sheets (OAuth)
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable **Google Sheets API**
4. Create OAuth 2.0 credentials:
   - Application type: Web application
   - Authorized redirect URIs: `http://localhost:5000/api/employees/google-sheets/callback`
5. Copy Client ID and Client Secret
6. Add to backend `.env` file:
   ```
   GOOGLE_CLIENT_ID=your_client_id_here
   GOOGLE_CLIENT_SECRET=your_client_secret_here
   GOOGLE_REDIRECT_URI=http://localhost:5000/api/employees/google-sheets/callback
   ```
7. Restart the backend server

## Data Format Examples

### CSV Format
```csv
firstName,lastName,email,phone,department,designation,joiningDate,employmentType,status
John,Doe,john.doe@example.com,1234567890,Engineering,Software Engineer,2024-01-01,full-time,active
Jane,Smith,jane.smith@example.com,9876543210,HR,HR Manager,2024-01-15,full-time,active
```

### Excel Format
Same structure as CSV, but in .xlsx or .xls format with proper column headers.

### Google Sheets Format
Create a sheet with the same column structure. First row should contain headers matching the field names.

## Validation Rules

### Email
- Must be a valid email format
- Must be unique across all employees
- Example: `john.doe@example.com`

### Phone
- Must contain 10-15 digits
- Spaces, hyphens, and parentheses are allowed
- Examples: `1234567890`, `123-456-7890`, `(123) 456-7890`

### Date Fields
- Format: `YYYY-MM-DD`
- Examples: `2024-01-01`, `1990-05-15`

### Department
- Can be department name (case-insensitive) or department ID
- Must exist in the system
- Create departments first if needed

### Enums
- **Gender**: male, female, other
- **Blood Group**: A+, A-, B+, B-, AB+, AB-, O+, O-
- **Marital Status**: single, married, divorced, widowed
- **Employment Type**: full-time, part-time, contract, intern
- **Status**: active, inactive, terminated, on-leave

## Error Handling

### Common Errors and Solutions

1. **"Email already exists"**
   - Solution: Check for duplicate emails in your file and database

2. **"Department not found"**
   - Solution: Create the department first or use exact department name

3. **"Invalid email format"**
   - Solution: Ensure email follows format: name@domain.com

4. **"Invalid phone format"**
   - Solution: Use 10-15 digit phone numbers

5. **"Invalid date format"**
   - Solution: Use YYYY-MM-DD format (e.g., 2024-01-01)

6. **"Google Sheets permission denied"**
   - Solution: Make sheet public or configure OAuth

## Best Practices

1. **Start Small**: Test with 5-10 records first
2. **Validate Departments**: Ensure all departments exist before upload
3. **Clean Data**: Remove empty rows and columns
4. **Consistent Format**: Use the same date and phone formats throughout
5. **Backup**: Keep a backup of your original data
6. **Review Preview**: Always review the preview before uploading
7. **Fix Errors**: Address all validation errors before upload

## API Endpoints

### Validate Bulk Data
```
POST /api/employees/bulk/validate
Body: { employees: [...] }
```

### Create Bulk Employees
```
POST /api/employees/bulk/create
Body: { employees: [...] }
```

### Get Template
```
GET /api/employees/bulk/template
```

### Fetch Google Sheets Data
```
POST /api/employees/google-sheets/fetch
Body: { spreadsheetId, range, accessToken }
```

## Troubleshooting

### Upload Not Working
- Check internet connection
- Verify file format (CSV, XLSX, XLS)
- Ensure file is not corrupted
- Check browser console for errors

### Validation Failing
- Review error messages carefully
- Check required fields are filled
- Verify data formats match requirements
- Ensure departments exist in system

### Google Sheets Not Loading
- Verify sheet URL is correct
- Check sheet permissions
- Ensure Google Sheets API is enabled (for OAuth)
- Check backend logs for detailed errors

## Support

For additional help or to report issues:
1. Check validation error messages
2. Review this guide
3. Contact system administrator
4. Check backend logs for detailed error information

## Version History

- **v1.0.0** - Initial release with CSV, Excel, and Google Sheets support
  - Comprehensive validation
  - Preview and confirmation
  - Progress tracking
  - Error handling
