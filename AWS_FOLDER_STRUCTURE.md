# AWS S3 Folder Structure - ECG Application

## 📁 Overview

The ECG application uploads data to AWS S3 using a structured folder hierarchy organized by date. All files are uploaded under the `ecg-reports/` prefix.

---

## 🗂️ Folder Structure

### Base Structure
```
s3://your-bucket-name/
└── ecg-reports/
    └── {YYYY}/
        └── {MM}/
            └── {DD}/
                ├── ECG_Report_*.pdf          (ECG PDF Reports)
                ├── ECG_Report_*.json         (Report Metadata JSON)
                ├── user_signup_*.json        (User Signup Data)
                └── {timestamp}/              (Complete Report Packages)
                    ├── ECG_Report_*.pdf
                    ├── patient_details_*.json
                    └── ecg_data_*.json
```

---

## 📋 Detailed Folder Structure

### 1. **Regular ECG Reports** (Simple Upload)

**Path Pattern:**
```
ecg-reports/{YYYY}/{MM}/{DD}/{filename}
```

**Example:**
```
ecg-reports/2026/01/24/ECG_Report_20260124_121815.pdf
ecg-reports/2026/01/24/ECG_Report_20260124_121815.json
```

**Code Location:** `cloud_uploader.py` line 429
```python
timestamp = datetime.now().strftime("%Y/%m/%d")
s3_key = f"ecg-reports/{timestamp}/{filename}"
```

**Files Uploaded:**
- PDF Reports: `ECG_Report_YYYYMMDD_HHMMSS.pdf`
- JSON Metadata: `ECG_Report_YYYYMMDD_HHMMSS.json` (if exists)

---

### 2. **Complete Report Packages** (Full Package Upload)

**Path Pattern:**
```
ecg-reports/{YYYY}/{MM}/{DD}/{YYYYMMDD_HHMMSS}/
```

**Example:**
```
ecg-reports/2026/01/24/20260124_121815/
├── ECG_Report_20260124_121815.pdf
├── patient_details_20260124_121815.json
└── ecg_data_20260124_121815.json
```

**Code Location:** `cloud_uploader.py` line 492-504
```python
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
date_prefix = datetime.now().strftime("%Y/%m/%d")
pdf_s3_key = f"ecg-reports/{date_prefix}/{timestamp}/{pdf_filename}"
```

**Files in Package:**
1. **PDF Report**: `ECG_Report_YYYYMMDD_HHMMSS.pdf`
   - Full ECG report with waveforms
   - Location: `ecg-reports/{date}/{timestamp}/{pdf_filename}`

2. **Patient Details**: `patient_details_YYYYMMDD_HHMMSS.json`
   - Patient information (name, age, gender, etc.)
   - Location: `ecg-reports/{date}/{timestamp}/patient_details_{timestamp}.json`

3. **ECG Data**: `ecg_data_YYYYMMDD_HHMMSS.json`
   - 12-lead ECG waveform data (10 seconds)
   - Sampling rate, lead data (I, II, III, aVR, aVL, aVF, V1-V6)
   - Location: `ecg-reports/{date}/{timestamp}/{ecg_filename}`

---

### 3. **User Signup Data**

**Path Pattern:**
```
ecg-reports/{YYYY}/{MM}/{DD}/{filename}
```

**Example:**
```
ecg-reports/2026/01/24/user_signup_ptr_20260124_120419.json
ecg-reports/2026/01/24/user_signup_Divyansh_20260124_120419.json
```

**Code Location:** `cloud_uploader.py` line 372
```python
# Uses same _upload_to_s3 method, so same structure
s3_key = f"ecg-reports/{timestamp}/{filename}"
```

**File Format:**
```json
{
  "username": "ptr",
  "full_name": "",
  "phone": "",
  "serial_id": "ptr",
  "email": "",
  "age": "",
  "gender": "",
  "registration_date": "2026-01-24 12:04:19",
  "last_sync": "2026-01-24 12:04:19",
  "uploaded_at": "2026-01-24T12:04:19.123456"
}
```

---

## 🔧 How Upload Works

### Upload Methods

#### 1. **Simple Report Upload** (`upload_report`)
- **Trigger**: When a PDF report is generated
- **Path**: `ecg-reports/{YYYY}/{MM}/{DD}/{filename}`
- **Files**: PDF + JSON (if exists)

#### 2. **Complete Package Upload** (`upload_complete_report_package`)
- **Trigger**: When full report package is requested
- **Path**: `ecg-reports/{YYYY}/{MM}/{DD}/{timestamp}/`
- **Files**: PDF + Patient JSON + ECG Data JSON

#### 3. **User Signup Upload** (`upload_user_signup`)
- **Trigger**: When new user registers
- **Path**: `ecg-reports/{YYYY}/{MM}/{DD}/user_signup_{username}_{timestamp}.json`
- **Files**: User data JSON

---

## 📊 Real-World Examples

### Example 1: Single Report Upload (January 24, 2026)
```
s3://my-ecg-bucket/
└── ecg-reports/
    └── 2026/
        └── 01/
            └── 24/
                ├── ECG_Report_20260124_121815.pdf
                └── ECG_Report_20260124_121815.json
```

### Example 2: Complete Package Upload (January 24, 2026, 12:18:15)
```
s3://my-ecg-bucket/
└── ecg-reports/
    └── 2026/
        └── 01/
            └── 24/
                └── 20260124_121815/
                    ├── ECG_Report_20260124_121815.pdf
                    ├── patient_details_20260124_121815.json
                    └── ecg_data_20260124_121815.json
```

### Example 3: User Signups (January 24, 2026)
```
s3://my-ecg-bucket/
└── ecg-reports/
    └── 2026/
        └── 01/
            └── 24/
                ├── user_signup_ptr_20260124_120419.json
                ├── user_signup_Divyansh_20260124_120419.json
                └── user_signup_12345_20260124_120419.json
```

---

## 🔍 File Metadata

All uploaded files include S3 metadata with:
- `type`: File type (ecg_report_pdf, patient_details, ecg_12lead_data, user_signup)
- `uploaded_at`: ISO timestamp of upload
- `filename`: Original filename
- Additional metadata based on file type

---

## 🚀 Auto-Sync Service

The application runs an **Auto-Sync Service** that:
- Monitors `reports/` directory every 15 seconds
- Uploads new/modified PDF reports automatically
- Uploads user signups from `users.json`
- Prevents duplicate uploads (checks upload log)

**Code Location:** `auto_sync_service.py`

---

## 📝 Configuration

### Environment Variables (`.env` file)

```env
# Enable cloud upload
CLOUD_UPLOAD_ENABLED=true

# Choose cloud service
CLOUD_SERVICE=s3

# AWS S3 Configuration
AWS_S3_BUCKET=your-bucket-name
AWS_S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
```

---

## 🔐 Security & Access

### S3 Bucket Policy Example
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT:user/ecg-uploader"
      },
      "Action": [
        "s3:PutObject",
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name/ecg-reports/*"
    }
  ]
}
```

---

## 📈 Benefits of This Structure

1. **Date-Based Organization**: Easy to find files by date
2. **Scalability**: Can handle millions of files efficiently
3. **Cost Optimization**: Can use S3 lifecycle policies to move old files to Glacier
4. **Backup & Recovery**: Easy to restore specific date ranges
5. **Compliance**: Organized structure helps with healthcare data compliance

---

## 🔄 Upload Flow

```
1. Application generates report
   ↓
2. Report saved to local: reports/ECG_Report_*.pdf
   ↓
3. Auto-sync service detects new file
   ↓
4. Cloud uploader checks if already uploaded
   ↓
5. If new, uploads to: s3://bucket/ecg-reports/{YYYY}/{MM}/{DD}/{filename}
   ↓
6. Upload logged to: reports/upload_log.json
   ↓
7. File marked as synced (prevents duplicates)
```

---

## 📌 Key Points

- ✅ All files go under `ecg-reports/` prefix
- ✅ Date structure: `YYYY/MM/DD/`
- ✅ Complete packages use timestamp subfolder: `{YYYY}/{MM}/{DD}/{timestamp}/`
- ✅ User signups use same date structure
- ✅ Duplicate uploads are prevented
- ✅ Metadata stored with each file
- ✅ Auto-sync runs every 15 seconds

---

## 🛠️ Code References

- **Main Uploader**: `ECGdiv/src/utils/cloud_uploader.py`
- **Auto-Sync Service**: `ECGdiv/src/utils/auto_sync_service.py`
- **S3 Upload Method**: `_upload_to_s3()` (line 413)
- **Complete Package**: `upload_complete_report_package()` (line 455)
- **User Signup**: `upload_user_signup()` (line 284)

---

**Last Updated**: January 2026


