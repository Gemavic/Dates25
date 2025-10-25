# API Documentation - Dates.care

**Version:** 1.0.0
**Last Updated:** October 25, 2025
**Base URL:** `https://dates.care/api`
**Database:** Supabase PostgreSQL
**Authentication:** Supabase Auth (JWT)

---

## Table of Contents

1. [Authentication](#authentication)
2. [User Management](#user-management)
3. [Profile Management](#profile-management)
4. [Discovery & Matching](#discovery--matching)
5. [Messaging](#messaging)
6. [Credits & Payments](#credits--payments)
7. [Verification](#verification)
8. [Gifts](#gifts)
9. [Community Features](#community-features)
10. [Error Handling](#error-handling)

---

## Authentication

### Base Configuration

```typescript
import { supabaseClient } from '@/lib/supabase';

// Supabase URL and Keys are configured via environment variables
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_ANON_KEY;
```

### Sign Up

**Endpoint:** `supabaseClient.auth.signUp()`

```typescript
const { data, error } = await supabaseClient.auth.signUp({
  email: 'user@example.com',
  password: 'secure_password',
  options: {
    data: {
      full_name: 'John Doe',
    },
  },
});
```

**Response:**
```typescript
{
  data: {
    user: {
      id: 'uuid',
      email: 'user@example.com',
      user_metadata: { full_name: 'John Doe' }
    },
    session: { access_token: 'jwt_token', refresh_token: 'refresh_token' }
  },
  error: null
}
```

### Sign In

**Endpoint:** `supabaseClient.auth.signInWithPassword()`

```typescript
const { data, error } = await supabaseClient.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'secure_password',
});
```

### Sign Out

**Endpoint:** `supabaseClient.auth.signOut()`

```typescript
const { error } = await supabaseClient.auth.signOut();
```

### Get Session

```typescript
const { data: { session } } = await supabaseClient.auth.getSession();
```

### Auth State Changes

```typescript
supabaseClient.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    console.log('User signed in:', session.user);
  } else if (event === 'SIGNED_OUT') {
    console.log('User signed out');
  }
});
```

---

## User Management

### Create User Profile

**Function:** `ProfileManager.createProfile()`

```typescript
import { ProfileManager } from '@/lib/database';

const profile = await ProfileManager.createProfile(userId, {
  email: 'user@example.com',
  full_name: 'John Doe',
  age: 28,
  location: 'New York, NY',
  occupation: 'Software Engineer',
  education: 'Bachelor\'s Degree',
  bio: 'Looking for meaningful connections',
  interests: ['Travel', 'Technology', 'Music']
});
```

**Database Table:** `user_profiles`

**Response:**
```typescript
{
  user_id: 'uuid',
  email: 'user@example.com',
  full_name: 'John Doe',
  first_name: 'John',
  age: 28,
  location: 'New York, NY',
  occupation: 'Software Engineer',
  education: 'Bachelor\'s Degree',
  bio: 'Looking for meaningful connections',
  interests: ['Travel', 'Technology', 'Music'],
  profile_visibility: 'public',
  is_verified: false,
  is_online: false,
  created_at: '2025-10-25T00:00:00Z',
  updated_at: '2025-10-25T00:00:00Z'
}
```

### Get User Profile

```typescript
const profile = await ProfileManager.getProfile(userId);
```

### Update User Profile

```typescript
const updatedProfile = await ProfileManager.updateProfile(userId, {
  bio: 'Updated bio',
  location: 'Los Angeles, CA',
  interests: ['Travel', 'Photography']
});
```

### Delete User Profile

**Database Query:**
```sql
DELETE FROM user_profiles WHERE user_id = 'uuid';
```

**Note:** Must be authenticated as the profile owner.

---

## Profile Management

### Upload Photos

**Database Table:** `user_photos`

```typescript
// Upload to Supabase Storage
const { data, error } = await supabaseClient
  .storage
  .from('user-photos')
  .upload(`${userId}/${photoId}.jpg`, file);

// Save photo record
const { data: photo } = await supabaseClient
  .from('user_photos')
  .insert({
    user_id: userId,
    photo_url: data.path,
    is_primary: false,
    is_approved: false
  })
  .select()
  .single();
```

### Get User Photos

```typescript
const { data: photos } = await supabaseClient
  .from('user_photos')
  .select('*')
  .eq('user_id', userId)
  .eq('is_approved', true)
  .order('is_primary', { ascending: false });
```

### Set Primary Photo

```typescript
// Unset all primary photos
await supabaseClient
  .from('user_photos')
  .update({ is_primary: false })
  .eq('user_id', userId);

// Set new primary photo
await supabaseClient
  .from('user_photos')
  .update({ is_primary: true })
  .eq('id', photoId);
```

---

## Discovery & Matching

### Get Discovery Profiles

**Function:** `ProfileManager.getDiscoveryProfiles()`

```typescript
const profiles = await ProfileManager.getDiscoveryProfiles(currentUserId, 50);
```

**Database Query:**
```sql
SELECT * FROM user_profiles
WHERE user_id != $1
  AND profile_visibility = 'public'
LIMIT $2;
```

**Response:** Array of profile objects

### Like a User

**Function:** `MatchManager.likeUser()`

```typescript
import { MatchManager } from '@/lib/database';

const like = await MatchManager.likeUser(
  userId,
  targetUserId,
  'like' // or 'super_like', 'pass', 'blink'
);
```

**Database Table:** `user_likes`

### Get Matches

**Function:** `MatchManager.getUserMatches()`

```typescript
const matches = await MatchManager.getUserMatches(userId);
```

**Database Query:**
```sql
SELECT * FROM matches
WHERE (user1_id = $1 OR user2_id = $1)
  AND is_active = true
ORDER BY last_activity DESC;
```

### Get Likes Received

```typescript
const likes = await MatchManager.getLikesReceived(userId);
```

**Database Query:**
```sql
SELECT ul.*, up.*
FROM user_likes ul
JOIN user_profiles up ON ul.user_id = up.user_id
WHERE ul.target_user_id = $1
  AND ul.like_type IN ('like', 'super_like')
ORDER BY ul.created_at DESC;
```

---

## Messaging

### Chat System

#### Get Chat Threads

**Function:** `MessagingManager.getChatThreads()`

```typescript
import { MessagingManager } from '@/lib/database';

const threads = await MessagingManager.getChatThreads(userId);
```

**Database Table:** `chat_threads`

#### Send Chat Message

```typescript
const message = await MessagingManager.sendChatMessage(
  threadId,
  senderId,
  'Hello! How are you?',
  0 // credits_spent
);
```

**Database Table:** `chat_messages`

**Response:**
```typescript
{
  id: 'uuid',
  thread_id: 'uuid',
  sender_id: 'uuid',
  message_text: 'Hello! How are you?',
  credits_spent: 0,
  is_read: false,
  created_at: '2025-10-25T00:00:00Z'
}
```

#### Get Chat Messages

```typescript
const { data: messages } = await supabaseClient
  .from('chat_messages')
  .select('*')
  .eq('thread_id', threadId)
  .order('created_at', { ascending: true });
```

### Mail System

#### Get Mail Threads

```typescript
const threads = await MessagingManager.getMailThreads(userId);
```

**Database Table:** `mail_threads`

#### Send Mail Message

```typescript
const message = await MessagingManager.sendMailMessage(
  threadId,
  senderId,
  'Re: Your message',
  'Thank you for your message...',
  10 // credits_spent
);
```

**Database Table:** `mail_messages`

---

## Credits & Payments

### Get User Credits

**Function:** `CreditManager.getUserCredits()`

```typescript
import { CreditManager } from '@/lib/database';

const credits = await CreditManager.getUserCredits(userId);
```

**Database Table:** `user_credits`

**Response:**
```typescript
{
  user_id: 'uuid',
  complimentary_credits: 20,
  purchased_credits: 0,
  total_kobos: 20,
  created_at: '2025-10-25T00:00:00Z',
  updated_at: '2025-10-25T00:00:00Z'
}
```

### Initialize User Credits

```typescript
const credits = await CreditManager.initializeUserCredits(userId);
```

**Creates:** 20 complimentary credits for new users

### Spend Credits

```typescript
await CreditManager.spendCredits(
  userId,
  10,
  'Mail message sent'
);
```

**Deduction Order:**
1. Purchased credits first
2. Then complimentary credits

### Add Credits

```typescript
await CreditManager.addCredits(
  userId,
  100,
  'Credit package purchase',
  true // isPurchased
);
```

### Get Transaction History

```typescript
const transactions = await CreditManager.getTransactions(userId, 50);
```

**Database Table:** `credit_transactions`

**Response:**
```typescript
[
  {
    id: 'uuid',
    user_id: 'uuid',
    transaction_type: 'spend', // or 'earn'
    amount: 10,
    description: 'Mail message sent',
    created_at: '2025-10-25T00:00:00Z'
  }
]
```

### Credit Pricing

| Feature | Cost (Credits) |
|---------|---------------|
| Browse Profiles | FREE |
| Like | FREE |
| Blink | FREE |
| Super Like | 5 |
| Chat Message | Per minute |
| Mail Message | 10 |
| Send Photo (First) | FREE |
| Send Photo (Additional) | 10 |
| Send Gift | Varies |
| Video Call | Per minute |
| Audio Call | Per minute |

---

## Verification

### Submit Verification Request

**Database Table:** `verification_requests`

```typescript
const { data: request } = await supabaseClient
  .from('verification_requests')
  .insert({
    user_id: userId,
    verification_type: 'photo_id',
    status: 'pending'
  })
  .select()
  .single();
```

### Upload Verification Documents

**Database Table:** `verification_documents`

```typescript
// Upload ID document
const { data: idDoc } = await supabaseClient
  .storage
  .from('verification-docs')
  .upload(`${userId}/id.jpg`, idFile);

// Upload selfie
const { data: selfie } = await supabaseClient
  .storage
  .from('verification-docs')
  .upload(`${userId}/selfie.jpg`, selfieFile);

// Save document records
await supabaseClient
  .from('verification_documents')
  .insert([
    {
      request_id: requestId,
      document_type: 'photo_id',
      document_url: idDoc.path,
      status: 'pending'
    },
    {
      request_id: requestId,
      document_type: 'selfie',
      document_url: selfie.path,
      status: 'pending'
    }
  ]);
```

### Get Verification Status

```typescript
const { data: verification } = await supabaseClient
  .from('user_verification')
  .select('*')
  .eq('user_id', userId)
  .single();
```

**Response:**
```typescript
{
  user_id: 'uuid',
  is_verified: false,
  verification_date: null,
  verification_method: null,
  biometric_verified: false,
  photo_verified: false,
  created_at: '2025-10-25T00:00:00Z'
}
```

---

## Gifts

### Get Gift Catalog

**Function:** `GiftManager.getGiftCatalog()`

```typescript
import { GiftManager } from '@/lib/database';

const gifts = await GiftManager.getGiftCatalog();
```

**Database Table:** `virtual_gifts`

**Response:**
```typescript
[
  {
    id: 'uuid',
    gift_name: 'Red Rose',
    gift_description: 'A classic romantic gesture',
    gift_image_url: 'https://...',
    credit_cost: 10,
    is_active: true,
    popularity_score: 95
  }
]
```

### Send Gift

```typescript
const gift = await GiftManager.sendGift(
  senderId,
  recipientId,
  giftId,
  10, // credits_spent
  'You are amazing!' // optional message
);
```

**Database Table:** `sent_gifts`

### Get Received Gifts

```typescript
const gifts = await GiftManager.getReceivedGifts(userId);
```

**Database Query:**
```sql
SELECT sg.*, vg.*, up.*
FROM sent_gifts sg
JOIN virtual_gifts vg ON sg.gift_id = vg.id
JOIN user_profiles up ON sg.sender_id = up.user_id
WHERE sg.recipient_id = $1
ORDER BY sg.created_at DESC;
```

---

## Community Features

### Newsletter Subscription

**Database Table:** `newsletter_subscriptions`

```typescript
const { data } = await supabaseClient
  .from('newsletter_subscriptions')
  .insert({
    email: 'user@example.com',
    subscribed: true
  })
  .select()
  .single();
```

### Get Newsfeed Posts

**Database Table:** `user_comments` (used for posts)

```typescript
const { data: posts } = await supabaseClient
  .from('user_comments')
  .select('*, user_profiles(*)')
  .order('created_at', { ascending: false })
  .limit(50);
```

### Create Post

```typescript
const { data: post } = await supabaseClient
  .from('user_comments')
  .insert({
    user_id: userId,
    comment_text: 'Just had an amazing date!',
    is_public: true
  })
  .select()
  .single();
```

### Like Post

**Database Table:** `comment_likes`

```typescript
const { data: like } = await supabaseClient
  .from('comment_likes')
  .insert({
    comment_id: postId,
    user_id: userId
  })
  .select()
  .single();
```

---

## Error Handling

### Error Logging

**Database Table:** `error_logs`

```typescript
const { data: log } = await supabaseClient
  .from('error_logs')
  .insert({
    user_id: userId,
    error_type: 'authentication_error',
    error_message: 'Invalid credentials',
    error_stack: JSON.stringify(error.stack),
    request_url: window.location.href,
    user_agent: navigator.userAgent
  })
  .select()
  .single();
```

### Common Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| `PGRST116` | No rows returned | Resource not found |
| `23505` | Unique violation | Duplicate entry |
| `23503` | Foreign key violation | Referenced record missing |
| `42501` | Insufficient privilege | RLS policy blocks access |
| `42P01` | Table doesn't exist | Run migrations |

### Error Response Format

```typescript
{
  error: {
    message: 'Error description',
    code: 'ERROR_CODE',
    details: 'Additional information'
  }
}
```

### Handling Errors

```typescript
try {
  const result = await supabaseClient
    .from('user_profiles')
    .select('*')
    .eq('user_id', userId)
    .single();

  if (result.error) {
    console.error('Database error:', result.error);
    // Handle error appropriately
  }
} catch (error) {
  console.error('Unexpected error:', error);
  // Log to error tracking service
}
```

---

## Rate Limiting

### Supabase Limits

- **API Requests:** 500 requests/second per project
- **Database Connections:** 60 concurrent connections (Free tier)
- **Storage:** 1GB (Free tier)
- **Bandwidth:** 2GB/month (Free tier)

### Recommendations

1. **Implement client-side caching** for frequently accessed data
2. **Use real-time subscriptions** instead of polling
3. **Batch operations** where possible
4. **Implement request debouncing** for user inputs

---

## Real-Time Subscriptions

### Subscribe to Chat Messages

```typescript
const subscription = supabaseClient
  .channel('chat_messages')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'chat_messages',
      filter: `thread_id=eq.${threadId}`
    },
    (payload) => {
      console.log('New message:', payload.new);
    }
  )
  .subscribe();

// Cleanup
subscription.unsubscribe();
```

### Subscribe to Matches

```typescript
const subscription = supabaseClient
  .channel('matches')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'matches',
      filter: `user1_id=eq.${userId},user2_id=eq.${userId}`
    },
    (payload) => {
      console.log('New match:', payload.new);
    }
  )
  .subscribe();
```

---

## Security Best Practices

1. **Never expose service role key** - Only use anon key in client
2. **Always use RLS policies** - Protect all tables
3. **Validate user input** - Both client and server side
4. **Use HTTPS only** - Enforced by Vercel
5. **Implement rate limiting** - Prevent abuse
6. **Log security events** - Monitor for suspicious activity
7. **Keep dependencies updated** - Regular security patches

---

## Support & Resources

### Documentation
- Supabase Docs: https://supabase.com/docs
- React Docs: https://react.dev
- TypeScript Docs: https://typescriptlang.org/docs

### Support
- Email: support@dates.care
- Supabase Dashboard: https://supabase.com/dashboard

### Database Schema
See Supabase Dashboard → Database → Tables for complete schema

---

*Last Updated: October 25, 2025*
*Version: 1.0.0*
*Platform: Dates.care*
