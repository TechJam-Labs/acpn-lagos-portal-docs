# GitBook Setup Guide for ACPN Lagos Portal Documentation

> **Copyright © 2024 TechJamLabs**  
> Website: [www.techjamlabs.com](https://www.techjamlabs.com)  
> Phone: +234 201 330 9089 | +1 206 710 0170  
> **Authors:** Ben Adenle, Odinaka Solomon, Dare Oloruntoba

---

## 🎯 **Overview**

This guide provides step-by-step instructions for setting up and uploading the ACPN Lagos Portal documentation to GitBook. The documentation is organized into 6 core applications with comprehensive architectural diagrams and business requirements.

---

## 📂 **Documentation Structure**

The documentation is organized as follows:

```
@/gitbook-docs/
├── README.md                          # Main introduction
├── SUMMARY.md                         # Navigation structure
├── system-overview.md                 # System overview
├── .gitbook.yaml                      # GitBook configuration
├── applications/                      # 6 Core Applications
│   ├── membership-management/
│   ├── pharmacy-stores/
│   ├── learning-cpd/
│   ├── internal-trade-b2b/
│   ├── external-trade-b2c/
│   └── events-management/
├── shared/                           # Shared Components
│   ├── architecture/
│   ├── data-models/
│   ├── security/
│   ├── integrations/
│   ├── infrastructure/
│   └── api-reference/
└── [other legacy folders]            # Existing documentation
```

---

## 🚀 **GitBook Setup Steps**

### **Step 1: GitBook Account Setup**

1. **Create GitBook Account**
   - Visit [GitBook.com](https://www.gitbook.com)
   - Sign up with TechJamLabs email account
   - Choose the appropriate plan (Team or Business)

2. **Create Organization**
   - Organization Name: "TechJamLabs"
   - Description: "Innovative Technology Solutions"
   - Website: www.techjamlabs.com

### **Step 2: Create New Space**

1. **Space Configuration**
   - Space Name: "ACPN Lagos Portal Documentation"
   - Description: "Comprehensive documentation for the Association of Community Pharmacists of Nigeria (Lagos Chapter) Portal"
   - Visibility: Private (initially) or Public
   - Template: Import from Git

2. **Import Settings**
   - Source: GitHub Repository
   - Repository: techjamlabs/acpn-portal-docs
   - Branch: main
   - Root Directory: `/`

### **Step 3: Git Repository Setup**

1. **Initialize Git Repository**
   ```bash
   cd @/gitbook-docs
   git init
   git add .
   git commit -m "Initial commit: ACPN Portal Documentation"
   ```

2. **Connect to GitHub**
   ```bash
   git remote add origin https://github.com/techjamlabs/acpn-portal-docs.git
   git branch -M main
   git push -u origin main
   ```

3. **Configure GitBook Integration**
   - Go to GitBook Space Settings
   - Navigate to "Integrations" tab
   - Connect GitHub repository
   - Set up automatic sync

---

## ⚙️ **GitBook Configuration**

### **Space Settings**

1. **General Settings**
   - Title: "ACPN Lagos Portal Documentation"
   - Description: "Complete technical and business documentation for the ACPN Lagos pharmaceutical management platform"
   - Logo: Upload TechJamLabs/ACPN logo
   - Favicon: Upload custom favicon

2. **Navigation Settings**
   - Use SUMMARY.md for navigation structure
   - Enable search functionality
   - Configure table of contents

3. **Theme & Branding**
   - Primary Color: #1E40AF (Professional Blue)
   - Secondary Color: #059669 (Pharmaceutical Green)
   - Font: Inter or System Default
   - Custom CSS (if needed)

### **Advanced Features**

1. **Enable Plugins**
   - Search: Enhanced search functionality
   - Mermaid: For architectural diagrams
   - Code Highlighting: For code blocks
   - Page TOC: Table of contents for pages
   - Back to Top: Navigation helper

2. **Custom Domain (Optional)**
   - Domain: docs.acpnlagos.org
   - SSL Certificate: Automatic via GitBook
   - DNS Configuration: Point CNAME to GitBook

---

## 📋 **Pre-Upload Checklist**

### **Content Verification**
- [ ] All 6 application overviews are complete
- [ ] Copyright information is on all major documents
- [ ] SUMMARY.md navigation structure is correct
- [ ] All internal links are working
- [ ] Mermaid diagrams are rendering properly
- [ ] Images and assets are properly referenced

### **Technical Verification**
- [ ] .gitbook.yaml configuration is correct
- [ ] All markdown files use consistent formatting
- [ ] Code blocks have proper syntax highlighting
- [ ] No broken external links
- [ ] File structure matches SUMMARY.md

### **Business Verification**
- [ ] All contact information is accurate
- [ ] TechJamLabs branding is consistent
- [ ] Author credits are properly attributed
- [ ] Copyright notices are complete
- [ ] Business requirements are comprehensive

---

## 🔄 **Upload Process**

### **Method 1: Git Integration (Recommended)**

1. **Push to GitHub**
   ```bash
   cd @/gitbook-docs
   git add .
   git commit -m "Complete documentation ready for GitBook"
   git push origin main
   ```

2. **GitBook Auto-Sync**
   - GitBook will automatically detect changes
   - Review the sync status in GitBook dashboard
   - Verify all content is properly imported

### **Method 2: Direct Upload**

1. **Export Documentation**
   - Create ZIP file of @/gitbook-docs content
   - Exclude .git folder and other unnecessary files

2. **Import to GitBook**
   - Use GitBook's import feature
   - Upload ZIP file
   - Configure navigation structure

---

## 🔧 **Post-Upload Configuration**

### **Content Organization**

1. **Verify Navigation**
   - Check all menu items are working
   - Ensure proper hierarchy
   - Test internal linking

2. **Update Page Settings**
   - Set page descriptions
   - Configure page visibility
   - Add page tags for better organization

### **SEO & Analytics**

1. **SEO Settings**
   - Meta titles and descriptions
   - Open Graph tags
   - Sitemap generation

2. **Analytics Setup**
   - Google Analytics integration
   - GitBook built-in analytics
   - User engagement tracking

---

## 👥 **Team Access & Permissions**

### **Team Members**
- **Ben Adenle**: Admin (Full access)
- **Odinaka Solomon**: Editor (Content editing)
- **Dare Oloruntoba**: Editor (Content editing)
- **ACPN Lagos Team**: Reviewer (Comment only)

### **Permission Levels**
- **Admin**: Full space management
- **Editor**: Content creation and editing
- **Reviewer**: Comments and suggestions only
- **Viewer**: Read-only access

---

## 📚 **Content Maintenance**

### **Regular Updates**
- Weekly review of content accuracy
- Monthly updates to technical specifications
- Quarterly review of business requirements
- Annual comprehensive documentation review

### **Version Control**
- Use semantic versioning (e.g., v2.0, v2.1)
- Document all major changes
- Maintain changelog
- Archive old versions

---

## 🔍 **Quality Assurance**

### **Content Review Process**
1. **Technical Review**: Verify all technical details
2. **Business Review**: Ensure business requirements are met
3. **Editorial Review**: Check grammar and formatting
4. **Final Approval**: TechJamLabs leadership sign-off

### **Testing Checklist**
- [ ] All links are functional
- [ ] Diagrams render correctly
- [ ] Search functionality works
- [ ] Mobile responsiveness
- [ ] Page load times are acceptable

---

## 📞 **Support & Troubleshooting**

### **Common Issues**
1. **Mermaid Diagrams Not Rendering**
   - Check syntax validity
   - Ensure Mermaid plugin is enabled
   - Verify diagram complexity limits

2. **Broken Links**
   - Use relative paths for internal links
   - Check file and folder names
   - Verify SUMMARY.md structure

3. **Images Not Loading**
   - Ensure images are in repository
   - Use proper relative paths
   - Check file formats (PNG, JPG, SVG)

### **Contact Support**
- **GitBook Support**: support@gitbook.com
- **TechJamLabs Team**: info@techjamlabs.com
- **Documentation Team**: docs@techjamlabs.com

---

## 🎯 **Success Metrics**

### **Launch Targets**
- [ ] Documentation published within 24 hours
- [ ] All navigation working correctly
- [ ] Search functionality operational
- [ ] Mobile optimization complete
- [ ] Team access configured

### **Ongoing Metrics**
- User engagement and page views
- Search query analytics
- User feedback and ratings
- Content freshness and accuracy
- Team collaboration effectiveness

---

**Next Steps**: Once uploaded, share the GitBook URL with stakeholders and begin regular content maintenance schedule.

---

**Copyright © 2024 TechJamLabs. All Rights Reserved.** 