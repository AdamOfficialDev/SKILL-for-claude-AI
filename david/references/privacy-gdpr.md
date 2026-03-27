# DAVID — GDPR & Privacy Compliance Reference (Scanner AE)

Full pattern library for privacy compliance. Load for any backend handling user personal data.

---

## GDPR Articles Quick Reference

| Article | Topic | DAVID Checks |
|---------|-------|-------------|
| Art. 5 | Data processing principles | Minimization, purpose limitation, retention |
| Art. 6 | Lawful basis | Consent/contract/legitimate interest documented |
| Art. 7 | Consent | Explicit, unbundled, revocable |
| Art. 13/14 | Privacy notice | Disclosed at collection |
| Art. 17 | Right to erasure | Delete/anonymize on request |
| Art. 20 | Data portability | Export user data functionality |
| Art. 25 | Privacy by design | Default minimal data collection |
| Art. 32 | Security measures | Encryption at rest and in transit |
| Art. 33 | Breach notification | Incident response plan exists |

---

## PII Field Recognition

DAVID automatically identifies these field names as containing PII:

**Direct identifiers (highest sensitivity):**
`ssn`, `social_security`, `passport`, `national_id`, `tax_id`, `driver_license`
`credit_card`, `card_number`, `cvv`, `bank_account`, `iban`
`email`, `phone`, `phone_number`, `mobile`, `tel`
`ip_address`, `device_id`, `cookie_id`, `fingerprint`

**Indirect identifiers (medium sensitivity):**
`name`, `first_name`, `last_name`, `full_name`, `username`
`address`, `street`, `city`, `zip`, `postal_code`, `location`
`date_of_birth`, `dob`, `birth_date`, `age`
`gender`, `race`, `ethnicity`, `religion`
`salary`, `income`, `financial_data`

**Sensitive categories (Art. 9 — extra protection):**
`health`, `medical`, `diagnosis`, `medication`, `disability`
`biometric`, `genetic`, `dna`
`sexual_orientation`, `political`, `union_membership`

---

## Right to Erasure Implementation Patterns

### Django Pattern

```python
# ❌ Incomplete deletion
def delete_account(user_id: int):
    User.objects.filter(id=user_id).delete()
    # Missing: orders, comments, session data, audit logs, email_history

# ✅ Comprehensive erasure
from django.db import transaction

def handle_erasure_request(user_id: int, requestor: str):
    """GDPR Art. 17 — Right to Erasure"""
    with transaction.atomic():
        user = User.objects.get(id=user_id)

        # Option A: Anonymize (preferred — preserves aggregate data)
        user.email = f'deleted_{user_id}@gdpr.removed'
        user.first_name = 'Deleted'
        user.last_name = 'User'
        user.phone = None
        user.date_of_birth = None
        user.ip_address = None
        user.gdpr_erased_at = timezone.now()
        user.save()

        # Cascade to related PII
        user.orders.update(customer_name='Deleted User', shipping_address='[REMOVED]')
        user.comments.update(author_name='Deleted User')
        user.profile_photos.all().delete()  # Hard delete photos

        # Keep financial records (legal requirement) but anonymize
        user.invoices.update(customer_name='Deleted User', customer_email='[REMOVED]')

        # Delete session/tracking data
        user.sessions.all().delete()
        user.analytics_events.filter(
            # Keep last 90 days for fraud investigation
            created_at__lt=timezone.now() - timedelta(days=90)
        ).delete()

        # Log the erasure (itself must not contain PII)
        ErasureLog.objects.create(
            user_id=user_id,  # ID only, not name/email
            requested_by=requestor,
            completed_at=timezone.now(),
            tables_processed=['users', 'orders', 'comments', 'invoices'],
        )
```

### Node.js / Prisma Pattern

```typescript
async function handleErasureRequest(userId: number): Promise<void> {
    await prisma.$transaction(async (tx) => {
        // Anonymize user record
        await tx.user.update({
            where: { id: userId },
            data: {
                email: `deleted_${userId}@gdpr.removed`,
                name: 'Deleted User',
                phone: null,
                dateOfBirth: null,
                gdprErasedAt: new Date(),
            },
        });

        // Cascade anonymization
        await tx.order.updateMany({
            where: { userId },
            data: { customerName: 'Deleted User', shippingAddress: '[REMOVED]' },
        });

        // Hard delete non-essential data
        await tx.session.deleteMany({ where: { userId } });
        await tx.analyticsEvent.deleteMany({ where: { userId } });

        // Log the erasure audit trail
        await tx.erasureLog.create({
            data: { userId, completedAt: new Date() },
        });
    });
}
```

---

## Data Retention Enforcement

```python
# ❌ No retention policy — data accumulates forever
# audit_logs: 3.2M rows dating back to 2019

# ✅ Scheduled retention cleanup
from celery import shared_task
from django.utils import timezone
from datetime import timedelta

@shared_task
def enforce_data_retention():
    """Run daily. Enforces retention policy per data category."""
    now = timezone.now()

    # Session data: 30 days
    deleted_sessions = Session.objects.filter(
        last_active__lt=now - timedelta(days=30)
    ).delete()

    # General audit logs: 90 days
    deleted_logs = AuditLog.objects.filter(
        category='general',
        created_at__lt=now - timedelta(days=90)
    ).delete()

    # Financial records: 7 years (legal requirement — DO NOT DELETE)
    # Medical records: varies by jurisdiction — flag for legal review
    # Marketing consent records: retain while customer + 3 years

    # Analytics events: anonymize after 13 months (Google Analytics standard)
    AnalyticsEvent.objects.filter(
        created_at__lt=now - timedelta(days=395)
    ).update(user_id=None, ip_address=None, user_agent='[ANONYMIZED]')

    return {
        'sessions_deleted': deleted_sessions[0],
        'logs_deleted': deleted_logs[0],
    }
```

---

## Consent Management

```typescript
// ❌ Implicit consent — no record, no revocation
async function registerUser(data: RegisterDto) {
    const user = await createUser(data);
    analytics.identify(user.id, { email: user.email });  // Tracking without consent
    return user;
}

// ✅ Explicit consent with audit trail
async function registerUser(data: RegisterDto) {
    const user = await createUser(data);

    // Record explicit consents at registration
    await recordConsent(user.id, {
        type: 'terms_of_service',
        version: '2024-01',
        granted: true,
        ipAddress: data.ipAddress,
        userAgent: data.userAgent,
        timestamp: new Date(),
    });

    // Analytics only with explicit opt-in
    if (data.analyticsConsent === true) {
        await recordConsent(user.id, { type: 'analytics', granted: true, ... });
        analytics.identify(user.id, { email: user.email });
    }

    // Marketing emails only with explicit opt-in
    if (data.marketingConsent === true) {
        await recordConsent(user.id, { type: 'marketing', granted: true, ... });
    }

    return user;
}

// Consent must be revocable
async function updateConsent(userId: number, type: string, granted: boolean) {
    await recordConsent(userId, { type, granted, timestamp: new Date() });

    if (!granted && type === 'analytics') {
        analytics.optOut(userId);  // Stop all tracking
    }
    if (!granted && type === 'marketing') {
        await emailList.unsubscribe(userId);
    }
}
```

---

## Encryption at Rest

```python
# ❌ PII stored in plaintext database columns
class User(Base):
    ssn = Column(String(11))           # Plaintext SSN — massive breach risk
    date_of_birth = Column(Date)       # Acceptable for DOB
    credit_card_last4 = Column(String(4))  # Acceptable for last 4 only

# ✅ Field-level encryption for sensitive PII
from cryptography.fernet import Fernet
import os

ENCRYPTION_KEY = Fernet(os.environ['FIELD_ENCRYPTION_KEY'].encode())

class EncryptedField(TypeDecorator):
    impl = LargeBinary

    def process_bind_param(self, value, dialect):
        if value is None: return None
        return ENCRYPTION_KEY.encrypt(str(value).encode())

    def process_result_value(self, value, dialect):
        if value is None: return None
        return ENCRYPTION_KEY.decrypt(value).decode()

class User(Base):
    ssn            = Column(EncryptedField)  # Encrypted at rest
    bank_account   = Column(EncryptedField)  # Encrypted at rest
    date_of_birth  = Column(Date)            # OK unencrypted (low sensitivity)
```

---

## Data Portability (GDPR Art. 20)

```python
# ✅ Export all user data in machine-readable format
def export_user_data(user_id: int) -> dict:
    """GDPR Art. 20 — Data portability. Returns all data we hold about the user."""
    user = User.objects.get(id=user_id)
    return {
        'exported_at': timezone.now().isoformat(),
        'profile': {
            'email': user.email,
            'name': user.name,
            'created_at': user.created_at.isoformat(),
        },
        'orders': list(user.orders.values(
            'id', 'created_at', 'total', 'status', 'items'
        )),
        'comments': list(user.comments.values('id', 'created_at', 'content')),
        'consents': list(ConsentLog.objects.filter(user_id=user_id).values(
            'type', 'granted', 'timestamp'
        )),
        # Include ALL data you hold — hiding data violates Art. 20
    }

# Deliver as downloadable JSON or ZIP
# Must be provided within 30 days of request (Art. 12)
```

---

## Privacy by Design Checklist (Art. 25)

DAVID checks these patterns in new feature code:

```
□ Default settings are privacy-preserving (opt-in, not opt-out)
□ Collecting minimum data needed (not "might be useful later")
□ Data access is role-based (support can't see payment details)
□ PII not in URLs (no /users?email=alice@example.com)
□ PII not in logs (no console.log(user))
□ HTTPS enforced (no HTTP endpoints for user data)
□ Data stored in correct jurisdiction (EU data in EU region)
□ Third-party sharing disclosed and consented to
□ Children's data has extra protections (COPPA/GDPR-K)
```
