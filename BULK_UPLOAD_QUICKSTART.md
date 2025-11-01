# Bulk Employee Upload - Quick Start Guide

## 🚀 Getting Started in 5 Minutes

### Step 1: Access the Feature
1. Start your HRMS application
2. Login as Admin or HR user
3. Navigate to **Employees** page
4. Click the **Bulk Upload** button (available on both the Employee List and Add Employee pages)

### Step 2: Download Template
1. On the Bulk Upload page, click **Download Template**
2. A CSV file will be downloaded with sample data
3. Open the file in Excel, Google Sheets, or any spreadsheet application

### Step 3: Prepare Your Data

#### Minimum Required Fields
Fill in these columns for each employee:
- `firstName` - e.g., John
- `lastName` - e.g., Doe
- `email` - e.g., john.doe@example.com (must be unique)
- `phone` - e.g., 1234567890 (10-15 digits)
- `department` - e.g., Engineering (must exist in system)
- `designation` - e.g., Software Engineer
- `joiningDate` - e.g., 2024-01-01 (YYYY-MM-DD format)

#### Example Row
```csv
firstName,lastName,email,phone,department,designation,joiningDate
John,Doe,john.doe@example.com,1234567890,Engineering,Software Engineer,2024-01-01
```

### Step 4: Upload Your File

#### Option A: CSV/Excel Upload
1. Click on the **Upload File** tab
2. Drag and drop your file OR click to browse
3. Supported formats: `.csv`, `.xlsx`, `.xls`
4. Wait for the file to parse (you'll see a success message)

#### Option B: Google Sheets
1. Click on the **Google Sheets** tab
2. Make your Google Sheet publicly accessible:
   - Open your sheet
   - Click **Share** → **Get link**
   - Set to "Anyone with the link can view"
3. Copy the URL
4. Paste it in the input field
5. Click **Fetch Data**

### Step 5: Validate Your Data
1. After upload/fetch, you'll see a data preview table
2. Click the **Validate** button
3. Review the validation summary:
   - ✅ **Green checkmarks** = Valid records
   - ❌ **Red X marks** = Invalid records
4. If there are errors:
   - Scroll down to see detailed error messages
   - Fix the errors in your source file
   - Re-upload and validate again

### Step 6: Upload Employees
1. Once all records are valid, click **Upload Employees**
2. Watch the progress bar
3. Wait for the success message
4. You'll be automatically redirected to the employee list

## 📋 Quick Reference

### Date Format
Always use: `YYYY-MM-DD`
- ✅ Correct: `2024-01-15`
- ❌ Wrong: `01/15/2024`, `15-01-2024`

### Phone Format
Use 10-15 digits (spaces/hyphens optional)
- ✅ Correct: `1234567890`, `123-456-7890`
- ❌ Wrong: `12345`, `abcd123456`

### Email Format
Standard email format
- ✅ Correct: `john@example.com`
- ❌ Wrong: `john@`, `@example.com`

### Department
Use exact department name (case-insensitive)
- ✅ Correct: `Engineering`, `engineering`
- ❌ Wrong: `Eng`, `Tech` (if not in system)

## 🎯 Pro Tips

1. **Start Small**: Test with 2-3 employees first
2. **Check Departments**: Ensure all departments exist before upload
3. **Use Template**: Always start from the downloaded template
4. **Validate First**: Always click Validate before Upload
5. **Keep Backup**: Save a copy of your original data
6. **Use Help**: Click the floating help button (?) for detailed guidance

## ⚠️ Common Mistakes to Avoid

1. ❌ Using wrong date format
2. ❌ Duplicate email addresses
3. ❌ Non-existent department names
4. ❌ Invalid phone numbers (too short/long)
5. ❌ Missing required fields
6. ❌ Uploading without validation

## 🆘 Troubleshooting

### "Department not found"
**Solution**: Create the department first in Administration → Departments

### "Email already exists"
**Solution**: Check if employee already exists or remove duplicate from file

### "Invalid date format"
**Solution**: Use YYYY-MM-DD format (e.g., 2024-01-01)

### "Google Sheets permission denied"
**Solution**: Make the sheet public or configure OAuth (see BULK_UPLOAD_GUIDE.md)

### File won't upload
**Solution**: 
- Check file format (CSV, XLSX, XLS only)
- Ensure file is not corrupted
- Try a smaller file first

## 📊 Sample Data

Here's a complete example with 2 employees:

```csv
firstName,lastName,email,phone,department,designation,joiningDate,employmentType,status,gender,basicSalary
John,Doe,john.doe@example.com,1234567890,Engineering,Software Engineer,2024-01-01,full-time,active,male,50000
Jane,Smith,jane.smith@example.com,9876543210,HR,HR Manager,2024-01-15,full-time,active,female,60000
```

## 🎓 Next Steps

After successfully uploading employees:
1. Verify employees in the Employee List
2. Check employee details pages
3. Set up additional information (if needed)
4. Configure access and permissions

## 📚 More Information

- **Detailed Guide**: [BULK_UPLOAD_GUIDE.md](./BULK_UPLOAD_GUIDE.md)
- **Technical Docs**: [BULK_UPLOAD_IMPLEMENTATION.md](./BULK_UPLOAD_IMPLEMENTATION.md)
- **Help Button**: Click the floating (?) button on the bulk upload page

## ✅ Success Checklist

Before uploading, ensure:
- [ ] All required fields are filled
- [ ] Dates are in YYYY-MM-DD format
- [ ] Emails are unique and valid
- [ ] Phone numbers are 10-15 digits
- [ ] Departments exist in the system
- [ ] File is in correct format (CSV/Excel)
- [ ] Data has been validated
- [ ] No validation errors remain

---

**Need Help?** Click the floating help button (?) on the bulk upload page for instant guidance!
