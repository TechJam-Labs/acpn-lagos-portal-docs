# BigBlueButton Integration for Virtual Learning

> **Self-Hosted Virtual Classroom Platform for ACPN CPD Programs**
> 
> Complete implementation guide for BigBlueButton integration with the ACPN Lagos Portal, enabling live virtual learning sessions, recordings, and seamless CPD credit management.

---

## 🎯 **Integration Overview**

### **BigBlueButton Platform**
BigBlueButton (BBB) is an open-source web conferencing system designed specifically for online learning. Its integration with the ACPN portal provides a comprehensive virtual classroom environment for continuing professional development.

#### **Key Features**
```yaml
Virtual Classroom Capabilities:
  Real-time Audio/Video: HD quality with echo cancellation
  Screen Sharing: Desktop and application sharing
  Interactive Whiteboard: Collaborative drawing and annotation
  Breakout Rooms: Small group discussions and activities
  Polls & Quizzes: Real-time audience engagement
  Chat: Text-based communication and Q&A
  
Recording & Playback:
  Automatic Recording: Session capture for later access
  Multiple Formats: MP4 video and presentation modes
  Searchable Content: Indexed recordings for easy discovery
  Download Options: Offline access for mobile learning
  
Integration Features:
  Single Sign-On: ACPN portal authentication
  Attendance Tracking: Automatic participant monitoring
  CPD Credit Assignment: Seamless credit calculation
  Learning Analytics: Engagement and performance metrics
```

---

## 🖥️ **Server Setup & Configuration**

### **Hardware Requirements**
```yaml
Minimum Requirements:
  CPU: 8 cores (Intel Xeon or AMD equivalent)
  RAM: 32GB (minimum for production)
  Storage: 1TB SSD (for recordings and system)
  Network: 1Gbps dedicated bandwidth
  OS: Ubuntu 20.04 LTS (specific BBB requirement)
  
Recommended for High Load:
  CPU: 16 cores with hyper-threading
  RAM: 64GB for improved performance
  Storage: 2TB NVMe SSD with backup storage
  Network: Multiple 1Gbps interfaces for redundancy
  
Concurrent User Capacity:
  50 Users: 4 cores, 16GB RAM
  100 Users: 8 cores, 32GB RAM
  200 Users: 12 cores, 48GB RAM
  500+ Users: 16+ cores, 64GB+ RAM
```

### **Installation Process**
```bash
# 1. Clean Ubuntu 20.04 LTS Installation
# Ensure fresh installation with no other web servers

# 2. Update System Packages
sudo apt update && sudo apt upgrade -y
sudo apt install wget curl software-properties-common

# 3. Install BigBlueButton
# Use official installation script
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v focal-270 -s bbb.acpnlagos.org -e admin@acpnlagos.org

# 4. Configure SSL Certificate
# Let's Encrypt integration for HTTPS
sudo bbb-conf --setip bbb.acpnlagos.org
sudo certbot --nginx -d bbb.acpnlagos.org

# 5. Optimize Configuration
sudo bbb-conf --restart
sudo bbb-conf --check
```

### **Network Configuration**
```yaml
Firewall Rules (UFW):
  SSH: 22/tcp (admin access)
  HTTP: 80/tcp (redirect to HTTPS)
  HTTPS: 443/tcp (web interface)
  FreeSWITCH: 16384-32768/udp (audio)
  Kurento: 3478/udp (STUN)
  Turn Server: 49152-65535/udp (WebRTC)
  
Port Mapping:
  Nginx: 443 (HTTPS proxy)
  BigBlueButton Web: 8090
  FreeSWITCH: 5060/5080 (SIP)
  Redis: 6379 (session storage)
  MongoDB: 27017 (recording metadata)
  
Domain Configuration:
  Primary Domain: bbb.acpnlagos.org
  SSL Certificate: Let's Encrypt wildcard
  DNS Records: A record pointing to server IP
  CDN: Optional CloudFlare for global access
```

---

## 🔌 **API Integration with ACPN Portal**

### **BBB API Implementation**
```typescript
// BigBlueButton API Service
import crypto from 'crypto';
import axios from 'axios';

export class BigBlueButtonService {
  private readonly apiUrl: string;
  private readonly sharedSecret: string;

  constructor() {
    this.apiUrl = process.env.BBB_API_URL || 'https://bbb.acpnlagos.org/bigbluebutton/api';
    this.sharedSecret = process.env.BBB_SHARED_SECRET || '';
  }

  // Create Meeting
  async createMeeting(params: CreateMeetingParams): Promise<Meeting> {
    const meetingParams = {
      name: params.name,
      meetingID: params.meetingId,
      attendeePW: params.attendeePassword,
      moderatorPW: params.moderatorPassword,
      welcome: params.welcomeMessage,
      dialNumber: '',
      voiceBridge: params.voiceBridge,
      maxParticipants: params.maxParticipants,
      logoutURL: `${process.env.PORTAL_URL}/cpd/session-complete`,
      record: 'true',
      allowStartStopRecording: 'true',
      autoStartRecording: 'true',
      webcamsOnlyForModerator: 'false',
      muteOnStart: 'true',
      meta_title: params.courseTitle,
      meta_instructor: params.instructorName,
      meta_course_id: params.courseId,
      meta_session_id: params.sessionId
    };

    const queryString = this.buildQueryString('create', meetingParams);
    const url = `${this.apiUrl}/create?${queryString}`;
    
    const response = await axios.get(url);
    return this.parseXMLResponse(response.data);
  }

  // Join Meeting URL
  generateJoinURL(params: JoinMeetingParams): string {
    const joinParams = {
      fullName: params.fullName,
      meetingID: params.meetingId,
      password: params.password,
      redirect: 'true',
      userID: params.userId,
      role: params.role,
      joinViaHtml5: 'true',
      guest: 'false'
    };

    const queryString = this.buildQueryString('join', joinParams);
    return `${this.apiUrl}/join?${queryString}`;
  }

  // Get Meeting Info
  async getMeetingInfo(meetingId: string): Promise<MeetingInfo> {
    const params = { meetingID: meetingId };
    const queryString = this.buildQueryString('getMeetingInfo', params);
    const url = `${this.apiUrl}/getMeetingInfo?${queryString}`;
    
    const response = await axios.get(url);
    return this.parseXMLResponse(response.data);
  }

  // End Meeting
  async endMeeting(meetingId: string, moderatorPassword: string): Promise<void> {
    const params = {
      meetingID: meetingId,
      password: moderatorPassword
    };

    const queryString = this.buildQueryString('end', params);
    const url = `${this.apiUrl}/end?${queryString}`;
    
    await axios.get(url);
  }

  // Get Recordings
  async getRecordings(meetingId?: string): Promise<Recording[]> {
    const params = meetingId ? { meetingID: meetingId } : {};
    const queryString = this.buildQueryString('getRecordings', params);
    const url = `${this.apiUrl}/getRecordings?${queryString}`;
    
    const response = await axios.get(url);
    return this.parseRecordingsResponse(response.data);
  }

  // Publish/Unpublish Recording
  async publishRecording(recordId: string, publish: boolean): Promise<void> {
    const params = {
      recordID: recordId,
      publish: publish.toString()
    };

    const queryString = this.buildQueryString('publishRecordings', params);
    const url = `${this.apiUrl}/publishRecordings?${queryString}`;
    
    await axios.get(url);
  }

  // Build query string with checksum
  private buildQueryString(apiCall: string, params: Record<string, string>): string {
    const paramString = Object.keys(params)
      .sort()
      .map(key => `${key}=${encodeURIComponent(params[key])}`)
      .join('&');
    
    const queryString = `${apiCall}${paramString}${this.sharedSecret}`;
    const checksum = crypto.createHash('sha1').update(queryString).digest('hex');
    
    return `${paramString}&checksum=${checksum}`;
  }

  // Parse XML response
  private parseXMLResponse(xmlData: string): any {
    // XML parsing implementation
    // Return parsed meeting data
  }
}
```

### **Meeting Management Integration**
```typescript
// CPD Session Management Service
export class CPDSessionService {
  private bbb: BigBlueButtonService;
  private db: DatabaseService;

  constructor() {
    this.bbb = new BigBlueButtonService();
    this.db = new DatabaseService();
  }

  async createLiveSession(sessionData: CPDSessionData): Promise<LiveSession> {
    // Create BBB meeting
    const meeting = await this.bbb.createMeeting({
      name: sessionData.title,
      meetingId: `cpd-${sessionData.id}`,
      attendeePassword: this.generatePassword(),
      moderatorPassword: this.generatePassword(),
      welcomeMessage: this.buildWelcomeMessage(sessionData),
      maxParticipants: sessionData.maxParticipants,
      courseTitle: sessionData.courseTitle,
      instructorName: sessionData.instructor,
      courseId: sessionData.courseId,
      sessionId: sessionData.id
    });

    // Store session in database
    const liveSession = await this.db.createLiveSession({
      id: sessionData.id,
      bbbMeetingId: meeting.meetingId,
      attendeePassword: meeting.attendeePassword,
      moderatorPassword: meeting.moderatorPassword,
      status: 'scheduled',
      startTime: sessionData.startTime,
      endTime: sessionData.endTime,
      enrolledUsers: sessionData.enrolledUsers
    });

    return liveSession;
  }

  async joinSession(sessionId: string, userId: string): Promise<string> {
    const session = await this.db.getLiveSession(sessionId);
    const user = await this.db.getUser(userId);
    
    if (!this.canUserJoinSession(session, user)) {
      throw new Error('User not authorized to join session');
    }

    // Determine user role
    const role = this.getUserRole(session, user);
    const password = role === 'moderator' 
      ? session.moderatorPassword 
      : session.attendeePassword;

    // Generate join URL
    const joinUrl = this.bbb.generateJoinURL({
      fullName: `${user.firstName} ${user.lastName}`,
      meetingId: session.bbbMeetingId,
      password: password,
      userId: user.id,
      role: role
    });

    // Log attendance
    await this.logAttendance(sessionId, userId, 'joined');

    return joinUrl;
  }

  async endSession(sessionId: string, instructorId: string): Promise<void> {
    const session = await this.db.getLiveSession(sessionId);
    
    if (!this.canUserEndSession(session, instructorId)) {
      throw new Error('User not authorized to end session');
    }

    // End BBB meeting
    await this.bbb.endMeeting(session.bbbMeetingId, session.moderatorPassword);

    // Update session status
    await this.db.updateLiveSession(sessionId, {
      status: 'completed',
      actualEndTime: new Date()
    });

    // Process recordings asynchronously
    this.processSessionRecordings(sessionId);
  }

  private async processSessionRecordings(sessionId: string): Promise<void> {
    // Wait for BBB to process recordings
    setTimeout(async () => {
      const session = await this.db.getLiveSession(sessionId);
      const recordings = await this.bbb.getRecordings(session.bbbMeetingId);

      for (const recording of recordings) {
        await this.db.createRecording({
          sessionId: sessionId,
          bbbRecordId: recording.recordId,
          name: recording.name,
          url: recording.playbackUrl,
          duration: recording.duration,
          size: recording.size,
          format: recording.format,
          status: 'available'
        });

        // Assign CPD credits for attendance
        await this.assignCPDCredits(sessionId);
      }
    }, 30000); // Wait 30 seconds for processing
  }

  private async assignCPDCredits(sessionId: string): Promise<void> {
    const session = await this.db.getLiveSession(sessionId);
    const attendance = await this.db.getSessionAttendance(sessionId);
    
    for (const attendee of attendance) {
      if (attendee.duration >= session.minimumDuration) {
        await this.db.assignCPDCredit({
          userId: attendee.userId,
          sessionId: sessionId,
          credits: session.cpdCredits,
          type: 'live_session',
          completedAt: new Date()
        });
      }
    }
  }
}
```

---

## 🎓 **Virtual Classroom Management**

### **Session Workflow**
```yaml
Pre-Session Phase:
  1. Course Scheduling: Instructor creates session in ACPN portal
  2. BBB Meeting Creation: Automatic meeting setup via API
  3. Enrollment: Students register for live session
  4. Reminders: Email and WhatsApp notifications
  5. Pre-Materials: Course materials distributed
  
During Session:
  1. Instructor Login: Moderator access to BBB room
  2. Room Setup: Audio/video testing, material upload
  3. Student Access: Attendee access via portal integration
  4. Automatic Recording: Session capture for later access
  5. Attendance Tracking: Real-time participant monitoring
  6. Interactive Features: Polls, breakout rooms, whiteboard
  
Post-Session Phase:
  1. Automatic Recording Processing: BBB processing
  2. Recording Publication: Available in portal
  3. CPD Credit Assignment: Automatic credit calculation
  4. Feedback Collection: Session evaluation forms
  5. Follow-up Materials: Additional resources distribution
```

### **Instructor Dashboard Features**
```typescript
// Instructor Dashboard Component
export const InstructorDashboard: React.FC = () => {
  const [sessions, setSessions] = useState<LiveSession[]>([]);
  const [currentSession, setCurrentSession] = useState<LiveSession | null>(null);

  return (
    <div className="instructor-dashboard">
      <div className="session-controls">
        <h2>Live Session Management</h2>
        
        {/* Session List */}
        <div className="session-list">
          {sessions.map(session => (
            <SessionCard
              key={session.id}
              session={session}
              onStart={() => startSession(session.id)}
              onEnd={() => endSession(session.id)}
              onJoin={() => joinAsInstructor(session.id)}
            />
          ))}
        </div>

        {/* Current Session Controls */}
        {currentSession && (
          <div className="current-session">
            <h3>{currentSession.title}</h3>
            <div className="session-stats">
              <span>Participants: {currentSession.participantCount}</span>
              <span>Duration: {currentSession.duration}</span>
              <span>Recording: {currentSession.isRecording ? 'ON' : 'OFF'}</span>
            </div>
            
            <div className="session-actions">
              <button onClick={() => toggleRecording(currentSession.id)}>
                {currentSession.isRecording ? 'Stop Recording' : 'Start Recording'}
              </button>
              <button onClick={() => createBreakoutRooms(currentSession.id)}>
                Create Breakout Rooms
              </button>
              <button onClick={() => startPoll(currentSession.id)}>
                Start Poll
              </button>
            </div>
          </div>
        )}
      </div>

      {/* Participant Management */}
      <div className="participant-management">
        <h3>Participants</h3>
        <ParticipantList
          sessionId={currentSession?.id}
          onMute={(userId) => muteParticipant(userId)}
          onKick={(userId) => kickParticipant(userId)}
          onPromote={(userId) => promoteToModerator(userId)}
        />
      </div>

      {/* Session Analytics */}
      <div className="session-analytics">
        <h3>Session Analytics</h3>
        <EngagementChart sessionId={currentSession?.id} />
        <AttendanceChart sessionId={currentSession?.id} />
      </div>
    </div>
  );
};
```

### **Student Learning Interface**
```typescript
// Student Learning Portal
export const StudentLearningPortal: React.FC = () => {
  const [enrolledSessions, setEnrolledSessions] = useState<EnrolledSession[]>([]);
  const [upcomingSessions, setUpcomingSessions] = useState<LiveSession[]>([]);

  return (
    <div className="student-portal">
      <div className="upcoming-sessions">
        <h2>Upcoming Live Sessions</h2>
        {upcomingSessions.map(session => (
          <SessionPreview
            key={session.id}
            session={session}
            onJoin={() => joinSession(session.id)}
            onEnroll={() => enrollInSession(session.id)}
          />
        ))}
      </div>

      <div className="session-history">
        <h2>Completed Sessions</h2>
        {enrolledSessions.map(session => (
          <CompletedSessionCard
            key={session.id}
            session={session}
            onViewRecording={() => viewRecording(session.recordingUrl)}
            onDownloadCertificate={() => downloadCertificate(session.id)}
          />
        ))}
      </div>

      <div className="cpd-progress">
        <h2>CPD Progress</h2>
        <CPDProgressChart />
        <CPDCreditSummary />
      </div>
    </div>
  );
};
```

---

## 🎬 **Recording Management & Distribution**

### **Automatic Recording Processing**
```typescript
// Recording Processing Service
export class RecordingService {
  private bbb: BigBlueButtonService;
  private storage: StorageService;

  async processSessionRecordings(sessionId: string): Promise<void> {
    const session = await this.db.getLiveSession(sessionId);
    
    // Wait for BBB processing (typically 5-15 minutes)
    const recordings = await this.waitForRecordings(session.bbbMeetingId);

    for (const recording of recordings) {
      // Create database record
      const recordingData = await this.db.createRecording({
        sessionId: sessionId,
        bbbRecordId: recording.recordId,
        title: recording.name,
        duration: recording.duration,
        size: recording.size,
        formats: recording.playbackFormats,
        processedAt: new Date(),
        status: 'available'
      });

      // Generate thumbnails
      await this.generateThumbnails(recordingData);

      // Create video chapters
      await this.createVideoChapters(recordingData);

      // Index for search
      await this.indexRecordingContent(recordingData);

      // Notify participants
      await this.notifyRecordingAvailable(sessionId, recordingData);
    }
  }

  async generateDownloadableFormats(recordingId: string): Promise<void> {
    const recording = await this.db.getRecording(recordingId);
    
    // Generate MP4 for offline viewing
    const mp4Url = await this.convertToMP4(recording.bbbRecordId);
    
    // Generate audio-only version
    const audioUrl = await this.extractAudio(recording.bbbRecordId);
    
    // Update recording with download links
    await this.db.updateRecording(recordingId, {
      downloadUrls: {
        video: mp4Url,
        audio: audioUrl,
        slides: recording.slidesUrl
      }
    });
  }

  private async waitForRecordings(meetingId: string): Promise<Recording[]> {
    let attempts = 0;
    const maxAttempts = 20; // 10 minutes max wait
    
    while (attempts < maxAttempts) {
      const recordings = await this.bbb.getRecordings(meetingId);
      
      if (recordings.length > 0 && recordings[0].state === 'published') {
        return recordings;
      }
      
      await new Promise(resolve => setTimeout(resolve, 30000)); // Wait 30 seconds
      attempts++;
    }
    
    throw new Error('Recording processing timeout');
  }
}
```

### **Recording Distribution System**
```yaml
Distribution Channels:
  Portal Access: Immediate access via ACPN portal
  Mobile App: Offline download for mobile learning
  WhatsApp Sharing: Direct links to enrolled participants
  Email Notifications: Recording available alerts
  
Access Control:
  Enrollment Required: Only enrolled participants can access
  Time-Limited Access: Optional expiration for sensitive content
  Download Permissions: Configurable download rights
  Analytics Tracking: View and engagement analytics
  
Format Options:
  Web Playback: HTML5 video player with chapters
  Mobile Download: MP4 format for offline viewing
  Audio Only: MP3 extraction for audio learning
  Transcript: Searchable text version of content
```

---

## 📊 **Analytics & Engagement Tracking**

### **Real-Time Session Analytics**
```typescript
// Session Analytics Service
export class SessionAnalyticsService {
  async trackParticipantEngagement(sessionId: string): Promise<EngagementMetrics> {
    const meetingInfo = await this.bbb.getMeetingInfo(sessionId);
    
    return {
      totalParticipants: meetingInfo.participantCount,
      averageAttendance: this.calculateAverageAttendance(sessionId),
      engagementScore: this.calculateEngagementScore(sessionId),
      interactionMetrics: {
        chatMessages: await this.getChatMessageCount(sessionId),
        pollResponses: await this.getPollResponseCount(sessionId),
        questionsAsked: await this.getQuestionCount(sessionId),
        handsRaised: await this.getHandRaiseCount(sessionId)
      },
      attentionMetrics: {
        averageViewTime: this.calculateAverageViewTime(sessionId),
        dropoffPoints: this.identifyDropoffPoints(sessionId),
        reengagementEvents: this.trackReengagementEvents(sessionId)
      }
    };
  }

  async generateSessionReport(sessionId: string): Promise<SessionReport> {
    const session = await this.db.getLiveSession(sessionId);
    const attendance = await this.db.getSessionAttendance(sessionId);
    const engagement = await this.trackParticipantEngagement(sessionId);

    return {
      sessionInfo: {
        title: session.title,
        instructor: session.instructor,
        date: session.startTime,
        duration: session.actualDuration
      },
      attendanceMetrics: {
        registered: session.registeredCount,
        attended: attendance.length,
        completionRate: this.calculateCompletionRate(attendance),
        averageDuration: this.calculateAverageDuration(attendance)
      },
      engagementMetrics: engagement,
      cpdCredits: {
        eligible: attendance.filter(a => a.qualifiesForCredit).length,
        assigned: await this.getCPDCreditsAssigned(sessionId)
      },
      feedback: await this.getSessionFeedback(sessionId)
    };
  }
}
```

### **Learning Analytics Dashboard**
```typescript
// Analytics Dashboard Component
export const AnalyticsDashboard: React.FC = () => {
  return (
    <div className="analytics-dashboard">
      {/* Session Overview */}
      <div className="session-overview">
        <MetricCard
          title="Total Sessions"
          value={totalSessions}
          trend="+12%"
        />
        <MetricCard
          title="Total Participants"
          value={totalParticipants}
          trend="+18%"
        />
        <MetricCard
          title="Average Engagement"
          value={`${averageEngagement}%`}
          trend="+5%"
        />
        <MetricCard
          title="CPD Credits Issued"
          value={cpdCreditsIssued}
          trend="+22%"
        />
      </div>

      {/* Engagement Charts */}
      <div className="engagement-charts">
        <EngagementTrendsChart />
        <AttendanceHeatmap />
        <ParticipationDistribution />
      </div>

      {/* Session Performance */}
      <div className="session-performance">
        <SessionPerformanceTable />
        <InstructorRankings />
        <PopularTopics />
      </div>
    </div>
  );
};
```

---

## 🔒 **Security & Privacy**

### **Meeting Security Configuration**
```yaml
Access Control:
  Authentication: ACPN portal SSO required
  Guest Access: Disabled by default
  Waiting Room: Optional for additional security
  Meeting Lock: Prevent unauthorized entry
  
Content Protection:
  Recording Encryption: Encrypted storage and transmission
  Watermarking: User identification on screen shares
  Download Restrictions: Configurable download permissions
  Access Logging: Comprehensive audit trail
  
Privacy Settings:
  Anonymous Participation: Optional anonymous mode
  Data Retention: Configurable recording retention period
  GDPR Compliance: EU data protection compliance
  Consent Management: Recording consent tracking
```

### **Data Protection Implementation**
```typescript
// Privacy & Security Service
export class BBBSecurityService {
  async createSecureMeeting(params: SecureMeetingParams): Promise<SecureMeeting> {
    // Generate cryptographically secure passwords
    const attendeePassword = this.generateSecurePassword();
    const moderatorPassword = this.generateSecurePassword();
    
    // Create meeting with security settings
    const meeting = await this.bbb.createMeeting({
      ...params,
      attendeePassword,
      moderatorPassword,
      guestPolicy: 'ALWAYS_DENY',
      lockSettingsDisableCam: false,
      lockSettingsDisableMic: true,
      lockSettingsDisablePrivateChat: false,
      lockSettingsDisablePublicChat: false,
      lockSettingsDisableNote: false,
      lockSettingsLockedLayout: false,
      allowModsToUnmuteUsers: true,
      allowModsToEjectUsers: true,
      meetingKeepEvents: true
    });

    // Log meeting creation for audit
    await this.auditLog.log({
      action: 'meeting_created',
      meetingId: meeting.meetingId,
      createdBy: params.createdBy,
      timestamp: new Date(),
      securityLevel: 'high'
    });

    return meeting;
  }

  async validateParticipantAccess(
    meetingId: string, 
    userId: string
  ): Promise<boolean> {
    // Check enrollment status
    const enrollment = await this.db.getSessionEnrollment(meetingId, userId);
    if (!enrollment) return false;

    // Check user status
    const user = await this.db.getUser(userId);
    if (!user.isActive || user.isSuspended) return false;

    // Check time constraints
    const session = await this.db.getLiveSession(meetingId);
    const now = new Date();
    const sessionStart = new Date(session.startTime);
    const sessionEnd = new Date(session.endTime);
    
    // Allow 15 minutes before and after session
    const accessStart = new Date(sessionStart.getTime() - 15 * 60 * 1000);
    const accessEnd = new Date(sessionEnd.getTime() + 15 * 60 * 1000);
    
    return now >= accessStart && now <= accessEnd;
  }
}
```

---

## 🚀 **Performance Optimization**

### **Server Optimization**
```bash
#!/bin/bash
# BBB Performance Optimization Script

# CPU Governor Setting
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Memory Optimization
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf

# Network Optimization
echo 'net.core.rmem_default = 262144' | sudo tee -a /etc/sysctl.conf
echo 'net.core.rmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
echo 'net.core.wmem_default = 262144' | sudo tee -a /etc/sysctl.conf
echo 'net.core.wmem_max = 16777216' | sudo tee -a /etc/sysctl.conf

# FreeSWITCH Optimization
sudo sed -i 's/DAEMON_ARGS="-nonat"/DAEMON_ARGS="-nonat -nort"/' /etc/default/freeswitch

# Kurento Optimization
echo 'kurento.MaxConcurrentConnections=500' | sudo tee -a /etc/kurento/modules/kurento/WebRtcEndpoint.conf.ini

# Apply changes
sudo sysctl -p
sudo systemctl restart freeswitch
sudo systemctl restart kurento-media-server
sudo bbb-conf --restart
```

### **Monitoring & Health Checks**
```typescript
// BBB Health Monitoring
export class BBBHealthMonitor {
  async checkSystemHealth(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkBBBService(),
      this.checkFreeSWITCH(),
      this.checkKurento(),
      this.checkNginx(),
      this.checkRedis(),
      this.checkMongoDB(),
      this.checkDiskSpace(),
      this.checkMemoryUsage(),
      this.checkCPULoad()
    ]);

    return {
      overall: this.calculateOverallHealth(checks),
      services: this.formatServiceStatus(checks),
      timestamp: new Date(),
      recommendations: this.generateRecommendations(checks)
    };
  }

  async monitorActiveSession(meetingId: string): Promise<SessionHealth> {
    const meetingInfo = await this.bbb.getMeetingInfo(meetingId);
    
    return {
      participantCount: meetingInfo.participantCount,
      audioQuality: await this.assessAudioQuality(meetingId),
      videoQuality: await this.assessVideoQuality(meetingId),
      networkLatency: await this.measureNetworkLatency(meetingId),
      resourceUsage: await this.getResourceUsage(),
      issues: await this.detectIssues(meetingId)
    };
  }
}
```

---

## 🔄 **Backup & Disaster Recovery**

### **Recording Backup Strategy**
```yaml
Backup Levels:
  Real-time Replication: Live recording duplication
  Daily Backups: Complete recording archive
  Weekly Snapshots: System configuration backup
  Monthly Archives: Long-term storage migration
  
Storage Locations:
  Primary: Local SSD storage for active recordings
  Secondary: Network attached storage for recent recordings
  Tertiary: Cloud storage for long-term archival
  Geographic: Off-site backup for disaster recovery
  
Recovery Procedures:
  Recording Recovery: Restore from multiple backup sources
  System Recovery: Complete BBB server restoration
  Configuration Recovery: Settings and customization restore
  Data Integrity: Checksum validation for all backups
```

### **High Availability Setup**
```yaml
Cluster Configuration:
  Load Balancer: HAProxy for meeting distribution
  Multiple BBB Servers: Horizontal scaling for capacity
  Database Clustering: MongoDB replica set
  Shared Storage: Network file system for recordings
  
Failover Process:
  Health Monitoring: Continuous service health checks
  Automatic Failover: Seamless server switching
  Session Migration: Transfer active sessions if possible
  Notification System: Admin alerts for service issues
```

---

This BigBlueButton integration provides a robust, secure, and scalable virtual learning environment that seamlessly integrates with the ACPN portal for comprehensive continuing professional development delivery.