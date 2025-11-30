---
description: This section covers how to configure OpenVPN user authentication and secrets.
icon: key
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Secrets

{% hint style="info" %}
OpenVPN secrets (users) are used for authentication. You can use certificate-only authentication or combine certificates with username/password.
{% endhint %}

In WinBox you can configure OpenVPN secrets in **PPP -> Secrets**, there you can click to <kbd>**+**</kbd> to create new user or you can use terminal with command `/ppp secret`

***

## Authentication methods

### Certificate-only authentication

When using **certificate-only authentication**, no PPP secrets are needed. Client authentication is handled entirely through certificates.

This method is enabled by setting `require-client-certificate=yes` on the OpenVPN server and not configuring any PPP secrets.

**Advantages:**
- More secure (no password to compromise)
- Easier to manage for small number of users
- Certificate revocation is possible

**Disadvantages:**
- Each client needs individual certificate
- More complex initial setup

### Certificate + Username/Password

This method combines certificate authentication with traditional username/password authentication.

**Advantages:**
- Additional security layer
- Easier user management
- Can use same certificate for multiple users

**Disadvantages:**
- Passwords can be compromised
- More complex for users

***

## Creating PPP secrets

### Basic user configuration

Click on **`+`** to add new PPP secret

You will need to fill out:

* **Name** - Username for the VPN connection
* **Password** - User's password
* **Service** - Select "ovpn" 
* **Profile** - Select appropriate VPN profile
* **Local Address** - (Optional) Router's tunnel endpoint IP
* **Remote Address** - (Optional) Specific IP for this user

{% code overflow="wrap" %}
```bash
/ppp secret add name=user1 password=SecurePassword123 service=ovpn profile=vpn-users
```
{% endcode %}

### Advanced user settings

For more control over individual users:

* **Routes** - Static routes to be added when user connects
* **Rate Limit** - Bandwidth limitation for the user
* **Comment** - Description for the user

{% code overflow="wrap" %}
```bash
/ppp secret add name=user1 password=SecurePassword123 service=ovpn profile=vpn-users rate-limit=10M/5M comment="John Doe - Marketing dept"
```
{% endcode %}

### Assigning specific IP addresses

You can assign specific IP addresses to users:

{% code overflow="wrap" %}
```bash
/ppp secret add name=admin password=AdminPass123 service=ovpn profile=vpn-users remote-address=10.10.10.100 comment="Admin user"
```
{% endcode %}

***

## User management

### Disabling users

To temporarily disable a user without deleting:

{% code overflow="wrap" %}
```bash
/ppp secret disable user1
```
{% endcode %}

### Enabling users

To re-enable a disabled user:

{% code overflow="wrap" %}
```bash
/ppp secret enable user1
```
{% endcode %}

### Changing passwords

To update a user's password:

{% code overflow="wrap" %}
```bash
/ppp secret set user1 password=NewPassword456
```
{% endcode %}

***

## Multiple authentication profiles

### Different user groups

Create different profiles for different user types:

{% code overflow="wrap" %}
```bash
# Admin users with full access
/ppp secret add name=admin1 password=AdminPass123 service=ovpn profile=admin-users

# Regular users with limited access  
/ppp secret add name=employee1 password=EmpPass123 service=ovpn profile=employee-users

# Guest users with restricted access
/ppp secret add name=guest1 password=GuestPass123 service=ovpn profile=guest-users
```
{% endcode %}

### Rate limiting per user type

Set different bandwidth limits for different user groups:

{% code overflow="wrap" %}
```bash
# Premium users - high bandwidth
/ppp secret add name=premium1 password=PremPass123 service=ovpn profile=vpn-users rate-limit=50M/20M

# Standard users - medium bandwidth
/ppp secret add name=standard1 password=StdPass123 service=ovpn profile=vpn-users rate-limit=20M/10M

# Basic users - low bandwidth
/ppp secret add name=basic1 password=BasicPass123 service=ovpn profile=vpn-users rate-limit=5M/2M
```
{% endcode %}

***

## Security considerations

### Password policy

**Strong password requirements:**
- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, and symbols
- Avoid dictionary words
- Regular password changes

### Certificate management

When using certificate + password authentication:

1. Each user should have unique certificate
2. Certificates should have reasonable expiration dates
3. Revoked certificates should be properly managed
4. Certificate common names should match usernames

### Monitoring access

Check active connections and user activity:

{% code overflow="wrap" %}
```bash
# View active PPP connections
/ppp active print

# Check specific user sessions
/ppp active print where name=user1

# Monitor connection logs
/log print where topics~"ppp"
```
{% endcode %}

***

<details>

<summary>Show complete user setup examples</summary>

{% code overflow="wrap" %}
```bash
# Create different user types with various configurations

# Administrator with specific IP and no rate limit
/ppp secret add name=admin password=AdminSecure2024! service=ovpn profile=admin-users remote-address=10.10.10.10 comment="Network Administrator"

# Regular employee with rate limit
/ppp secret add name=jdoe password=Employee2024@ service=ovpn profile=employee-users rate-limit=10M/5M comment="John Doe - Sales"

# Contractor with time-limited access and rate limit
/ppp secret add name=contractor1 password=Temp2024# service=ovpn profile=contractor-users rate-limit=5M/2M comment="Temporary contractor - expires 2024-12-31"

# Guest user with heavy restrictions
/ppp secret add name=guest password=Guest2024$ service=ovpn profile=guest-users rate-limit=2M/1M comment="Guest access only"

# Service account for monitoring
/ppp secret add name=monitoring password=Monitor2024% service=ovpn profile=service-users remote-address=10.10.10.200 comment="Monitoring service"
```
{% endcode %}

</details>

## Best practices

### User account management

1. **Regular audits** - Review and remove unused accounts
2. **Password rotation** - Implement regular password changes
3. **Access logging** - Monitor and log all access attempts
4. **Principle of least privilege** - Give users minimum required access

### Certificate management

1. **Individual certificates** - Each user should have unique certificate
2. **Certificate expiration** - Set appropriate validity periods
3. **Revocation process** - Have clear process for certificate revocation
4. **Backup certificates** - Securely backup CA and certificates

### Troubleshooting authentication

**Authentication failures:**
- Check username and password spelling
- Verify user account is enabled
- Check certificate validity and trust
- Review server logs for detailed error messages

**Connection issues:**
- Verify profile configuration
- Check IP pool availability
- Test network connectivity
- Review firewall rules

