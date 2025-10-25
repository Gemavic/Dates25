# Dates.care Admin Guide

**Administrator & Staff Operations Manual**

**Version:** 1.0.0
**Last Updated:** October 25, 2025
**Access Level:** Staff Only

---

## Table of Contents

1. [Admin Access](#admin-access)
2. [User Management](#user-management)
3. [Content Moderation](#content-moderation)
4. [Verification System](#verification-system)
5. [Credit & Payment Management](#credit--payment-management)
6. [Monitoring & Analytics](#monitoring--analytics)
7. [Support Operations](#support-operations)
8. [Security & Compliance](#security--compliance)
9. [System Maintenance](#system-maintenance)
10. [Emergency Procedures](#emergency-procedures)

---

## Admin Access

### Staff Panel Login

**URL:** `https://dates.care/#staff-panel`

**Access Credentials:**
- Reserved for authorized staff members only
- Contact IT administrator for account setup

**Database Table:** `staff_members`

### Staff Roles

| Role | Permissions | Description |
|------|-------------|-------------|
| **Super Admin** | Full access | System configuration, staff management |
| **Admin** | Most features | User management, moderation, reports |
| **Moderator** | Content moderation | Review photos, handle reports |
| **Support** | Limited access | View profiles, assist users |

### Adding Staff Members

```sql
INSERT INTO staff_members (
  staff_id,
  email,
  role,
  is_active
) VALUES (
  'staff-uuid',
  'admin@dates.care',
  'admin',
  true
);
```

**Required Information:**
- Email address
- Role assignment
- Full name
- Department

---

## User Management

### Viewing User Profiles

**Access:** Staff Panel → Users → Search

**Available Information:**
- Profile details
- Activity history
- Credit balance
- Transaction history
- Match history
- Reported incidents
- Verification status

### User Search Filters

- **Email address**
- **User ID**
- **Name**
- **Registration date**
- **Verification status**
- **Account status** (Active/Suspended/Banned)
- **Last active**

### User Actions

#### Suspend User Account

**Reason Required:**
- Policy violation
- Suspicious activity
- User request
- Payment fraud
- Safety concern

**Process:**
1. Access user profile
2. Click "Suspend Account"
3. Select reason
4. Enter duration (days) or permanent
5. Add admin notes
6. Confirm suspension

**Database Query:**
```sql
UPDATE user_profiles
SET account_status = 'suspended',
    suspension_reason = 'Policy violation',
    suspended_until = NOW() + INTERVAL '30 days',
    suspended_by = 'admin-id'
WHERE user_id = 'user-id';
```

#### Ban User Account

**Permanent removal for:**
- Severe policy violations
- Illegal activity
- Repeated offenses
- Scamming/fraud
- Safety threats

**Process:**
1. Review violation evidence
2. Verify with supervisor (for permanent bans)
3. Access user profile
4. Click "Ban Account"
5. Document reason
6. Confirm ban

**Effect:**
- Account permanently disabled
- Profile hidden
- Cannot create new account with same email
- All active sessions terminated

#### Reinstate User Account

**For:**
- Suspension period ended
- False positive suspension
- User appeal approved

**Process:**
1. Review original suspension reason
2. Check for repeat violations
3. Click "Reinstate Account"
4. Document decision
5. Optional: Send notification to user

### Bulk Actions

**Export User Data:**
```sql
SELECT user_id, email, full_name, registration_date, account_status
FROM user_profiles
WHERE registration_date >= '2025-01-01'
ORDER BY registration_date DESC;
```

**Bulk Suspension:**
- Use for coordinated spam attacks
- Requires supervisor approval
- Document all affected accounts

---

## Content Moderation

### Photo Approval Queue

**Access:** Staff Panel → Moderation → Photos

**Review Process:**

1. **Check Photo Quality**
   - Clear, not blurry
   - Good lighting
   - Face visible (if profile photo)

2. **Verify Compliance**
   - No nudity or explicit content
   - No offensive gestures
   - No weapons or drugs
   - No copyrighted material
   - No minors in photo

3. **Take Action**
   - ✅ Approve
   - ❌ Reject (with reason)
   - 🚫 Report for review

**Database Table:** `user_photos`

```sql
-- Approve photo
UPDATE user_photos
SET is_approved = true,
    reviewed_by = 'staff-id',
    reviewed_at = NOW()
WHERE id = 'photo-id';

-- Reject photo
UPDATE user_photos
SET is_approved = false,
    rejection_reason = 'Inappropriate content',
    reviewed_by = 'staff-id',
    reviewed_at = NOW()
WHERE id = 'photo-id';
```

### Profile Content Review

**Red Flags:**
- Incomplete profiles with suspicious activity
- Copy-paste bios
- External contact info in bio
- Solicitation language
- Fake or stock photos
- Age mismatches

**Actions:**
- Request profile update
- Temporary visibility restriction
- Manual verification requirement
- Account suspension

### Reported Content

**Access:** Staff Panel → Reports → Review Queue

**Report Categories:**
1. Fake Profile
2. Inappropriate Content
3. Harassment
4. Scam Attempt
5. Underage User
6. Other

**Review Process:**

1. **Examine Report**
   - Reporter's description
   - Evidence provided
   - Report history

2. **Investigate**
   - View reported user's profile
   - Check message history (if applicable)
   - Review user's history

3. **Take Action**
   - Dismiss (false report)
   - Warning to user
   - Remove content
   - Suspend account
   - Ban account

4. **Document**
   - Record decision
   - Update report status
   - Notify reporter (if appropriate)

**Database Table:** `user_reports` (to be created)

---

## Verification System

### Verification Request Queue

**Access:** Staff Panel → Verification → Pending Requests

**Review Process:**

#### Step 1: ID Document Verification

**Check:**
- ✅ Document is valid government-issued ID
- ✅ Document is not expired
- ✅ Photo matches selfie
- ✅ Name matches profile
- ✅ Date of birth confirms 18+
- ❌ Document is clear and legible

**Red Flags:**
- Blurry or obscured documents
- Photoshopped documents
- Screenshots instead of original
- Information doesn't match profile

#### Step 2: Selfie Verification

**Check:**
- ✅ Face matches ID photo
- ✅ Person is holding ID or verification code
- ✅ Photo is recent (not old photo)
- ✅ Clear, well-lit, not blurry

**Red Flags:**
- Different person than ID
- Screenshot of someone else
- Obvious editing/filters
- Covering face

#### Step 3: Decision

**Approve:**
```sql
UPDATE user_verification
SET is_verified = true,
    verification_date = NOW(),
    verification_method = 'photo_id',
    verified_by = 'staff-id'
WHERE user_id = 'user-id';

UPDATE user_profiles
SET is_verified = true
WHERE user_id = 'user-id';
```

**Reject:**
```sql
UPDATE verification_requests
SET status = 'rejected',
    rejection_reason = 'ID and selfie do not match',
    reviewed_by = 'staff-id',
    reviewed_at = NOW()
WHERE id = 'request-id';
```

**Request Resubmission:**
- Poor photo quality
- Unclear documents
- Missing information

### Biometric Verification

**Advanced Verification (Optional):**
- Facial recognition matching
- Liveness detection
- Document authenticity check

**Database Table:** `biometric_data`

---

## Credit & Payment Management

### Transaction Monitoring

**Access:** Staff Panel → Transactions → Monitor

**Watch For:**
- Unusual purchase patterns
- Chargebacks
- Failed transactions
- Refund requests
- High-value transactions

### Refund Processing

**Eligibility Check:**
1. Request within 14 days of purchase
2. Credits unused
3. No policy violations
4. Valid payment record

**Process:**
1. Verify purchase
2. Check credit usage
3. Calculate refund amount
4. Process through payment provider
5. Deduct credits from account
6. Document transaction

**Database:**
```sql
-- Check credit usage
SELECT
  purchased_credits,
  (SELECT COUNT(*) FROM credit_transactions
   WHERE user_id = 'user-id'
   AND created_at > purchase_date) as transactions_since_purchase
FROM user_credits
WHERE user_id = 'user-id';

-- Process refund
UPDATE credit_transactions
SET transaction_status = 'refunded',
    refund_date = NOW(),
    refund_processed_by = 'staff-id'
WHERE transaction_id = 'transaction-id';
```

### Fraudulent Activity

**Indicators:**
- Multiple failed payment attempts
- Stolen credit card usage
- Chargebacks
- Unusual spending patterns
- Account sharing

**Response:**
1. Flag account for review
2. Suspend credit purchases
3. Contact payment processor
4. Document evidence
5. Consider account suspension/ban

---

## Monitoring & Analytics

### Key Metrics Dashboard

**User Metrics:**
- Total registered users
- Daily active users (DAU)
- Monthly active users (MAU)
- New registrations (today, week, month)
- User retention rates

**Engagement Metrics:**
- Profile views
- Likes sent/received
- Matches made
- Messages sent
- Average session duration

**Revenue Metrics:**
- Credit purchases (daily, weekly, monthly)
- Average purchase value
- Revenue per user
- Refund rate
- Payment success rate

**System Health:**
- Error rate
- API response time
- Database performance
- Storage usage
- Bandwidth usage

### SQL Queries for Analytics

**Daily Active Users:**
```sql
SELECT COUNT(DISTINCT user_id) as dau
FROM user_activity
WHERE activity_date = CURRENT_DATE;
```

**New Registrations Today:**
```sql
SELECT COUNT(*) as new_users
FROM user_profiles
WHERE DATE(created_at) = CURRENT_DATE;
```

**Matches Made Today:**
```sql
SELECT COUNT(*) as matches_today
FROM matches
WHERE DATE(created_at) = CURRENT_DATE;
```

**Revenue Today:**
```sql
SELECT SUM(amount) as revenue
FROM credit_transactions
WHERE transaction_type = 'purchase'
  AND DATE(created_at) = CURRENT_DATE;
```

### Monitoring Tools

**Recommended:**
- **Sentry** - Error tracking
- **Google Analytics** - User behavior
- **Supabase Dashboard** - Database monitoring
- **Vercel Analytics** - Performance monitoring

---

## Support Operations

### Support Ticket System

**Access:** Staff Panel → Support → Tickets

**Ticket Categories:**
1. Account Issues
2. Payment Problems
3. Technical Support
4. Profile/Photo Issues
5. Verification Questions
6. Safety Concerns
7. General Inquiries

**Priority Levels:**
- **Critical** - Account access, payment errors (< 4 hours)
- **High** - Verification delays, matching issues (< 24 hours)
- **Medium** - Profile questions, feature requests (< 48 hours)
- **Low** - General questions (< 72 hours)

### Response Templates

**Account Access:**
```
Subject: Account Access Assistance

Hi [Name],

Thank you for contacting Dates.care support.

I've reviewed your account and [specific action taken].

[Resolution details]

If you continue to experience issues, please reply to this email.

Best regards,
[Staff Name]
Dates.care Support Team
```

**Payment Issues:**
```
Subject: Payment Issue Resolution

Hi [Name],

I've investigated your payment issue regarding [transaction details].

[Explanation of issue]
[Resolution or next steps]
[Refund information if applicable]

Transaction ID: [ID]
Amount: [Amount]

Please allow 5-7 business days for refunds to appear.

Best regards,
[Staff Name]
```

### Escalation Procedures

**Escalate When:**
- Legal issues mentioned
- Safety threats
- Media inquiries
- Technical issues beyond support scope
- Repeated complaints not resolved

**Escalation Path:**
1. Support → Senior Support
2. Senior Support → Department Manager
3. Department Manager → Director
4. Director → Legal/Executive (if needed)

---

## Security & Compliance

### Data Access Logs

**Monitoring:**
- All admin actions are logged
- Regular audit reviews
- Suspicious activity alerts

**Database Table:** `security_audit_log`

```sql
-- Log admin action
INSERT INTO security_audit_log (
  staff_id,
  action_type,
  target_user_id,
  action_description,
  ip_address,
  created_at
) VALUES (
  'staff-id',
  'account_suspension',
  'user-id',
  'Suspended account for policy violation',
  '192.168.1.1',
  NOW()
);
```

### Privacy Compliance

**PIPEDA Requirements:**
- User consent for data collection
- Right to access personal data
- Right to correct information
- Right to delete account
- Data breach notification (72 hours)

**Data Access Requests:**
1. Verify user identity
2. Compile user data from all tables
3. Export in machine-readable format
4. Deliver within 30 days

**Data Deletion Requests:**
1. Verify user identity
2. Backup data (legal requirement)
3. Anonymize user data
4. Delete personal information
5. Confirm completion

### Security Incident Response

**If Breach Detected:**
1. **Contain** - Isolate affected systems
2. **Assess** - Determine scope and impact
3. **Notify** - Alert security team and management
4. **Document** - Log all details
5. **Remediate** - Fix vulnerability
6. **Report** - Notify affected users within 72 hours
7. **Review** - Post-incident analysis

---

## System Maintenance

### Regular Maintenance Tasks

**Daily:**
- Review error logs
- Monitor system performance
- Check payment processing
- Review moderation queue

**Weekly:**
- User growth report
- Revenue report
- Top issues review
- Staff performance review

**Monthly:**
- Database optimization
- Security audit
- Compliance review
- Feature usage analysis
- Backup verification

### Database Maintenance

**Backup Schedule:**
- Automatic daily backups (Supabase)
- Weekly manual verification
- Monthly archive to cold storage

**Optimization:**
```sql
-- Vacuum tables
VACUUM ANALYZE user_profiles;
VACUUM ANALYZE chat_messages;
VACUUM ANALYZE credit_transactions;

-- Rebuild indexes
REINDEX TABLE user_profiles;
REINDEX TABLE matches;
```

### Performance Monitoring

**Key Indicators:**
- API response time < 200ms
- Database query time < 50ms
- Page load time < 2 seconds
- Error rate < 0.5%
- Uptime > 99.9%

**Alerts Setup:**
- Error rate spike
- Slow query detection
- Storage usage > 80%
- CPU usage > 80%
- Failed payment spike

---

## Emergency Procedures

### Site Down

**Immediate Actions:**
1. Check Vercel status: https://vercel.com/status
2. Check Supabase status: https://status.supabase.com/
3. Review error logs in Vercel dashboard
4. Check recent deployments
5. Notify team via Slack/emergency channel

**Resolution:**
- If deployment issue: Rollback to previous version
- If database issue: Contact Supabase support
- If DDoS attack: Enable Vercel protection
- Document incident for post-mortem

### Database Issues

**Symptoms:**
- Slow queries
- Connection timeouts
- Data inconsistencies

**Actions:**
1. Check Supabase dashboard
2. Review slow query log
3. Check connection pool
4. Verify RLS policies
5. Contact Supabase support if needed

### Payment System Failure

**Actions:**
1. Check payment processor status
2. Review recent transactions
3. Disable credit purchases temporarily
4. Notify users via in-app message
5. Document affected transactions
6. Process refunds if necessary

### Data Breach

**CRITICAL - Follow Security Incident Response**

1. **Immediate Containment**
2. **Notify Management**
3. **Assess Impact**
4. **Preserve Evidence**
5. **Remediate**
6. **User Notification** (within 72 hours)
7. **Report to Privacy Commissioner** (if required)

### Contact Information

**Emergency Contacts:**
- CTO: [Contact Info]
- Security Lead: [Contact Info]
- Legal Counsel: [Contact Info]
- Supabase Support: https://supabase.com/dashboard/support
- Vercel Support: https://vercel.com/support

---

## Best Practices

### Admin Responsibilities

1. **User Privacy** - Never share user information
2. **Professionalism** - Maintain professional communication
3. **Documentation** - Document all actions and decisions
4. **Consistency** - Apply policies uniformly
5. **Security** - Protect admin credentials
6. **Continuous Learning** - Stay updated on policies

### Common Mistakes to Avoid

❌ Making decisions without documentation
❌ Discussing users publicly
❌ Sharing admin credentials
❌ Acting on emotions
❌ Ignoring escalation procedures
❌ Skipping verification steps

### Tips for Success

✅ Use templates for common responses
✅ Keep detailed notes
✅ Double-check before banning users
✅ Be empathetic with users
✅ Ask for help when unsure
✅ Follow up on escalated issues

---

## Training Resources

### Required Reading
- Terms of Service
- Privacy Policy
- Misconduct Prevention Policy
- This Admin Guide
- API Documentation

### Training Modules
1. Platform Overview
2. Content Moderation
3. Verification Process
4. Support Best Practices
5. Security & Privacy

### Ongoing Education
- Monthly team meetings
- Policy update reviews
- Industry best practices
- Legal compliance updates

---

## Appendix

### Keyboard Shortcuts

**Staff Panel:**
- `Ctrl/Cmd + K` - Quick search
- `Ctrl/Cmd + M` - Moderation queue
- `Ctrl/Cmd + R` - Reports
- `Ctrl/Cmd + V` - Verification queue
- `Ctrl/Cmd + U` - User search

### SQL Quick Reference

**Find user by email:**
```sql
SELECT * FROM user_profiles WHERE email = 'user@example.com';
```

**Check user credits:**
```sql
SELECT * FROM user_credits WHERE user_id = 'user-id';
```

**View recent matches:**
```sql
SELECT * FROM matches
WHERE created_at > NOW() - INTERVAL '7 days'
ORDER BY created_at DESC;
```

**Active users today:**
```sql
SELECT COUNT(DISTINCT user_id) FROM user_activity
WHERE activity_date = CURRENT_DATE;
```

---

**This guide is confidential and for authorized staff only.**

*Last Updated: October 25, 2025*
*Version: 1.0.0*
*Classification: Internal Use Only*
