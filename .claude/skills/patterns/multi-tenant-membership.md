# Multi-Tenant User Membership Pattern

**Pattern:** Users can belong to multiple tenants with different roles, accept invitations, and switch between tenants.

## Schema Design

### Core Models (Prisma)

```prisma
model User {
  id            String              @id @default(cuid())
  email         String              @unique
  name          String?
  emailVerified DateTime?
  image         String?
  createdAt     DateTime            @default(now())

  memberships   TenantMembership[]
  invitationsSent TenantInvitation[] @relation("InvitedBy")
  sessions      Session[]
}

model Tenant {
  id          String              @id @default(cuid())
  name        String
  slug        String              @unique
  domain      String?             @unique
  createdAt   DateTime            @default(now())

  memberships TenantMembership[]
  invitations TenantInvitation[]
}

model TenantMembership {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  tenantId  String
  tenant    Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)

  role      String   // 'owner', 'admin', 'member', 'viewer'
  invitedBy String?  // userId who invited
  invitedAt DateTime @default(now())
  joinedAt  DateTime?
  status    String   @default("active") // 'pending', 'active', 'suspended'

  // User's last activity in this tenant
  lastAccessedAt DateTime @default(now())

  @@unique([userId, tenantId])
  @@index([tenantId])
  @@index([userId])
  @@index([status])
}

model TenantInvitation {
  id         String    @id @default(cuid())
  tenantId   String
  tenant     Tenant    @relation(fields: [tenantId], references: [id], onDelete: Cascade)

  email      String
  role       String    // Role to assign when accepted
  token      String    @unique @default(cuid())

  invitedBy  String
  inviter    User      @relation("InvitedBy", fields: [invitedBy], references: [id])

  createdAt  DateTime  @default(now())
  expiresAt  DateTime  // 7 days from creation
  acceptedAt DateTime?

  @@index([email])
  @@index([token])
  @@index([tenantId])
}

model Session {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Active tenant for this session
  activeTenantId  String?

  expiresAt       DateTime
  createdAt       DateTime @default(now())

  @@index([userId])
}
```

## Session Management

### Auth Helpers with Active Tenant

```typescript
// lib/auth.ts
import { getServerSession } from 'next-auth'
import { authOptions } from '@/app/api/auth/[...nextauth]/route'
import { db } from '@/lib/db'

export async function requireAuth() {
  const session = await getServerSession(authOptions)
  if (!session?.user) {
    throw new Error('Unauthorized')
  }
  return session.user
}

export async function getActiveTenant() {
  const user = await requireAuth()

  // Get active tenant from session
  const session = await db.session.findFirst({
    where: { userId: user.id },
    include: { activeTenant: true }
  })

  if (!session?.activeTenantId) {
    // Default to user's first tenant
    const membership = await db.tenantMembership.findFirst({
      where: { userId: user.id, status: 'active' },
      include: { tenant: true }
    })

    if (!membership) {
      throw new Error('No tenant access')
    }

    return membership.tenant
  }

  return session.activeTenant
}

export async function getUserMemberships(userId: string) {
  return db.tenantMembership.findMany({
    where: { userId, status: 'active' },
    include: { tenant: true },
    orderBy: { lastAccessedAt: 'desc' }
  })
}

export async function requireTenantRole(
  tenantId: string,
  allowedRoles: string[]
) {
  const user = await requireAuth()

  const membership = await db.tenantMembership.findUnique({
    where: {
      userId_tenantId: {
        userId: user.id,
        tenantId
      }
    }
  })

  if (!membership || !allowedRoles.includes(membership.role)) {
    throw new Error('Insufficient permissions')
  }

  return { user, membership }
}
```

## Invitation System

### Server Actions

```typescript
// app/actions/invitations.ts
'use server'

import { db } from '@/lib/db'
import { requireAuth, requireTenantRole } from '@/lib/auth'
import { sendInvitationEmail } from '@/lib/email'
import { z } from 'zod'

const inviteSchema = z.object({
  email: z.string().email(),
  role: z.enum(['admin', 'member', 'viewer']),
  tenantId: z.string()
})

export async function inviteUserToTenant(formData: FormData) {
  const user = await requireAuth()

  // Parse and validate
  const parsed = inviteSchema.safeParse({
    email: formData.get('email'),
    role: formData.get('role'),
    tenantId: formData.get('tenantId')
  })

  if (!parsed.success) {
    return { error: 'Invalid input', details: parsed.error }
  }

  const { email, role, tenantId } = parsed.data

  // Check permissions (only admins/owners can invite)
  await requireTenantRole(tenantId, ['owner', 'admin'])

  // Check if user already member
  const inviteeUser = await db.user.findUnique({ where: { email } })
  if (inviteeUser) {
    const existing = await db.tenantMembership.findUnique({
      where: {
        userId_tenantId: {
          userId: inviteeUser.id,
          tenantId
        }
      }
    })

    if (existing) {
      return { error: 'User already member of this tenant' }
    }
  }

  // Check for existing pending invitation
  const pendingInvite = await db.tenantInvitation.findFirst({
    where: {
      email,
      tenantId,
      acceptedAt: null,
      expiresAt: { gt: new Date() }
    }
  })

  if (pendingInvite) {
    return { error: 'Invitation already sent' }
  }

  // Create invitation
  const invitation = await db.tenantInvitation.create({
    data: {
      email,
      role,
      tenantId,
      invitedBy: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
    },
    include: {
      tenant: true,
      inviter: true
    }
  })

  // Send email
  await sendInvitationEmail({
    to: email,
    tenantName: invitation.tenant.name,
    inviterName: invitation.inviter.name,
    token: invitation.token,
    role
  })

  return { success: true, invitationId: invitation.id }
}

export async function acceptInvitation(token: string) {
  const user = await requireAuth()

  // Find invitation
  const invitation = await db.tenantInvitation.findUnique({
    where: { token },
    include: { tenant: true }
  })

  if (!invitation) {
    return { error: 'Invalid invitation' }
  }

  if (invitation.acceptedAt) {
    return { error: 'Invitation already accepted' }
  }

  if (invitation.expiresAt < new Date()) {
    return { error: 'Invitation expired' }
  }

  if (invitation.email !== user.email) {
    return { error: 'Invitation sent to different email' }
  }

  // Check if already member
  const existing = await db.tenantMembership.findUnique({
    where: {
      userId_tenantId: {
        userId: user.id,
        tenantId: invitation.tenantId
      }
    }
  })

  if (existing) {
    return { error: 'Already member of this tenant' }
  }

  // Create membership and mark invitation as accepted
  await db.$transaction([
    db.tenantMembership.create({
      data: {
        userId: user.id,
        tenantId: invitation.tenantId,
        role: invitation.role,
        invitedBy: invitation.invitedBy,
        invitedAt: invitation.createdAt,
        joinedAt: new Date(),
        status: 'active'
      }
    }),
    db.tenantInvitation.update({
      where: { id: invitation.id },
      data: { acceptedAt: new Date() }
    })
  ])

  return {
    success: true,
    tenant: invitation.tenant
  }
}

export async function declineInvitation(token: string) {
  const user = await requireAuth()

  const invitation = await db.tenantInvitation.findUnique({
    where: { token }
  })

  if (!invitation || invitation.email !== user.email) {
    return { error: 'Invalid invitation' }
  }

  // Delete invitation
  await db.tenantInvitation.delete({
    where: { id: invitation.id }
  })

  return { success: true }
}

export async function getPendingInvitations() {
  const user = await requireAuth()

  return db.tenantInvitation.findMany({
    where: {
      email: user.email,
      acceptedAt: null,
      expiresAt: { gt: new Date() }
    },
    include: {
      tenant: true,
      inviter: true
    },
    orderBy: { createdAt: 'desc' }
  })
}
```

## Tenant Switching

### Server Actions

```typescript
// app/actions/tenant-switch.ts
'use server'

import { db } from '@/lib/db'
import { requireAuth } from '@/lib/auth'
import { revalidatePath } from 'next/cache'

export async function switchActiveTenant(tenantId: string) {
  const user = await requireAuth()

  // Verify user has access to this tenant
  const membership = await db.tenantMembership.findUnique({
    where: {
      userId_tenantId: {
        userId: user.id,
        tenantId
      },
      status: 'active'
    }
  })

  if (!membership) {
    return { error: 'No access to this tenant' }
  }

  // Update session's active tenant
  await db.session.updateMany({
    where: { userId: user.id },
    data: { activeTenantId: tenantId }
  })

  // Update lastAccessedAt
  await db.tenantMembership.update({
    where: { id: membership.id },
    data: { lastAccessedAt: new Date() }
  })

  // Revalidate all pages
  revalidatePath('/', 'layout')

  return { success: true }
}

export async function leaveTenant(tenantId: string) {
  const user = await requireAuth()

  // Check if user is owner
  const membership = await db.tenantMembership.findUnique({
    where: {
      userId_tenantId: {
        userId: user.id,
        tenantId
      }
    }
  })

  if (!membership) {
    return { error: 'Not a member of this tenant' }
  }

  if (membership.role === 'owner') {
    // Check if there are other owners
    const ownerCount = await db.tenantMembership.count({
      where: {
        tenantId,
        role: 'owner',
        status: 'active'
      }
    })

    if (ownerCount === 1) {
      return { error: 'Cannot leave: you are the only owner. Transfer ownership first.' }
    }
  }

  // Remove membership
  await db.tenantMembership.delete({
    where: { id: membership.id }
  })

  // If this was active tenant, switch to another
  const otherMembership = await db.tenantMembership.findFirst({
    where: { userId: user.id, status: 'active' }
  })

  if (otherMembership) {
    await switchActiveTenant(otherMembership.tenantId)
  }

  return { success: true }
}
```

## UI Components

### Tenant Switcher

```tsx
// components/tenant-switcher.tsx
'use client'

import { useState } from 'react'
import { Check, ChevronsUpDown, Plus } from 'lucide-react'
import { switchActiveTenant } from '@/app/actions/tenant-switch'
import { useRouter } from 'next/navigation'

interface Tenant {
  id: string
  name: string
  slug: string
}

interface TenantSwitcherProps {
  tenants: Tenant[]
  activeTenantId: string
}

export function TenantSwitcher({ tenants, activeTenantId }: TenantSwitcherProps) {
  const router = useRouter()
  const [open, setOpen] = useState(false)
  const [loading, setLoading] = useState(false)

  const activeTenant = tenants.find(t => t.id === activeTenantId)

  async function handleSwitch(tenantId: string) {
    setLoading(true)
    const result = await switchActiveTenant(tenantId)

    if (result.success) {
      setOpen(false)
      router.refresh()
    } else {
      alert(result.error)
    }

    setLoading(false)
  }

  return (
    <div className="relative">
      <button
        onClick={() => setOpen(!open)}
        className="flex items-center gap-2 px-3 py-2 border rounded-md hover:bg-gray-50"
        disabled={loading}
      >
        <span className="font-medium">{activeTenant?.name}</span>
        <ChevronsUpDown className="h-4 w-4 text-gray-500" />
      </button>

      {open && (
        <div className="absolute top-full mt-1 w-64 bg-white border rounded-md shadow-lg z-50">
          <div className="p-2 space-y-1">
            {tenants.map(tenant => (
              <button
                key={tenant.id}
                onClick={() => handleSwitch(tenant.id)}
                className="w-full flex items-center justify-between px-3 py-2 text-left hover:bg-gray-100 rounded-md"
              >
                <span>{tenant.name}</span>
                {tenant.id === activeTenantId && (
                  <Check className="h-4 w-4 text-blue-600" />
                )}
              </button>
            ))}
          </div>

          <div className="border-t p-2">
            <button
              onClick={() => router.push('/tenants/new')}
              className="w-full flex items-center gap-2 px-3 py-2 text-left hover:bg-gray-100 rounded-md text-sm"
            >
              <Plus className="h-4 w-4" />
              Create new workspace
            </button>
          </div>
        </div>
      )}
    </div>
  )
}
```

### Invitation List

```tsx
// app/invitations/page.tsx
import { getPendingInvitations } from '@/app/actions/invitations'
import { AcceptButton, DeclineButton } from './buttons'

export default async function InvitationsPage() {
  const invitations = await getPendingInvitations()

  if (invitations.length === 0) {
    return (
      <div className="text-center py-12">
        <p className="text-gray-500">No pending invitations</p>
      </div>
    )
  }

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">Pending Invitations</h1>

      {invitations.map(invite => (
        <div
          key={invite.id}
          className="border rounded-lg p-4 flex items-center justify-between"
        >
          <div>
            <h3 className="font-medium">{invite.tenant.name}</h3>
            <p className="text-sm text-gray-600">
              Invited by {invite.inviter.name} as {invite.role}
            </p>
            <p className="text-xs text-gray-500 mt-1">
              Expires {new Date(invite.expiresAt).toLocaleDateString()}
            </p>
          </div>

          <div className="flex gap-2">
            <AcceptButton token={invite.token} />
            <DeclineButton token={invite.token} />
          </div>
        </div>
      ))}
    </div>
  )
}
```

## Security Considerations

### Always Verify Tenant Access

```typescript
// ✅ GOOD: Check membership before operations
export async function getProjectById(projectId: string) {
  const { user, membership } = await requireTenantRole(tenantId, ['owner', 'admin', 'member'])

  const project = await db.project.findFirst({
    where: {
      id: projectId,
      tenantId: membership.tenantId  // Ensure project belongs to user's tenant
    }
  })

  if (!project) {
    throw new Error('Project not found')
  }

  return project
}

// ❌ BAD: Direct access without tenant check
export async function getProjectById(projectId: string) {
  return db.project.findUnique({
    where: { id: projectId }
  })
}
```

### Role-Based Permissions

```typescript
const ROLE_HIERARCHY = {
  owner: 4,
  admin: 3,
  member: 2,
  viewer: 1
}

export function canPerformAction(userRole: string, requiredRole: string): boolean {
  return ROLE_HIERARCHY[userRole] >= ROLE_HIERARCHY[requiredRole]
}

// Usage
export async function deleteProject(projectId: string, tenantId: string) {
  const { membership } = await requireTenantRole(tenantId, ['owner', 'admin'])

  if (!canPerformAction(membership.role, 'admin')) {
    return { error: 'Insufficient permissions' }
  }

  // Proceed with deletion
}
```

## Common Patterns

### Member Management Page

```typescript
// app/[tenant]/settings/members/page.tsx
import { getActiveTenant } from '@/lib/auth'
import { db } from '@/lib/db'
import { InviteMemberForm } from './invite-form'
import { MemberList } from './member-list'

export default async function MembersPage() {
  const tenant = await getActiveTenant()

  const members = await db.tenantMembership.findMany({
    where: { tenantId: tenant.id },
    include: { user: true },
    orderBy: { role: 'desc' }
  })

  const pendingInvites = await db.tenantInvitation.findMany({
    where: {
      tenantId: tenant.id,
      acceptedAt: null,
      expiresAt: { gt: new Date() }
    }
  })

  return (
    <div className="space-y-8">
      <InviteMemberForm tenantId={tenant.id} />
      <MemberList members={members} pendingInvites={pendingInvites} />
    </div>
  )
}
```

### First-Time User Flow

```typescript
// middleware.ts - Redirect if no tenant access
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { getToken } from 'next-auth/jwt'

export async function middleware(request: NextRequest) {
  const token = await getToken({ req: request })

  if (token && request.nextUrl.pathname === '/') {
    // Check if user has any tenant memberships
    const memberships = await db.tenantMembership.count({
      where: { userId: token.sub, status: 'active' }
    })

    if (memberships === 0) {
      // No tenants - redirect to onboarding
      return NextResponse.redirect(new URL('/onboarding', request.url))
    }
  }

  return NextResponse.next()
}
```

## Edge Cases

1. **User invited to tenant they're already in:** Check for existing membership before creating invitation
2. **User leaves tenant while it's active:** Switch to another tenant automatically
3. **Last owner trying to leave:** Prevent unless ownership transferred
4. **Expired invitations:** Clean up periodically with cron job
5. **Email change:** Handle invitation email mismatch gracefully
6. **Invitation to non-existent user:** Create account on first login, then auto-accept invitations

## Best Practices

- **Default to user's most recently accessed tenant** on login
- **Store tenant in session** for performance (avoid DB query per request)
- **Revalidate paths** after tenant switch to update UI
- **Show pending invitations count** in navbar
- **Set invitation expiry** (7 days recommended)
- **Send email notifications** for invitations and role changes
- **Audit log** membership changes for compliance
- **Graceful degradation** if user loses tenant access mid-session
