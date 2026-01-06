# Plan: Fix Admin Dashboard Sign-In Issue

## Problem Identified

The user can authenticate successfully (login works), but cannot access the admin dashboard because:
- The `user_roles` table is **empty**
- No admin role has been assigned to any user
- The app correctly checks for admin role before granting dashboard access

## Current State

- User `admin@restaurant.com` exists with ID: `6c5180b1-f8e1-4794-97e1-05aa9d1d7ac0`
- Authentication is working (successful logins in auth logs)
- `user_roles` table exists but has no entries

## Solution

### Step 1: Add Admin Role via Database Migration

Create a migration that inserts the admin role for the existing user:

```sql
INSERT INTO user_roles (user_id, role)
VALUES ('6c5180b1-f8e1-4794-97e1-05aa9d1d7ac0', 'admin')
ON CONFLICT (user_id, role) DO NOTHING;
```

### Step 2: Improve Login UX (Optional Enhancement)

Update the AdminLoginPage to show a clearer message when a user is authenticated but doesn't have admin privileges:

- Show "Access Denied: You don't have admin privileges" message
- Provide instructions to contact the owner for access
- Add a logout button so they can try a different account

### Step 3: Add First-User-Is-Admin Logic (Optional)

Create a database trigger that automatically grants admin role to the first user who signs up (useful for initial setup). This prevents the chicken-and-egg problem.

## Files to Modify

1. **Database Migration** - Insert admin role for existing user
2. **src/pages/admin/AdminLoginPage.tsx** - Better UX for non-admin users
3. **src/hooks/useAuth.tsx** - No changes needed (already working correctly)

## Implementation Order

1. Run database migration to add admin role
2. Update login page UX (optional but recommended)
3. Test login flow

## Expected Outcome

After implementation:
- User `admin@restaurant.com` will have admin role
- Login will redirect to `/admin/dashboard` successfully
- Non-admin users will see a clear "access denied" message

## Critical Files for Implementation

- Database migration (new file) - Adds admin role to user_roles table
- src/pages/admin/AdminLoginPage.tsx - Improve UX for access denied state
- src/hooks/useAuth.tsx - Reference for understanding auth flow (no changes needed)
