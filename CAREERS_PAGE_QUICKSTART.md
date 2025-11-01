# Careers Page - Quick Start Guide

## 🚀 Getting Started in 5 Minutes

### For HR Personnel

#### Step 1: Post a Job
1. Login to HRMS portal
2. Navigate to **Job Desk**
3. Click **"Create Job Posting"**
4. Fill in job details:
   - Title (e.g., "Software Engineer")
   - Department
   - Location
   - Employment Type
   - Description
   - Requirements
   - Skills
5. Click **"Publish"** to set status to "Active"
6. Job is now live on the careers page!

#### Step 2: View Applications
1. Navigate to **Job Desk**
2. Click on the job posting
3. Click **"View Applicants"** or go to `/job-desk/[jobId]/applicants`
4. See all candidates who applied
5. Applications from careers page will show:
   - Source: "job-portal"
   - Stage: "applied"
   - Complete candidate details

### For Candidates (Public Users)

#### Step 1: Browse Jobs
1. Visit `http://localhost:5173/careers` (or your deployed URL)
2. No login required!
3. Browse available job openings
4. Use search and filters to find relevant jobs

#### Step 2: Apply for a Job
1. Click **"Apply Now"** on desired job
2. Fill in the application form:
   - **Required**: Name, Email, Phone
   - **Optional**: Experience, Skills, Education, etc.
3. Click **"Submit Application"**
4. Success! You'll receive a candidate code

#### Step 3: Track Your Application
Your application is now in the HRMS system and HR will review it soon!

## 📋 Quick Reference

### Public URLs
- **Careers Page**: `/careers` or `/jobs`
- **No Authentication Required**: Anyone can view and apply

### Application Form Fields

**Required:**
- First Name
- Last Name
- Email
- Phone

**Optional but Recommended:**
- Experience (years/months)
- Current Company & Designation
- Current & Expected CTC
- Notice Period
- Skills
- Education Details
- Preferred Locations

### Data Validation Rules

**Email:**
- Must be valid format (e.g., john@example.com)
- Cannot apply twice for same job

**Phone:**
- Must be 10-15 digits
- Spaces, hyphens, parentheses allowed

**Experience:**
- Years: 0-50
- Months: 0-11

## 🎯 Common Use Cases

### Use Case 1: Fresh Graduate Applying
```
Name: Jane Smith
Email: jane@email.com
Phone: 9876543210
Experience: 0 years, 0 months
Education: B.Tech in Computer Science
Skills: Python, Java, SQL
```

### Use Case 2: Experienced Professional
```
Name: John Doe
Email: john@email.com
Phone: 1234567890
Experience: 5 years, 3 months
Current Company: ABC Corp
Current Designation: Senior Developer
Current CTC: 1500000
Expected CTC: 2000000
Notice Period: 60 days
Skills: JavaScript, React, Node.js, AWS
```

### Use Case 3: Career Change
```
Name: Sarah Johnson
Email: sarah@email.com
Phone: 5555555555
Experience: 3 years, 0 months
Current Company: XYZ Ltd
Current Designation: Marketing Manager
Preferred Locations: New York, Remote
Skills: Digital Marketing, SEO, Content Strategy
```

## ⚠️ Common Issues & Solutions

### Issue: "You have already applied for this position"
**Solution**: You can only apply once per job. If you need to update your application, contact HR directly.

### Issue: "Invalid email format"
**Solution**: Ensure email follows format: name@domain.com

### Issue: "Invalid phone format"
**Solution**: Use 10-15 digit phone number (e.g., 1234567890)

### Issue: Job not appearing on careers page
**Solution (HR)**: Ensure job status is set to "Active" in Job Desk

### Issue: Application not showing in HRMS
**Solution**: Check the specific job's applicants page at `/job-desk/[jobId]/applicants`

## 🔍 Finding the Right Job

### Using Search
- Search by job title (e.g., "Engineer")
- Search by department (e.g., "Sales")
- Search by keywords in description

### Using Filters
- **Department**: Filter by specific department
- **Location**: Filter by job location
- **Employment Type**: Full-time, Part-time, Contract, Intern

### Clearing Filters
- Click "Clear Filters" to reset all filters and see all jobs

## 📊 Understanding Job Cards

Each job card shows:
- **Job Title**: Position name
- **Department**: Which team you'll join
- **Location**: Where the job is based
- **Employment Type**: Full-time, Part-time, etc.
- **Experience**: Required years of experience
- **Openings**: Number of positions available
- **Description**: Brief job overview

## ✅ Application Checklist

Before submitting, ensure:
- [ ] All required fields filled (Name, Email, Phone)
- [ ] Email is valid and accessible
- [ ] Phone number is correct
- [ ] Experience details are accurate
- [ ] Skills are relevant to the job
- [ ] Education information is complete
- [ ] Preferred locations are specified (if any)

## 🎉 After Applying

### What Happens Next?
1. **Immediate**: You receive a success message with candidate code
2. **Within 24-48 hours**: HR reviews your application
3. **If shortlisted**: You'll receive an email/call for next steps
4. **Interview Process**: Multiple rounds based on role
5. **Offer**: If selected, you'll receive an offer letter

### Your Candidate Code
- Format: `CAN00123`
- Use this to reference your application
- Keep it safe for future communication

## 🔐 Privacy & Data

### What We Collect
- Personal information (name, email, phone)
- Professional details (experience, skills)
- Educational background
- Preferred locations

### How We Use It
- To review your application
- To contact you for interviews
- To assess job fit
- To maintain candidate database

### Data Security
- All data is securely stored
- Only HR personnel have access
- Not shared with third parties

## 📞 Need Help?

### For Candidates
- Check this guide first
- Review validation error messages
- Ensure all required fields are filled

### For HR
- Check Job Desk for posted jobs
- Verify job status is "Active"
- Review applicants page for submissions

## 🚀 Pro Tips

### For Better Application Success
1. **Complete Profile**: Fill all optional fields
2. **Relevant Skills**: Add skills matching job requirements
3. **Clear Information**: Use professional language
4. **Accurate Details**: Ensure all information is correct
5. **Timely Application**: Apply as soon as you see the job

### For HR Efficiency
1. **Clear Job Descriptions**: Write detailed requirements
2. **Accurate Details**: Specify experience, skills needed
3. **Regular Updates**: Keep job postings current
4. **Quick Response**: Review applications promptly
5. **Status Updates**: Update candidate stages regularly

---

**Ready to get started?** Visit `/careers` to see all open positions!
