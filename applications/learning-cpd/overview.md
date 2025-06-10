# Learning & CPD Application - Overview

> **Copyright © 2024 TechJamLabs**  
> Website: [www.techjamlabs.com](https://www.techjamlabs.com)  
> Phone: +234 201 330 9089 | +1 206 710 0170  
> **Authors:** Ben Adenle, Odinaka Solomon, Dare Oloruntoba

---

## 🎯 **Application Overview**

The **Learning & CPD (Continuing Professional Development) Application** provides comprehensive training and professional development opportunities for pharmacists, enabling them to maintain their professional credentials and advance their careers.

### **Primary Functions**

1. **Course Management**
   - Course creation and curation
   - Content delivery and streaming
   - Multi-format content support
   - Course catalog organization

2. **Learning Paths**
   - Structured learning journeys
   - Prerequisite management
   - Progress tracking
   - Competency mapping

3. **Assessment & Certification**
   - Online assessments and quizzes
   - Practical evaluations
   - Certificate generation
   - Credential verification

4. **BigBlueButton Integration**
   - Live virtual classrooms
   - Interactive webinars
   - Recording and playback
   - Real-time collaboration

5. **CPD Credit Tracking**
   - Automated credit calculation
   - PCN requirement compliance
   - Credit history management
   - Annual requirement tracking

6. **Learning Analytics**
   - Student progress monitoring
   - Course effectiveness analysis
   - Engagement metrics
   - Performance reporting

---

## 🏗️ **Architecture Overview**

### **System Architecture**

```mermaid
graph TB
    subgraph "Learning & CPD Application"
        COURSE_MGMT[Course Management Service]
        LEARNING_PATH[Learning Path Engine]
        ASSESSMENT[Assessment Engine]
        BBB_INT[BigBlueButton Integration]
        CPD_TRACK[CPD Tracking Service]
        ANALYTICS[Learning Analytics]
    end
    
    subgraph "External Integrations"
        BBB_SERVER[BigBlueButton Server]
        PCN_API[PCN Database]
        CONTENT_CDN[Content Delivery Network]
        VIDEO_STREAM[Video Streaming]
    end
    
    subgraph "Data Layer"
        COURSE_DB[(Course Database)]
        PROGRESS_DB[(Progress Database)]
        ASSESSMENT_DB[(Assessment Database)]
        CPD_DB[(CPD Records Database)]
    end
    
    COURSE_MGMT --> CONTENT_CDN
    COURSE_MGMT --> COURSE_DB
    
    LEARNING_PATH --> COURSE_DB
    LEARNING_PATH --> PROGRESS_DB
    
    ASSESSMENT --> ASSESSMENT_DB
    BBB_INT --> BBB_SERVER
    BBB_INT --> VIDEO_STREAM
    
    CPD_TRACK --> PCN_API
    CPD_TRACK --> CPD_DB
    
    ANALYTICS --> COURSE_DB
    ANALYTICS --> PROGRESS_DB
    ANALYTICS --> ASSESSMENT_DB
```

This Learning & CPD Application ensures continuous professional development and regulatory compliance for pharmacists. 