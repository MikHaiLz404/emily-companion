# AIRI Social Network - Agent-to-Agent Communication

## Context

The user wants AIRI to be a **personal AI agent** that:
1. **Represents each user** - AIRI is a "harness for everyone" with personal traits
2. **Connects to other AIRIs** - forming a social network of agents
3. **Shares daily life & news** - agents share local information and events
4. **Acts as a proxy** - contact family, doctor on behalf of the user (but not medical advice)
5. **Brings back invitations** - nearby events, social opportunities

This is like a **decentralized social network for personal AI agents** - similar to Matrix/ActivityPub but for AI agents representing humans.

## Core Concept: AIRI as Personal Proxy

```
┌─────────────────────────────────────────────────────────────┐
│                        Your AIRI                            │
│  - Your personal trait & personality                        │
│  - Your voice to the world                                  │
│  - Your digital proxy for communication                     │
└─────────────────────────────────────────────────────────────┘
         │                                    ▲
         ▼                                    │
┌─────────────────────┐        ┌─────────────────────┐
│   Neighbor's AIRI    │◄──────►│   Doctor's AIRI     │
│   - Local news      │        │   - Appointment     │
│   - Community info  │        │   - Reminders       │
└─────────────────────┘        └─────────────────────┘
         │                                    ▲
         ▼                                    │
┌─────────────────────┐        ┌─────────────────────┐
│   Family's AIRI      │◄──────►│   Friend's AIRI     │
│   - Check-ins        │        │   - Invitations     │
│   - Health updates   │        │   - Activities      │
└─────────────────────┘        └─────────────────────┘
```

## Key Properties

### 1. Personal Trait
- Each AIRI has a unique personality reflecting its user
- Can be configured based on user's preferences, culture, values
- Maintains user's communication style

### 2. Proxy Capabilities
- Can contact others on behalf of the user
- Can receive messages intended for the user
- NOT a medical professional - just a communication conduit

### 3. Social Network
- Discover nearby agents (local network)
- Share relevant daily life information
- Exchange invitations and events
- Maintain relationships with trusted agents

### 4. Privacy-First
- Agents talk privately to their owners only
- No public broadcasting
- User controls what AIRI shares

## MVP Design (Based on Your Answers)

### Architecture for MVP

```
┌─────────────────────────────────────────────────────────────┐
│  Your AIRI                                              │
│  ┌──────────────────────────────────────────────────┐    │
│  │              Social Module                        │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐         │    │
│  │  │Contact  │  │ Social  │  │ Social  │         │    │
│  │  │ Store   │  │ Message │  │ Identity │         │    │
│  │  │         │  │ Store   │  │ Service  │         │    │
│  │  └────┬────┘  └────┬────┘  └────┬────┘         │    │
│  │       │            │            │                │    │
│  │       └────────────┼────────────┘                │    │
│  │                    ▼                             │    │
│  │         ┌──────────────────┐                    │    │
│  │         │  Sync Engine      │ ← Polls contacts  │    │
│  │         └──────────────────┘   every N minutes  │    │
│  └──────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
       │                    │                    │
       ▼                    ▼                    ▼
   ┌─────────┐         ┌─────────┐         ┌─────────┐
   │Neighbor │         │ Family  │         │ Doctor  │
   │  AIRI   │◄────────│  AIRI   │────────►│  AIRI   │
   └─────────┘         └─────────┘         └─────────┘
```

### MVP Properties

| Aspect | MVP Choice | Why |
|--------|-----------|-----|
| **Discovery** | User-initiated | Simplest - users add contacts they trust |
| **Identity** | Phone number style | Unique, memorable, familiar (like Signal) |
| **Communication** | Interval polling | NOT real-time - agents sync periodically |
| **Relationships** | Location-based | Neighbors, local community, trusted contacts |

### MVP Protocol Design

#### Identity Format
```
airis:<country-code><phone-number>
Example: airis:+1234567890 (US)
          airis:+66812345678 (Thailand)
          airis:+819012345678 (Japan)
```

#### Communication Pattern (Polling)
```
Every N minutes (configurable):
  1. AIRI collects recent events from contacts
  2. Fetch new invitations, news, status updates
  3. Present aggregated info to user
  4. Push own updates to contacts (if changed)
```

#### Message Types for MVP

1. **News/Update** - Local community information
2. **Invitation** - Events, gatherings, activities
3. **Status** - "I'm doing well" / "Need help with X"
4. **Message** - Direct communication via proxy

### Data Storage

```
Local IndexedDB:
├── contacts/          - Your trusted AIRI contacts
├── outbox/            - Messages waiting to be sent
├── inbox/             - Received messages (last 100)
├── news/              - Collected news from network
├── invitations/       - Received invitations
└── sync-state/        - Last sync timestamps per contact
```

### MVP Features Priority

1. **Phase 1: Core Messaging**
   - Add contact by phone number
   - Send/receive text messages
   - View conversation history

2. **Phase 2: News Collection**
   - Poll contacts for updates
   - Aggregate news in local feed
   - Filter by relevance/location

3. **Phase 3: Invitations**
   - Share invitations with contacts
   - RSVP tracking
   - Calendar integration

4. **Phase 4: Location Features**
   - Nearest neighbors discovery
   - Community events
   - Local business integrations

### Technical Stack for MVP

| Component | Technology | Reason |
|-----------|------------|--------|
| **Transport** | HTTP/HTTPS + JSON | Simple, firewall-friendly |
| **Discovery** | Contact list + optional mDNS for local | User-initiated simplicity |
| **Storage** | IndexedDB | Already used by AIRI |
| **Identity** | Phone number hash + public key | Privacy-preserving |
| **Sync** | Background fetch (WorkManager/Battery) | Battery-efficient |

### Example MVP Flow

```
User A wants to connect with User B:
1. User A → "Add contact: +66812345678"
2. User B → Receives notification: "Someone wants to connect"
3. User B → Accepts/Declines
4. If accepted → Contacts linked
5. Every 15 minutes:
   - A polls B for new messages/news
   - B polls A for new messages/news
6. New content → Presented to users
```

### Privacy Considerations

- Phone numbers hashed for discovery (not stored in plain)
- Messages encrypted end-to-end
- User controls what AIRI shares with contacts
- Location shared only at user's discretion

---

## Implementation Architecture

### Module Structure (New `social` module)

```
packages/stage-ui/src/stores/modules/
├── social/                    # NEW: Social module
│   ├── index.ts              # Module entry
│   ├── contacts.ts            # Contact store
│   ├── messages.ts            # Message store
│   ├── sync.ts                # Sync engine
│   └── identity.ts            # Local identity

apps/stage-tamagotchi/src/renderer/components/stage-islands/
├── ContactsIsland.vue         # NEW: Contacts overlay
├── SocialFeedIsland.vue       # NEW: News/invitations feed

packages/stage-pages/src/pages/settings/
├── social/                    # NEW: Social settings
│   ├── index.vue             # Social settings hub
│   ├── contacts.vue          # Contact management
│   └── privacy.vue           # Privacy controls
```

### Data Schema

```typescript
// Contact
interface SocialContact {
  id: string // UUID
  phoneHash: string // Hashed phone for discovery
  displayName: string // User-defined name
  airiEndpoint: string // Contact's HTTP endpoint
  publicKey: string // For E2E encryption
  lastSyncAt: number // Timestamp
  trustLevel: 'pending' | 'trusted'
}

// Message
interface SocialMessage {
  id: string
  type: 'news' | 'invitation' | 'status' | 'message'
  from: string // Contact ID
  to: string // Contact ID
  content: string // Encrypted content
  metadata?: {
    location?: string
    eventDate?: string
    rsvp?: boolean
  }
  createdAt: number
  readAt?: number
}

// Local Identity
interface LocalIdentity {
  phoneNumber: string // User's phone
  phoneHash: string // Discovery hash
  publicKey: string // For E2E
  privateKey: string // Stored securely
}
```

### Key Implementation Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Sync Protocol** | Simple HTTP POST/GET with JSON | Firewall-friendly, easy to debug |
| **Identity** | Phone number + Ed25519 keypair | Familiar + secure |
| **Encryption** | X25519 + AES-GCM | Modern, fast |
| **Discovery** | Manual add only (MVP) | Simplest trust model |
| **Offline handling** | Queue messages in outbox | Reliable delivery |
| **Contact endpoint** | User configures publicly | No central server needed |

### Bootstrapping Flow

```
1. User A generates identity (phone + keypair)
2. User A shares their identifier with User B (out-of-band)
3. User B adds contact: identifier + endpoint
4. User B sends connection request
5. User A approves/declines
6. If approved: symmetric key exchange via X3DH
7. Contacts linked, polling begins
```

### Weak Points Addressed

| Weak Point | Mitigation |
|------------|------------|
| How do AIRIs find each other? | User-initiated contact sharing (MVP) |
| What if contact is offline? | Messages queued in outbox, delivered on next poll |
| Security of phone numbers? | Hashed for storage, only shared with contacts |
| How to bootstrap first contact? | Manual sharing via any channel (copy/paste) |
| What if endpoint changes? | Re-discovery via phone hash registry (future) |

## Next Steps

1. Confirm this MVP design fits your vision
2. Start with Phase 1: Core Messaging
3. Create `social` module structure
4. Add ContactsIsland UI component
5. Implement sync engine
6. Add Settings page for social module
