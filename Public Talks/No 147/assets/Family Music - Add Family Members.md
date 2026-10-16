# Add family members to Family Music

This guide covers giving family members access to Navidrome at
<https://music.xprojects.ca> through Cloudflare Access. Each person should use
their own Cloudflare email login and a separate, non-admin Navidrome account.
Never share the Navidrome administrator account.

## Before you begin

- Get each family member's explicit approval to use their exact email address.
- Have access to the Cloudflare dashboard and the Dell server.
- Keep Navidrome and Cloudflare Access enabled; do not create a public bypass.
- The Dell, NAS, and router must stay on for music access.

## 1. Allow the email address in Cloudflare Access

1. Sign in to the Cloudflare dashboard and select the `xprojects.ca` account.
2. Open **Zero Trust** and go to **Access** → **Applications**.
3. Open the **Family Music** application for `music.xprojects.ca`.
4. Edit its **Invited family emails** Allow policy.
5. Add the person's exact, approved email address. Keep the policy restricted to
   the invited email addresses; do not add a broad email-domain rule or a
   bypass policy.
6. Save the policy. Repeat for each person who has approved access.

## 2. Create a separate Navidrome account

1. Open <https://music.xprojects.ca> and complete the Cloudflare email
   one-time PIN login as the administrator.
2. In Navidrome, open the user/account administration area and create a new
   user for the family member.
3. Give the account a unique username and password, and leave administrator
   privileges disabled.
4. Share that person's Navidrome credentials with them privately. Do not send
   the administrator credentials.
5. Repeat for each family member.

## 3. Have them test access

1. Ask the family member to open <https://music.xprojects.ca>.
2. They should sign in to Cloudflare Access using the same email address that
   was added to the policy, then enter their own Navidrome username and
   password.
3. Confirm they can browse and play music.

If Cloudflare rejects the email, verify the exact address in the Access policy.
If Cloudflare login succeeds but Navidrome rejects the credentials, reset that
person's Navidrome password using the administrator account.

## Native music apps

The current setup is verified for browser access. Native Subsonic-compatible
music apps may not support the Cloudflare Access email login. Test compatibility
separately if desired; do not disable or bypass Cloudflare Access to make an
app work.
