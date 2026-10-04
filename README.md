# 📸 Attend AI Class — AI-Powered Attendance System

> **Making attendance faster, smarter, and more convenient with AI.**

**Attend AI Class** is a multi-modal AI-powered attendance management system that automatically records student attendance using **Face Recognition** and **Voice Recognition**.

It provides separate dashboards for **teachers and students**, allowing teachers to create subjects, manage classes, record attendance, and share enrollment links, while students can register their biometric data and track their attendance.

---

## ✨ Features

### 👨‍🏫 Teacher Features

* 🔐 Secure teacher registration and login
* 🔒 Passwords protected using **bcrypt hashing**
* 📚 Create and manage subjects
* 🔑 Generate unique subject codes
* 📷 **Image-Based Attendance**

  * Upload a classroom photo
  * Detect multiple faces automatically
  * Identify enrolled students
  * Mark attendance automatically
* 🎙️ **Voice-Based Attendance**

  * Upload a bulk classroom audio recording
  * Detect and segment individual speech
  * Match voices with enrolled students
  * Automatically mark attendance
* 📊 View detailed attendance records
* 🕐 Attendance timestamps
* 🔗 Share subjects using:

  * QR codes
  * Shareable enrollment links
  * Subject codes

### 👨‍🎓 Student Features

* 🔐 Student registration and login
* 📸 Upload a face photo for recognition enrollment
* 🎤 Upload a voice sample for voice recognition enrollment
* 📚 Join subjects using subject codes
* 📱 Join subjects by scanning a QR code
* 📈 View attendance statistics for each subject
* 🚪 Unenroll from subjects

---

# 🧠 AI / ML Architecture

Attend AI uses two independent recognition pipelines for automated attendance.

## 📷 Face Recognition Pipeline

**File:** `src/pipelines/face_pipeline.py`

| Step               | Technology                         |
| ------------------ | ---------------------------------- |
| Face Detection     | `dlib` frontal face detector       |
| Landmark Detection | `dlib` 68-point shape predictor    |
| Feature Extraction | dlib ResNet face recognition model |
| Face Embedding     | 128-dimensional embedding          |
| Classification     | `scikit-learn` SVM                 |
| Kernel             | Linear                             |
| Matching Threshold | Euclidean distance ≤ `0.6`         |

### Face Attendance Flow

```text
Classroom Image
      ↓
Face Detection
      ↓
68-Point Landmark Detection
      ↓
128-D Face Embedding
      ↓
SVM Classification
      ↓
Student Identification
      ↓
Attendance Marked
```

---

## 🎙️ Voice Recognition Pipeline

**File:** `src/pipelines/voice_pipeline.py`

| Step                     | Technology                   |
| ------------------------ | ---------------------------- |
| Audio Loading            | `librosa`                    |
| Sample Rate              | 16 kHz                       |
| Preprocessing            | `resemblyzer.preprocess_wav` |
| Embedding Extraction     | Resemblyzer `VoiceEncoder`   |
| Voice Embedding          | 256-dimensional d-vector     |
| Speaker Identification   | Cosine similarity            |
| Matching Threshold       | `0.65`                       |
| Speech Segmentation      | `librosa.effects.split`      |
| Minimum Segment Duration | `0.5 seconds`                |
| Silence Threshold        | `top_db=30`                  |

### Voice Attendance Flow

```text
Classroom Audio
      ↓
Audio Loading & Resampling
      ↓
Speech Segmentation
      ↓
Voice Embedding Extraction
      ↓
Cosine Similarity Matching
      ↓
Student Identification
      ↓
Attendance Marked
```

---

# 🗄️ Database Architecture

Attend AI uses **Supabase with PostgreSQL** for storing user accounts, subjects, enrollments, biometric embeddings, and attendance records.

### Database Structure

```text
┌─────────────────┐
│    teachers     │
├─────────────────┤
│ teacher_id      │
│ username        │
│ password        │
│ name            │
└────────┬────────┘
         │
         │
         ▼
┌─────────────────┐
│    subjects     │
├─────────────────┤
│ subject_id      │
│ subject_code    │
│ name            │
│ section         │
│ teacher_id      │
└────────┬────────┘
         │
         │
         ▼
┌──────────────────────┐
│  subject_students    │
├──────────────────────┤
│ subject_id           │
│ student_id           │
└──────────┬───────────┘
           │
           ▼
┌─────────────────┐
│    students     │
├─────────────────┤
│ student_id      │
│ name            │
│ face_embedding  │
│ voice_embedding │
└────────┬────────┘
         │
         ▼
┌──────────────────────┐
│   attendance_logs    │
├──────────────────────┤
│ id                   │
│ timestamp            │
│ subject_id           │
│ student_id           │
│ is_present           │
└──────────────────────┘
```

### Database Setup

The repository includes `supabase.sql`, which contains the SQL required to initialize the database tables.

Run the SQL script in your **Supabase SQL Editor** before running the application.

> ⚠️ **Security:** Never commit your Supabase credentials or `.streamlit/secrets.toml` to GitHub.

---

# 📁 Project Structure

```text
Attend AI/
│
├── app.py
├── requirements.txt
├── supabase.sql
│
├── .streamlit/
│   └── secrets.toml
│
└── src/
    │
    ├── screens/
    │   ├── home_screen.py
    │   ├── teacher_screen.py
    │   └── student_screen.py
    │
    ├── pipelines/
    │   ├── face_pipeline.py
    │   └── voice_pipeline.py
    │
    ├── components/
    │   ├── dialog_add_photo.py
    │   ├── dialog_voice_attendance.py
    │   ├── dialog_attendance_results.py
    │   ├── dialog_create_subject.py
    │   ├── dialog_share_subject.py
    │   ├── dialog_enroll.py
    │   ├── dialog_auto_enroll.py
    │   ├── subject_card.py
    │   ├── header.py
    │   └── footer.py
    │
    ├── database/
    │   └── db.py
    │
    └── ui/
        └── base_layout.py
```

### Main Components

| Component           | Purpose                                      |
| ------------------- | -------------------------------------------- |
| `app.py`            | Application entry point and routing          |
| `home_screen.py`    | Landing page and login selection             |
| `teacher_screen.py` | Teacher dashboard                            |
| `student_screen.py` | Student dashboard                            |
| `face_pipeline.py`  | Face detection and recognition               |
| `voice_pipeline.py` | Voice recognition and speaker identification |
| `db.py`             | Supabase database operations                 |
| `components/`       | Reusable UI dialogs and components           |
| `base_layout.py`    | Global UI styles and layout                  |

---

# 🛠️ Tech Stack

| Category              | Technology                               |
| --------------------- | ---------------------------------------- |
| Application Framework | **Streamlit**                            |
| Programming Language  | **Python**                               |
| Face Detection        | **dlib**                                 |
| Face Recognition      | **dlib ResNet / face recognition model** |
| Voice Recognition     | **Resemblyzer**                          |
| Audio Processing      | **librosa**                              |
| Machine Learning      | **scikit-learn**                         |
| Classification        | **SVM**                                  |
| Database              | **Supabase / PostgreSQL**                |
| Authentication        | **bcrypt**                               |
| QR Code Generation    | **Segno**                                |
| Image Processing      | **Pillow**                               |
| Numerical Computing   | **NumPy**                                |

---


# 🔐 Security Considerations

Attend AI handles authentication and biometric information, so security is important.

* Passwords are stored using **bcrypt hashing**.
* Supabase credentials should be stored using **Streamlit Secrets**.
* Never commit API keys or passwords to GitHub.
* Biometric embeddings should be handled securely.
* Access to attendance records should be restricted to authorized users.

> **Note:** This project is intended for educational and demonstration purposes. For production deployment, additional security, privacy, consent, and data-protection measures should be implemented.

---

# 🎯 How Attend AI Works

### Teacher Workflow

```text
Teacher Registration/Login
          ↓
Create Subject
          ↓
Share Subject
    ┌─────┴─────┐
    ↓           ↓
   QR Code   Subject Code
    └─────┬─────┘
          ↓
   Students Enroll
          ↓
Teacher Starts Attendance
     ┌────┴────┐
     ↓         ↓
   Image     Voice
     ↓         ↓
Face Model  Voice Model
     └────┬────┘
          ↓
 Student Identification
          ↓
 Attendance Database
          ↓
 Attendance Records
```

### Student Workflow

```text
Student Registration/Login
          ↓
Add Face Photo
          +
Add Voice Sample
          ↓
Enroll in Subject
          ↓
Attend Class
          ↓
AI Recognition
          ↓
Attendance Recorded
          ↓
View Attendance Statistics
```

---

# 🚀 Future Improvements

Some possible improvements for future versions include:

* 📱 Mobile-friendly interface
* ⚡ Faster recognition for large classrooms
* 🎯 Improved face and speaker recognition accuracy
* 🧑‍🤝‍🧑 Better handling of multiple speakers
* 📊 Advanced attendance analytics
* 📥 Export attendance reports to CSV/PDF
* 🔔 Attendance notifications
* 📅 Attendance calendar and monthly reports
* 🛡️ Role-based access control
* ☁️ Scalable cloud-based AI processing
* 🔒 Stronger biometric data protection
* 📈 Admin dashboard for institution-wide attendance management


---

# 📄 License

This project is open source and available under the **MIT License**.

See the [`LICENSE`](LICENSE) file for more information.

---

# 👨‍💻 Author

**Rohit Singh Rawat**

Built with Python, Streamlit, Machine Learning, Computer Vision, and Speech Recognition.

---

## ❤️ Acknowledgements

This project uses several open-source technologies and libraries, including:

* Streamlit
* dlib
* scikit-learn
* Resemblyzer
* librosa
* Supabase
* PostgreSQL
* Pillow
* NumPy
* Segno

---

<p align="center">
  <b>📸 Attend AI — Smarter Attendance with AI</b>
  <br>
  Made with ❤️ using Python & AI
</p>
